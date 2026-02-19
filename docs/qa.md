## Задача: Уточнение требований для реализации Ansible-плейбука
- **Статус**: В процессе
- **Описание**: Собрать полный набор входных данных для безопасной и идемпотентной реализации Kubernetes-кластера на RedOS 8.0.2 в изолированной среде.

# qa.md

## 1. Закрытые вопросы (получены ответы)

### 1.1 VMware и инфраструктура
- vCenter: `7.0.3`
- ESXi: `7.0.3`
- Доступ к изменению сетевых настроек в vCenter: отсутствует
- Provisioning: `clone_from_template`
- Template: `k8s-pcp-template`
- Datacenter: `Eleksir`
- Cluster: `North`
- Datastore: `North_Datastore01_vol_02`
- Portgroup: `10.255.106.0/26_424`
- DRS anti-affinity для control-plane: `no`
- Ресурсы:
  - control-plane: `2 vCPU / 4 GB RAM / 50 GB`
  - worker: `4 vCPU / 8 GB RAM / 50 GB`
  - metallb: `2 vCPU / 4 GB RAM / 50 GB`

### 1.2 Сеть и DNS/NTP
- Management subnet: `10.255.106.0/26`
- Pod subnet: `10.245.0.0/16`
- Service subnet: `10.246.0.0/16`
- MetalLB external pool: `10.255.106.21-10.255.106.31`
- Узлы:
  - `k8s-scp-01` `10.255.106.10`
  - `k8s-scp-02` `10.255.106.11`
  - `k8s-scp-03` `10.255.106.12`
  - `k8s-wkn-01` `10.255.106.13`
  - `k8s-wkn-02` `10.255.106.14`
  - `k8s-wkn-03` `10.255.106.15`
  - `k8s-mlb-01` `10.255.106.16`
  - `k8s-mlb-02` `10.255.106.17`
  - `k8s-mlb-03` `10.255.106.18`
  - `k8s-nfs-01` `10.255.106.19`
- DNS servers: `10.255.85.13`, `10.7.5.10`, `1.1.1.1`, `1.0.0.1`, `8.8.8.8`, `8.8.4.4`
- Search domains: пусто
- NTP: `timeserver.ru`

### 1.3 Proxy и репозитории
- mode: `local_on_each_node`
- `http_proxy`: `http://127.0.0.1:12334`
- `https_proxy`: `http://127.0.0.1:12334`
- `no_proxy`: `localhost,127.0.0.1,10.0.0.0/8,.eleksir.net,.eleksir.finance,.cr.yandex`
- RPM mirror: `external_via_proxy`
- Registry mirror: `external_via_proxy`
- Proxy auth: `no`
- Registry auth: `no`
- Хранение credentials: `ansible-vault`

### 1.4 Kubernetes / MetalLB / NFS / Security / Ops
- Kubernetes version: `1.30.14`
- Runtime: `containerd`
- CNI: `calico`
- Control plane endpoint: `10.255.106.20`
- Control plane endpoint port: `8443`
- Control-plane endpoint HA: `keepalived + haproxy` на control-plane узлах
- MetalLB mode: `l2`
- Address pool: `pcidss-lan`, range `10.255.106.21-10.255.106.30`
- Ingress controller: `nginx`
- MetalLB node placement:
  - labels: `node-role.kubernetes.io/metallb=true`
  - taints: отсутствуют
- NFS:
  - OS: `RedOS 8.0.2`
  - export: `/srv/storage`
  - clients: `10.255.106.0/26`
  - options: `rw,sync,no_root_squash,no_subtree_check`
  - storage class: `nfs-sp` (`Delete`, default=true)
- Security:
  - ansible user: `enemirov`
  - ssh auth: `password`
  - bastion: `no`
  - selinux: `Enforcing`
  - secrets store: `ansible-vault`
- Operations:
  - ansible control node: `Debian 13`
  - ansible-core: `2.19.4`
  - quality gate: `ansible-lint`
  - require_check_diff_support: `yes`
  - unified `site.yml`: `true`
  - split playbooks: `true`
  - tags strategy: `true`
  - runbook required: `true`
- NFS performance requirements:
  - min IOPS: `>= 2000`
  - min throughput: `>= 150 MB/s`
  - max latency: `<= 5 ms`
- NFS backup/snapshot strategy:
  - daily snapshot
  - retention: `14 days`
- Acceptance criteria:
  - Успешный полный deploy (`playbooks/site.yml`) на чистом контуре.
  - Идемпотентный повторный запуск без незапланированных изменений.
  - Все ноды `Ready`, системные поды `Running`.
  - Проверка MetalLB + ingress (IP из пула и доступность 80/443).
  - Проверка NFS PVC (bind + RW тест).
  - Проверка failover одной control-plane ноды.

## 2. Открытые вопросы (блокируют старт реализации)
Блокеры отсутствуют.

## 3. Варианты закрытия неоднозначностей
Выбран вариант 3 (production baseline + расширенные критерии приемки).
