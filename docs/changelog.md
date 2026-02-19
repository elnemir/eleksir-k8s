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
