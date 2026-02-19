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
| T-019 | Reinstall | Добавить деструктивный сценарий полной переустановки кластера (`cluster_and_nfs`) | Критический | В процессе |

## Собранные данные (2026-02-19)
- VMware: `vCenter 7.0.3`, `ESXi 7.0.3`, `clone_from_template`, шаблон `k8s-pcp-template`.
- Kubernetes stack: `k8s 1.30.14`, `containerd`, `calico`, `control_plane_endpoint 10.255.106.20`.
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
