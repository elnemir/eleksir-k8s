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
- Доступ по SSH к узлам из `k8s/inventories/prod/hosts.ini`.
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

Важно:
- в роли `base_os` перед кластерной установкой выполняется обновление всех установленных пакетов (`base_os_update_all_packages: true` по умолчанию).
- при необходимости этот шаг можно отключить только осознанно: `-e base_os_update_all_packages=false`.

## 4. Предпроверки перед деплоем
```bash
cd /Users/eln/Documents/k8s-redos
source .venv/bin/activate

ansible-inventory -i k8s/inventories/prod/hosts.ini --graph
ansible -i k8s/inventories/prod/hosts.ini all -m ping

ansible-playbook -i k8s/inventories/prod/hosts.ini --syntax-check k8s/playbooks/k8s_site.yml
ansible-playbook -i k8s/inventories/prod/hosts.ini --syntax-check k8s/playbooks/k8s_bootstrap.yml
ansible-playbook -i k8s/inventories/prod/hosts.ini --syntax-check k8s/playbooks/k8s_metallb.yml
ansible-playbook -i k8s/inventories/prod/hosts.ini --syntax-check k8s/playbooks/k8s_scale_out.yml
ansible-playbook -i k8s/inventories/prod/hosts.ini --syntax-check k8s/playbooks/k8s_hardening.yml
ansible-playbook -i k8s/inventories/prod/hosts.ini --syntax-check k8s/playbooks/k8s_storage.yml
ansible-playbook -i k8s/inventories/prod/hosts.ini --syntax-check k8s/playbooks/k8s_validate.yml
ansible-playbook -i k8s/inventories/prod/hosts.ini --syntax-check k8s/playbooks/k8s_reinstall_cluster_and_nfs.yml
ansible-playbook -i k8s/inventories/prod/hosts.ini --syntax-check k8s/playbooks/k8s_manage_proxy.yml
```

Проверить параметры control-plane VIP в `k8s/inventories/prod/group_vars`:
- `control_plane_endpoint: 10.255.106.20`
- `control_plane_endpoint_port: 8443`
- `control_plane_vip_enabled: true`
- `kubeadm_config_api_version: kubeadm.k8s.io/v1beta3`
- `kubeadm_init_allow_experimental_api: false`
- `container_runtime_install_cri_tools: false` (рекомендуемо для текущего набора repo на RedOS)
- `kubernetes_repo_validate_before_install: true`
- `kubernetes_package_install_retries: 4`
- `kubernetes_package_install_nobest: true`
- `kubernetes_packages_version_override: ""` (пусто = использовать текущую версию из repo channel)
- `security_hardening_enabled: false|true` (управление запуском роли hardening)

## 5. Режим A: Изолированная сеть (proxy включен)
Убедитесь, что в `k8s/inventories/prod/group_vars/all.yml`:
- `proxy_enabled: true`
- `http_proxy/https_proxy/no_proxy` заданы корректно.

Запуск:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_site.yml
```

## 6. Режим B: Неизолированная сеть (proxy выключен)
Вариант 1 (временный override на запуск):
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_site.yml -e proxy_enabled=false
```

Вариант 2 (постоянно для окружения):
- установить `proxy_enabled: false` в `k8s/inventories/prod/group_vars/all.yml`
- затем запускать обычной командой:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_site.yml
```

Управление hardening/SELinux шагами:
- для отключения hardening-этапа (включая SELinux/firewalld шаги роли `security_hardening`) используйте:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_site.yml -e security_hardening_enabled=false
```
- для обратного включения:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_site.yml -e security_hardening_enabled=true
```

## 7. Проверка hostname по inventory
Системные hostname должны совпадать с именами узлов в `inventory_hostname`.

Проверка:
```bash
ansible -i k8s/inventories/prod/hosts.ini all -m command -a hostname
```

Ожидается:
- `k8s-scp-01`, `k8s-scp-02`, `k8s-scp-03`
- `k8s-wkn-01`, `k8s-wkn-02`, `k8s-wkn-03`
- `k8s-mlb-01`, `k8s-mlb-02`, `k8s-mlb-03`
- `k8s-nfs-01`

## 7.1 Проверка `/etc/hosts` на узлах
Роль `base_os` управляет `/etc/hosts` блоком `ANSIBLE MANAGED INVENTORY HOSTS`.

Проверка:
```bash
ansible -i k8s/inventories/prod/hosts.ini all -m shell -a "sed -n '/ANSIBLE MANAGED INVENTORY HOSTS/,+15p' /etc/hosts"
```

Ожидается:
- в управляемом блоке присутствуют соответствия `IP -> inventory hostname` для всех узлов кластера;
- обновление выполняется из inventory (при изменении IP/hostname в `k8s/inventories/prod/hosts.ini` блок перегенерируется при следующем запуске `bootstrap`).

## 8. Проверка идемпотентности
После успешного деплоя выполнить повторный запуск:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_site.yml
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
7. SELinux/firewalld соответствуют текущему контуру (в диагностическом режиме `firewalld` отключен).
8. Повторный прогон плейбука не создает нежелательных изменений.

## 10. MetalLB L2 без доступа к vCenter
Ограничение: сетевые изменения в `vCenter` недоступны.

Что обязательно проверить до публикации:
1. Для `node_ip` режима: клиенты имеют маршрут/доступ к ingress-нодам `k8s-mlb-*` в сети `10.255.106.0/26`.
2. Для `node_ip` режима: на межсетевых экранах разрешен доступ на IP ingress-нод по `80/443`.
3. Для `LoadBalancer` режима: VIP-пул `10.255.106.21-10.255.106.30` зарезервирован и не выдается DHCP.
4. Для `LoadBalancer` режима: DNS указывает на ingress VIP.

Текущий ingress-режим проекта: `DaemonSet + hostNetwork` (`node_ip`), без обязательного `LoadBalancer`-VIP.

Проверка публикации в текущем режиме:
```bash
# DaemonSet ingress контроллера должен быть готов
kubectl --kubeconfig /etc/kubernetes/admin.conf -n ingress-nginx get ds ingress-nginx-controller -o wide

# Поды ingress должны быть размещены на выделенных ingress-нодах
kubectl --kubeconfig /etc/kubernetes/admin.conf -n ingress-nginx get pods -o wide -l app.kubernetes.io/component=controller

# Проверка доступности через node IP ingress-нод (пример)
curl -I http://10.255.106.16
curl -I http://10.255.106.17
curl -I http://10.255.106.18
```

Опциональный режим `LoadBalancer` (режим совместимости):
```bash
# Используется только при явном переключении validation_ingress_validation_mode=loadbalancer
kubectl --kubeconfig /etc/kubernetes/admin.conf -n ingress-nginx get svc ingress-nginx-controller -o wide
```

Автоматизация в `k8s/playbooks/k8s_validate.yml`:
- Роль `validation` проверяет доступность API через control-plane VIP (`/readyz` через `kubectl --server=https://10.255.106.20:8443`).
- Роль `validation` проверяет `keepalived/haproxy` на всех control-plane нодах и наличие ровно одного владельца VIP.
- Роль `validation` в текущем режиме `node_ip` проверяет готовность ingress `DaemonSet` и размещение controller-подов на ingress-нодах.
- В режиме совместимости `loadbalancer` проверяются `type=LoadBalancer`, external address и (опционально) `validation_ingress_expected_vip`.
- Опциональный failover-test VIP (по умолчанию выключен): `-e validation_enable_control_plane_vip_failover_test=true`.
- Для отключения API/VIP проверок: `-e validation_enable_api_vip_check=false -e validation_enable_control_plane_vip_state_check=false`.
- Для отключения проверки: `-e validation_enable_ingress_vip_check=false`.
- Текущее значение в inventory: `validation_enable_ingress_vip_check=true`, `validation_ingress_validation_mode=node_ip`.

Пример запуска:
```bash
# Стандартная безопасная валидация
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_validate.yml

# Стендовый failover-check control-plane VIP
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_validate.yml \
  -e validation_enable_control_plane_vip_failover_test=true
```

Запуск только failover-сценария по тегу:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_validate.yml --tags failover
```

Рекомендованный запуск failover в текущем контуре (явно включить проверку):
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_validate.yml \
  --tags failover \
  -e validation_enable_control_plane_vip_failover_test=true
```

## 11. Полезные команды диагностики
```bash
# Kubernetes
kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes -o wide
kubectl --kubeconfig /etc/kubernetes/admin.conf get pods -A
kubectl --kubeconfig /etc/kubernetes/admin.conf get svc -A
kubectl --kubeconfig /etc/kubernetes/admin.conf get storageclass

# Проверка режима firewall
ansible -i k8s/inventories/prod/hosts.ini all -m command -a "systemctl is-enabled firewalld || true"
ansible -i k8s/inventories/prod/hosts.ini all -m command -a "systemctl is-active firewalld || true"

# Control-plane VIP
ansible -i k8s/inventories/prod/hosts.ini control_plane -m command -a "systemctl is-active haproxy"
ansible -i k8s/inventories/prod/hosts.ini control_plane -m command -a "systemctl is-active keepalived"
ansible -i k8s/inventories/prod/hosts.ini control_plane -m shell -a "ip -4 addr show | grep -F 10.255.106.20 || true"
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
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml
```

Если на этапе runtime возникает `Depsolve Error` по `cri-tools`:
1. Убедиться, что `container_runtime_install_cri_tools: false`.
2. Перезапустить bootstrap:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml
```

Если на этапе `kubernetes_core` возникает `Cannot download, all mirrors were already tried`:
1. Проверить доступность repo metadata на хосте:
```bash
sudo dnf -q makecache --refresh --disablerepo='*' --enablerepo=kubernetes
```
2. Проверить, что включен fallback по `nobest`:
- `kubernetes_package_install_nobest: true`
3. При недоступности нужного patch-релиза временно указать доступную версию:
- `kubernetes_packages_version_override: "1.30.1-150500.1.1"` (пример)
4. Повторить bootstrap:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml
```

Если `kubeadm init` падает на `wait-control-plane` (`kubelet 10248`):
1. В текущей версии роли включен авто-сбор диагностики (`kubelet/runtime state`, `journalctl -u kubelet`, `crictl ps`) в `rescue`.
2. Проверить в выводе:
   - что swap выключен;
   - что runtime-сокет доступен (`/run/containerd/containerd.sock`);
   - что `kubelet` в `active` и не падает по cgroup/конфигу.
3. После исправления причины перезапустить:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml --tags k8s
```

Если в `kube-system events` есть `FailedCreatePodSandBox` с `pause:3.9` и `registry.k8s.io ... Forbidden`:
1. Проверить применение proxy-env к systemd-сервисам:
```bash
ansible -i k8s/inventories/prod/hosts.ini all -m shell -a "systemctl show containerd -p Environment --no-pager"
ansible -i k8s/inventories/prod/hosts.ini all -m shell -a "systemctl show kubelet -p Environment --no-pager"
```
2. Проверить, что включен mirror `registry.k8s.io` для `containerd`:
```bash
ansible -i k8s/inventories/prod/hosts.ini all -m shell -a "cat /etc/containerd/certs.d/registry.k8s.io/hosts.toml || true"
ansible -i k8s/inventories/prod/hosts.ini all -m shell -a "grep -n 'config_path' /etc/containerd/config.toml"
```
3. Убедиться в inventory:
- `containerd_registry_k8s_io_mirror_enabled: true`
- `containerd_registry_k8s_io_mirror_endpoint: https://cr.yandex/mirror/k8s.gcr.io`
4. Перезапустить runtime и сетевой этап:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml --tags proxy,runtime,network
```

Если на этапе `networking` возникает таймаут `calico-node rollout`:
1. В текущей версии роли включен controlled self-heal:
   - первичный `rollout status`;
   - при таймауте: сбор диагностик (`get pods`, `describe ds`, `get events`);
   - автоматический `rollout restart daemonset/calico-node` и повторный `rollout status`.
2. Проверить параметры ожидания:
- `network_cni_rollout_timeout: "600s"`
- `network_cni_rollout_retries: 3`
- `network_cni_rollout_retry_after_restart: true`
- `network_cni_rollout_timeout_retry: "600s"`
3. Повторить bootstrap:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml --tags network
```

Если failover-валидация падала с `validation_failover_source_host is undefined`:
1. Убедиться, что используется актуальная версия роли `validation` с guard/fallback для `delegate_to`.
2. Запускать failover по команде с явным флагом:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_validate.yml \
  --tags failover \
  -e validation_enable_control_plane_vip_failover_test=true
```
3. Проверить, что в конце play `failed=0`, а `keepalived` на source-host возвращен в `started`.

## 12. Опция data-диска для NFS
Если нужен отдельный диск под `nfs_export_path`, настройте в `k8s/inventories/prod/group_vars/nfs.yml`:
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
- если указан `storage_nfs_data_disk_device`, но `storage_nfs_data_disk_enabled=false`, playbook завершится ошибкой (защита от silent-skip).
- после монтирования выполняется проверка, что `nfs_export_path` смонтирован именно с ожидаемого partition.

## 13. Рекомендованный порядок запуска
1. `k8s/playbooks/k8s_bootstrap.yml`
2. `k8s/playbooks/k8s_metallb.yml`
3. `k8s/playbooks/k8s_hardening.yml`
4. `k8s/playbooks/k8s_storage.yml`
5. `k8s/playbooks/k8s_validate.yml`

Отдельный запуск только этапа MetalLB:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_metallb.yml
```

Либо единым запуском:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_site.yml
```

## 13.1 Масштабирование существующего кластера (добавление нод)
Используйте отдельный playbook `k8s/playbooks/k8s_scale_out.yml`.

Шаги:
1. Добавьте новые хосты в inventory в их целевые роли (`control_plane` или `workers` или `metallb`).
2. Добавьте эти же хосты в группу `scale_out_nodes`.
3. Запустите:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_scale_out.yml
```

Ключевые особенности:
- playbook валидирует, что `scale_out_nodes` не пуст и что все хосты в нем входят в одну из групп `control_plane/workers/metallb`;
- выполняет подготовку ОС/runtime только для `scale_out_nodes`;
- выполняет join через `control_plane[0]` + целевые новые ноды;
- автоматически пересобирает `control_plane_vip`, если добавляются control-plane ноды;
- автоматически пересобирает `metallb`, если добавляются metallb-ноды.

Защитный механизм:
- если на целевой ноде уже существует `/etc/kubernetes/kubelet.conf`, playbook завершится с ошибкой;
- принудительный режим допускается только с `-e scale_out_allow_reprepare_existing_nodes=true`.

Сценарий replacement-ноды (старая нода удалена из кластера, новая VM поднята с тем же hostname/IP):
- запускайте `scale_out` с `-e scale_out_allow_reprepare_existing_nodes=true`;
- playbook выполнит cleanup stale state (`kubeadm reset`, очистка `/etc/kubernetes`, `/var/lib/kubelet`, CNI-state), затем выполнит повторный join.

Пример:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_scale_out.yml \
  -e scale_out_allow_reprepare_existing_nodes=true
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
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_reinstall_cluster_and_nfs.yml \
  -e reinstall_scope=cluster_and_nfs \
  -e reinstall_confirm_token=DESTROY_CLUSTER_AND_NFS_DATA
```

После выполнения:
1. Повторно развернуть кластер через `k8s/playbooks/k8s_site.yml`.
2. Проверить, что `StorageClass nfs-sp` пересоздан и PVC создаются заново.
3. Проверить readiness всех нод и системных подов.

## 15. Оперативное управление proxy-режимом
Для переключения proxy без полного bootstrap используйте отдельный playbook `k8s/playbooks/k8s_manage_proxy.yml`.

Включить proxy (применить profile/dnf/systemd proxy конфигурацию):
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_manage_proxy.yml -e proxy_state=present
```

Отключить proxy (удалить profile/dnf/systemd proxy конфигурацию):
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_manage_proxy.yml -e proxy_state=absent
```

Быстрая проверка после переключения:
```bash
ansible -i k8s/inventories/prod/hosts.ini all -m shell -a "systemctl show kubelet -p Environment --no-pager"
ansible -i k8s/inventories/prod/hosts.ini all -m shell -a "grep -n 'ANSIBLE MANAGED PROXY' /etc/dnf/dnf.conf || true"
```

## 16. Автодеплой приложений: Prometheus, GitLab Runner, k8tz
Автодеплой выполняется из `k8s/playbooks/k8s_bootstrap.yml` на `control_plane[0]` и управляется переменными в `k8s/inventories/prod/group_vars/all.yml`:
- `deploy_prometheus_stack: true|false`
- `deploy_gitlab_runner: true|false`
- `deploy_k8tz: true|false`

### 16.1 Настройка параметров деплоя
Ключевые переменные:
- `prometheus_stack_*`: repo/chart/version, namespace/release, ingress hosts.
- `gitlab_runner_*`: `gitlab_runner_gitlab_url`, `gitlab_runner_registration_token`, теги, parallelism.
- `k8tz_*`: repo/chart/version, namespace/release, `k8tz_timezone`.

Секреты (GitLab runner token) храните в vault-файле.

Создать vault-файл из примера:
```bash
cp k8s/inventories/prod/group_vars/vault.yml.example k8s/inventories/prod/group_vars/vault.yml
ansible-vault encrypt k8s/inventories/prod/group_vars/vault.yml
ansible-vault edit k8s/inventories/prod/group_vars/vault.yml
```

### 16.2 Запуск только новых приложений по тегам
Только Prometheus:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml --tags prometheus
```

Только GitLab Runner:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml --tags gitlab-runner
```

Только k8tz:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml --tags k8tz
```

Все три приложения:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml --tags observability,gitlab-runner,k8tz
```

Пример запуска с vault:
```bash
ansible-playbook -i k8s/inventories/prod/hosts.ini k8s/playbooks/k8s_bootstrap.yml \
  --tags gitlab-runner \
  --ask-vault-pass
```
