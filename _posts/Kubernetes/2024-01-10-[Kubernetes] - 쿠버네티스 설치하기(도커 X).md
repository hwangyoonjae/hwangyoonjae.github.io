---
title: "[Kubernetes] - 쿠버네티스 설치하기(도커 X)"
categories:
  - Kubernetes
tags:
  - [Kubernetes, Container]

toc: true
toc_sticky: true

date: 2024-01-10
last_modified_at: 2024-01-10
---

## Kubernetes Docker 지원 중단:
- Kubernetes에서 V1.20.0이후부터 Docker 지원 중단되어 Docker는 설치하지 않고 진행한다.

* * *

## Kubernetes 설치 전 준비사항:
### Containerd 설치하기:
- Kubernetes Pod에서 컨테이너를 실행하기 위해 설치한다.
```bash
$ yum install yum-utils -y
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ yum install containerd.io -y
```

### Containerd 구성 파일 생성하기:
```bash
$ mkdir -p /etc/containerd
$ containerd config default | sudo tee /etc/containerd/config.toml
```

## Containerd 서비스 시작하기:
```bash
$ systemctl enable containerd
$ systemctl start containerd
```

* * *

### Selinux permissive 모드로 변경하기:
- 컨테이너가 호스트 파일시스템에 접근하는 것을 용이하게 하게 위해서 **"enforcing -> permissive"**로 변경한다.
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
$ systemctl stop firewalld
$ systemctl disable firewalld
```

* * *

### br_netfilter활성화 & iptables 설정
- kubernetes는 iptables를 이용하여 pod간 통신을 가능하게 하기에 정상 동작하도록 설정한다.
```bash
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ sysctl --system
```

- iptables는 커널상에서의 netfilter 패킷필터링 기능을 사용자 공간에서 제어하는 수준으로 사용할 수 있어 iptables의 설정을 따라가기 위해서는 br_netfilter를 enable 시켜줘야 한다.
```bash
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
$ modprobe overlay
$ modprobe br_netfilter
```

* * *

### 서버 재부팅하기:
- 변경된 설정값으로 적용하기 위해 서버를 재부팅한다.
```bash
$ reboot
```

* * *

## Kubernetes 설치하기:
### kubernetes YUM Repository 설정하기:
- kubernetes.repo 등록한다.
```bash
$ cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

* * *

### kubeadm, kubelet, kubectl 패키지 설치하기:
- kubeadm: kubernetes cluster를 구축하기 위한 명령 도구이다.
- kubelet: master node, worker node에서 데몬 프로세스로 기동되어 있으면서, container와 pod를 생성/삭제/상태를 감시한다.
- kubectl: 사용자가 kubernetes cluster에게 작업 요청하기 위한 명령 도구이다.
```bash
$ yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
$ systemctl enable kubelet
$ systemctl start kubelet
```

* * *

### Flannel 설치하기:
- 서로 다른 노드에 있는 pod 간 통신을 완성하기 위해서 network plugin인 Flannel을 설치한다.
```bash
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml 
$ systemctl restart kubelet
```

* * *

## Kubernetes Master Node 작업하기:
- **kubeadm init** 명령어를 통해 초기화를 진행한다.
```bash
$ kubeadm reset
$ kubeadm init --apiserver-advertise-address {k8s-master IP} --pod-network-cidr=10.244.0.0/16
```
[![Master Node 구성완료](/assets/images/kubernetes/Master%20Node%20%EA%B5%AC%EC%84%B1%EC%99%84%EB%A3%8C.PNG)](/assets/images/kubernetes/Master%20Node%20%EA%B5%AC%EC%84%B1%EC%99%84%EB%A3%8C.PNG)

- 혹시나 Kubernetes 설치 간에 문제 발생 시 아래 링크를 참고하여 오류를 해결하면된다.
> * [Kubernetes 오류 해결](https://hwangyoonjae.github.io/kubernetes/Kubernetes-쿠버네티스-설치-중-오류-해결하기/ "Kubernetes 오류 해결")<br>
<span style="color:#FA5858; font-size:12px">※ 필자가 설치한 과정에서 문제 발생 시 조치한 사항으로 독자의 테스트 과정에서 문제 발생에 대한 조치사항은 없을 수 있다.</span>

* * *

## Kubernetes Worker Node 작업하기:
- **kubeadm join** 명령어를 통해 노드서버를 쿠버네티스 클러스터로 결합한다.
```bash
$ kubeadm join {k8s-master IP}:6443 --token {token 값} --discovery-token-ca-cert-hash {hash 값}
```
[![Worker Node 구성완료](/assets/images/kubernetes/Worker%20Node%20%EA%B5%AC%EC%84%B1%EC%99%84%EB%A3%8C.PNG)](/assets/images/kubernetes/Worker%20Node%20%EA%B5%AC%EC%84%B1%EC%99%84%EB%A3%8C.PNG)

* * *