---
layout: post
title: "Harbor 구축하기"
date: 2024-02-05
categories: [DevOps, Harbor]
tags: [Harbor, Dockerhub, Image]
image: /assets/img/post-title/harbor-wallpaper.jpg
---

## 1. Harbor란? :
- Docker 이미지를 저장, 관리 및 배포하기 위한 오픈 소스 컨테이너 레지스트리입니다.

* * *

### 1.1 Harbor를 사용하는 이유? :
- 회사에서 쿠버네티스 Pod 구성을 위해 컨테이너 이미지를 DockerHub에서 이미지 Pull 진행하였으나, ***Too Many Requests. You have reached your pull rate limit.*** 오류가 발생하여 private docker registry 운영하기 위해 Harbor를 사용하게 되었습니다.

* * *

## 2. Harbor 사전설치하기 :
### 2.1 인증서 폴더 생성하기 :
```bash
$ mkdir -p ~/certs
$ cd ~/certs
```

* * *

### 2.2 CA Certificates(비밀키) 생성하기 :
```bash
$ openssl genrsa -out ca.key 4096
```

* * *

### 2.3 Root CA의 비밀키와 짝을 이룰 공개키 생성하기 :
```bash
$ openssl req -x509 -new -nodes -sha512 -days 365 \
-subj "/C=CN/ST=seoul/L=seoul/O=kyh0703/OU=tester/CN=[도메인 또는 서버주소]" \
-key ca.key \
-out ca.crt
```

* * *

### 2.4 Server Certificates(비밀키) 생성하기 :
```bash
$ openssl genrsa -out yourdomain.com.key 4096
```

* * *

### 2.5 Server의 CSR 파일 생성하기 :
```bash
# * CN은 도메인이나 아이피 입력
$ openssl req -sha512 -new \
-subj "/C=CN/ST=seoul/L=seoul/O=kyh0703/OU=tester/CN=[도메인 또는 서버주소]" \
-key [도메인 또는 서버주소].key \
-out [도메인 또는 서버주소].csr
```

* * *

### 2.6 인증하기 :
```bash
$ cat > v3ext.cnf <<-EOF
subjectAltName = IP:[도메인 또는 서버주소],IP:127.0.0.1
EOF
```

* * *

### 2.7 인증키 생성하기 :
```bash
$ openssl x509 -req -sha512 -days 3650 \
-extfile v3.ext \
-CA ca.crt -CAkey ca.key -CAcreateserial \
-in [도메인 또는 서버주소].csr \
-out [도메인 또는 서버주소].crt
```

* * *

### 2.8 인증서 복사하기 :
```bash
$ mkdir -p /data/cert
$ cp [도메인 또는 서버주소].crt /data/cert/
$ cp [도메인 또는 서버주소].key /data/cert/
```

* * *

### 2.9 Cert 파일 생성하기 :
```bash
$ openssl x509 -inform PEM -in [도메인 또는 서버주소].crt -out [도메인 또는 서버주소].cert
```

* * *

### 2.10 Docker 인증서 복사하기 :
```bash
$ mkdir -p /etc/docker/certs.d/[도메인 또는 서버주소]
$ cp [도메인 또는 서버주소].cert /etc/docker/certs.d/[도메인 또는 서버주소]/
$ cp [도메인 또는 서버주소].key /etc/docker/certs.d/[도메인 또는 서버주소]/
$ cp ca.crt /etc/docker/certs.d/[도메인 또는 서버주소]/
```

* * *

### 2.11 Docker 재시작 :
```bash
$ systemctl restart docker
```

* * *

## 3. Harbor 설치하기 :
### 3.1 Harbor 파일 다운로드 :
```bash
$ wget https://github.com/goharbor/harbor/releases/download/v2.3.1/harbor-offline-installer-v2.3.1.tgz
$ tar -zxvf harbor-offline-installer-v2.3.1.tgz
```

* * *

### 3.2 yaml 파일 복사하기 :
```bash
$ cp harbor.yml.tmpl harbor.yml
```

* * *

### 3.3 파일 수정하기 :
```bash
$ vi harbor.yml
# 아래내용 변경
hostname: [도메인 또는 서버주소]

certificate: /etc/docker/certs.d/[도메인 또는 서버주소]/[도메인 또는 서버주소].cert
private_key: /etc/docker/certs.d/[도메인 또는 서버주소]/[도메인 또는 서버주소].key
```

* * *

### 3.4 Harbor 설치하기 :
```bash
$ ./prepare
$ ./install.sh
```

* * *

### 3.5 linux 신뢰할 수  있는 인증서 적용하기 :
```bash
$ cp [도메인 또는 서버주소].crt /etc/pki/ca-trust/source/anchors/harbor-server.crt
$ cp ca.crt /etc/pki/ca-trust/source/anchors/harbor-ca.crt
$ update-ca-trust
```

* * *

## 4. Harbor 접속하기 :
- 위 설정 완료 후 ***https://서버주소:포트***로 입력하여 접속합니다.

![Harbor 초기화면](/assets/img/post/docker/Harbor%20초기화면.png)

```html
⚠️ ca.crt 인증서를 로컬(데스크탑)의 신뢰할 수 있는 루트 인증기관에 등록 시 신뢰할 수 있는 인증서로 보여집니다.
```

* * *