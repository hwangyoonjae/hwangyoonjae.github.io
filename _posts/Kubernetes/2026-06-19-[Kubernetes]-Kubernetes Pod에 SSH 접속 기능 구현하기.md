---
layout: post
title: "Kubernetes Pod에 SSH 접속 기능 구현하기"
date: 2026-06-19
categories: [Kubernetes]
tags: [Kubernetes, SSH]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. 기능 구현하기 :

> 기본적으로 Kubernetes는 kubectl exec 명령을 통해 Pod 내부에 접근하는 방식을 사용합니다.
> 
> 하지만 기존 VM 운영 환경과 동일하게 SSH를 통해 Pod에 직접 접속해야 하는 요구사항이 발생할 수 있습니다.
> 
> 고객사 요청에 의해 컨테이너에 OpenSSH Server를 추가하여 SSH 접속 기능 테스트를 진행해봤습니다.
{: .prompt-info}

* * *

### 1.1 Dockerfile 생성하기 :

- SSH 서비스를 제공하기 위해 OpenSSH Server를 설치하고 관련 설정을 추가합니다.

```bash
# Dockerfile
FROM nginx:1.29

RUN apt-get update && \
    apt-get install -y openssh-server && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir -p /run/sshd

RUN ssh-keygen -A

RUN echo 'root:Passw0rd!' | chpasswd

RUN sed -i 's/#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/^UsePAM yes/UsePAM no/' /etc/ssh/sshd_config

RUN usermod -s /bin/sh root

EXPOSE 80 22

CMD ["/bin/bash", "-c", "/usr/sbin/sshd && nginx -g 'daemon off;'"]

```

* * *

### 1.2 컨테이너 이미지 빌드 및 배포하기 :

- Dockerfile 작성 후 컨테이너 이미지를 빌드합니다.

```bash
$ docker build -t harbor.example.com/library/nginx-ssh:1.0 .
```

* * *

- Harbor에 업로드합니다.

```bash
$ docker push harbor.example.com/library/nginx-ssh:1.0
```

* * *

## 2. Kubernetes 리소스 생성하기 :
### 2.1 Deployment 생성하기 :

- SSH 기능을 사용하기 위해 Container Capability를 추가합니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    apps: apps
  name: pod-ssh-test
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pod-ssh-test
  template:
    metadata:
      labels:
        app: pod-ssh-test
    spec:
      containers:
        - image: harbor.test.com/library/nginx-ssh:1.1
          name: container
          ports:
            - containerPort: 80 # Nginx 웹 서비스가 사용하는 HTTP 포트
              protocol: TCP
            - containerPort: 22 # OpenSSH Server(sshd)가 사용하는 SSH 포트
              protocol: TCP
          # SecurityContext 설정
          securityContext:
            capabilities:
              add:
                - SYS_CHROOT # SSH 인증 과정에서 OpenSSH가 사용하는 Sandbox(chroot) 기능 수행 권한
                - AUDIT_WRITE # SSH 세션 생성 과정에서 Linux Audit 로그를 기록하기 위한 권한
```

```bash
$ kubectl apply -f pod-ssh-deploy.yaml
```

> 파드 생성 시 권한을 추가해야하는 이유
>
> OpenSSH Server는 일반적인 웹 애플리케이션보다 추가적인 시스템 권한을 요구합니다.
>
> 따라서 Kubernetes 환경에서 SSH 서비스를 제공하기 위해서는 SYS_CHROOT, AUDIT_WRITE Capability를 추가하여 SSH 인증 및 세션 생성 과정이 정상적으로 수행될 수 있도록 구성해야 합니다.
{: .prompt-warning}

* * *

### 2.2 Service 생성하기 :

- 외부에서 SSH로 접속할 수 있도록 Pod의 22번 포트를 노드의 30022번 포트로 연결하는 Service를 생성합니다.

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-ssh-service
  namespace: default
spec:
  type: NodePort
  selector:
    app: pod-ssh-test
  ports:
    - name: ssh
      protocol: TCP
      port: 22
      targetPort: 22
      nodePort: 30022
```

```bash
$ kubectl apply -f pod-ssh-svc.yaml
```

* * *

## 3. SSH 접속 테스트하기 :

- 배포 완료 후 Worker Node IP와 NodePort를 이용하여 SSH 접속을 시도합니다.

```bash
# ssh <계정>@<워커노드 IP> -p <서비스 NodePort>
$ ssh root@192.168.1.100 -p 32222
```

![ssh 접속 화면](/assets/img/post/kubernetes/ssh%20접속%20화면.png)

* * *