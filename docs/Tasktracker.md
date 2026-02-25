## Задача: План и трекинг реализации Ansible-плейбука Kubernetes на RedOS
- **Статус**: В процессе
- **Описание**: Отслеживание прогресса по этапам реализации, контроль приоритетов и зависимостей по архитектуре из Project.md.

# Tasktracker.md

## Текущий статус проекта
- Общий статус: **В процессе**
- Текущий этап: **Готовность к валидации на control host**
- Следующий этап: **Стендовые проверки, идемпотентность, устранение дефектов**

## Бэклог и прогресс

| ID | Раздел | Задача | Приоритет | Статус |
|---|---|---|---|---|
| T-001 | Архитектура | Подготовить и согласовать `docs/Project.md` | Критический | Завершена |
| T-002 | Аналитика | Собрать недостающие входные данные (сеть, прокси, DNS/NTP, vCenter, версии) | Критический | Завершена |
| T-003 | Инвентори | Подготовить `inventories/prod/hosts.yml` с группами `control_plane`, `workers`, `metallb`, `nfs` | Критический | Завершена |
| T-004 | Переменные | Сформировать `group_vars` для прокси, сети, k8s, MetalLB, NFS | Критический | Завершена |
| T-005 | Базовая ОС | Реализовать роль `base_os` для RedOS 8.0.2 | Высокий | Завершена (без стендовой валидации) |
| T-006 | Прокси | Реализовать роль `proxy` (dnf/systemd/runtime) | Критический | Завершена (без стендовой валидации) |
| T-007 | Runtime | Реализовать роль `container_runtime` (containerd/CRI-O) | Критический | Завершена (без стендовой валидации) |
| T-008 | Kubernetes | Реализовать роль `kubernetes_core` (init/join/control-plane/worker) | Критический | Завершена (без стендовой валидации) |
| T-009 | CNI | Внедрить сетевой плагин и базовые политики | Высокий | Завершена (без стендовой валидации) |
| T-010 | MetalLB | Развернуть MetalLB и адресные пулы | Критический | Завершена (без стендовой валидации) |
| T-011 | NFS | Настроить NFS и StorageClass/PV/PVC | Критический | Завершена (без стендовой валидации) |
| T-012 | Безопасность | Настроить firewalld и SELinux правила | Критический | Завершена (без стендовой валидации) |
| T-013 | Идемпотентность | Проверить повторный прогон без нежелательных изменений | Высокий | Не начата |
| T-014 | Качество | Настроить `ansible-lint`, `yamllint`, pre-commit | Средний | Не начата |
| T-015 | Валидация | Добавить smoke-тесты и post-checks | Высокий | Не начата |
| T-016 | Документация | Поддерживать `Project.md`, `Tasktracker.md`, `Diary.md`, `changelog.md` в актуальном состоянии | Высокий | В процессе |
| T-017 | Унификация | Синхронизировать hostname узлов с именами из inventory + параметризовать включение proxy | Критический | Завершена (без стендовой валидации) |
| T-018 | Эксплуатация | Подготовить runbook для запуска на control host (режимы с proxy/без proxy + checklist) | Высокий | Завершена |
| T-019 | Reinstall | Добавить деструктивный сценарий полной переустановки кластера (`cluster_and_nfs`) | Критический | Завершена (без стендовой валидации) |
| T-020 | NFS Disk | Добавить опцию выбора диска для разметки и монтирования под NFS export path | Критический | Завершена (без стендовой валидации) |
| T-021 | NFS Guardrail | Добавить защиту от использования системного диска в NFS data-disk сценарии | Критический | Завершена (без стендовой валидации) |
| T-022 | MetalLB Ops | Зафиксировать сценарий публикации ingress VIP в L2 без доступа к vCenter | Высокий | Завершена |
| T-023 | Validation | Добавить автоматизированный post-check ingress VIP (`LoadBalancer`/EXTERNAL-IP) | Высокий | Завершена (без стендовой валидации) |
| T-024 | Control Plane HA | Реализовать control-plane endpoint VIP через `keepalived + haproxy` | Критический | Завершена (без стендовой валидации) |
| T-025 | Validation API HA | Добавить автоматическую проверку API VIP (`10.255.106.20:8443`) и базовый failover-check сценарий | Критический | Завершена (без стендовой валидации) |
| T-026 | Validation Tagging | Добавить отдельный тег `failover` для запуска только failover-сценария валидации | Высокий | Завершена |
| T-027 | Kubeadm Compat | Исправить совместимость `kubeadm` config API (`v1beta4` -> совместимый параметризуемый вариант) | Критический | Завершена |
| T-028 | Runtime Depsolve | Устранить конфликт `cri-tools` при установке runtime-пакетов на RedOS | Критический | Завершена |
| T-029 | K8s Repo Resilience | Добавить precheck/retry для Kubernetes repo и параметр override версии пакетов | Критический | Завершена |
| T-030 | Kubeadm Init Diagnostics | Добавить preflight и rescue-диагностику для сбоя `wait-control-plane` (kubelet 10248) | Критический | Завершена (без стендовой валидации) |
| T-031 | Kubeadm Init Hardening | Добавить precheck доступности controlPlaneEndpoint и исправить idempotency `kubeadm init` по `readyz` | Критический | Завершена (без стендовой валидации) |
| T-032 | VIP Backend Guard | Добавить явный guard для `kubeadm init`: endpoint доступен, но backend `:6443` недоступны | Критический | Завершена (без стендовой валидации) |
| T-033 | Ansible Callback Cleanup | Удалить `stdout_callback = yaml` из `ansible.cfg` для совместимости с актуальным Ansible | Высокий | Завершена |
| T-034 | Kubeadm Stale-State Guard | Добавить precheck/обработку частично инициализированного control-plane перед повторным `kubeadm init` | Критический | Завершена (без стендовой валидации) |
| T-035 | Proxy NO_PROXY Hardening | Добавить явные IP control-plane endpoint/нод в `no_proxy` для исключения обращения kubelet через proxy | Критический | Завершена (без стендовой валидации) |
| T-036 | Control Plane VIP SELinux Hardening | Добавить idempotent-настройку `haproxy_connect_any` и безопасный post-check VIP в роли `control_plane_vip` | Критический | Завершена (без стендовой валидации) |
| T-037 | SELinux Mode Input Normalization | Нормализовать `selinux_target_mode` к lowercase в `security_hardening`, сохранив совместимость со значениями вида `Enforcing` | Высокий | Завершена (без стендовой валидации) |
| T-038 | Kubeadm Join Resilience Hardening | Добавить precheck endpoint + retry и последовательный join для control-plane/worker узлов | Критический | Завершена (без стендовой валидации) |
| T-039 | Kubeadm Join Endpoint Fallback | Добавить fallback `primary_control_plane:6443` для `kubeadm join` при недоступности VIP endpoint | Критический | Завершена (без стендовой валидации) |
| T-040 | SELinux Temporary Disable | Временно отключить SELinux на всех узлах для диагностики нестабильности join/VIP | Высокий | Завершена (без стендовой валидации) |
| T-041 | Kubeadm Join Forced Primary Endpoint | Принудительно выполнять `kubeadm join` через `primary_control_plane:6443` без VIP precheck в проблемном контуре | Критический | Завершена (без стендовой валидации) |
| T-042 | Join Endpoint Diagnostics | Добавить auto-диагностику сети при падении `wait_for` выбранного `kubeadm join` endpoint | Высокий | Завершена (без стендовой валидации) |
| T-043 | Bootstrap Security Sequencing | Добавить запуск `security_hardening` в `bootstrap.yml` перед `kubernetes_core` для гарантии портов/API reachability | Критический | Завершена (без стендовой валидации) |
| T-044 | Calico Rollout Diagnostics Hardening | Добавить retry/timeout и auto-диагностику в `networking` при таймауте rollout `calico-node` | Высокий | Завершена (без стендовой валидации) |
| T-045 | Calico VXLAN Migration | Переключить Calico с BGP на VXLAN в Ansible-автоматизации для устранения зависимости от BGP peering | Критический | Завершена (без стендовой валидации) |
| T-046 | Calico VXLAN Full Migration | Убрать зависимость `calico-node` от BIRD в VXLAN-режиме (`CLUSTER_TYPE`, probes) | Критический | Завершена (без стендовой валидации) |
| T-047 | MetalLB Webhook Readiness Hardening | Добавить precheck webhook endpoints и retry/rescue-диагностику для `metallb` pool apply | Высокий | Завершена (без стендовой валидации) |
| T-048 | MetalLB Webhook FailurePolicy Workaround | Добавить временный patch `failurePolicy=Ignore` для webhook MetalLB на время `pool apply` с последующим возвратом в `Fail` | Высокий | Завершена (без стендовой валидации) |
| T-049 | MetalLB Webhook Delete/Recreate Workaround | Временно удалять `ValidatingWebhookConfiguration` MetalLB на время `pool apply` и восстанавливать после | Высокий | Завершена (без стендовой валидации) |
| T-050 | MetalLB Webhook Restore Proxy Fix | Добавить proxy environment в шаг восстановления webhook-конфигурации MetalLB | Высокий | Завершена (без стендовой валидации) |
| T-051 | Control Plane Helm Tooling | Добавить установку `helm` и `helmfile` на control-plane ноды через `kubernetes_core` | Высокий | Завершена (без стендовой валидации) |
| T-052 | Proxy Mode Management Playbook | Добавить отдельный playbook для включения/отключения proxy-режима и доработать роль `proxy` под `present/absent` | Высокий | Завершена (без стендовой валидации) |
| T-053 | Runbook Proxy Operations | Добавить в runbook раздел по оперативному переключению proxy-режима через `playbooks/manage_proxy.yml` | Средний | Завершена |
| T-054 | Validation Failover Source Guard | Устранить падение failover-проверки при undefined `validation_failover_source_host` через безопасную инициализацию и guards | Высокий | Завершена |
| T-055 | Validation Delegate Fallback Guard | Исключить падение шаблонизации `delegate_to` в failover-блоке через безопасный fallback `default(inventory_hostname)` | Высокий | Завершена |
| T-056 | Documentation Full Sync | Полностью синхронизировать `Project.md`, `runbook.md`, `qa.md`, `Diary.md` с актуальным состоянием ролей и эксплуатационных сценариев | Высокий | Завершена |
| T-057 | Validation Ingress Diagnostics Hardening | Добавить явный диагностический вывод `kubectl get svc` (rc/stdout/stderr) перед ingress assert и расширить fail message | Высокий | Завершена |

## Собранные данные (2026-02-19)
- VMware: `vCenter 7.0.3`, `ESXi 7.0.3`, `clone_from_template`, шаблон `k8s-pcp-template`.
- Kubernetes stack: `k8s 1.30.14`, `containerd`, `calico`, `control_plane_endpoint 10.255.106.20:8443`.
- Узлы и роли: 3 `control-plane`, 3 `worker`, 3 `metallb`, 1 `nfs`; IP-адреса зафиксированы.
- MetalLB: `l2`, пул `10.255.106.21-10.255.106.30`, ingress `nginx`.
- NFS: `/srv/storage`, CIDR `10.255.106.0/26`, StorageClass `nfs-sp` по умолчанию.
- Proxy: `local_on_each_node`, `http://127.0.0.1:12334`, расширенный `no_proxy`.
- Management subnet: `10.255.106.0/26`.
- MetalLB placement: label `node-role.kubernetes.io/metallb=true`, без taints.

## Реализованный каркас (2026-02-19)
- Создан `inventories/prod/hosts.yml` с группами `control_plane`, `workers`, `metallb`, `nfs`, `k8s_cluster`.
- Созданы `group_vars` для `all`, `control_plane`, `workers`, `metallb`, `nfs`.
- Создан набор playbook-файлов: `site.yml`, `bootstrap.yml`, `hardening.yml`, `storage.yml`, `validate.yml`.
- Созданы роли-каркасы: `base_os`, `proxy`, `container_runtime`, `kubernetes_core`, `networking`, `metallb`, `security_hardening`, `storage_nfs`, `validation`.
- Добавлены `ansible.cfg` и `requirements.yml`.

## Реализованная логика ролей (2026-02-19)
- `base_os`: пакеты, modules-load, sysctl, swap off, chrony.
- `proxy`: `/etc/profile.d`, `dnf.conf`, systemd drop-in для `containerd/kubelet/crio`.
- `container_runtime`: установка runtime, `containerd` config, `crictl`, сервис.
- `kubernetes_core`: repo, пакеты, `kubeadm init`, генерация join-команд, join control-plane/worker/metallb.
- `networking`: применение CNI manifest (calico/cilium/flannel), ожидание readiness.
- `metallb`: установка компонентов, IP pool/L2 advertisement, labeling нод.
- `storage_nfs`: экспорт NFS, сервисы, `nfs-subdir-external-provisioner`, StorageClass.
- `security_hardening`: SELinux mode, firewalld и role-based порты.
- `validation`: базовые post-checks по readiness, MetalLB controller и NFS StorageClass.
- `base_os`: добавлена синхронизация системного hostname с `inventory_hostname` и управление `/etc/hosts` по inventory.
- Параметр `proxy_enabled` добавлен в `group_vars/all.yml`; роль `proxy` и proxy-env для `kubectl apply` стали условными.
- Добавлен `docs/runbook.md` с командами запуска для `proxy_enabled=true/false`, предчеками и checklist приемки.
- Добавлен `playbooks/reinstall_cluster_and_nfs.yml` для полной деструктивной переустановки (`cluster_and_nfs`) с guardrail-токеном подтверждения.
- В `storage_nfs` добавлена опция выделенного data-диска: разметка, форматирование и монтирование в `nfs_export_path`.
- Добавлен guardrail в `storage_nfs`: блокировка системного диска по умолчанию с явным override-флагом.
- В документации зафиксирован операционный сценарий MetalLB L2 без прав на сетевые изменения в `vCenter`.
- В роль `validation` добавлена автоматическая проверка ingress `LoadBalancer` и ожидаемого VIP.
- Добавлена роль `control_plane_vip`: реализация API endpoint VIP через `keepalived + haproxy` на control-plane нодах.
- Роль `validation` дополнена проверками API VIP (`/readyz`), состояния `keepalived/haproxy` и опциональным failover-test для control-plane VIP.
- В `playbooks/validate.yml` добавлен отдельный on-demand путь запуска failover-проверки по тегу `failover`.
- Исправлена совместимость bootstrap: kubeadm config API параметризован и по умолчанию переключен на `kubeadm.k8s.io/v1beta3`.
- В роли `container_runtime` установка `cri-tools` сделана опциональной и отключена по умолчанию для устранения depsolve-конфликта.
- В роли `kubernetes_core` добавлены precheck/retry для repo и установка пакетов с retry, а также override версии пакетов через `kubernetes_packages_version_override`.
- В роли `kubernetes_core` добавлены preflight перед `kubeadm init` (runtime service/socket, swap-check) и `rescue`-диагностика (`kubelet/runtime state`, `journalctl`, `crictl ps`).
- В роли `kubernetes_core` добавлены precheck доступности `controlPlaneEndpoint` перед `kubeadm init`, переключение idempotency на проверку `kubectl ... /readyz`, а также параметризованная детализация логов `kubeadm init`.
- В роли `kubernetes_core` добавлен явный guard в `rescue`: если `controlPlaneEndpoint` доступен, но все backend `control-plane` на `:6443` недоступны, play завершается с целевым сообщением о проблеме VIP/HAProxy-цепочки.
- В `ansible.cfg` удален `stdout_callback = yaml` как устаревшая настройка для актуальной версии Ansible.
- В роли `kubernetes_core` добавлен guard частично-инициализированного состояния (`/etc/kubernetes/manifests`, `/var/lib/etcd`) с опциональным auto-reset через `kubeadm_init_auto_reset_on_stale_state`.
- В `inventories/prod/group_vars/all.yml` в `no_proxy` добавлены явные IP `controlPlaneEndpoint` и control-plane нод для исключения доступа Kubernetes-компонентов к API через HTTP(S)-proxy.
- В роли `control_plane_vip` добавлена idempotent-настройка SELinux boolean `haproxy_connect_any` (persisted) на control-plane нодах при включенном SELinux.
- В роли `control_plane_vip` добавлен условный post-check API VIP (`/readyz`) после применения конфигурации, выполняемый только при наличии `admin.conf`.
- В роли `security_hardening` добавлена case-insensitive валидация `selinux_target_mode` и нормализация значения `state` в lowercase перед вызовом `ansible.posix.selinux`.
- В роли `kubernetes_core` добавлен precheck доступности `controlPlaneEndpoint` перед `kubeadm join` на non-primary нодах.
- В `kubernetes_core` join-таски переведены на retry (`until` + `retries/delay`) и последовательное выполнение (`throttle: 1`) для снижения ошибок `context deadline exceeded`/`rate limiter`.
- В роли `kubernetes_core` добавлен выбор endpoint для `kubeadm join`: сначала `controlPlaneEndpoint`, при недоступности — fallback на `primary_control_plane:6443`.
- Join-команды выполняются через адресную подмену `kubeadm join <endpoint>`, что позволяет прозрачно использовать fallback без пересоздания токена.
- В `inventories/prod/group_vars/all.yml` добавлен `selinux_target_mode: Disabled` как временный диагностический режим для всех узлов (требуется reboot для полного применения).
- В роли `kubernetes_core` добавлен режим `kubeadm_join_force_primary_endpoint`, отключающий VIP probe и принудительно выбирающий endpoint `primary_control_plane:6443` для join.
- В `inventories/prod/group_vars/all.yml` включен `kubeadm_join_force_primary_endpoint: true` для текущего troubleshooting-контура.
- В `roles/kubernetes_core/tasks/main.yml` добавлен `block/rescue` и авто-диагностика сети на joining-ноде при недоступности выбранного endpoint (`ip route get`, TCP probe, `curl /readyz`) с агрегированным fail-сообщением.
- В `playbooks/bootstrap.yml` добавлен запуск роли `security_hardening` на `k8s_cluster` до роли `kubernetes_core`, чтобы исключить повторный bootstrap без примененных firewall-правил.
- В роли `networking` ожидание `calico-node` rollout сделано параметризуемым и переведено в `block/rescue` с автосбором диагностик (`pods`, `describe ds`, `events`) при таймауте.
- В роли `networking` добавлена параметризация backend-режима Calico и применение VXLAN-параметров (`calico_backend=vxlan`, `IPIP=Never`, `VXLAN=Always`) после `kubectl apply`.
- В `inventories/prod/group_vars/all.yml` включен VXLAN-режим Calico и отключен IPIP для текущего production-контура.
- В роли `networking` добавлен patch `CLUSTER_TYPE=k8s` и felix-only health checks для `calico-node` в VXLAN-режиме, чтобы исключить ожидание BIRD и CrashLoop/Unhealthy по `bird.*`.
- В роли `metallb` добавлены precheck готовности `metallb-webhook-service` endpoints, retry при `kubectl apply -f /tmp/metallb-config.yaml` и rescue-диагностика (`metallb-system` pods/events) для сбоев webhook.
- В роли `metallb` добавлен временный workaround `failurePolicy=Ignore` для `ipaddresspool`/`l2advertisement` webhook на время `apply` pool-конфига и автоматический rollback в `failurePolicy=Fail` в `always`.
- В роли `metallb` добавлен fallback-workaround удаления `ValidatingWebhookConfiguration` перед `pool apply` и восстановления webhook-конфигурации через повторный `apply` базового манифеста.
- В роли `metallb` шаг восстановления webhook-конфигурации переведен на проксируемое окружение (`proxy_environment`) для устранения `network is unreachable` при загрузке remote manifest URL.
- В роли `kubernetes_core` добавлена параметризуемая установка `helm` и `helmfile` на control-plane нодах из официальных release-артефактов с учетом архитектуры и proxy-настроек.
- В роли `proxy` добавлена поддержка `proxy_state=present|absent` для управляемого включения/отключения system/profile/dnf proxy-конфигурации.
- Добавлен отдельный `playbooks/manage_proxy.yml` для оперативного переключения proxy-режима без полного bootstrap.
- В `docs/runbook.md` добавлены эксплуатационные команды переключения proxy (`present/absent`) через `playbooks/manage_proxy.yml` и базовые проверки результата.
- В роли `validation` добавлена безопасная прединициализация `validation_failover_source_host` и защитные `when`-условия для failover-операций `keepalived`, чтобы исключить `undefined` в `delegate_to`.
- В роли `validation` для failover-операций `keepalived` добавлен fallback в `delegate_to` через `validation_failover_source_host | default(inventory_hostname)`, чтобы исключить падение до вычисления `when`.
- Выполнена полная синхронизация эксплуатационной и архитектурной документации (`Project.md`, `runbook.md`, `qa.md`, `Diary.md`) с текущим фактическим состоянием кластера и последних фиксов validation failover.
- В роли `validation` добавлена расширенная диагностика ingress-проверки: при `rc != 0` выводятся `rc/stdout/stderr`, а итоговый assert содержит `stderr` для упрощения устранения причин (`NotFound`/namespace/service mismatch/доступ).

## Декомпозиция ближайшего этапа (сбор данных)
- [x] Утвердить схему IP-адресов и список нод/ролей.
- [x] Утвердить консистентность подсетей management/VLAN относительно фактических IP нод.
- [x] Утвердить параметры прокси (`http_proxy`, `https_proxy`, `no_proxy`) и модель размещения прокси.
- [x] Утвердить источник пакетов/образов (локальные mirror/registry или внешние через proxy).
- [x] Утвердить версию Kubernetes и runtime.
- [x] Утвердить диапазоны IP для MetalLB.
- [x] Утвердить параметры NFS (performance/backup).
- [x] Зафиксировать labels/taints для выделенных `metallb` нод.
- [x] Утвердить приемочные критерии (`acceptance_criteria`) и политику `--check/--diff`.

## Риски и контроль
- Риск: недоопределенный `no_proxy` может сломать кластерные коммуникации.
  - Контроль: отдельная валидация на всех нодах до bootstrap.
- Риск: loopback proxy (`127.0.0.1`) не будет работать на всех хостах без локального форвардера на каждой ноде.
  - Контроль: подтвердить FQDN/IP прокси или внедрить единый механизм локального прокси-сервиса.
- Риск: конфликт firewalld/SELinux политик с CNI/MetalLB/NFS.
  - Контроль: таблица портов и контекстов + автоматические post-checks.
- Риск: нестабильный источник образов в изолированной сети.
  - Контроль: fallback на локальный registry mirror.
