## Задача: Эксплуатационный runbook для запуска Ansible-плейбука
- **Статус**: Завершена
- **Описание**: Стандартизировать запуск и проверку плейбуков на control host для режимов изолированной и неизолированной сети.

# runbook.md

## 1. Назначение
Документ описывает порядок запуска проекта на control host:
- подготовка окружения;
- запуск деплоя в двух режимах (`proxy_enabled=true/false`);
- проверка идемпотентности;
- базовый чек-лист приемки.

## 2. Предварительные требования (control host)
- ОС control host: Debian 13 (или совместимая Linux-среда).
- Доступ по SSH к узлам из `inventories/prod/hosts.yml`.
- Python3 и pip установлены.
- Доступ к репозиторию проекта и `sudo` при необходимости.

## 3. Подготовка окружения
```bash
cd /Users/eln/Documents/k8s-redos

python3 -m venv .venv
source .venv/bin/activate

pip install --upgrade pip
pip install "ansible-core==2.19.4" ansible-lint yamllint

ansible-galaxy collection install -r requirements.yml
```

Проверка инструментов:
```bash
ansible --version
ansible-playbook --version
ansible-lint --version
yamllint --version
```

## 4. Предпроверки перед деплоем
```bash
cd /Users/eln/Documents/k8s-redos
source .venv/bin/activate

ansible-inventory -i inventories/prod/hosts.yml --graph
ansible -i inventories/prod/hosts.yml all -m ping

ansible-playbook -i inventories/prod/hosts.yml --syntax-check playbooks/site.yml
ansible-playbook -i inventories/prod/hosts.yml --syntax-check playbooks/bootstrap.yml
ansible-playbook -i inventories/prod/hosts.yml --syntax-check playbooks/hardening.yml
ansible-playbook -i inventories/prod/hosts.yml --syntax-check playbooks/storage.yml
ansible-playbook -i inventories/prod/hosts.yml --syntax-check playbooks/validate.yml
ansible-playbook -i inventories/prod/hosts.yml --syntax-check playbooks/reinstall_cluster_and_nfs.yml
```

## 5. Режим A: Изолированная сеть (proxy включен)
Убедитесь, что в `inventories/prod/group_vars/all.yml`:
- `proxy_enabled: true`
- `http_proxy/https_proxy/no_proxy` заданы корректно.

Запуск:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml
```

## 6. Режим B: Неизолированная сеть (proxy выключен)
Вариант 1 (временный override на запуск):
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml -e proxy_enabled=false
```

Вариант 2 (постоянно для окружения):
- установить `proxy_enabled: false` в `inventories/prod/group_vars/all.yml`
- затем запускать обычной командой:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml
```

## 7. Проверка hostname по inventory
Системные hostname должны совпадать с именами узлов в `inventory_hostname`.

Проверка:
```bash
ansible -i inventories/prod/hosts.yml all -m command -a hostname
```

Ожидается:
- `k8s-scp-01`, `k8s-scp-02`, `k8s-scp-03`
- `k8s-wkn-01`, `k8s-wkn-02`, `k8s-wkn-03`
- `k8s-mlb-01`, `k8s-mlb-02`, `k8s-mlb-03`
- `k8s-nfs-01`

## 8. Проверка идемпотентности
После успешного деплоя выполнить повторный запуск:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml
```

Критерий:
- отсутствуют незапланированные изменения на стабильной конфигурации.

## 9. Базовый чек-лист приемки
1. Все хосты доступны через `ansible -m ping`.
2. `kubeadm`-кластер поднят, все ноды `Ready`.
3. CNI установлен и рабочий.
4. MetalLB controller/speaker в `Running`, IP-пул применен.
5. `StorageClass nfs-sp` создан, PVC успешно bind.
6. SELinux/firewalld применены по ролям.
7. Повторный прогон плейбука не создает нежелательных изменений.

## 10. MetalLB L2 без доступа к vCenter
Ограничение: сетевые изменения в `vCenter` недоступны.

Что обязательно проверить до публикации:
1. VIP-пул `10.255.106.21-10.255.106.30` зарезервирован и не выдается DHCP.
2. Клиенты имеют маршрут/доступ в сеть `10.255.106.0/26`.
3. На межсетевых экранах разрешен доступ к VIP на `80/443`.
4. DNS указывает на VIP ingress.

Рекомендованная фиксация VIP ingress:
```bash
# Проверить, что ingress service имеет тип LoadBalancer
kubectl --kubeconfig /etc/kubernetes/admin.conf -n ingress-nginx get svc ingress-nginx-controller -o wide

# Зафиксировать VIP (пример: 10.255.106.21)
kubectl --kubeconfig /etc/kubernetes/admin.conf -n ingress-nginx patch svc ingress-nginx-controller \
  --type merge \
  -p '{"metadata":{"annotations":{"metallb.io/loadBalancerIPs":"10.255.106.21"}}}'
```

Проверка публикации:
```bash
# Проверить EXTERNAL-IP у ingress-сервиса
kubectl --kubeconfig /etc/kubernetes/admin.conf -n ingress-nginx get svc ingress-nginx-controller -o wide

# Проверить с клиентской сети (где должен быть доступ)
curl -I http://10.255.106.21
curl -Ik https://10.255.106.21
```

## 11. Полезные команды диагностики
```bash
# Kubernetes
kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes -o wide
kubectl --kubeconfig /etc/kubernetes/admin.conf get pods -A
kubectl --kubeconfig /etc/kubernetes/admin.conf get svc -A
kubectl --kubeconfig /etc/kubernetes/admin.conf get storageclass

# Проверка MetalLB
kubectl --kubeconfig /etc/kubernetes/admin.conf -n metallb-system get all

# Проверка NFS provisioner
kubectl --kubeconfig /etc/kubernetes/admin.conf -n nfs-storage get all
```

## 12. Опция data-диска для NFS
Если нужен отдельный диск под `nfs_export_path`, настройте в `inventories/prod/group_vars/nfs.yml`:
- `storage_nfs_data_disk_enabled: true`
- `storage_nfs_data_disk_device: /dev/sdb` (или другой нужный диск)
- `storage_nfs_data_disk_partition_number: 1`
- `storage_nfs_data_disk_fs_type: xfs`
- `storage_nfs_data_disk_mount_opts: defaults`
- `storage_nfs_data_disk_allow_system_disk: false`

Важно:
- сценарий может переформатировать выбранный диск (в зависимости от `storage_nfs_data_disk_fs_force`);
- используйте только целевой data-диск.
- по умолчанию включен guardrail, блокирующий системный диск (`/`, `/boot`, `/boot/efi`, `/var`).
- принудительный обход возможен только явным `storage_nfs_data_disk_allow_system_disk=true` (не рекомендуется).

## 13. Рекомендованный порядок запуска
1. `playbooks/bootstrap.yml`
2. `playbooks/hardening.yml`
3. `playbooks/storage.yml`
4. `playbooks/validate.yml`

Либо единым запуском:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml
```

## 14. Деструктивный сценарий: полная переустановка (cluster_and_nfs)
Внимание: сценарий удаляет:
- весь Kubernetes state на узлах `k8s_cluster`;
- все данные из NFS export path (`/srv/storage`).

Обязательные guardrail-параметры:
- `reinstall_scope=cluster_and_nfs`
- `reinstall_confirm_token=DESTROY_CLUSTER_AND_NFS_DATA`

Запуск:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/reinstall_cluster_and_nfs.yml \
  -e reinstall_scope=cluster_and_nfs \
  -e reinstall_confirm_token=DESTROY_CLUSTER_AND_NFS_DATA
```

После выполнения:
1. Повторно развернуть кластер через `playbooks/site.yml`.
2. Проверить, что `StorageClass nfs-sp` пересоздан и PVC создаются заново.
3. Проверить readiness всех нод и системных подов.
