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

Проверить параметры control-plane VIP в `inventories/prod/group_vars`:
- `control_plane_endpoint: 10.255.106.20`
- `control_plane_endpoint_port: 8443`
- `control_plane_vip_enabled: true`
- `kubeadm_config_api_version: kubeadm.k8s.io/v1beta3`
- `kubeadm_init_allow_experimental_api: false`
- `container_runtime_install_cri_tools: false` (рекомендуемо для текущего набора repo на RedOS)
- `kubernetes_repo_validate_before_install: true`
- `kubernetes_package_install_retries: 4`
- `kubernetes_packages_version_override: ""` (пусто = использовать текущую версию из repo channel)

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
4. Kubernetes API доступен через `10.255.106.20:8443`.
5. MetalLB controller/speaker в `Running`, IP-пул применен.
6. `StorageClass nfs-sp` создан, PVC успешно bind.
7. SELinux/firewalld применены по ролям.
8. Повторный прогон плейбука не создает нежелательных изменений.

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

Автоматизация в `playbooks/validate.yml`:
- Роль `validation` проверяет доступность API через control-plane VIP (`/readyz` через `kubectl --server=https://10.255.106.20:8443`).
- Роль `validation` проверяет `keepalived/haproxy` на всех control-plane нодах и наличие ровно одного владельца VIP.
- Роль `validation` проверяет, что ingress-service имеет `type=LoadBalancer` и ненулевой external address.
- При заданном `validation_ingress_expected_vip` дополнительно валидируется точное совпадение VIP.
- Опциональный failover-test VIP (по умолчанию выключен): `-e validation_enable_control_plane_vip_failover_test=true`.
- Для отключения API/VIP проверок: `-e validation_enable_api_vip_check=false -e validation_enable_control_plane_vip_state_check=false`.
- Для отключения проверки: `-e validation_enable_ingress_vip_check=false`.

Пример запуска:
```bash
# Стандартная безопасная валидация
ansible-playbook -i inventories/prod/hosts.yml playbooks/validate.yml

# Стендовый failover-check control-plane VIP
ansible-playbook -i inventories/prod/hosts.yml playbooks/validate.yml \
  -e validation_enable_control_plane_vip_failover_test=true
```

Запуск только failover-сценария по тегу:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/validate.yml --tags failover
```

## 11. Полезные команды диагностики
```bash
# Kubernetes
kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes -o wide
kubectl --kubeconfig /etc/kubernetes/admin.conf get pods -A
kubectl --kubeconfig /etc/kubernetes/admin.conf get svc -A
kubectl --kubeconfig /etc/kubernetes/admin.conf get storageclass

# Control-plane VIP
ansible -i inventories/prod/hosts.yml control_plane -m command -a "systemctl is-active haproxy"
ansible -i inventories/prod/hosts.yml control_plane -m command -a "systemctl is-active keepalived"
ansible -i inventories/prod/hosts.yml control_plane -m shell -a "ip -4 addr show | grep -F 10.255.106.20 || true"
nc -vz 10.255.106.20 8443

# Проверка MetalLB
kubectl --kubeconfig /etc/kubernetes/admin.conf -n metallb-system get all

# Проверка NFS provisioner
kubectl --kubeconfig /etc/kubernetes/admin.conf -n nfs-storage get all
```

Если на bootstrap возникает ошибка `experimental API spec ... is not allowed`:
1. Убедиться, что используется `kubeadm_config_api_version: kubeadm.k8s.io/v1beta3`.
2. Перезапустить bootstrap после фикса:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/bootstrap.yml
```

Если на этапе runtime возникает `Depsolve Error` по `cri-tools`:
1. Убедиться, что `container_runtime_install_cri_tools: false`.
2. Перезапустить bootstrap:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/bootstrap.yml
```

Если на этапе `kubernetes_core` возникает `Cannot download, all mirrors were already tried`:
1. Проверить доступность repo metadata на хосте:
```bash
sudo dnf -q makecache --refresh --disablerepo='*' --enablerepo=kubernetes
```
2. При недоступности нужного patch-релиза временно указать доступную версию:
- `kubernetes_packages_version_override: "1.30.1-150500.1.1"` (пример)
3. Повторить bootstrap:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/bootstrap.yml
```

Если `kubeadm init` падает на `wait-control-plane` (`kubelet 10248`):
1. В текущей версии роли включен авто-сбор диагностики (`kubelet/runtime state`, `journalctl -u kubelet`, `crictl ps`) в `rescue`.
2. Проверить в выводе:
   - что swap выключен;
   - что runtime-сокет доступен (`/run/containerd/containerd.sock`);
   - что `kubelet` в `active` и не падает по cgroup/конфигу.
3. После исправления причины перезапустить:
```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/bootstrap.yml --tags k8s
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
