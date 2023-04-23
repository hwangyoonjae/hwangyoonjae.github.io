---
title: "[Kubernetes] - 쿠버네티스 설치하기"
categories:
  - Web
tags:
  - [Kubernetes, Docker, Container]

toc: true
toc_sticky: true

date: 2023-04-15
last_modified_at: 2023-04-15
---

## Kubernetes란?:
- 컨테이너화된 워크로드와 서비스를 관리하기 위한 이식성이 있고, 확장가능한 오픈소스 플랫폼이다.
- 선언적 구성과 자동화를 모두 용이하게 해주며 크고, 빠르게 성장하는 생태계를 가지고 있다.

* * *

## Kubernetes 설치 전 준비사항:
### Docker 설치하기:
- docker 설치를 진행해야하므로 아래 링크 참조하면된다.
> * [Docker 설치방법](https://hwangyoonjae.github.io/docker/Docker-Docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/ "Docker 설치방법")

* * *

### Selinux permissive 모드로 변경하기:
- 컨테이너가 호스트 파일시스템에 접근하는 것을 용이하게 하게 위해서 **"disabled -> permissive"**로 변경한다.
```bash
$ setenforce 0
$ sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

* * *

### SWAP 비활성화하기:
- kubernetes 개발팀의 swap 메모리 지원에 대한 우선순위가 낮기 때문에 지원하지 않아 비활성화한다.
```bash
$ swapoff -a
$ sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

* * *

### 방화벽 비활성화하기:
- kubernetes 운영상 반드시 열려있어야 하는 필수 포트가 있는데, 이것이 닫겨 있을 경우 쿠버네티스 운영 및 설치에 장애가 될 수 있기 떄문에 우선 설치 테스트에서는 비활성화한다.
```bash
$ systemctl disable firewalld
$ systemctl stop firewalld
```

* * *

### br_netfilter활성화 & iptables 설정
- kubernetes는 iptables를 이용하여 pod간 통신을 가능하게 하기에 정상 동작하도록 설정한다.
```bash
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

$ sudo sysctl --system
```

- iptables는 커널상에서의 netfilter 패킷필터링 기능을 사용자 공간에서 제어하는 수준으로 사용할 수 있어 iptables의 설정을 따라가기 위해서는 br_netfilter를 enable 시켜줘야 한다.
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```

* * *

### 서버 재부팅하기:
- 변경된 설정값으로 적용하기 위해 서버를 재부팅한다.
```bash
$ reboot
```

* * *