---
layout: post
title: "Kubernetes 설치하기"
date: 2023-04-15
categories: [컨테이너, Kubernetes]
tags: [Kubernetes, Docker, Container]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. Kubernetes란? :
- 컨테이너화된 워크로드와 서비스를 관리하기 위한 이식성이 있고, 확장가능한 오픈소스 플랫폼입니다.
- 선언적 구성과 자동화를 모두 용이하게 해주며 크고, 빠르게 성장하는 생태계를 가지고 있습니다.

* * *

## 2. Kubernetes 설치 전 준비사항 :
### 2.1 Docker 설치하기 :
- docker 설치를 진행해야하므로 아래 링크 참조하면됩니다.
> * [Docker 설치방법](https://hwangyoonjae.github.io/docker/Docker-Docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/ "Docker 설치방법")

* * *

### 2.2 Selinux permissive 모드로 변경하기 :
- 컨테이너가 호스트 파일시스템에 접근하는 것을 용이하게 하게 위해서 **"enforcing -> permissive"**로 변경합니다.

```bash
$ setenforce 0
$ sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

* * *

### 2.3 SWAP 비활성화하기 :
- kubernetes 개발팀의 swap 메모리 지원에 대한 우선순위가 낮기 때문에 지원하지 않아 비활성화합니다.

```bash
$ swapoff -a
$ sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

* * *

### 2.4 방화벽 비활성화하기 :
- kubernetes 운영상 반드시 열려있어야 하는 필수 포트가 있는데, 이것이 닫겨 있을 경우 쿠버네티스 운영 및 설치에 장애가 될 수 있기 떄문에 우선 설치 테스트에서는 비활성화합니다.

```bash
$ systemctl stop firewalld
$ systemctl disable firewalld
```

* * *

### 2.5 br_netfilter활성화 & iptables 설정하기 :
- kubernetes는 iptables를 이용하여 pod간 통신을 가능하게 하기에 정상 동작하도록 설정합니다.

```bash
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

$ sysctl --system
```

* * *

- iptables는 커널상에서의 netfilter 패킷필터링 기능을 사용자 공간에서 제어하는 수준으로 사용할 수 있어 iptables의 설정을 따라가기 위해서는 br_netfilter를 enable 시켜줘야 합니다.

```bash
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```

* * *

### 2.6 서버 재부팅하기 :
- 변경된 설정값으로 적용하기 위해 서버를 재부팅합니다.

```bash
$ reboot
```

* * *

## 3. Kubernetes 설치하기 :
### 3.1 kubernetes YUM Repository 설정하기 :
- kubernetes.repo 등록합니다.

```bash
$ vi /etc/yum.repos.d/kubernetes.repo
```
```bash
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```

* * *

### 3.2 kubeadm, kubelet, kubectl 패키지 설치하기 :
- kubeadm: kubernetes cluster를 구축하기 위한 명령 도구입니다.
- kubelet: master node, worker node에서 데몬 프로세스로 기동되어 있으면서, container와 pod를 생성/삭제/상태를 감시합니다.
- kubectl: 사용자가 kubernetes cluster에게 작업 요청하기 위한 명령 도구입니다.

```bash
$ yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
$ systemctl enable kubelet
$ systemctl start kubelet
```

* * *

### 3.3 daemon.json 편집하기 :
```bash
$ cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

* * *

## 4. Kubernetes Master Node 작업하기 :
- **kubeadm init** 명령어를 통해 초기화를 진행합니다.

```bash
$ kubeadm reset
$ kubeadm init --apiserver-advertise-address {k8s-master IP} --pod-network-cidr=172.16.0.0/16
```
![Master Node 구성완료](/assets/img/post/kubernetes/Master%20Node%20%EA%B5%AC%EC%84%B1%EC%99%84%EB%A3%8C.PNG)

- 혹시나 Kubernetes 설치 간에 문제 발생 시 아래 링크를 참고하여 오류를 해결하면됩니다.
> * [Kubernetes 오류 해결](https://hwangyoonjae.github.io/kubernetes/Kubernetes-쿠버네티스-설치-중-오류-해결하기/ "Kubernetes 오류 해결")<br>

> 필자가 설치한 과정에서 문제 발생 시 조치한 사항으로 독자의 테스트 과정에서 문제 발생에 대한 조치사항은 없을 수 있습니다.
{: .prompt-info}

* * *

## 5. Kubernetes Worker Node 작업하기 :
- **kubeadm join** 명령어를 통해 노드서버를 쿠버네티스 클러스터로 결합합니다.

```bash
$ kubeadm join {k8s-master IP}:6443 --token {token 값} --discovery-token-ca-cert-hash {hash 값}
```
![Worker Node 구성완료](/assets/img/post/kubernetes/Worker%20Node%20%EA%B5%AC%EC%84%B1%EC%99%84%EB%A3%8C.PNG)

* * *