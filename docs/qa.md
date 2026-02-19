## Задача: Уточнение требований для реализации Ansible-плейбука
- **Статус**: В процессе
- **Описание**: Собрать полный набор входных данных для безопасной и идемпотентной реализации Kubernetes-кластера на RedOS 8.0.2 в изолированной среде.

# qa.md

## 1. VMware и инфраструктура
1. Какая версия vCenter и ESXi используется?
2. Нужна ли автоматическая подготовка VM через Ansible (клонирование из шаблона) или только конфигурация уже созданных ВМ?
3. Какие ресурсы (vCPU, RAM, disk) для каждой группы нод (`control-plane`, `worker`, `metallb`)?
4. Есть ли требования к anti-affinity/DRS rules для control-plane?

## 2. Сеть и DNS/NTP
1. Какие подсети/VLAN используются для:
   - management/SSH;
   - pod network (CNI);
   - service network;
   - external IP pool MetalLB?
2. Какие конкретные IP/hostname планируются для 9 k8s-нод и NFS-сервера?
3. Какие DNS-серверы и search domain использовать на нодах?
4. Какой NTP-источник доступен в изолированной сети?

## 3. Прокси и доступ к пакетам/образам
1. Укажите `http_proxy`, `https_proxy`, `no_proxy` (целиком, включая pod/service CIDR и internal domains).
2. Планируется ли локальный mirror для RPM-пакетов RedOS?
3. Планируется ли локальный container registry mirror для образов Kubernetes/CNI/MetalLB?
4. Нужна ли авторизация на proxy/registry и где хранить учетные данные (ansible-vault)?

## 4. Kubernetes и runtime
1. Какую версию Kubernetes необходимо установить?
2. Какой container runtime выбрать: `containerd` или `CRI-O`?
3. Какой CNI планируется: Calico, Cilium или другой?
4. Нужны ли taints/labels для выделения 3 нод под MetalLB ingress?

## 5. MetalLB и ingress
1. Какой диапазон(ы) IP выделяется под MetalLB?
2. Какой режим MetalLB нужен: L2 или BGP?
3. Если BGP: какие peer-роутеры, ASN, пароли/MD5, policy?
4. Нужен ли конкретный ingress controller (nginx/traefik/haproxy) вместе с MetalLB?

## 6. NFS и хранилище
1. На каком узле размещается NFS и какая ОС/версия на нем?
2. Какой путь экспорта (`/srv/nfs/k8s` и т.п.) и модель прав доступа?
3. Какие требования по производительности и отказоустойчивости NFS?
4. Какие StorageClass-параметры нужны (reclaimPolicy, mountOptions, default class)?
5. Нужен ли backup/snapshot strategy для данных NFS?

## 7. Безопасность и доступ
1. Какие порты допустимо открыть на каждой роли нод (control-plane/worker/metallb/nfs)?
2. SELinux должен оставаться `Enforcing` во всех средах?
3. Нужны ли дополнительные SELinux booleans/modules для NFS/CNI/runtime?
4. Какая модель доступа Ansible: отдельный sudo-пользователь, ключи SSH, bastion?
5. Где хранить секреты и сертификаты (ansible-vault/внешний secret store)?

## 8. Эксплуатация и CI/CD
1. Где будет запускаться Ansible control node (OS, версия ansible-core)?
2. Нужны ли `ansible-lint`, `yamllint`, `pre-commit` как обязательный quality gate?
3. Нужна ли поддержка `--check` и `--diff` для всех ролей?
4. Какие приемочные критерии считаются обязательными перед вводом в эксплуатацию?

## 9. Формат результата
1. Нужно ли подготовить единый `site.yml` + отдельные плейбуки по этапам (`bootstrap`, `security`, `storage`, `validate`)?
2. Нужен ли режим частичного запуска через теги (`base`, `k8s`, `metallb`, `security`, `storage`)?
3. Нужна ли отдельная документация runbook (порядок запуска, rollback, troubleshooting)?
