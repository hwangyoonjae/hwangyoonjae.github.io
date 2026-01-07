---
layout: post
title: "Gitlab & Runner 설치하기"
date: 2024.06.18
categories: [DevOps, Gitlab]
tags: [Git, Gitlab, Runner]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## 인증서 생성하기:
- 인증 받을 서버 도메인을 환경 변수로 설정
```bash
export DOMAIN={Domain 주소}
```

- 인증 기관 인증서(CA 인증서 개인키) 생성
```bash
openssl genrsa -out ca.key 4096
## CA 인증서 생성
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=KR/ST=Seoul/CN=RootCA" \
 -key ca.key \
 -out ca.crt
# 서버 인증서 생성
## 개인키 생성
openssl genrsa -out $DOMAIN.key 4096
## 인증 서명 요청(CSR) 작성
openssl req -sha512 -new \
    -subj "/C=KR/ST=Seoul/L=Jung-gu/O=Innogrid/OU=IT/CN=$DOMAIN" \
    -key $DOMAIN.key \
    -out $DOMAIN.csr
```

- x509 v3 확장 파일 생성
```bash
cat > v3.ext <<-EOF
# 기본 제약 사항: 이 인증서는 CA가 아닌 서버용으로 사용됩니다.
basicConstraints=CA:FALSE
# 키 사용: 인증서의 공개 키가 어떻게 사용될 수 있는지를 제어합니다.
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
# 확장 키 사용: 이 인증서는 서버 인증용으로 사용될 수 있도록 합니다.
extendedKeyUsage = serverAuth
# 대체 이름: 인증서에 대한 대체 식별 정보를 제공합니다.
subjectAltName = @alt_names
[alt_names]
DNS.1=$DOMAIN
# DNS.2=example.com  # 추가 도메인을 사용하는 경우 주석 해제
# IP.1=127.0.0.1     # IP 주소를 사용하는 경우 주석 해제
EOF
```

- CA 인증서로 호스트에 대한 인증서 생성
```bash
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in $DOMAIN.csr \
    -out $DOMAIN.crt
```

* * *

## Gitlab 설치하기:
```yaml
# gitlab.yaml
version: "3.9"
services:
  secloudit-gitlab:
    image: harbor.com/gitlab-ce:15.11.3-ce.0
    container_name: secloudit-gitlab
    restart: always
    hostname: "gitlab.com"
    privileged: true
    cpus: "4.0"
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url "https://gitlab.com:8443" # 주소 지정
        gitlab_rails["initial_root_password"] = 'qwe1212!Q'
        gitlab_rails["gitlab_shell_ssh_port"] = 8022
        gitlab_rails["time_zone"] = "Asia/Seoul"
        nginx["ssl_certificate"] = "/etc/gitlab/certs/gitlab.com.crt"
        nginx["ssl_certificate_key"] = "/etc/gitlab/certs/gitlab.com.key"
        nginx["redirect_http_to_https"] = true
        registry["enable"] = false
    ports:
      - "8080:8080"
      - "8443:8443"
      - "8022:8022"
    volumes:
      - ./config:/etc/gitlab
      - ./logs:/var/log/gitlab
      - ./data:/var/opt/gitlab
      - ./certs:/etc/gitlab/certs
      - /etc/localtime/:/etc/localtime:ro
```

* * *

## Runner 설치하기:
```yaml
# gitlab-runner.yaml
version: '3.9'
services:
  gitlab-runner:
    image: harbor.com/gitlab-runner:v15.11.1 # 📌
    restart: unless-stopped
    container_name: gitlab-runner
    hostname: gitlab-runner
    privileged: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - $WORKDIR/volume/gitlab-runner:/etc/gitlab-runner
      - ./ssl:/etc/gitlab-runner/certs/
    ports:
      - "8093:8093"
```