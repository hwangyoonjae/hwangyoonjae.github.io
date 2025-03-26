---
layout: post
title: "[Kubernetes] - 쿠버네티스 HAproxy 구성하기"
date: 2024-02-19
categories: Kubernetes Install
tags: [Kubernetes, HAproxy, Loadbalance]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## HAproxy이란?:
- 고성능의 TCP/HTTP 로드 밸런서 및 프록시 서버로, 웹 서버의 로드 밸런싱과 고가용성을 제공하도록 설계되었다.

* * *

## HAproxy 특징:
1. ***고성능*** : HAProxy는 높은 네트워크 트래픽을 다룰 수 있으며, 일반적으로 한 서버에서 수십만 개의 동시 연결을 처리할 수 있다.
2. ***고가용성*** : HAProxy는 서버나 서비스가 실패하는 경우에도 계속 작동하도록 설계되었다.
이것은 트래픽을 여러 서버 간에 분산시키는 로드 밸런싱뿐만 아니라, 주 서버가 다운되는 경우를 대비한 대체 서버 (failover) 지정 기능을 통해 이루어진다. (keepalived와 같이 사용)
3. ***유연성*** : 다양한 로드 밸런싱 알고리즘을 지원하며, 트래픽 라우팅과 관련된 광범위한 옵션을 제공한다. 또한 HTTP, HTTPS, TCP 등 다양한 프로토콜을 지원한다.
4. ***보안*** : SSL/TLS 암호화를 지원하며, HTTP 요청 및 응답에 대한 상세한 통제를 제공함으로써 안전한 웹 통신을 가능하게 한다.

* * *

## Kubernetes HAproxy 구성도:
- Kubernetes HAproxy 구성은 아래 그림과 같이 구현한다.
[![Kubernetes HAproxy 구성도](/assets/img/post/kubernetes/Kubernetes%20HAproxy%20구성도.png)](/assets/img/post/kubernetes/Kubernetes%20HAproxy%20구성도.png)

* * *

## keepalived 설치 및 설정:
- master node간의 HA를 위해 설치한다.
```bash
$ apt install -y keepalived
$ cd /etc/keepalived
$ vi keepalived.conf
```

- k8s-master1에는 아래 내용을 입력한다.
```bash
# master/backup 구분
global_defs {
   router_id rtr_0
}
vrrp_instance VI_1 {
    state MASTER
    interface [인터페이스명]
    virtual_router_id 50
    priority 100
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass Password@
    }
    virtual_ipaddress {
        VIP address
    }
}
```

- k8s-master2에는 아래 내용을 입력한다.
```bash
# master/backup 구분
global_defs {
   router_id rtr_1
}
vrrp_instance VI_2 {
    state BACKUP
    interface [인터페이스명]
    virtual_router_id 50
    priority 99
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass Password@
    }
    virtual_ipaddress {
        VIP address
    }
}
```

- k8s-master3에는 아래 내용을 입력한다.
```bash
# master/backup 구분
global_defs {
   router_id rtr_1
}
vrrp_instance VI_3 {
    state BACKUP
    interface [인터페이스명]
    virtual_router_id 50
    priority 98
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass Password@
    }
    virtual_ipaddress {
        VIP address
    }
}
```

- 모든 Master에서 keepalived 재시작 후 ping 테스트한다.
```bash
$ systemctl restart keepalived
$ systemctl enable keepalived
# ping 테스트
$ ping [vip address]
```

* * *

## 로드밸런서 구성하기:
- 고가용성(HA)을 위해서는 로드밸런서가 필요한데, 로드 밸런서 뒤에 있는 apiserver 중 한개의 apiserver에 장애가 발생해도, 나머지 apiserver로 정상적인 서비스를 하도록 로드를 분배한다.
- 로드 밸런서는 ***HAproxy*** application을 설치한다.
```html
⚠️ 모든 Master Node의 설치한다.
```
```bash
$ apt install haproxy
```

- HAproxy 설정 파일의 내용을 입력한다.
```bash
$ cat << EOF | sudo tee -a /etc/haproxy/haproxy.cfg
frontend kubernetes-master-lb
      bind 0.0.0.0:16443
      mode tcp
      default_backend kubernetes-master-nodes
backend kubernetes-master-nodes
      mode tcp
      balance roundrobin
      server k8s-master1 {k8s-master1 IP}:6443 check
      server k8s-master2 {k8s-master2 IP}:6443 check
      server k8s-master3 {k8s-master3 IP}:6443 check
EOF
```

- HAproxy 서비스 재시작한다.
```bash
$ systemctl restart haproxy
$ systemctl enable haproxy
```

- LoadBalancer 설정을 확인한다.
```bash
$ nc -v [LoadBalancer_IP] [Port]
```

* * *

## Kubernetes 설치하기:
- Kubernetes HA 구성을 위해 Kubernetes 설치가 필요하다.
> * [Kubernetes 설치방법](https://hwangyoonjae.github.io/kubernetes/Kubernetes-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0(%EB%8F%84%EC%BB%A4-X)/ "Kubernetes 설치방법")

* * *

## Kubernetes 초기화 및 노드 조인하기:
### kubernetes 초기화:
- 컨트롤 플레인 노드 중 하나에서 kubernetes 초기화 작업을 진행한다.
```bash
$ kubeadm init --control-plane-endpoint=k8s-lb:6443 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address={k8s-master1 IP} --cri-socket unix:///var/run/crio/crio.sock --upload-certs
```
[![Kubernetes HAproxy 구성 초기화](/assets/img/post/kubernetes/Kubernetes%20HAproxy%20구성%20초기화.png)](/assets/img/post/kubernetes/Kubernetes%20HAproxy%20구성%20초기화.png)

- ***kubectl***은 항상 ***$HOME/.kube***를 참조하므로 root가 아닌 노멀 사용자는 반드시 인증을 해야한다.
```bash
# 아래 명령어를 통해서 인증한다.
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

* * *

### Kubernetes Master Node 조인하기:
- 다른 마스터노드(Master Node)들에서는 아래 명령어를 실행한다.
```html
⚠️ 토큰 값과 암호키 값은 다를 수 있으니 참고 바랍니다.
```
```bash
$ kubeadm join k8s-lb:6443 --token nd6wjc.6i4fgf5ig28mp4lz \
      --discovery-token-ca-cert-hash sha256:1bddd38ae800a09195c9d52868750a1c43200957e7a2b246251f44235d301c2c \
      --control-plane --certificate-key 4c637105d0e0edcaab6979975efa61a475c1db1944e76de2d391bb2fdb7b6f05
```

* * *

### Kubernetes Worker Node 조인하기:
- 워커노드(Worker Node)들에서는 아래 명령어를 실행한다.
```html
⚠️ 토큰 값과 암호키 값은 다를 수 있으니 참고 바랍니다.
```
```bash
$ kubeadm join k8s-lb:6443 --token nd6wjc.6i4fgf5ig28mp4lz \
      --discovery-token-ca-cert-hash sha256:1bddd38ae800a09195c9d52868750a1c43200957e7a2b246251f44235d301c2c
```

* * *

## Kubernetes 모든 노드 확인하기:
- 모든 노드에서 조인 성공했다면 확인한다.
```bash
$ kubectl get nodes
```
[![Kubernetes HAproxy 클러스터 구축 노드](/assets/img/post/kubernetes/Kubernetes%20HAproxy%20클러스터%20구축%20노드.png)](/assets/img/post/kubernetes/Kubernetes%20HAproxy%20클러스터%20구축%20노드.png)

* * *