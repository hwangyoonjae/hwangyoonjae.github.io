---
layout: post
title: "Ansible-playbook 이용하여 원격 서버 K8S Cluster 구성하기"
date: 2025-06-13
categories: [DevOps, Ansible]
tags: [Ansible, Kubernetes]
image: /assets/img/post-title/ansible-wallpaper.jpg
---

## 1. Ansible 파일 구조 :

- Ansible 통해서 Kubernetes Cluster 구성 파일 구조는 아래와 같습니다.

> 필자는 클러스터 구성 시 HAproxy를 사용하였습니다.
{: .prompt-info}

```bash
[root@yjhwang ansible]# tree
.
├── 1_replace_vars.sh
├── 2_ssh-key-copy.sh
├── 3_update_coredns.sh
├── cluster.yml
├── install_rpm
│   ├── ansible-7.7.0-1.el9.noarch.rpm
│   ├── ansible-core-2.14.18-1.el9.x86_64.rpm
│   ├── git-core-2.47.1-2.el9_6.x86_64.rpm
│   ├── python3-cffi-1.14.5-5.el9.x86_64.rpm
│   ├── python3-cryptography-36.0.1-4.el9.x86_64.rpm
│   ├── python3-packaging-20.9-5.el9.noarch.rpm
│   ├── python3-ply-3.11-14.el9.0.1.noarch.rpm
│   ├── python3-pycparser-2.20-6.el9.noarch.rpm
│   ├── python3-pyparsing-2.4.7-9.el9.noarch.rpm
│   ├── python3-pyyaml-5.4.1-6.el9.x86_64.rpm
│   ├── python3-resolvelib-0.5.4-5.el9.noarch.rpm
│   └── sshpass-1.09-4.el9.x86_64.rpm
├── inventory
│   ├── group_vars
│   │   ├── all.yml
│   │   ├── haproxy.yml
│   │   ├── kubernetes.yml
│   ├── inventory.ini
│   └── inventory.ini.bak
├── replace_vars.log
├── roles
│   ├── haproxy
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       └── haproxy.cfg.j2
│   ├── kubernetes
│   │   └── tasks
│   │       ├── crio.yml
│   │       ├── kube-init.yml
│   │       ├── kubernetes.yml
│   │       ├── main.yml
│   │       └── preinstall.yml
│   └── ping-test.yml
└── vars.sh
```

* * *

## 2. Ansible 설치하기 :

```bash
$ dnf install -y /root/ansible/install_rpm/*.rpm --disablerepo=* --nobest --skip-broken
```

```bash
# ansible 버전 확인
$ ansible --version
```
![Ansible 버전 확인-new](/assets/img/post/Ansible/Ansible%20버전확인-new.png)


* * *

## 3. Ansible로 K8S Cluster 구성하기 :
### 3.1 원격 서버 아이피 주소 및 설치 경로 변수 입력 :

```bash
# 모든 Script는 해당 변수 값을 사용합니다.

# 도메인 주소
DEFAULT_DOMAIN="<고객사명>.com"
HARBOR_DOMAIN="harbor.$DEFAULT_DOMAIN"
INGRESS_DOMAIN="$DEFAULT_DOMAIN"

# haproxy 서버 IP 주소 및 host명
LOADBALANCER_SERVER_IP=""
LOADBALANCER_SERVER_NAME=""

# 설치 파일 폴더 경로
INSTALL_BASE_PATH="/root"
INSTALL_PATH="$INSTALL_BASE_PATH/<Install_Directory>"

# 노드 IP 목록(개수 만큼 입력)
MASTER_IPS=("")
WORKER_IPS=("")

# yml 파일 경로
INVENTORY_PATH="$INSTALL_PATH/ansible/inventory"
INVENTORY_FILE="$INVENTORY_PATH/inventory.ini"
INVENTORY_BACKUP_FILE="${INVENTORY_FILE}.bak"

HAPROXY_YML_FILE="$INVENTORY_PATH/group_vars/haproxy.yml"
```

### 3.2 각 파일별 변수 변환 스크립트 실행 :
```bash
#!/bin/bash

# 공통 변수 파일을 불러오기
source ./vars.sh

# 오류 발생 시 즉시 종료
set -e 

# install.yml용 YAML 포맷
K8S_MASTER_NODES_YAML=""
K8S_WORKER_NODES_YAML=""

for i in "${!MASTER_IPS[@]}"; do
  index=$((i + 1))
  nodename="k8s-master$index"
  K8S_MASTER_NODES_YAML+="  - $nodename"$'\n'
done

for i in "${!WORKER_IPS[@]}"; do
  index=$((i + 1))
  nodename="k8s-worker$index"
  K8S_WORKER_NODES_YAML+="  - $nodename"$'\n'
done

# haproxy.yml 업데이트
awk -v masters="$K8S_MASTER_NODES_YAML" -v workers="$K8S_WORKER_NODES_YAML" '
{
  if ($0 ~ /<K8S_MASTER_NODES>/) {
    print masters
    next
  }
  if ($0 ~ /<K8S_WORKER_NODES>/) {
    print workers
    next
  }
  print
}' "$HAPROXY_YML_FILE" > "${HAPROXY_YML_FILE}.tmp" && mv "${HAPROXY_YML_FILE}.tmp" "$HAPROXY_YML_FILE"

echo -e "\033[32m[✔] haproxy.yml 멀티라인 치환 완료 → $HAPROXY_YML_FILE\033[0m"

# 2. inventory.ini 백업
cp "$INVENTORY_FILE" "$INVENTORY_BACKUP_FILE"
echo "[i] 기존 inventory.ini 백업 완료 → $INVENTORY_BACKUP_FILE"

# 3. 새로운 kubernetes 관련 섹션 생성
{
  # kubernetes 그룹을 children으로 정의
  echo "[kubernetes:children]"
  echo "kube_control_plane"
  echo "kube_node"
  echo ""

  # kube_control_plane 그룹 (마스터만)
  echo "[kube_control_plane]"
  for i in "${!MASTER_IPS[@]}"; do
    echo "k8s-master$((i+1)) ansible_host=${MASTER_IPS[$i]}"
  done
  echo ""

  # kube_node 그룹 (워커만)
  echo "[kube_node]"
  for i in "${!WORKER_IPS[@]}"; do
    echo "k8s-worker$((i+1)) ansible_host=${WORKER_IPS[$i]}"
  done
  echo ""
} > new_kubernetes_section.txt

# 4. 기존 [kubernetes], [kube_control_plane], [kube_node] 섹션만 제거 (나머지는 그대로 유지)
awk '
BEGIN { in_kube = 0; in_control_plane = 0; in_node = 0 }
/^\[kubernetes\]/ { in_kube = 1; next }
/^\[kubernetes:children\]/ { in_kube = 1; next }
/^\[kube_control_plane\]/ { in_control_plane = 1; next }
/^\[kube_node\]/ { in_node = 1; next }
/^\[/ && (in_kube || in_control_plane || in_node) { in_kube = in_control_plane = in_node = 0 }
!in_kube && !in_control_plane && !in_node
' "$INVENTORY_BACKUP_FILE" > tmp_inventory.ini

# 5. 새로 만든 섹션 추가
cat new_kubernetes_section.txt >> tmp_inventory.ini

# 6. 최종 파일 반영
mv tmp_inventory.ini "$INVENTORY_FILE"
rm new_kubernetes_section.txt

echo -e "\033[32m[✔] inventory.ini 의 [kubernetes:children], [kube_control_plane], [kube_node] 섹션 업데이트 완료!\033[0m"

# 치환할 키=값 매핑
mappings=(
  "<HARBOR_DOMAIN>=$HARBOR_DOMAIN"
  "<LOADBALANCER_SERVER_IP>=$LOADBALANCER_SERVER_IP"
  "<LOADBALANCER_SERVER_NAME>=$LOADBALANCER_SERVER_NAME"
  "<INSTALL_BASE_PATH>=$INSTALL_BASE_PATH"
  "<INSTALL_PATH>=$INSTALL_PATH"
  "<K8S_MASTER_NODES>=$K8S_MASTER_NODES_YAML"
  "<K8S_WORKER_NODES>=$K8S_WORKER_NODES_YAML"
)

# 대상 파일 목록 찾기
target_files=$(find $INSTALL_PATH -type f \( -name "*.yaml" -o -name "*.yml"  -o -name "*.ini" -o -name "config.js" -o -name "Corefile" -o -name "dev.env" \))

# 로그 초기화
log_file="replace_vars.log"
> "$log_file"

# 파일마다 반복
for file in $target_files; do
  should_process=false
  content=$(<"$file")

  # 바꿀 내용이 실제로 있는지 체크
  for pair in "${mappings[@]}"; do
    key="${pair%%=*}"
    if grep -q "$key" <<< "$content"; then
      should_process=true
      break
    fi
  done

  if [ "$should_process" = false ]; then
    continue
  fi

  echo -e "\e[33m🔧 처리 중: $file\e[0m"
  cp "$file" "$file.bak"

  for pair in "${mappings[@]}"; do
    key="${pair%%=*}"
    value="${pair#*=}"
    content="${content//$key/$value}"  # 문자열 치환
    echo "[`date '+%Y-%m-%d %H:%M:%S'`] $file: '$key' → '${value//$'\n'/\\n}'" >> "$log_file"
  done

  echo "$content" > "$file"
done

echo -e "\033[32m✅ 모든 변경 완료!\033[0m"
echo "📄 변경 로그: $log_file"
```

* * *

### 3.3 inventory.ini에 등록된 서버들 ssh-key 자동 생성 스크립트 실행 :

```bash
#!/bin/bash

# -------------------
INVENTORY_FILE="inventory/inventory.ini"
TMP_IP_LIST="./ssh_unique_ip_list.txt"
SSH_KEY="$HOME/.ssh/id_rsa.pub"
# -------------------

# 0. sshpass 확인
if ! command -v sshpass &> /dev/null; then
  echo "[!] sshpass가 설치되어 있지 않습니다. 설치 후 다시 실행하세요."
  exit 1
fi

# 1. SSH 비밀번호 입력 받기
read -s -p "[?] 원격 서버 SSH 비밀번호를 입력하세요: " SSH_PASSWORD
echo ""

# 2. SSH 키가 없으면 생성
if [ ! -f "$SSH_KEY" ]; then
  echo "[+] SSH 키가 없어서 생성합니다."
  ssh-keygen -t rsa -b 4096 -N "" -f "${SSH_KEY%.*}"
else
  echo "[+] SSH 키가 이미 존재합니다. 생성하지 않고 계속 진행합니다."
fi

# 3. inventory.ini 파일에서 ansible_host IP 추출
grep -oP 'ansible_host=\K[^\s]+' "$INVENTORY_FILE" | sort -u > "$TMP_IP_LIST"

echo "[+] 총 $(wc -l < "$TMP_IP_LIST")개의 고유 IP가 추출되었습니다."

# 4. ssh-copy-id 실행
while read -r IP; do
  if [ -n "$IP" ]; then
    echo "[-] SSH 키를 $IP에 복사 중..."
    sshpass -p "$SSH_PASSWORD" ssh-copy-id -o StrictHostKeyChecking=no "root@$IP"
  fi
done < "$TMP_IP_LIST"

# 5. 정리
rm -f "$TMP_IP_LIST"
echo "[+] SSH 키 공유 완료!"
```

* * *

### 3.4 Ansible 인벤토리 그룹별 변수 정의 :
- all.yml 파일에서는 원격서버들의 IP 주소와 설치 파일 경로들을 지정한다.
- ***1_replace_vars.sh*** 파일을 통해서 해당 변수에 값을 변경한다.

```yml
# ansible/inventory/group_vars/all.yml

# 도메인 주소
HARBOR_DOMAIN: "<HARBOR_DOMAIN>"
INGRESS_DOMAIN: "<INGRESS_DOMAIN>"

# haproxy 서버 IP 주소
USE_LB: true # haproxy 서버 사용여부
LOADBALANCER_SERVER_IP: "<LOADBALANCER_SERVER_IP>"

# 설치 경로
INSTALL_BASE_PATH: "/root"
INSTALL_PATH: "{{ INSTALL_BASE_PATH }}/secloudit-v2.2-install"

ALL_K8S_NODES: "{{ K8S_MASTER_NODES + K8S_WORKER_NODES }}"

# Kubernetes 설치 파일 경로
K8S_NODE_INSTALL_PATH: "{{ INSTALL_PATH }}/kubernetes/k8s-node-pack"
```

* * *

- haproxy.yml 파일에서는 haproxy 설정에 관련된 변수들을 지정한다.

```yml
# ansible/inventory/group_vars/hapoxy.yml

HAPROXY_INSTALL_PATH: "{{ INSTALL_PATH }}/kubernetes/lb/lb-pack"
HAPROXY_INSTALL_DIR: "{{ HAPROXY_INSTALL_PATH }}/haproxy"
HAPROXY_REMOTE_DIR: "/root/haproxy"

# HAProxy 설정
HAPROXY_CFG_FILE: "/etc/haproxy/haproxy.cfg"
LOADBALANCER_PORT: 6443
STATS_PORT: 9999
HTTPS_PORT: 443
HTTPS_NODEPORT: 30443
HTTP_PORT: 80
HTTP_NODEPORT: 30080

K8S_MASTER_NODES:
 <K8S_MASTER_NODES>
  
K8S_WORKER_NODES:
 <K8S_WORKER_NODES>

```

* * *

- kubernetes.yml 파일에서는 kubernetes 설정에 관련된 변수들을 지정한다.

```yml
# ansible/inventory/group_vars/kubernetes.yml

# crio 설치 파일 경로
CRIO_INSTALL_DIR: "{{ K8S_NODE_INSTALL_PATH }}/crio"
CRIO_REMOTE_DIR: "/root/crio"
CRIO_PATH: "/etc/crio"
CRIO_CONF: "{{ CRIO_PATH }}/crio.conf.d/10-crio.conf"
CRIO_ROOT: "{{ CRIO_PATH }}/crio-root"
CRIO_RUNROOT: "{{ CRIO_PATH }}/crio-runroot"
CRIO_VERSION_FILE: "{{ CRIO_PATH }}/crio-versions/versions"
CRIO_CLEAN_SHUTDOWN_FILE: "{{ CRIO_PATH }}/crio-clean/clean.shutdown"
PAUSE_IMAGE: "{{ HARBOR_DOMAIN }}/kubernetes-install/pause:3.9"

# Kubernetes 설치 파일 경로
K8S_INSTALL_DIR: "{{ K8S_NODE_INSTALL_PATH }}/kubeadm"
K8S_REMOTE_DIR: "/root/kubeadm"
K8S_INIT_FILE_DIR: "{{ K8S_NODE_INSTALL_PATH }}/config"
K8S_INIT_FILE_REMOTE_DIR: "/root/config"
```

* * *

### 3.5 Ansible 역할 단위 플레이북 구성 :
```yml
# ansible/roles/haproxy/handlers/main.yml

- name: Restart HAProxy
  service:
    name: haproxy
    state: restarted
```

```yml
# ansible/roles/haproxy/tasks/main.yml

- name: Add Kubernetes nodes to /etc/hosts
  blockinfile:
    path: /etc/hosts
    marker: "# {mark} ANSIBLE MANAGED K8S NODES"
    block: |
      {% for node in ALL_K8S_NODES %}
      {{ hostvars[node].ansible_host }} {{ node }}
      {% endfor %}

- name: Copy HAProxy directory to remote server (if not localhost)
  copy:
    src: "{{ HAPROXY_INSTALL_DIR }}/"
    dest: "{{ HAPROXY_REMOTE_DIR }}/"
    mode: '0755'
  when: inventory_hostname != 'localhost'

- name: Install HAProxy from the local files
  shell: "dnf install -y {{ HAPROXY_REMOTE_DIR }}/*.rpm --disablerepo=*"

- name: Copy HAProxy configuration file
  template:
    src: haproxy.cfg.j2
    dest: /etc/haproxy/haproxy.cfg
    owner: root
    group: root
    mode: '0644'
  notify: Restart HAProxy

- name: Enable and start HAProxy service
  service:
    name: haproxy
    state: started
    enabled: true
```

```yml
# ansible/roles/haproxy/templates/haproxy.cfg.j2

frontend k8s-loadbalancer
    bind 0.0.0.0:{{ LOADBALANCER_PORT }}
    option tcplog
    option forwardfor
    tcp-request inspect-delay 5s
    tcp-request content accept if { req.ssl_hello_type 1 }
    mode tcp
    default_backend k8s-masters

backend k8s-masters
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    {% for node in K8S_MASTER_NODES %}
server {{ node }} {{ node }}:6443 check
    {% endfor %}

listen stats
    bind 0.0.0.0:{{ STATS_PORT }}
    stats enable
    stats realm Haproxy Statistics
    stats uri /haproxy_stats

frontend https-ingress
    bind 0.0.0.0:{{ HTTPS_PORT }}
    option tcplog
    tcp-request inspect-delay 5s
    tcp-request content accept if { req.ssl_hello_type 1 }
    mode tcp
    default_backend https-nodeports

backend https-nodeports
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    {% for node in K8S_WORKER_NODES %}
server {{ node }} {{ node }}:{{ HTTPS_NODEPORT }} check
    {% endfor %}

frontend http-ingress
    bind 0.0.0.0:{{ HTTP_PORT }}
    option tcplog
    tcp-request inspect-delay 5s
    tcp-request content accept if { req.ssl_hello_type 1 }
    mode tcp
    default_backend http-nodeports

backend http-nodeports
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    {% for node in K8S_WORKER_NODES %}
server {{ node }} {{ node }}:{{ HTTP_NODEPORT }} check
    {% endfor %}
```

```yml
# ansible/kubernetes/tasks/main.yml

- name: Setting 
  import_tasks: preinstall.yml

- name: Install CRI-O
  import_tasks: crio.yml

- name: Install Kubernetes components
  import_tasks: kubernetes.yml

- name: Kubernetes init
  import_tasks: kube-init.yml
```

```yml
# ansible/kubernetes/tasks/preinstall.yml

# SELinux 설정
- name: Check and set SELinux to permissive (runtime)
  command: setenforce 0
  when: ansible_facts.selinux.status == "enabled"
  failed_when: false

- name: Set SELinux to permissive in config file (permanent)
  replace:
    path: /etc/sysconfig/selinux
    regexp: '^SELINUX=enforcing'
    replace: 'SELINUX=permissive'
  when: ansible_facts.selinux.status == "enabled"

# Swap 해제
- name: Disable swap immediately
  command: swapoff -a
  when: ansible_swaptotal_mb > 0
  failed_when: false

- name: Remove swap entry from /etc/fstab
  replace:
    path: /etc/fstab
    regexp: '^([^#].*\sswap\s)'
    replace: '#\1'
  when: ansible_swaptotal_mb > 0

# 방화벽 해제
- name: Stop firewalld
  systemd:
    name: firewalld
    state: stopped
    enabled: no
    masked: yes
  failed_when: false
```

```yml
# ansible/kubernetes/tasks/crio.yml

- name: Copy CRI-O directory to remote server (if not localhost)
  copy:
    src: "{{ CRIO_INSTALL_DIR }}/"
    dest: "{{ CRIO_REMOTE_DIR }}/"
    mode: '0755'
  when: inventory_hostname != 'localhost'

- name: Install CRI-O from the local files
  shell: "dnf install -y {{ CRIO_REMOTE_DIR }}/*.rpm --disablerepo=*"

- name: Ensure pause_image is set under [crio.image]
  blockinfile:
    path: "{{ CRIO_CONF }}"
    marker: "# {mark} PAUSE_IMAGE"
    block: |
      [crio.image]
      pause_image = "{{ PAUSE_IMAGE }}"
  when: pause_image is defined

- name: Ensure [crio] section settings are present
  blockinfile:
    path: "{{ CRIO_CONF }}"
    marker: "# {mark} CRIO_SECTION"
    block: |
      [crio]
      root = "{{ CRIO_ROOT }}"
      runroot = "{{ CRIO_RUNROOT }}"
      version_file = "{{ CRIO_VERSION_FILE }}"
      clean_shutdown_file = "{{ CRIO_CLEAN_SHUTDOWN_FILE }}"

- name: Load kernel modules for CRI-O
  copy:
    dest: "/etc/modules-load.d/crio.conf"
    content: |
      overlay
      br_netfilter

- name: Load kernel modules for Istio iptables
  copy:
    dest: "/etc/modules-load.d/istio-iptables.conf"
    content: |
      br_netfilter
      nf_nat
      xt_REDIRECT
      xt_owner
      iptable_nat
      iptable_mangle
      iptable_filter

- name: Ensure required kernel modules are loaded
  modprobe:
    name: "{{ item }}"
    state: present
  loop:
    - overlay
    - br_netfilter
    - nf_nat
    - xt_REDIRECT
    - xt_owner
    - iptable_nat
    - iptable_mangle
    - iptable_filter

- name: Configure sysctl for Kubernetes
  copy:
    dest: "/etc/sysctl.d/99-kubernetes-cri.conf"
    content: |
      net.bridge.bridge-nf-call-iptables = 1
      net.ipv4.ip_forward = 1
      net.bridge.bridge-nf-call-ip6tables = 1

- name: Apply sysctl settings
  command: sysctl --system

- name: Enable and start crio service
  systemd:
    name: crio
    enabled: yes
    state: started

- name: Add Harbor registry to CRI-O configuration
  blockinfile:
    path: /etc/containers/registries.conf.d/crio.conf
    block: |
      [[registry]]
      location = "{{ HARBOR_DOMAIN }}"
      insecure = true

- name: Restart CRI-O
  systemd:
    name: crio
    state: restarted
    enabled: true
```

* * *

```yml
# ansible/kubernetes/tasks/kubernetes.yml

- name: Add Kubernetes nodes to /etc/hosts
  blockinfile:
    path: /etc/hosts
    marker: "# {mark} ANSIBLE MANAGED K8S NODES"
    block: |
      {% for node in ALL_K8S_NODES %}
      {{ hostvars[node].ansible_host }} {{ node }}
      {% endfor %}

- name: Copy kubernetes directory to remote server (if not localhost)
  copy:
    src: "{{ K8S_INSTALL_DIR }}/"
    dest: "{{ K8S_REMOTE_DIR }}/"
    mode: '0755'
  when: inventory_hostname != 'localhost'

- name: Install kubernetes from the local files
  shell: "dnf install -y {{ K8S_REMOTE_DIR }}/*.rpm --disablerepo=*"

- name: Display kubelet version
  command: kubelet --version
  register: kubelet_version
  changed_when: false

- name: Display kubeadm version
  command: kubeadm version
  register: kubeadm_version
  changed_when: false

- name: Display kubectl version
  command: kubectl version --client
  register: kubectl_version
  changed_when: false

- name: Enable and start kubelet service
  systemd:
    name: kubelet
    enabled: yes
    state: started

- name: Restart kubelet service
  systemd:
    name: kubelet
    state: restarted
    enabled: true

- name: Log Kubernetes version details
  debug:
    msg: |
      kubelet version: "{{ kubelet_version.stdout }}"
      kubeadm version: "{{ kubeadm_version.stdout }}"
      kubectl version: "{{ kubectl_version.stdout }}"
```

```yml
# ansible/kubernetes/tasks/kube-init.yml

- name: Copy Kubernetes Config file to remote server (if not localhost)
  copy:
    src: "{{ K8S_INIT_FILE_DIR }}/"
    dest: "{{ K8S_INIT_FILE_REMOTE_DIR }}/"
    mode: '0755'
  when: inventory_hostname == 'k8s-master1'

- name: Set MASTER_IP variable to k8s-master1 IP
  set_fact:
    MASTER_IP: "{{ hostvars['k8s-master1'].ansible_host | default(hostvars['k8s-master1'].ansible_default_ipv4.address) }}"

- name: Replace <MASTER_IP> in config file with the determined IP
  lineinfile:
    path: "{{ K8S_INIT_FILE_REMOTE_DIR }}/config.yaml"
    regexp: '<MASTER_IP>'
    line: "  advertiseAddress: {{ MASTER_IP }}"
  when: inventory_hostname == 'k8s-master1'

- name: Check if /etc/kubernetes/admin.conf exists
  stat:
    path: /etc/kubernetes/admin.conf
  register: admin_conf_check

- name: Set exists_admin_conf fact
  set_fact:
    exists_admin_conf: "{{ admin_conf_check.stat.exists }}"

- name: Initialize Kubernetes master node using kubeadm
  command: kubeadm init --config={{ K8S_INIT_FILE_REMOTE_DIR }}/config.yaml --upload-certs
  register: kubeadm_init_output
  failed_when: "'error' in kubeadm_init_output.stderr"
  changed_when: "'kubeadm init' not in kubeadm_init_output.stdout"
  when:
    - inventory_hostname == 'k8s-master1'
    - not exists_admin_conf

- name: Create .kube directory for root
  file:
    path: "{{ ansible_env.HOME }}/.kube"
    state: directory
    owner: root
    group: root
    mode: '0755'
  become: yes
  when: inventory_hostname == 'k8s-master1'

- name: Copy kubeconfig to .kube/config
  copy:
    src: /etc/kubernetes/admin.conf
    dest: "{{ ansible_env.HOME }}/.kube/config"
    owner: root
    group: root
    mode: '0644'
    remote_src: yes
  become: yes
  when: inventory_hostname == 'k8s-master1'

- name: Generate join command on the first master node
  command: kubeadm token create --print-join-command
  register: join_command
  delegate_to: k8s-master1
  run_once: true

- name: Debug the join command output (only on master1)
  debug:
    msg:
      - "join_command stdout: {{ join_command.stdout }}"
      - "join_command stderr: {{ join_command.stderr }}"
  when: inventory_hostname == 'k8s-master1' and join_command is defined

- name: Set join_command_value for all nodes (on master1)
  set_fact:
    join_command_value: "{{ join_command.stdout }}"
  when: inventory_hostname == 'k8s-master1' and join_command is defined

- name: Distribute join command to other nodes
  set_fact:
    join_command_value: "{{ hostvars['k8s-master1'].join_command.stdout }}"
  when: inventory_hostname != 'k8s-master1' and hostvars['k8s-master1'].join_command is defined

- name: Join master nodes to the cluster as control-plane
  command: "{{ join_command_value }} --control-plane --certificate-key {{ join_command_value }}"
  become: yes
  when:
    - inventory_hostname != 'k8s-master1'
    - inventory_hostname in groups['kube_control_plane']

# -- Worker Node Join --
- name: Check if kubelet.conf exists (joined already)
  stat:
    path: /etc/kubernetes/kubelet.conf
  register: kubelet_conf
  become: yes

- name: Set fact worker already joined
  set_fact:
    worker_already_joined: "{{ kubelet_conf.stat.exists }}"

- name: Join worker nodes to the cluster
  command: "{{ join_command_value }}"
  become: yes
  when:
    - inventory_hostname in groups['kube_node']
    - not worker_already_joined

```

* * *

### 3.6 Ansible 통해서 K8S Cluster 배포 :

```bash
$ ansible-playbook -i inventory/inventory.ini cluster.yml -vvv
```

* * *