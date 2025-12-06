---
layout: post
title: "Keepalived를 통한 HAproxy 이중화 구성하기"
date: 2025-12-05
categories: [운영체제 & 네트워크, HAproxy]
tags: [Kubernetes, HAproxy, Loadbalancer, Keepalived]
image: /assets/img/post-title/haproxy-wallpaper.jpg
---

## 1. HAproxy 설치하기 :
### 1.1 설치하기 :

```bash
# 2대 노드(VM)에 진행한다.
$ dnf install -y haproxy
```

* * *

### 1.2 HAproxy 기본 설정하기 :

```bash
$ vi /etc/haproxy/haproxy.cfg

# 아래는 예시이고, 환경에 맞게 설정한다.
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

frontend k8s-api-server
  bind 0.0.0.0:6443
  #bind 0.0.0.0:26443  # k8s master에서 구축하는 경우 26443 포트 사용
  option tcplog
  mode tcp
  timeout client 30s
  default_backend k8s-api-server-backend

backend k8s-api-server-backend
   mode tcp
   balance roundrobin
   option tcp-check
   option tcplog
   server [HOSTNAME] [K8S_MASTER_IP]:6443 check 

# Ingress Contoroller(HTTP)
frontend ingress-http
  bind 0.0.0.0:80
  option tcplog
  mode tcp
  timeout client 30s
  default_backend ingress-http-backend

# Ingress Contoroller(HTTP)
backend ingress-http-backend
   mode tcp
   balance roundrobin
   option tcp-check
   option tcplog
   server [HOSTNAME] [K8S_WORKER_IP]:30080 check

# Ingress Contoroller(HTTPS)
frontend ingress-https
  bind 0.0.0.0:443
  option tcplog
  mode tcp
  timeout client 30s
  default_backend ingress-https-backend

# Ingress Contoroller(HTTPS)
backend ingress-https-backend
   mode tcp
   balance roundrobin
   option tcp-check
   option tcplog
   server [HOSTNAME] [K8S_WORKER_IP]:30443 check

listen stats
    bind 0.0.0.0:9999
    stats enable
    stats realm Haproxy Statistics  # 브라우저 타이틀
    stats uri /haproxy_stats  # stat 를 제공할 URI
```

* * *

## 2. Keepalived 설치하기 :
### 2.1 설치하기 :

```bash
# 2대 노드(VM)에 진행한다.
$ dnf install -y keepalived
```

* * *

### 2.2 Keepalived 기본 설정하기 :

- Loadbalancer1 (MASTER) 설정은 아래와 같다.

```bash
$ vi /etc/keepalived/keepalived.conf

# 아래는 예시이고, 환경에 맞게 설정한다.
vrrp_instance VI_1 {
    state MASTER          # LB1는 MASTER
    interface [NIC_NAME]
    virtual_router_id 200 # 두 서버 모두 동일해야 함
    priority 150          # MASTER가 더 높게
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        [VIP_Address]/[VIP_Netmask] dev [NIC_NAME]
    }
}
```

- Loadbalancer2 (WORKER) 설정은 아래와 같다.

```bash
$ vi /etc/keepalived/keepalived.conf

# 아래는 예시이고, 환경에 맞게 설정한다.
vrrp_instance VI_1 {
    state BACKUP          # LB2는 BACKUP
    interface [NIC_NAME]
    virtual_router_id 200 # 두 서버 모두 동일해야 함
    priority 100          # LB1보다 낮게
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        [VIP_Address]/[VIP_Netmask] dev [NIC_NAME]
    }
}
```

> virtual_ipaddress에서 dev 키워드를 사용하는 이유?
>
> virtual_ipaddress 항목에서는 Linux iproute2 문법을 따르므로 네트워크 인터페이스를 지정할 경우 반드시 dev <interface> 형식으로 명시해야 하며, 인터페이스 이름만 단독으로 기술하는 것은 유효한 문법이 아니다.
{: .prompt-info}

* * *

## 3. 서비스 시작 & 동작 확인하기 :
### 3.1 서비스 시작하기 :

- 2대 노드(VM)에서 진행한다.

```bash
# HAproxy 서비스 enabled 활성화 및 시작
$ systemctl enable haproxy
$ systemctl start haproxy

# keepalived 서비스 enabled 활성화 및 시작
$ systemctl enable keepalived
$ systemctl start keepalived
```

* * *

### 3.2 VIP 연결 확인하기 :

- Loadbalancer1 (MASTER)에서 VIP가 연결되었는지 확인한다.

```bash
$ ip addr show dev [NIC_NAME] | grep [VIP_Address]
```

* * *

## 4. Failover 테스트하기 :

- Loadbalancer1 (MASTER)에서 일부러 HAproxy나 Keepalived를 중지한다.

```bash
# 방법 1: haproxy 죽이기
$ systemctl stop haproxy

# 방법 2: keepalived 죽이기
$ systemctl stop keepalived
```

- Loadbalancer2 (WORKER)에서 지정한 네트워크 인터페이스의 VIP가 보이는지 확인한다.

```bash
$ ip addr show dev [NIC_NAME] | grep [VIP_Address]
```

* * *