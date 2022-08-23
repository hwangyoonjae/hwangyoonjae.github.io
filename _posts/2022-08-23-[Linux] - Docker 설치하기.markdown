---
title: "[Linux] - Docker 설치하기"
layout: post
date: 2022-08-23
image: /assets/images/Post/docker.png
headerImage: true
tag:
- Docker
- Kubernetes
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Docker란 무엇인가?:
- 컨테이너 기술을 활용하는 앱의 개발, 제공 및 실행을 위해 설계된 소프트웨어 컨테이너 플랫폼이다.

<span style="color:#FA5858; font-size:12px">※ 인터페이스란? : 서로 다른 두 개의 시스템, 장치 사이에서 정보나 신호를 주고받는 경우의 접점이나 경계면이다.</span>

* * *

## Docker 설치하기 :

### 레퍼지토리 등록 :
```bash
# 어플리케이션 설치를 위한 사전작업
$ yum install -y yum-utils
$ yum-config-manager \ --add-repo \ https://download.docker.com/linux/centos/docker-ce.repo
```

- docker 설치 진행
```bash
# yum으로 docker 설치 :
$ yum install docker-ce docker-ce-cli containerd.io
```

<span style="color:#FA5858; font-size:12px">※ 필자는 20.10.17버전을 사용했다.</span>

* * *

### docker 실행 :
```bash
$ systemctl start docker
$ systemctl enable docker
```

<span style="color:#FA5858; font-size:12px">※ systemctl enable 명령어를 사용하는 이유 : 서버 부팅 시 자동으로 서비스 구동하기 위해 사용한다.</span>

* * *

### docker 구동확인 :
- 아래 명령어로 구동이 되었는지 확인한다.
```bash
$ docker version
$ docker ps
```

[![텍스트](/assets/images/Linux/docker%20%EA%B5%AC%EB%8F%99%ED%99%95%EC%9D%B8.PNG)](/assets/images/Linux/docker%20%EA%B5%AC%EB%8F%99%ED%99%95%EC%9D%B8.PNG)

* * *

### 일반 유저 docker 실행 권한 부여 :
- 필자는 root 유저로 로그인하여 설치부터 실행까지 수행하였으나, 일반 유저의 경우 install 할때 sudo를 쓰거나 su 명령어로 root 권한으로 변경해야한다. 
```bash
$ usermod -a -G docker 유저계정
```

* * *