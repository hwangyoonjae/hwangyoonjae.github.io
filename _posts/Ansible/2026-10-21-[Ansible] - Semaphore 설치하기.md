---
layout: post
title: "Semaphore 설치하기"
date: 2026-10-21
categories: [DevOps, Ansible, Semaphore]
tags: [Ansible, Kubernetes]
image: /assets/img/post-title/semaphore-wallpaper.png
---

## 1. 사전 확인 사항 :
### 1.1 Rocky Linux 10 CPU 요구사항 :

- Rocky Linux 10은 x86_64에서 **x86-64-v3**를 최소 요건으로 하며, VM 환경이라면 CPU 모델을 host-passthrough 등으로 설정해야 합니다.

```bash
$ /usr/lib64/ld-linux-x86-64.so.2 --help | grep x86-64-v3   # "supported" 표시 확인
```
![rocky linux 10 cpu 지원 확인](/assets/img/post/Ansible/rocky%20linux%2010%20cpu%20지원%20확인.png)

---

### 1.2 버전 구성 :

| 구성 요소 | 버전 | 설치 방식 |
|---|---|---|
| Rocky Linux | 10.2 (Minimal ISO) | 다운로드용 VM과 대상 서버 모두 |
| Python | 3.12 (Rocky 10 기본) | RPM |
| ansible-core | 2.21.x | pip wheel → venv (`/opt/ansible`) |
| Semaphore UI | 2.19.x | RPM |
| DB | SQLite (Semaphore 내장) | 별도 설치 없음 |

> Rocky 10 AppStream의 ansible-core RPM은 2.16이므로, 2.21을 사용하기 위해 PyPI wheel로 받아 venv에 설치합니다. ansible-core 2.21은 컨트롤 노드에 Python 3.12 ~ 3.14가 필요하며, Rocky 10 기본 Python(3.12)으로 사용할 수 있습니다.
{: .prompt-info}

---

### 1.3 Ansible 컬렉션 구성 :

- Docker / Kubernetes 설치 자체는 대부분 ansible-core 기본 모듈(`ansible.builtin`: `dnf`, `systemd_service`, `copy`, `template`, `lineinfile`, `command`)로 처리하고, 그 외 작업에 아래 컬렉션을 사용합니다.

| 컬렉션 | 용도 | 주요 모듈 |
|---|---|---|
| community.docker | 컨테이너, 이미지, 네트워크, 볼륨 관리 | `docker_container`, `docker_image`, `docker_image_load`, `docker_compose_v2`, `docker_network`, `docker_volume` |
| kubernetes.core | 파드, 디플로이먼트, 서비스 등 K8s 리소스 생성/조회, Helm 배포 | `k8s`, `k8s_info`, `k8s_exec`, `k8s_log`, `helm` |
| ansible.posix | 설치 전 OS 설정 | `sysctl`, `selinux`, `firewalld`, `mount` |
| community.general | 커널 모듈 로드 | `modprobe` (overlay, br_netfilter) |

> `community.docker`는 `community.library_inventory_filtering_v1` 컬렉션에 의존합니다. `ansible-galaxy collection download`가 의존 컬렉션도 자동으로 받으므로 따로 지정할 필요는 없습니다.
{: .prompt-info}

- 컬렉션 모듈은 **태스크가 실행되는 호스트의 Python**에서 동작하므로, 필요한 Python 패키지를 실행 위치에 설치해야 합니다.

| 컬렉션 | 필요한 패키지 | 설치 위치 |
|---|---|---|
| community.docker | `requests`(필수), `docker`(Swarm 관련 등 일부 모듈) | Docker 호스트 |
| kubernetes.core | `kubernetes`, `PyYAML`, `jsonpatch` | Semaphore 서버의 `/opt/ansible` venv (`hosts: localhost` + kubeconfig 방식 기준) |
| ansible.posix (`firewalld`, `selinux`) | `python3-firewall`, `python3-libselinux` | 대상 서버 시스템 Python (Rocky 10 Minimal 기본 포함) |

> `kubernetes.core.helm` 모듈을 사용하려면 Python 패키지 외에 **helm 바이너리**가 별도로 필요합니다.
{: .prompt-tip}

---

## 2. 설치 목록 :

| 항목 | 출처 | 저장 위치 | 설치 위치 |
|---|---|---|---|
| python3, python3-pip, git | Rocky 10 BaseOS/AppStream | `rpms/base` | Semaphore 서버 |
| sshpass | EPEL 10 | `rpms/base` | Semaphore 서버 (비밀번호 방식 SSH 접속 시에만 필요) |
| ansible-core 2.21 + 의존성 wheel | PyPI | `pip/ansible` | Semaphore 서버 (venv) |
| kubernetes, PyYAML, jsonpatch wheel | PyPI | `pip/k8s` | Semaphore 서버 (venv) |
| Ansible 컬렉션 | Ansible Galaxy | `collections` | Semaphore 서버 |
| semaphore_2.19.x_linux_amd64.rpm | GitHub Releases | `semaphore` | Semaphore 서버 |
| Docker CE RPM | download.docker.com | `rpms/docker` | Docker 호스트 (플레이북으로 설치) |
| requests, docker wheel | PyPI | `pip/docker` | Docker 호스트 (플레이북으로 설치) |

---

## 3. 다운로드용 VM 준비 :

1. Rocky 10.2 **Minimal ISO**로 VM을 설치합니다. 대상 서버와 같은 설치 구성으로 맞춥니다.
2. 인터넷 연결을 확인합니다.
3. **이 시점에서 VM 스냅샷을 생성합니다.** 이후 패키지를 추가로 받을 때마다 이 스냅샷으로 복원해서 사용합니다.

> RPM 다운로드(4.2~4.3)를 마치기 전에는 git, python3-pip 등 **대상 패키지를 설치하지 않습니다.** 설치하면 해당 의존성이 다운로드에서 빠질 수 있습니다.
{: .prompt-warning}

---

## 4. 다운로드용 VM 작업 (인터넷 연결 환경) :
### 4.1 다운로드 도구 설치 및 디렉터리 생성하기 :

```bash
$ dnf install -y dnf-plugins-core epel-release
$ dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

$ mkdir -p ~/offline/{rpms/base,rpms/docker,pip/ansible,pip/k8s,pip/docker,semaphore,collections}
$ cd ~/offline
```

---

### 4.2 OS 패키지 RPM 다운로드 :

- ansible-core는 pip로 설치하므로 RPM 목록에서 제외합니다.

```bash
$ dnf download --resolve --destdir=rpms/base \
  python3 python3-pip git sshpass
```

---

### 4.3 Docker CE RPM 다운로드 :

```bash
$ dnf download --resolve --destdir=rpms/docker \
  docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

### 4.4 Semaphore 2.19 RPM 다운로드 :

```bash
$ VER=2.19.16   # GitHub Releases에서 2.19.x 최신 패치 버전 확인 후 변경
$ ARCH=amd64
$ BASE=https://github.com/semaphoreui/semaphore/releases/download/v${VER}

$ curl -L -o semaphore/semaphore_${VER}_linux_${ARCH}.rpm ${BASE}/semaphore_${VER}_linux_${ARCH}.rpm
$ curl -L -o semaphore/semaphore_${VER}_checksums.txt     ${BASE}/semaphore_${VER}_checksums.txt

$ cd semaphore
$ sha256sum --ignore-missing -c semaphore_${VER}_checksums.txt
$ cd ..
```
![Semaphore 2.19 RPM 다운로드](/assets/img/post/Ansible/Semaphore%202.19%20RPM%20다운로드.png)

> 체크섬 파일 이름은 릴리스 페이지의 Assets 목록에서 확인하세요.
{: .prompt-tip}

---

### 4.5 Python wheel 다운로드 :

- RPM 다운로드가 끝났으므로 이제 python3-pip를 설치합니다.

```bash
$ dnf install -y python3-pip
$ python3 --version    # 3.12.x 확인

# Semaphore 서버 venv: ansible-core 2.21
$ python3 -m pip download -d pip/ansible "ansible-core>=2.21,<2.22"

# Semaphore 서버 venv: kubernetes.core 모듈용
$ python3 -m pip download -d pip/k8s kubernetes PyYAML jsonpatch

# Docker 호스트: community.docker 모듈용
$ python3 -m pip download -d pip/docker requests docker
```

> `>=2.21,<2.22`로 지정하면 2.21 계열의 최신 패치 버전을 받습니다. 특정 버전으로 고정하려면 `"ansible-core==2.21.5"`처럼 지정합니다.
{: .prompt-tip}

> cryptography 등 일부 wheel은 Python 버전과 CPU 아키텍처별 바이너리이므로, 반드시 대상 서버와 같은 **Rocky 10.2 / Python 3.12 / x86_64** 환경에서 받아야 합니다.
{: .prompt-warning}

---

### 4.6 오프라인 설치 검증하기 :

- 받은 wheel만으로 설치되는지 검증합니다. 이 venv는 다음 단계의 컬렉션 다운로드에도 사용합니다.

```bash
$ python3 -m venv /tmp/ansible-test
$ /tmp/ansible-test/bin/pip install --no-index \
    --find-links=pip/ansible --find-links=pip/k8s \
    ansible-core kubernetes jsonpatch
$ /tmp/ansible-test/bin/ansible --version     # core 2.21.x 확인
```

> `--no-index` 설치가 실패하면 누락된 wheel이 있다는 뜻이므로 4.5를 다시 확인합니다.
{: .prompt-warning}

---

### 4.7 Ansible 컬렉션 다운로드 :

- ansible-core 2.21 venv로 다운로드해야 2.21과 호환되는 컬렉션 버전을 받습니다.

```bash
$ cat <<'EOF' > collections-list.yml
collections:
  - name: community.docker
  - name: kubernetes.core
  - name: ansible.posix
  - name: community.general
EOF

$ /tmp/ansible-test/bin/ansible-galaxy collection download \
    -r collections-list.yml -p collections

$ ls collections
```

> `collections` 디렉터리에 각 컬렉션의 `*.tar.gz`와 `requirements.yml`이 생성됩니다. 의존 컬렉션인 `community-library_inventory_filtering_v1-*.tar.gz`도 포함되어 있는지 확인합니다.
{: .prompt-info}

---

### 4.8 패키징 파일 생성하기 :

```bash
$ cd ~
$ tar -zcvf semaphore-offline-rl10.2-$(date +%Y%m%d).tar.gz offline/
$ sha256sum semaphore-offline-*.tar.gz > checksum.txt
```

- 압축 파일 내부 구조

```
offline/
├── rpms/
│   ├── base/              # python3, python3-pip, git, sshpass + 의존성
│   └── docker/            # Docker CE + 의존성
├── pip/
│   ├── ansible/           # ansible-core 2.21 + 의존성 wheel
│   ├── k8s/               # kubernetes, PyYAML, jsonpatch (Semaphore venv)
│   └── docker/            # requests, docker (Docker 호스트)
├── semaphore/             # semaphore RPM + checksums
├── collections/           # 컬렉션 tar.gz + requirements.yml
└── collections-list.yml   # 컬렉션 다운로드 목록
```

> 작업이 끝나면 다운로드용 VM을 **3단계에서 만든 스냅샷으로 복원**해 다음 다운로드에 대비합니다.
{: .prompt-warning}

---

## 5. 대상 서버 작업 (폐쇄망) :
### 5.1 반입 파일 검증 및 압축 해제 :

```bash
$ sha256sum -c checksum.txt
$ tar -zxvf semaphore-offline-*.tar.gz -C /opt
```

---

### 5.2 OS 패키지 설치하기 :

- `--disablerepo='*'`로 온라인 repo 접속 없이 디렉터리의 RPM만으로 설치합니다.
- 이미 같은 버전이 설치된 패키지는 건너뛰고, 더 새 버전이 있으면 업데이트됩니다.
- 충돌 오류가 나면 오류 메시지에 나온 RPM 파일(대체 관계 패키지)을 삭제한 뒤 다시 실행합니다.

```bash
$ cd /opt/offline/rpms/base
$ dnf install -y --disablerepo='*' ./*.rpm

$ python3 --version   # 3.12.x
$ git --version
```

---

### 5.3 ansible-core 2.21 설치하기 (venv) :

```bash
$ python3 -m venv --system-site-packages /opt/ansible
$ /opt/ansible/bin/pip install --no-index \
    --find-links=/opt/offline/pip/ansible --find-links=/opt/offline/pip/k8s \
    ansible-core kubernetes jsonpatch

# semaphore 서비스와 모든 사용자가 사용할 수 있도록 PATH에 링크
$ for f in /opt/ansible/bin/ansible*; do ln -sf "$f" /usr/local/bin/$(basename "$f"); done

$ which ansible-playbook     # /usr/local/bin/ansible-playbook
$ ansible --version          # core 2.21.x
$ /opt/ansible/bin/python -c "import kubernetes, selinux; print('ok')"
```

> `--system-site-packages` 옵션은 시스템 RPM으로 설치된 SELinux / firewalld Python 바인딩을 venv에서도 사용하기 위해 필요합니다. 이 옵션이 없으면 Semaphore 서버 자신(`localhost`)을 대상으로 `copy`, `template`, `selinux`, `firewalld` 태스크를 실행할 때 libselinux 바인딩 오류가 발생합니다.
{: .prompt-warning}

> systemd의 기본 PATH에 `/usr/local/bin`이 포함되어 있어 Semaphore 서비스도 이 ansible을 사용합니다. 시스템 RPM의 ansible-core(2.16)는 설치하지 않으므로 버전이 섞이지 않습니다.
{: .prompt-info}

---

### 5.4 Ansible 컬렉션 설치하기 :

- semaphore 서비스 계정도 사용할 수 있도록 시스템 경로에 설치합니다.

```bash
$ cd /opt/offline/collections
$ ansible-galaxy collection install -r requirements.yml -p /usr/share/ansible/collections

$ ansible-galaxy collection list
```

> `requirements.yml`은 같은 디렉터리의 `tar.gz` 파일을 가리키므로 **반드시 `collections` 디렉터리 안에서 실행**합니다. 다른 위치에서 실행하면 Galaxy 서버 접속을 시도하다 실패할 수 있습니다.
{: .prompt-warning}

---

### 5.5 Semaphore 설치하기 :

```bash
$ dnf install -y --disablerepo='*' /opt/offline/semaphore/semaphore_*.rpm
$ semaphore version
```

---

### 5.6 Semaphore 전용 계정 및 디렉터리 생성하기 :
 
- root로 setup을 실행하면 설정 파일과 DB 파일 소유자가 root가 되어, 서비스를 다른 계정으로 실행할 때 권한 문제가 생깁니다. 전용 계정을 먼저 만들고 그 계정으로 setup을 실행합니다.

```bash
$ useradd --system --create-home --home-dir /var/lib/semaphore --shell /bin/bash semaphore
$ mkdir -p /etc/semaphore /var/lib/semaphore/playbooks
$ chown -R semaphore:semaphore /etc/semaphore /var/lib/semaphore
```
 
> 홈 디렉터리(`/var/lib/semaphore`)는 반드시 쓰기 가능해야 합니다. Ansible이 실행 중 `~/.ansible` 아래에 임시 파일을 생성합니다.
{: .prompt-warning}
 
---
 
### 5.7 Semaphore 초기 설정하기 :
 
```bash
$ cd /var/lib/semaphore
$ sudo -u semaphore semaphore setup
```
 
| 순서 | 질문 | 입력값 | 설명 |
|---|---|---|---|
| 1 | What database to use | **SQLite에 해당하는 번호** | 버전에 따라 번호가 다르므로 화면 목록에서 확인 (2.19부터 BoltDB 없음) |
| 2 | DB 파일 경로 | `/var/lib/semaphore/database.sqlite` | 별도 DB 서버 설치 불필요 |
| 3 | Playbook path (default /tmp/semaphore) | `/var/lib/semaphore/playbooks` | `/tmp`는 재부팅·정리 시 삭제될 수 있음 |
| 4 | Public URL (optional) | `http://<서버IP>:3000` 또는 Enter | 알림 메시지 링크에 사용 |
| 5 | Enable email alerts? | `no` | 내부 SMTP 서버가 있다면 `yes` |
| 6 | Enable telegram / slack / rocket.chat / teams 등 alerts? | 모두 `no` | 폐쇄망에서 외부 메신저 사용 불가 |
| 7 | Enable LDAP authentication? | `no` | 사내 LDAP/AD 연동 시에만 `yes` |
| 8 | Config output directory | `/etc/semaphore` | `config.json` 생성 위치 |
| 9 | Username | `admin` | 관리자 로그인 ID |
| 10 | Email | `admin@example.local` | 형식만 맞으면 됨 |
| 11 | Your name | `Administrator` | 화면 표시 이름 |
| 12 | Password | 관리자 비밀번호 | |
 
> 알림 관련 질문은 버전에 따라 개수와 종류가 다릅니다. 모두 `no`로 답하고, 필요하면 나중에 `config.json`에서 활성화합니다.
{: .prompt-info}
 
- 설정 결과를 확인합니다.

```bash
$ cat /etc/semaphore/config.json      # dialect가 sqlite인지, 경로가 맞는지 확인
$ ls -l /var/lib/semaphore/           # database.sqlite 생성 확인
```
 
- 백업은 서비스를 중지한 뒤 `database.sqlite`와 `config.json`을 복사합니다.

> 여러 팀이 동시에 사용하거나 HA 구성 계획이 있다면 처음부터 외부 DB(MySQL/MariaDB, PostgreSQL)를 사용하는 것이 좋습니다. 나중에 DB 종류를 바꾸는 작업은 간단하지 않습니다.
{: .prompt-tip}
 
> setup이 중간에 꼬이면 `/etc/semaphore/config.json`과 DB 파일을 지우고 다시 실행합니다. 관리자 계정만 다시 만들려면 아래 명령을 사용합니다.
{: .prompt-tip}
 
```bash
$ sudo -u semaphore semaphore user add --admin --login admin --name Administrator \
    --email admin@example.local --password '<비밀번호>' --config /etc/semaphore/config.json
```
 
---
 
### 5.8 systemd 서비스 등록하기 :
 
- RPM에 systemd 유닛이 포함되어 있는지 먼저 확인합니다.

```bash
$ rpm -ql semaphore | grep -i systemd
$ systemctl cat semaphore 2>/dev/null
```
 
- 유닛이 없으면 직접 생성합니다.

```bash
$ cat <<'EOF' > /etc/systemd/system/semaphore.service
[Unit]
Description=Semaphore UI
After=network-online.target
Wants=network-online.target
 
[Service]
Type=simple
User=semaphore
Group=semaphore
WorkingDirectory=/var/lib/semaphore
ExecStart=/usr/bin/semaphore server --config /etc/semaphore/config.json
Restart=on-failure
RestartSec=5
 
[Install]
WantedBy=multi-user.target
EOF
 
$ systemctl daemon-reload
$ systemctl enable --now semaphore
$ systemctl status semaphore
```
 
> `ExecStart`의 바이너리 경로는 `which semaphore` 결과와 맞춥니다.
{: .prompt-warning}
 
---
 
### 5.9 방화벽 설정하기 :
 
```bash
$ firewall-cmd --add-port=3000/tcp --permanent
$ firewall-cmd --reload
```
 
---
 
### 5.10 (권장) 온라인 repo 비활성화하기 :
 
- `--disablerepo='*'`를 매번 붙이지 않고 dnf를 쓸 때 타임아웃을 피하려면 온라인 repo를 꺼 둡니다.
```bash
$ sed -i 's/^enabled=1/enabled=0/' /etc/yum.repos.d/*.repo
```
 
---

## 6. Semaphore 설정하기 :
### 6.1 Semaphore 초기 설정 :

- 웹 UI(`http://<서버IP>:3000`)에서 관리자 계정 입력하여 로그인합니다.

![semaphore 초기 화면](/assets/img/post/Ansible/semaphore%20초기%20화면.png)

---

### 6.2 Semaphore 프로젝트 생성하기 :

- 최초 로그인 후에는 Ansible 자동화 작업을 관리하기 위한 프로젝트(Project)를 생성해야 합니다.

![semaphore 프로젝트 생성](/assets/img/post/Ansible/semaphore%20프로젝트%20생성.png)
![semaphore 프로젝트 생성 후 화면](/assets/img/post/Ansible/semaphore%20프로젝트%20생성%20후%20화면.png)

---