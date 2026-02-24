# changelog.md

## 2026-02-19
### Добавлено
- Создан каталог документации `docs/`.
- Добавлен документ архитектуры проекта: `docs/Project.md`.
- Добавлен трекер задач с приоритетами: `docs/Tasktracker.md`.
- Добавлен проектный дневник наблюдений: `docs/Diary.md`.
- Добавлен перечень уточняющих вопросов: `docs/qa.md`.

### Изменено
- Зафиксирован процесс разработки: сначала сбор входных данных, затем реализация Ansible-плейбука.

## 2026-02-19 (сбор входных данных)
### Изменено
- Обновлен `docs/Project.md`: добавлен раздел 12 с зафиксированными параметрами среды (VMware, сеть, Kubernetes, MetalLB, NFS, security, ops).
- Обновлен `docs/Tasktracker.md`: отражен прогресс `T-002` (80%), добавлены блокирующие уточнения перед стартом реализации.
- Обновлен `docs/qa.md`: вопросы переведены в статусный формат (закрытые ответы, открытые блокеры, варианты решений).
- Обновлен `docs/Diary.md`: добавлена запись с наблюдениями и решениями по результатам сбора данных.

## 2026-02-19 (уточнение блокеров)
### Изменено
- Обновлен `docs/Project.md`: зафиксированы `management_subnet_cidr`, proxy-модель `local_on_each_node`, полный формат proxy URL, label/taints для MetalLB-нод.
- Обновлен `docs/qa.md`: закрыты вопросы по management subnet/proxy/metallb placement, оставлены только незаполненные финальные блокеры.
- Обновлен `docs/Tasktracker.md`: прогресс `T-002` повышен до `90%`, отмечены закрытые подпункты по сети и прокси.
- Обновлен `docs/Diary.md`: добавлена запись по новым подтвержденным параметрам и оставшимся блокерам.

## 2026-02-19 (закрытие финальных блокеров)
### Изменено
- Обновлен `docs/Project.md`: зафиксированы production-требования NFS (`IOPS >= 2000`, `>=150 MB/s`, `<=5 ms`), стратегия backup (`daily snapshot`, `retention 14 days`), `require_check_diff_support: yes`, расширенные acceptance criteria.
- Обновлен `docs/qa.md`: блокеры закрыты, зафиксирован выбранный вариант 3.
- Обновлен `docs/Tasktracker.md`: задача `T-002` переведена в `Завершена`, чек-лист сбора данных закрыт.
- Обновлен `docs/Diary.md`: добавлена запись о завершении этапа сбора данных.

## 2026-02-19 (старт реализации каркаса)
### Изменено
- Обновлен `docs/Tasktracker.md`: зафиксирован старт этапа реализации Ansible-каркаса.
- Задачи `T-003` и `T-004` переведены в статус `В процессе`.

## 2026-02-19 (реализация Ansible-каркаса)
### Добавлено
- Создан `inventories/prod/hosts.yml` с группами `control_plane`, `workers`, `metallb`, `nfs`, `k8s_cluster`.
- Созданы файлы `inventories/prod/group_vars/{all,control_plane,workers,metallb,nfs}.yml`.
- Созданы плейбуки `playbooks/{site,bootstrap,hardening,storage,validate}.yml`.
- Созданы роли-каркасы с `defaults`, `tasks`, `handlers` для `base_os`, `proxy`, `container_runtime`, `kubernetes_core`, `networking`, `metallb`, `security_hardening`, `storage_nfs`, `validation`.
- Добавлены `ansible.cfg` и `requirements.yml`.

### Изменено
- Обновлен `docs/Tasktracker.md`: задачи `T-003` и `T-004` переведены в `Завершена`.

## 2026-02-19 (старт реализации ролей)
### Изменено
- Обновлен `docs/Tasktracker.md`: задачи `T-005`...`T-012` переведены в `В процессе`.
- Зафиксирован переход от каркаса к реализации рабочей логики ролей.

## 2026-02-19 (реализация логики ролей)
### Изменено
- Реализована роль `base_os`: пакеты, modules-load, sysctl, swap, chrony.
- Реализована роль `proxy`: профили окружения, `dnf` proxy, systemd drop-in для `containerd/kubelet/crio`.
- Реализована роль `container_runtime`: установка runtime, конфиг `containerd`, `crictl`, управление сервисом.
- Реализована роль `kubernetes_core`: установка компонентов Kubernetes, `kubeadm init/join`, подготовка kubeconfig.
- Реализованы роли `networking`, `metallb`, `storage_nfs`, `security_hardening`, `validation` с базовой рабочей логикой.
- Обновлен `playbooks/bootstrap.yml`: роль `kubernetes_core` запускается на всей группе `k8s_cluster`.
- Обновлен `inventories/prod/group_vars/all.yml`: дополнен `no_proxy`, добавлены runtime-переменные.
- Обновлен `docs/Tasktracker.md`: задачи `T-005`...`T-012` переведены в `Завершена (без стендовой валидации)`.

## 2026-02-19 (доработка параметризации сети)
### Изменено
- В архитектурных требованиях зафиксирована параметризация включения proxy для неизолированных контуров.
- В трекер добавлена задача `T-017`: hostname из inventory + переключаемый proxy.

## 2026-02-19 (hostname + switchable proxy)
### Изменено
- Обновлена роль `base_os`: добавлена установка системного hostname по `inventory_hostname` и управление `/etc/hosts` из inventory.
- Обновлен `group_vars/all.yml`: добавлен параметр `proxy_enabled` и условная карта `proxy_environment`.
- Обновлен `playbooks/bootstrap.yml`: роль `proxy` теперь запускается только при `proxy_enabled=true`.
- Обновлены роли `networking`, `metallb`, `storage_nfs`: proxy-переменные для `kubectl` применяются условно через `proxy_environment`.
- Обновлен `docs/Tasktracker.md`: `T-017` переведена в `Завершена (без стендовой валидации)`.

## 2026-02-19 (старт подготовки runbook)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-018` в статусе `В процессе`.

## 2026-02-19 (runbook для control host)
### Добавлено
- Добавлен `docs/runbook.md` с пошаговым сценарием запуска на control host:
  - подготовка окружения;
  - режим `proxy_enabled=true` (изолированная сеть);
  - режим `proxy_enabled=false` (неизолированная сеть);
  - проверка hostname по inventory;
  - checklist идемпотентности и приемки.

### Изменено
- Обновлен `docs/Tasktracker.md`: задача `T-018` переведена в `Завершена`.

## 2026-02-19 (деструктивный reinstall cluster_and_nfs)
### Добавлено
- Добавлен `playbooks/reinstall_cluster_and_nfs.yml` для полной переустановки с удалением Kubernetes-state и очисткой NFS данных.
- В `docs/runbook.md` добавлен отдельный раздел запуска деструктивного сценария с guardrail-параметрами подтверждения.

### Изменено
- Обновлен `docs/Project.md`: зафиксирован отдельный playbook переустановки.
- Обновлен `docs/Tasktracker.md`: задача `T-019` переведена в `Завершена (без стендовой валидации)`.

## 2026-02-19 (старт NFS disk option)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-020` в статусе `В процессе`.
- Обновлен `docs/Project.md`: зафиксировано требование опциональной разметки/монтирования выделенного диска для NFS.

## 2026-02-19 (NFS data disk option)
### Изменено
- Обновлен `roles/storage_nfs/defaults/main.yml`: добавлены параметры управления выделенным data-диском.
- Обновлен `inventories/prod/group_vars/nfs.yml`: добавлены переменные конфигурации data-диска для NFS.
- Обновлен `roles/storage_nfs/tasks/main.yml`: реализованы шаги разметки диска, создания ФС и монтирования в `nfs_export_path` (опционально).
- Обновлен `docs/runbook.md`: добавлен раздел по использованию опции data-диска.
- Обновлен `docs/Tasktracker.md`: задача `T-020` переведена в `Завершена (без стендовой валидации)`.

## 2026-02-19 (старт NFS system-disk guardrail)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-021` в статусе `В процессе`.
- Обновлен `docs/Project.md`: добавлено требование guardrail для NFS data-диска.

## 2026-02-19 (NFS system-disk guardrail)
### Изменено
- Обновлен `roles/storage_nfs/tasks/main.yml`: добавлен guardrail, блокирующий выбор системного диска для data-диска NFS по умолчанию.
- Обновлен `roles/storage_nfs/defaults/main.yml`: добавлены параметры `storage_nfs_data_disk_allow_system_disk` и `storage_nfs_data_disk_guard_mounts`.
- Обновлен `inventories/prod/group_vars/nfs.yml`: добавлен параметр `storage_nfs_data_disk_allow_system_disk`.
- Обновлен `docs/runbook.md`: добавлены инструкции по guardrail и override-поведению.
- Обновлен `docs/Tasktracker.md`: задача `T-021` переведена в `Завершена (без стендовой валидации)`.

## 2026-02-19 (MetalLB L2 без доступа к vCenter)
### Изменено
- Обновлен `docs/Project.md`: зафиксирована модель публикации ingress через VIP в `MetalLB L2` при отсутствии прав на сетевые изменения в `vCenter`.
- Обновлен `docs/runbook.md`: добавлен checklist внешних предпосылок (IPAM/DHCP, маршрутизация, firewall, DNS), команды фиксации VIP и проверки доступности.
- Обновлен `docs/qa.md`: добавлено архитектурное ограничение по отсутствию доступа к сетевым настройкам в `vCenter`.
- Обновлен `docs/Tasktracker.md`: добавлена и завершена задача `T-022`.

## 2026-02-19 (старт ingress VIP post-check)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-023` в статусе `В процессе`.

## 2026-02-19 (ingress VIP post-check)
### Изменено
- Обновлен `roles/validation/tasks/main.yml`: добавлены проверки ingress service (`type=LoadBalancer`, external address, optional expected VIP).
- Обновлен `roles/validation/defaults/main.yml`: добавлены параметры `validation_enable_ingress_vip_check`, `validation_ingress_namespace`, `validation_ingress_service_name`, `validation_ingress_expected_vip`.
- Обновлен `inventories/prod/group_vars/all.yml`: зафиксированы параметры проверки ingress VIP и ожидаемый VIP `10.255.106.21`.
- Обновлен `docs/Project.md`: добавлено описание автоматического ingress VIP post-check в архитектурной части.
- Обновлен `docs/runbook.md`: добавлено описание автоматизации проверки в `playbooks/validate.yml`.
- Обновлен `docs/Tasktracker.md`: задача `T-023` переведена в `Завершена (без стендовой валидации)`.

## 2026-02-19 (старт control-plane VIP keepalived+haproxy)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-024` в статусе `В процессе`.

## 2026-02-19 (control-plane VIP keepalived+haproxy)
### Изменено
- Добавлена роль `roles/control_plane_vip` (установка `keepalived/haproxy`, настройка VRRP unicast, VIP listener для Kubernetes API).
- Обновлен `playbooks/bootstrap.yml`: добавлен запуск роли `control_plane_vip` до `kubernetes_core`.
- Обновлен `roles/kubernetes_core/templates/kubeadm-config.yaml.j2`: `controlPlaneEndpoint` параметризован через `control_plane_endpoint_port`.
- Обновлены `inventories/prod/group_vars/all.yml` и `inventories/prod/group_vars/control_plane.yml`: включен control-plane VIP, задан endpoint port `8443`, добавлены VRRP/firewall параметры.
- Обновлены `roles/security_hardening/defaults/main.yml` и `roles/security_hardening/tasks/main.yml`: добавлена поддержка `firewall_allowed_protocols` (для `vrrp`).
- Обновлены `docs/Project.md`, `docs/runbook.md`, `docs/qa.md`: зафиксирована архитектура и эксплуатация control-plane VIP.
- Обновлен `docs/Tasktracker.md`: задача `T-024` переведена в `Завершена (без стендовой валидации)`.

## 2026-02-19 (старт validation API VIP + failover-check)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-025` в статусе `В процессе`.

## 2026-02-19 (validation API VIP + failover-check)
### Изменено
- Обновлен `roles/validation/defaults/main.yml`: добавлены параметры API VIP-проверки и опционального failover-test.
- Обновлен `roles/validation/tasks/main.yml`: добавлены проверки API `readyz` через `control_plane_endpoint`, проверки `keepalived/haproxy` на всех control-plane узлах, проверка единственного владельца VIP и опциональный failover-test.
- Обновлен `docs/runbook.md`: добавлены флаги запуска валидации для API VIP и failover-test.
- Обновлен `docs/Project.md`: отражены новые проверки в архитектурном описании и статусе реализации.
- Обновлен `docs/Tasktracker.md`: задача `T-025` переведена в `Завершена (без стендовой валидации)`.

## 2026-02-19 (старт failover tag в validate.yml)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-026` в статусе `В процессе`.

## 2026-02-19 (failover tag в validate.yml)
### Изменено
- Обновлен `playbooks/validate.yml`: добавлен отдельный on-demand play c тегами `never,failover` для запуска только failover-сценария роли `validation`.
- Обновлен `docs/runbook.md`: добавлена команда запуска `ansible-playbook ... --tags failover`.
- Обновлен `docs/Project.md`: зафиксирован механизм on-demand failover-проверки по тегу.
- Обновлен `docs/Tasktracker.md`: задача `T-026` переведена в `Завершена`.

## 2026-02-19 (старт kubeadm config API compatibility fix)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-027` в статусе `В процессе`.

## 2026-02-19 (kubeadm config API compatibility fix)
### Изменено
- Обновлен `roles/kubernetes_core/defaults/main.yml`: добавлены параметры `kubeadm_config_api_version` и `kubeadm_init_allow_experimental_api`.
- Обновлен `roles/kubernetes_core/templates/kubeadm-config.yaml.j2`: `apiVersion` переведен на параметр `kubeadm_config_api_version`.
- Обновлен `roles/kubernetes_core/tasks/main.yml`: добавлена проверка переменной API-версии и опциональный флаг `--allow-experimental-api`.
- Обновлен `inventories/prod/group_vars/all.yml`: зафиксированы безопасные значения `kubeadm_config_api_version: kubeadm.k8s.io/v1beta3` и `kubeadm_init_allow_experimental_api: false`.
- Обновлены `docs/Project.md` и `docs/runbook.md`: добавлены эксплуатационные инструкции по совместимости kubeadm API.
- Обновлен `docs/Tasktracker.md`: задача `T-027` переведена в `Завершена`.

## 2026-02-19 (старт container runtime depsolve fix)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-028` в статусе `В процессе`.

## 2026-02-19 (container runtime depsolve fix)
### Изменено
- Обновлен `roles/container_runtime/defaults/main.yml`: установка `cri-tools` вынесена в опциональные параметры (`container_runtime_install_cri_tools`, `container_runtime_cri_tools_package`), из дефолтного списка runtime-пакетов удален `cri-tools`.
- Обновлен `roles/container_runtime/tasks/main.yml`: добавлен probe `crictl` и условная установка `cri-tools` только при явном включении.
- Обновлен `inventories/prod/group_vars/all.yml`: зафиксирован безопасный default `container_runtime_install_cri_tools: false`.
- Обновлены `docs/Project.md` и `docs/runbook.md`: добавлены указания по runtime packaging policy и troubleshooting depsolve для `cri-tools`.
- Обновлен `docs/Tasktracker.md`: задача `T-028` переведена в `Завершена`.

## 2026-02-19 (старт k8s repo resilience fix)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-029` в статусе `В процессе`.

## 2026-02-19 (k8s repo resilience fix)
### Изменено
- Обновлен `roles/kubernetes_core/defaults/main.yml`: добавлены параметры `kubernetes_packages_version_override`, `kubernetes_repo_validate_*`, `kubernetes_package_install_*`.
- Обновлен `roles/kubernetes_core/tasks/main.yml`: добавлены шаги precheck repo (`dnf makecache` c retry), вычисление списка устанавливаемых пакетов с учетом override-версии и retry установки пакетов.
- Обновлен `inventories/prod/group_vars/all.yml`: зафиксированы параметры resilience для Kubernetes package install.
- Обновлены `docs/Project.md` и `docs/runbook.md`: добавлены правила эксплуатации и troubleshooting для недоступности зеркал Kubernetes repo.
- Обновлен `docs/Tasktracker.md`: задача `T-029` переведена в `Завершена`.

## 2026-02-19 (старт kubeadm init diagnostics fix)
### Изменено
- Обновлен `docs/Tasktracker.md`: добавлена задача `T-030` в статусе `В процессе`.

## 2026-02-19 (kubeadm init diagnostics fix)
### Изменено
- Обновлен `roles/kubernetes_core/tasks/main.yml`: добавлены preflight-проверки перед `kubeadm init` (запуск runtime service, ожидание runtime socket, проверка выключенного swap).
- Обновлен `roles/kubernetes_core/tasks/main.yml`: `kubeadm init` переведен в `block/rescue` с автоматическим сбором диагностики (`systemctl is-active kubelet/runtime`, `journalctl -u kubelet`, `crictl ps`).
- Обновлен `docs/runbook.md`: добавлен troubleshooting-раздел для сбоя `wait-control-plane` (`127.0.0.1:10248`) и команды повторного запуска.
- Обновлен `docs/Tasktracker.md`: задача `T-030` переведена в `Завершена (без стендовой валидации)`.

## 2026-02-24 (kubeadm init endpoint/idempotency hardening)
### Изменено
- Обновлен `roles/kubernetes_core/defaults/main.yml`: добавлены параметры управления precheck endpoint (`kubeadm_init_endpoint_check_*`) и управляемой детализацией логов `kubeadm` (`kubeadm_init_verbose`, `kubeadm_init_verbosity`).
- Обновлен `roles/kubernetes_core/tasks/main.yml`: перед `kubeadm init` добавлена проверка доступности `controlPlaneEndpoint` (`wait_for` по `host:port`) с retry.
- Обновлен `roles/kubernetes_core/tasks/main.yml`: idempotency `kubeadm init` переведена с `creates` на проверку фактической готовности API (`kubectl ... /readyz`) для исключения ложного пропуска после частичного init.
- Обновлен `roles/kubernetes_core/tasks/main.yml`: расширена диагностика `rescue` (увеличен tail `kubelet` logs, добавлен контекст pre-init проверок в fail message).

## 2026-02-24 (kubeadm init VIP backend guard)
### Изменено
- Обновлен `roles/kubernetes_core/tasks/main.yml`: в `rescue` добавлен probe доступности backend control-plane узлов на `:6443`.
- Обновлен `roles/kubernetes_core/tasks/main.yml`: добавлен явный `fail` для случая, когда `controlPlaneEndpoint` доступен, но ни один backend API не отвечает, с диагностическим сообщением по VIP/HAProxy-цепочке.
- Обновлен `docs/Tasktracker.md`: задача `T-032` переведена в `Завершена (без стендовой валидации)`.

## 2026-02-24 (ansible callback cleanup)
### Изменено
- Обновлен `ansible.cfg`: удален параметр `stdout_callback = yaml` как избыточный для актуальных версий Ansible.
- Обновлен `docs/Tasktracker.md`: добавлена и завершена задача `T-033`.
