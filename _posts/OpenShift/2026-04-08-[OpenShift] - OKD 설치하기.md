---
layout: post
title: "OKD 설치하기"
date: 2026-04-08
categories: [컨테이너, Openshift]
tags: [Openshift, OKD]
image: /assets/img/post-title/openshift-wallpaper.jpg
---

## 1. OKD를 설치하게 된 계기 :

> 온프레미스 및 폐쇄망 환경에서 비용 부담 없이 OpenShift 아키텍처를 검증하기 위해 OKD를 활용하여 Kubernetes 환경을 구축하였습니다.
{: .prompt-warning}

* * *

## 2. VM 생성하기 :

| 구분        | 호스트                 | IP                     | 수량 | OS           | CPU | MEM | DISK |
|------------|----------------------|------------------------|------|--------------|-----|-----|------|
| 관리 서버   | manager.okd.local     | 192.168.1.164        | 1    | Rocky 9.6    | 4   | 8   | 50   |
| LB 서버     | api.lb.okd.local      | 192.168.1.163        | 1    | Rocky 9.6    | 4   | 8   | 50   |
| 부트스트랩 서버 | bootstrap.okd.local   | 192.168.1.165        | 1    | rhcos-4.14.0 | 4   | 8   | 50   |
| 마스터 노드 | master01~03.okd.local | 192.168.1.166 ~ 168  | 3    | rhcos-4.14.0 | 8   | 16  | 70   |

> 일반적으로 OKD 클러스터는 마스터 노드와 워커 노드를 분리하여 구성하는 것이 권장되나, 본 테스트 환경은 제한된 리소스 내에서 단순 기능 검증을 목적으로 구성하였습니다.
> 
> 이에 따라 별도의 워커 노드를 구성하지 않고, 3대의 마스터 노드에 워크로드(파드)를 직접 배치하는 방식으로 클러스터를 구성하였습니다.
{: .prompt-warning}

* * *

## 3. DNS 설치하기 :

> 해당 과정은 관리서버에서 진행합니다.
{: .prompt-info}

- Openshift 사용을 위해 내부 DNS 를 준비해야 하며, OCP 서버에서 DNS, LoadBalancer 및 http 파일 서버 역할을 담당하게 됩니다.

* * *

### 3.1 DNS 설치 및 세팅하기 :

- SELinux, 방화벽 비활성화 설정을 하고 적용합니다.

```bash
# SElinux 비활성화
$ vi /etc/selinux/config

SELINUX=disabled
```

```bash
# 방화벽 비활성화
$ systemctl stop firewalld
$ systemctl disable firewalld
```

* * *

- bind 패키지를 설치합니다.

```bash
$ dnf install bind bind-utils
```

* * *

### 3.2 DNS 서버 설정하기 :

- 정방향 및 역방향 zone 파일을 생성합니다.

```bash
# 정방향 DMZ Zone 파일
$ vi /var/named/okd.local.zone

# 아래와 같이 내용을 작성합니다.
$TTL 60
@   IN  SOA okd-manager.okd.local. root (
        2026040201 ; serial
        1D         ; refresh
        1H         ; retry
        1W         ; expire
        3H )       ; minimum

    IN  NS  okd-manager.okd.local.

okd-manager     IN  A   192.168.1.164

; LB (api / ingress)
api             IN  A   192.168.1.163
api.lb          IN  A   192.168.1.163
api-int.lb      IN  A   192.168.1.163
*.lb            IN  A   192.168.1.163

; bootstrap
bootstrap       IN  A   192.168.1.165

; master
master01        IN  A   192.168.1.166
master02        IN  A   192.168.1.167
master03        IN  A   192.168.1.168

; worker
worker01        IN  A   192.168.1.161
worker02        IN  A   192.168.1.162
```

* * *

```bash
# 역방향 DNS Zone 파일
$ vi /var/named/1.168.192.rev.zone

# 아래와 같이 내용을 작성합니다.
$TTL 60
@   IN  SOA okd-manager.okd.local. root (
        2026040201 ; serial
        1D
        1H
        1W
        3H )
    IN  NS  okd-manager.okd.local.
164 IN PTR okd-manager.okd.local.
163 IN PTR api-lb.okd.local.
163 IN PTR api-int-lb.okd.local.
165 IN PTR bootstrap.okd.local.
166 IN PTR master01.okd.local.
167 IN PTR master02.okd.local.
168 IN PTR master03.okd.local.
161 IN PTR worker01.okd.local.
162 IN PTR worker02.okd.local.
```

* * *

- 외부에서도 질의가 가능하도록 하고 zone 파일을 config 파일에 연결 합니다.

```bash
$ vi /etc/named.conf

options {
        listen-on port 53 { any; }; # 해당 부분 변경
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; }; # 해당 부분 변경

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};
```

* * *

- DNS Zone 선언(정의) 파일의 zone 파일을 연결합니다.

```bash
$ vi /etc/named.rfc1912.zones

...

zone "okd.local" IN {
    type master;
    file "okd.local.zone";
    allow-update { none; };
};

zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "1.168.192.rev.zone";
    allow-update { none; };
};
```

* * *

- named 서비스 자동 시작 활성화 및 서비스 재시작합니다.

```bash
$ systemctl enable named
$ systemctl start named
```

* * *

- 설정한 정방향과 역방향 질의가 잘되는지 확인합니다.

```bash
$ nslookup okd-manager.okd.local localhost
```

![정방향역방향 질의 테스트](/assets/img/post/Openshift/정방향역방향%20질의%20테스트.png)

* * *

## 4. HTTP 설정하기 :

> 해당 과정은 관리서버에서 진행합니다.
{: .prompt-info}

### 4.1 Nginx 설치하기 :
- nginx 패키지를 설치합니다.

```bash
$ dnf install nginx
```

* * *

- nginx 설정 파일을 열고 아래 내용을 수정합니다.

```bash
$ vi /etc/nginx/conf.d/default.conf

server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html/files;
        index  index.html index.htm;
        autoindex on;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

* * *

- 파일 서버로 사용할 디렉토리를 생성하여 적용합니다.

```bash
$ mkdir /usr/share/nginx/html/files
$ systemctl reload nginx
```

* * *

- nginx 서비스 자동 시작 활성화 및 서비스 재시작합니다.

```bash
$ systemctl enable nginx
$ systemctl start nginx
```

* * *

## 5. LB(HAproxy) 설정하기 :

> 해당 과정은 LB서버에서 진행합니다.
{: .prompt-info}

### 5.1 HAproxy 설치하기 :

- HAproxy 패키지를 설치합니다.

```bash
$ dnf install haproxy
```

* * *

- haproxy 설정 파일을 백업합니다.

```bash
$ cp -arp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
```

* * *

- haproxy 설정 파일을 열고 아래 내용을 수정합니다.

```bash
$ vi /etc/nginx/conf.d/default.conf

...

listen api-server-6443
    bind *:6443
    mode tcp
    option tcplog
    balance roundrobin
    server bootstrap  bootstrap.okd.local:6443 check inter 1s backup
    server master01   master01.okd.local:6443 check inter 1s
    server master02   master02.okd.local:6443 check inter 1s
    server master03   master03.okd.local:6443 check inter 1s

listen machine-config-server-22623
    bind *:22623
    mode tcp
    option tcplog
    balance roundrobin
    server bootstrap  bootstrap.okd.local:22623 check inter 1s backup
    server master01   master01.okd.local:22623 check inter 1s
    server master02   master02.okd.local:22623 check inter 1s
    server master03   master03.okd.local:22623 check inter 1s

listen ingress-router-443
    bind *:443
    mode tcp
    option tcplog
    balance source
    server master01   master01.okd.local:443 check inter 1s
    server master02   master02.okd.local:443 check inter 1s
    server master03   master03.okd.local:443 check inter 1s

listen ingress-router-80
    bind *:80
    mode tcp
    option tcplog
    balance source
    server master01   master01.okd.local:80 check inter 1s
    server master02   master02.okd.local:80 check inter 1s
    server master03   master03.okd.local:80 check inter 1s
```

* * *

- haproxy 서비스 자동 시작 활성화 및 서비스 재시작합니다.

```bash
$ systemctl enable haproxy
$ systemctl start haproxy
```

* * *

## 6. Openshift Installer 설치 :

> 해당 과정은 관리서버에서 진행합니다.
{: .prompt-info}

### 6.1 Openshift Installer 설치 및 Client 파일 다운로드 :
- Openshift Installer 파일은 아래 다운로드 미러 서버에서 다운받습니다.

> * [다운로드 미러 서버](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/ "다운로드 미러 서버")

> 필자는 4.14.0 버전으로 다운받았습니다.
{: .prompt-tip}

* * *

- 다운받은 install 및 client 파일을 서버에 업로드하고, 압축 해제하여 설치합니다.

```bash
$ tar -zxvf openshift-install-linux-4.14.0-0.okd-2024-01-26-175629.tar.gz
$ ./openshift-install version
```
![install 버전 정보](/assets/img/post/Openshift/install%20버전%20정보.png)

* * *

```bash
$ tar -zxvf openshift-client-linux-4.14.0-0.okd-2024-01-26-175629.tar.gz
$ mv oc kubectl /usr/local/bin/
$ oc version
```
![client 버전 정보](/assets/img/post/Openshift/client%20버전%20정보.png)

* * *

### 6.2 SSH Key 생성하기 :

- 클러스터 노드 SSH 접근을 위해 키를 생성하며, 생성된 키가 노드에 전달되면 클러스터 각 노드 SSH 원격 접속을 할 수 있습니다.

```bash
$ ssh-keygen -t ed25519 -N '' -f ~/.ssh/id_ed25519
$ cat ~/.ssh/id_ed25519.pub
```
![ssh key 생성](/assets/img/post/Openshift/ssh%20key%20생성.png)

* * *

### 6.3 설치 구성 파일 생성하기 :

- OKD 클러스터 설치에 필요한 기본 환경 정보를 정의하기 위해 install-config.yaml 파일을 생성합니다.

```bash
# 폴더 생성
$ mkdir installation_directory
$ cd installation_directory

# config 파일 수정
$ vi install-config.yaml
apiVersion: v1
baseDomain: okd.local
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: lb
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: '{Redhat 홈페이지에서 복사한 Pull secret 키}'
sshKey: '[ssh-keygen 으로 생성한 public key]'
```
![install config 파일](/assets/img/post/Openshift/install%20config%20파일.png)

> - install-config.yaml에서 pullSecret과 sshKey 값은 아래 방법 대로 진행하면됩니다.
>
>   - pullSecret 값 확인 방법 : 아래 가이드 참고
> 
>   - sshKey 값 확인 방법 : “6.2 SSH Key 생성하기” 가이드에서 확인한 ssh key 값
{: .prompt-example}


* * *

### 6.4 Manifest 파일 생성하기 :

- manifest 파일 생성시 install-config.yaml 파일이 삭제되므로 만일을 위해 백업합니다.

```bash
$ cp -arp install-config.yaml install-config.yaml.bak
```

* * *

- manifest 파일 생성합니다.

```bash
$ cd ..
$ ./openshift-install create manifests --dir installation_directory/
$ cd installation_directory
$ ll
```
![manifest 생성](/assets/img/post/Openshift/manifest%20생성.png)

* * *

### 6.5 ignition 파일 생성하기 :
- CoreOS 를 초기 구성하는데 필요한 파일 ignition 파일을 생성합니다.

```bash
$ cd ..
$ ./openshift-install create ignition-configs --dir installation_directory/
$ cd installation_directory
$ ll
```
![ignition 파일 생성](/assets/img/post/Openshift/ignition%20파일%20생성.png)

* * *

- 생성된 ignition 파일들을 nginx 파일 서버 디렉토리로 복사하고 다운로드가 가능하도록 퍼미션을 조정합니다.

```bash
$ cp -arp *.ign /usr/share/nginx/html/files/

$ chmod 644 /usr/share/nginx/html/files/*
```

* * *

### 6.6 Hash 파일 생성하기 :

- 추후 ign 검증을 위해 해시값이 필요한데, 해시값의 길이가 길어 타이핑이 어려우므로 hash 파일에 해시값을 넣습니다.

```bash
$ cd /usr/share/nginx/html/files
$ sha512sum bootstrap.ign |awk {'print $1'} > bootstrap.hash
$ sha512sum master.ign |awk {'print $1'} > master.hash
$ sha512sum worker.ign |awk {'print $1'} > worker.hash
```

* * *

## 7. bootstrap 서버 설치하기 :

> 해당 과정은 부트스트랩서버에서 진행합니다.
{: .prompt-info}

### 7.1 bootstrap OS 생성 및 설치하기 :
- RedHat CoreOS ISO 파일을 다운로드 하고, OS 설치를 진행하며, 해당 서버 (bootstrap, master, worker)에 설치를 해야 합니다.

> * [ISO 다운로드](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/ "ISO 다운로드")

> 필자는 4.14.0 버전으로 다운받았습니다.
{: .prompt-tip}

* * *

- 위 URL 통해서 다운받은 ISO 파일로 VM 부팅 시 아래와 같습니다.

![rhcos 부팅 시 첫 화면](/assets/img/post/Openshift/rhcos%20부팅%20시%20첫%20화면.png)

* * *

- 네트워크 설정을 진행합니다.

```bash
$ nmtui
```

> 환경에 맞게 IP정보를 입력해주고, DNS는 관리서버 IP주소로 입력합니다.
{: .prompt-info}

![VM 네트워크 설정](/assets/img/post/Openshift/VM%20네트워크%20설정.png)

* * *

- 네트워크 설정 후 sudo 를 붙여 root 권한으로 데몬을 재시작 해줍니다.

```bash
$ sudo systemctl restart NetworkManager
```

* * *

- ignition 파일의 해시값을 변수에 저장하고 이를 이용하여 OS 설치를 진행합니다.

```bash
$ hash=`curl http://192.168.1.164/bootstrap.hash`
$ sudo coreos-installer install --copy-network --ignition-url http://192.168.1.164/bootstrap.ign /dev/sda --ignition-hash sha512-${hash}
```

> 설치 완료 후 재부팅 진행해야합니다.
{: .prompt-warning}

* * *

### 7.2 bootstrap 서버와 Kubernetes API 여결되는지 확인하기 :

- bootstrap 서버 설치를 완료한 것이므로 Kubernetes API 연결이 잘 되는지 아래 명령으로 확인해봅니다.

```bash
$ ./openshift-install --dir installation_directory wait-for bootstrap-complete --log-level=info
```
![bootstrap 서버와 Kubernetes API 연결](/assets/img/post/Openshift/bootstrap%20서버와%20Kubernetes%20API%20연결.png)

* * *

## 8. Master 서버 설치하기 :

> 해당 과정은 마스터서버에서 진행합니다.
{: .prompt-info}

### 8.1 Master 서버 설치하기 :

- master 서버도 설치 방법은 bootstrap 서버와 동일하게 OS 생성 후, ignition 파일의 해시값을 변수에 저장하고 이를 이용하여 OS 설치를 진행합니다.

```bash
$ hash=`curl http://192.168.1.164/master.hash`
$ sudo coreos-installer install --copy-network --ignition-url http://192.168.1.164/master.ign /dev/sda --ignition-hash sha512-${hash}
```

> 설치 완료 후 재부팅 진행해야합니다.
{: .prompt-warning}

* * *

- master 서버 설치를 완료한 것이므로 Kubernetes API 연결이 잘 되는지 아래 명령으로 확인해봅니다.

```bash
$ ./openshift-install --dir installation_directory wait-for bootstrap-complete --log-level=debug
```

![master 서버 설치 확인](/assets/img/post/Openshift/master%20서버%20설치%20확인.png)

* * *

### 8.2 Master Node 조회하기 :

> 해당 과정은 관리서버에서 진행합니다.
{: .prompt-warning}

- OCP 로그인시 자동으로 쿠버네티스 관리자 계정을 설정하게 조치하고 현재 세션에도 적용합니다.

```bash
$ echo "export KUBECONFIG=/root/installation_directory/auth/kubeconfig" >> ~/.bash_profile

$ export KUBECONFIG=/root/installation_directory/auth/kubeconfig
```

* * *

- Openshift CLI로 현재 사용자를 확인합니다.

```bash
$ oc whoami
```

![Openshift 사용자 확인](/assets/img/post/Openshift/Openshift%20사용자%20확인.png)

* * *

- 연결된 노드를 확인합니다.

```bash
$ oc get nodes
```

![ 마스터 노드 연결 확인](/assets/img/post/Openshift/마스터%20노드%20연결%20확인.png)

* * *