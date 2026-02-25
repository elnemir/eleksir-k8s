## Задача: Дневник проектных решений и технических наблюдений
- **Статус**: В процессе
- **Описание**: Фиксировать ключевые технические решения, выявленные проблемы и выбранные подходы для бесшовной передачи контекста между разработчиками.

# Diary.md

## Дата: 2026-02-19
### Наблюдения
- Проект стартует с нуля, в репозитории отсутствуют playbook/roles/inventory.
- Архитектура должна учитывать изолированную сеть и работу через прокси.
- Кластерная топология задана: 3 control-plane, 3 worker, 3 dedicated MetalLB nodes.
- Есть строгие требования к идемпотентности, безопасности (firewalld/SELinux) и поддержке NFS persistent storage.

### Решения
- Зафиксирована базовая целевая архитектура и поэтапный план в `docs/Project.md`.
- Сформирован стартовый task backlog с приоритетами в `docs/Tasktracker.md`.
- Выбран подход "сначала сбор входных данных, затем реализация playbook".
- Вынесены открытые вопросы в `docs/qa.md` для формализации параметров среды.

### Проблемы
- Отсутствуют обязательные входные параметры (сети, версии, политика прокси/no_proxy, детали NFS, порты и security baseline).
- Не подтвержден стек Kubernetes (версия kubeadm/kubelet и container runtime).
- Не определена стратегия источников пакетов/образов для полностью изолированного контура.

## Дата: 2026-02-19 (сессия 2)
### Наблюдения
- Получены конкретные параметры VMware, сетевой схемы, версии Kubernetes, CNI, MetalLB, NFS и security baseline.
- Подтверждена целевая модель: `kubeadm + containerd + calico + metallb(l2) + nginx ingress + nfs`.
- Большинство входных данных закрыто, но остаются критичные неоднозначности по сети и прокси.

### Решения
- Зафиксированы параметры среды в `docs/Project.md` (раздел 12).
- Обновлен прогресс в `docs/Tasktracker.md` (T-002 = 80%).
- `docs/qa.md` переведен в формат "закрытые вопросы + блокеры + варианты закрытия".

### Проблемы
- Несоответствие `management_subnet_cidr` и фактической подсети узлов.
- Не подтверждена топология прокси (`127.0.0.1` локально/централизованно).
- Не заполнены `acceptance_criteria`, `performance_requirements`, `backup_snapshot_strategy`.

## Дата: 2026-02-19 (сессия 3)
### Наблюдения
- Подтверждена management-подсеть `10.255.106.0/26`.
- Подтверждена прокси-модель `local_on_each_node` и формат proxy URL со схемой.
- Зафиксирован label для выделенных MetalLB-нод без taints.

### Решения
- Актуализированы `docs/Project.md`, `docs/qa.md` и `docs/Tasktracker.md`.
- Список блокеров сужен до 4 незаполненных полей: NFS performance, NFS backup strategy, `require_check_diff_support`, `acceptance_criteria`.

### Проблемы
- Пользователь оставил пустыми целевые метрики NFS производительности.
- Пользователь оставил пустой backup/snapshot strategy.
- Пользователь не зафиксировал `require_check_diff_support` и `acceptance_criteria`.

## Дата: 2026-02-19 (сессия 4)
### Наблюдения
- Выбран вариант 3 для закрытия блокеров: production baseline + расширенные acceptance criteria.
- Сбор входных данных завершен, архитектурные блокеры сняты.

### Решения
- Зафиксированы NFS performance требования: `IOPS >= 2000`, `throughput >= 150 MB/s`, `latency <= 5 ms`.
- Зафиксирована стратегия backup: `daily snapshot`, `retention 14 days`.
- Зафиксирована политика `require_check_diff_support: yes`.
- Зафиксирован расширенный набор критериев приемки (deploy, идемпотентность, health, MetalLB/ingress, NFS PVC, failover control-plane).

### Проблемы
- Критичные блокеры для начала реализации отсутствуют.

## Дата: 2026-02-19 (сессия 5)
### Наблюдения
- Пользователь подтвердил переход к шагу реализации Ansible-каркаса.
- Входные данные согласованы и достаточны для генерации inventory/group_vars/playbooks/roles.

### Решения
- Этап реализации каркаса официально стартовал.
- В трекере задачи `T-003` и `T-004` переведены в `В процессе`.

### Проблемы
- На момент старта каркаса критичные блокеры отсутствуют.

## Дата: 2026-02-19 (сессия 6)
### Наблюдения
- Каркас Ansible-репозитория успешно создан в соответствии с архитектурой `Project.md`.
- Все базовые сущности (inventory, group_vars, playbooks, roles, ansible.cfg, requirements.yml) присутствуют.
- Локальная проверка `ansible-playbook --syntax-check` не выполнена из-за отсутствия `ansible-playbook` в окружении.

### Решения
- Этапы `T-003` и `T-004` закрыты как завершенные.
- В ролях добавлены базовые `assert`-проверки входных переменных и безопасные skeleton-задачи (`debug` с `changed_when: false`).
- Следующий этап: наполнение ролей рабочей логикой установки и конфигурации.

### Проблемы
- В текущем окружении отсутствует `ansible-playbook`, поэтому syntax-check нужно выполнить после установки Ansible.

## Дата: 2026-02-19 (сессия 7)
### Наблюдения
- Пользователь подтвердил переход к следующему этапу: реализация рабочей логики ролей.
- Приоритет реализации: `base_os -> proxy -> container_runtime -> kubernetes_core -> networking -> metallb -> storage_nfs -> security_hardening -> validation`.

### Решения
- Этап реализации ролей официально стартован.
- Задачи `T-005`...`T-012` переведены в статус `В процессе`.

### Проблемы
- Для финальной проверки потребуется среда с установленным `ansible-playbook`.

## Дата: 2026-02-19 (сессия 8)
### Наблюдения
- Реализация ролей выполнена в полном согласованном порядке.
- Логика плейбуков переведена с skeleton-задач на реальные операции развертывания.
- Проверка `ansible-playbook --syntax-check` по-прежнему недоступна из-за отсутствия Ansible в окружении разработки.

### Решения
- Реализованы роли `base_os`, `proxy`, `container_runtime`, `kubernetes_core`, `networking`, `metallb`, `storage_nfs`, `security_hardening`, `validation`.
- Обновлен `bootstrap.yml`: `kubernetes_core` применяется ко всей группе `k8s_cluster` для корректного join всех нод.
- Расширен `group_vars/all.yml` для runtime и `no_proxy` диапазонов кластера.

### Проблемы
- Фактическая проверка на стенде (деплой, идемпотентность, post-check) отложена до установки `ansible-playbook`.

## Дата: 2026-02-19 (сессия 9)
### Наблюдения
- Появились новые требования: hostname узлов должен устанавливаться по именам из inventory.
- Деплой должен поддерживать два режима сети: с proxy и без proxy.

### Решения
- Стартована доработка `T-017` для hostname-синхронизации и параметризации proxy.
- Сначала обновлена документация, затем начинается изменение ролей/переменных.

### Проблемы
- После изменения логики прокси потребуется повторная стендовая валидация в обоих режимах (`proxy_enabled=true/false`).

## Дата: 2026-02-19 (сессия 10)
### Наблюдения
- Требование по hostname из inventory реализовано на уровне `base_os`.
- Прокси-настройка теперь управляется параметром `proxy_enabled`.

### Решения
- Добавлен параметр `proxy_enabled` в `group_vars/all.yml`.
- Роль `proxy` сделана условной в `playbooks/bootstrap.yml`.
- Команды `kubectl apply` в ролях `networking`, `metallb`, `storage_nfs` переведены на условный `proxy_environment`.
- Добавлена генерация `/etc/hosts` из inventory для консистентного name-resolution внутри кластера.

### Проблемы
- Нужна последующая проверка на control host в двух профилях: `proxy_enabled=true` и `proxy_enabled=false`.

## Дата: 2026-02-19 (сессия 11)
### Наблюдения
- Подтверждено требование: проверки будут выполняться позже на control host.
- Необходим отдельный runbook с командами запуска для двух сетевых профилей.

### Решения
- Стартована задача `T-018` по подготовке эксплуатационного runbook.

### Проблемы
- До публикации runbook отсутствует единый стандартизованный сценарий запуска/проверок для control host.

## Дата: 2026-02-19 (сессия 12)
### Наблюдения
- Runbook для control host подготовлен как отдельный документ в `docs`.
- В runbook включены оба сетевых режима (`proxy_enabled=true/false`) и блок проверок hostname по inventory.

### Решения
- Завершена задача `T-018` по эксплуатационной документации.
- В качестве единого operational entrypoint принят `docs/runbook.md`.

### Проблемы
- Практический прогон шагов runbook будет выполнен позже на control host (по плану пользователя).

## Дата: 2026-02-19 (сессия 13)
### Наблюдения
- Добавлено новое требование: сценарий полной переустановки кластера с уничтожением всех данных (`cluster_and_nfs`).
- Требование относится к деструктивным операциям и требует guardrail-механизмов.

### Решения
- Добавлен отдельный playbook `playbooks/reinstall_cluster_and_nfs.yml`.
- В playbook реализованы обязательные подтверждения через переменные:
  - `reinstall_scope=cluster_and_nfs`
  - `reinstall_confirm_token=DESTROY_CLUSTER_AND_NFS_DATA`
- В `docs/runbook.md` добавлен раздел запуска деструктивного сценария.

### Проблемы
- Стендовая проверка деструктивного сценария отложена до выполнения на control host.

## Дата: 2026-02-19 (сессия 14)
### Наблюдения
- Добавлено требование: при развертывании NFS нужна опция выбора диска для разметки и монтирования.
- Требование должно работать как опциональный сценарий, без влияния на существующий путь с готовой файловой системой.

### Решения
- В роль `storage_nfs` добавлены параметры data-диска и условные задачи:
  - `parted` (разметка),
  - `filesystem` (создание ФС),
  - `mount` (монтирование в `nfs_export_path`).
- В `group_vars/nfs.yml` добавлены переменные для настройки этого поведения.
- В runbook добавлен раздел с примером включения опции.

### Проблемы
- Перед использованием на production требуется валидация на стенде с тестовым диском.

## Дата: 2026-02-19 (сессия 15)
### Наблюдения
- Запрошено усиление безопасности: запрет выбора системного диска в сценарии NFS data-диска.

### Решения
- Стартована задача `T-021` на реализацию guardrail с возможностью явного override.

### Проблемы
- Без guardrail остается риск ошибочного форматирования системного диска.

## Дата: 2026-02-19 (сессия 16)
### Наблюдения
- Guardrail для NFS data-диска реализован в роли `storage_nfs`.
- По умолчанию системный диск блокируется для выбора как `storage_nfs_data_disk_device`.

### Решения
- Добавлены проверки критических mount-sources (`/`, `/boot`, `/boot/efi`, `/var`) и их parent-дисков.
- Добавлен explicit override-параметр `storage_nfs_data_disk_allow_system_disk` (по умолчанию `false`).
- Обновлен runbook с описанием guardrail и условиями обхода.

### Проблемы
- Для production нужен обязательный dry-run/стендовый прогон перед использованием override-флага.

## Дата: 2026-02-19 (сессия 17)
### Наблюдения
- Подтверждено архитектурное ограничение: нет доступа к изменению сетевых настроек в `vCenter`.
- Для публикации сервисов в `MetalLB L2` требуется операционный сценарий без зависимости от vCenter-изменений.

### Решения
- В `Project.md` зафиксирована модель публикации: `Service type=LoadBalancer` + VIP из пула MetalLB.
- В `runbook.md` добавлен отдельный checklist для `MetalLB L2` без доступа к `vCenter`.
- Добавлены команды для фиксации статического VIP ingress (`metallb.io/loadBalancerIPs`) и проверки доступности по `80/443`.

### Проблемы
- Внешние предпосылки (IPAM/DHCP reservation, маршрутизация, firewall, DNS) зависят от сетевой команды и должны быть выполнены до приемки.

## Дата: 2026-02-19 (сессия 18)
### Наблюдения
- Требуется автоматизировать проверку публикации ingress через VIP в рамках роли `validation`.
- Текущая валидация проверяет readiness нод, MetalLB controller и NFS StorageClass, но не проверяет ingress EXTERNAL-IP.

### Решения
- Стартована задача `T-023` на добавление post-check для `Service type=LoadBalancer` и выделенного VIP ingress.
- В `roles/validation/tasks/main.yml` добавлен блок проверок ingress-сервиса:
  - тип сервиса `LoadBalancer`,
  - наличие внешнего адреса (`ip` или `hostname`),
  - опциональная сверка с `validation_ingress_expected_vip`.
- В `group_vars/all.yml` зафиксирован ожидаемый VIP `10.255.106.21`.
- В `runbook.md` добавлена инструкция по включению/отключению автоматической проверки.

### Проблемы
- Значение ожидаемого ingress VIP должно быть параметризуемым, чтобы не ломать сценарии без фиксированного VIP.
- Стендовая проверка `playbooks/validate.yml` отложена до запуска на control host.

## Дата: 2026-02-19 (сессия 19)
### Наблюдения
- Выявлен архитектурный пробел: `control_plane_endpoint` задан, но отсутствует реализованный механизм владения VIP.
- Требуется HA endpoint для Kubernetes API без изменений сетевой инфраструктуры в `vCenter`.

### Решения
- Стартована задача `T-024` на реализацию control-plane VIP через `keepalived + haproxy`.
- План реализации: отдельная роль, подключение в `bootstrap.yml` до `kubeadm init/join`, обновление firewall-политик для VRRP.

### Проблемы
- Нужно исключить конфликт портов между `haproxy` и `kube-apiserver` на control-plane узлах.

## Дата: 2026-02-19 (сессия 20)
### Наблюдения
- Реализован control-plane VIP endpoint для Kubernetes API без зависимости от изменений в `vCenter`.
- Для исключения портового конфликта с `kube-apiserver:6443` VIP listener вынесен на `8443`.

### Решения
- Добавлена роль `control_plane_vip` и подключена в `playbooks/bootstrap.yml` перед `kubernetes_core`.
- Включены `keepalived + haproxy` на control-plane узлах, включен `net.ipv4.ip_nonlocal_bind=1`.
- Обновлен `kubeadm`-шаблон на использование `control_plane_endpoint_port`.
- В `security_hardening` добавлена поддержка `firewall_allowed_protocols`, для control-plane разрешен `vrrp`.

### Проблемы
- Нужна обязательная стендовая проверка failover VIP между control-plane нодами на control host.

## Дата: 2026-02-19 (сессия 21)
### Наблюдения
- После реализации `control_plane_vip` не хватает автоматизированной проверки доступности API через VIP в роли `validation`.
- Нужен базовый сценарий проверки failover control-plane VIP с безопасным default-поведением.

### Решения
- Стартована задача `T-025`: добавить post-check API endpoint `10.255.106.20:8443`.
- Стартована доработка `validation` для базового failover-check сценария (опционально, по флагу).
- Добавлена проверка API через `kubectl --server=https://10.255.106.20:8443 get --raw=/readyz`.
- Добавлены проверки сервисов `keepalived/haproxy` на всех control-plane узлах.
- Добавлена проверка наличия ровно одного владельца control-plane VIP.
- Добавлен опциональный failover-test (`validation_enable_control_plane_vip_failover_test=true`) с возвратом `keepalived` в `started`.

### Проблемы
- Авто-failover проверка должна быть выключена по умолчанию, чтобы избежать нежелательных переключений в production.
- Практический запуск failover-test нужно делать только на стенде/в согласованное окно.

## Дата: 2026-02-19 (сессия 22)
### Наблюдения
- Для операционного запуска нужен отдельный тег, позволяющий запускать только failover-сценарий, без полного набора validate-задач.

### Решения
- Стартована задача `T-026` на добавление отдельного тега `failover` в `playbooks/validate.yml`.
- Добавлен отдельный play в `playbooks/validate.yml` с тегами `never,failover`.
- Для failover-play отключены нецелевые проверки (`storage`, `ingress`) и включен целевой `validation_enable_control_plane_vip_failover_test=true`.
- В runbook добавлена отдельная команда запуска `--tags failover`.

### Проблемы
- Важно сохранить безопасный default: failover-сценарий не должен запускаться без явного `--tags failover`.

## Дата: 2026-02-19 (сессия 23)
### Наблюдения
- На стенде получен фатальный сбой `kubeadm init`: `experimental API spec: kubeadm.k8s.io/v1beta4 is not allowed`.
- Ошибки `Join command is not available` на остальных узлах являются каскадными после неуспешного bootstrap primary control-plane.

### Решения
- Стартована задача `T-027` для фикса совместимости версии API в `kubeadm-config`.
- В `kubernetes_core` API-версия kubeadm-конфига параметризована через `kubeadm_config_api_version`.
- Дефолт переключен на совместимый `kubeadm.k8s.io/v1beta3`.
- Добавлен опциональный флаг `kubeadm_init_allow_experimental_api` для редких случаев использования experimental API.

### Проблемы
- До фикса bootstrap Kubernetes не может пройти, а join-фаза блокируется.
- Практическая проверка исправления требует повторного запуска `playbooks/bootstrap.yml` на control host.

## Дата: 2026-02-19 (сессия 24)
### Наблюдения
- На этапе `container_runtime` получен `Depsolve Error`: конфликт установленного `cri-tools1.34` с пакетом `cri-tools` из `kubernetes` repo (`1.30.x`).
- Сбой воспроизводится на всех control-plane узлах и блокирует дальнейший bootstrap.

### Решения
- Стартована задача `T-028`: убрать жесткую зависимость роли runtime от пакета `cri-tools` и сделать установку `crictl` опциональной/безопасной.
- Из дефолтного списка `container_runtime_packages_map` удален `cri-tools`; оставлена установка только базового runtime (`containerd`/`cri-o`).
- Добавлен probe `command -v crictl` и условная установка `cri-tools` только при `container_runtime_install_cri_tools=true`.
- В `group_vars/all.yml` зафиксирован безопасный default `container_runtime_install_cri_tools: false`.

### Проблемы
- До фикса роль `container_runtime` не может стабильно отрабатывать на RedOS с текущим набором репозиториев.
- После фикса требуется повторный запуск `playbooks/bootstrap.yml` на control host для подтверждения устранения depsolve.

## Дата: 2026-02-19 (сессия 25)
### Наблюдения
- На этапе `kubernetes_core` зафиксирован сбой загрузки `kubeadm` (`Cannot download, all mirrors were already tried without success`).
- Требуется повысить устойчивость роли к временной недоступности зеркал и добавить параметр явного выбора версии пакетов.

### Решения
- Стартована задача `T-029` на доработку `kubernetes_core`: precheck repo, retries установки и override версии Kubernetes-пакетов.
- Добавлен precheck `dnf makecache --enablerepo=kubernetes` с retry.
- Добавлен retry для установки Kubernetes-пакетов.
- Добавлен параметр `kubernetes_packages_version_override` для ручного выбора доступного patch-релиза.

### Проблемы
- Без этой доработки bootstrap остается чувствительным к сетевой деградации/нестабильным зеркалам.
- Для подтверждения фикса нужен повторный запуск `bootstrap` на control host.

## Дата: 2026-02-19 (сессия 26)
### Наблюдения
- На `kubeadm init` получен сбой `wait-control-plane`: kubelet не становится healthy по `127.0.0.1:10248`.
- Текущий вывод недостаточно структурирован для быстрого определения причины (swap/runtime/kubelet/cri).

### Решения
- Стартована задача `T-030`: добавить preflight-проверки перед `kubeadm init` и `rescue`-сбор диагностики при падении.
- В `kubernetes_core` добавлены preflight-шаги на primary control-plane: запуск runtime-сервиса, ожидание runtime socket, явная проверка `swapoff`.
- `kubeadm init` переведен в `block/rescue`: при падении автоматически собираются `kubelet/runtime state`, `journalctl -u kubelet`, `crictl ps -a` и выводятся в одной ошибке Ansible.
- В runbook добавлен отдельный troubleshooting-сценарий для `wait-control-plane` с фокусом на swap/runtime/kubelet/cgroup.

### Проблемы
- Без авто-диагностики разбор причины требует ручного сбора логов на каждой ноде.
- Требуется повторный запуск `playbooks/bootstrap.yml --tags k8s` на control host для подтверждения исправления на стенде.

## Дата: 2026-02-24/25 (сессия 27, bootstrap/networking/metallb/proxy stabilization)
### Наблюдения
- Серия взаимосвязанных сбоев в bootstrap/join была обусловлена комбинацией сетевой доступности, SELinux/firewalld, CNI backend и webhook-достижимости.
- Для текущего диагностического контура применен временный `selinux_target_mode: Disabled`; подтверждена необходимость возврата к `Enforcing` после стабилизации.
- Для failover-валидации обнаружен edge-case: шаблонизация `delegate_to` могла падать до вычисления `when`.

### Решения
- Усилены роли `kubernetes_core`, `networking`, `metallb` (precheck/retry/rescue-диагностика, endpoint fallback/force-primary mode, Calico VXLAN, webhook delete/recreate + proxy environment).
- Добавлена установка `helm` и `helmfile` на control-plane в `kubernetes_core`.
- Добавлен отдельный operational playbook `playbooks/manage_proxy.yml` и поддержка `proxy_state=present|absent` в роли `proxy`.
- В `validation` последовательно реализованы:
  - безопасная прединициализация `validation_failover_source_host` (`T-054`);
  - fallback в `delegate_to` через `default(inventory_hostname)` для исключения ошибки шаблонизации (`T-055`).

### Проблемы
- Нужен финальный стендовый цикл подтверждения: `bootstrap -> validate -> failover --tags failover` и проверка стабильности без ручных workaround.
- После завершения диагностики требуется плановый возврат SELinux к `Enforcing` с подтверждением совместимости ролей.
