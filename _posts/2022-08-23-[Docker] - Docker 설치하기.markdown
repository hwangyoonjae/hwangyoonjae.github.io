---
title: "[Docker] - Docker 설치하기"
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
- 컨테이너 기반의 오픈소스 가상화 플랫폼이다.

<span style="color:#FA5858; font-size:12px">※ 컨테이너란? : 격리된 공간에서 프로세스가 동작하는 기술이다.</span>

* * *

### 가상화 방식과 컨테이너 기술의 차이:
- 기존의 가상화 방식은 OS 가상화 방식이며, 예를들어 VMWare나 VirtualBox와 같다.간단하지만 너무 무겁고 느리기 때문에 운영에 쓸 수 없는 문제점을 해결하고자 CPU 가상화 기술(HVM)을 이용한 Kernel-based Virtual Machine과 반가상화 기술이 등장하였다.
이러한 기술들은 OpenStack이나 AWS와 같은 클라우드 서비스의 기반 기술이 되었다.

[![텍스트](/assets/images/Linux/%EA%B0%80%EC%83%81%ED%99%94vs%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%20%EC%B0%A8%EC%9D%B4.PNG)](/assets/images/Linux/%EA%B0%80%EC%83%81%ED%99%94vs%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%20%EC%B0%A8%EC%9D%B4.PNG)

- 리눅스에서는 이 방식을 리눅스 컨테이너라 하고, 단순히 프로세스를 격리시키기 때문에 가볍고 빠르게 동작한다.
- 하나의 서버에 여러개의 컨테이너를 실행하면 서로 영향을 미치지 않고 독립적으로 실행된다.

* * *

### Docker 장점:
- 도커는 컨테이너를 가벼운 모듈식(경량화) 가상 머신처럼 사용할 수 있다는 특징이 있고, 컨테이너를 구축하고 배포하는 것을 비교적 쉽게 할 수 있기 때문에 어플리케이션을 클라우드에 최적화할 수 있도록 지원한다.

- 여러 프로세스와 어플리케이션을 따로따로 실행해서 인프라를 더욱 효율적으로 활용하고 기존처럼 한 환경에서 어플리케이션을 실행할 때와 동일한 보안을 유지할 수 있다.

* * *

## Docker 구조:
- Docker는 **클라이언트와 서버 구조**로 되어있고, 클라이언트인 CLI(Docker)를 통해 각 커맨드로 데몬 프로세스와 통신하며, 데몬 프로세스가 Docker 레지스트리에 저장되어 있는 이미지들을 불러와 컨테이너를 실행하거나 커스터마이징한 이미지를 Docker 레지스트리에 다시 업로드하는 흐름을 가진다.
[![텍스트](/assets/images/Linux/docker%20%EA%B5%AC%EC%A1%B0.PNG)](/assets/images/Linux/docker%20%EA%B5%AC%EC%A1%B0.PNG)

- **Docker 데몬** : 다른 Docker 데몬과 통신하거나 Docker API 요청을 기다리고 이미지, 컨테이너, 네트워크, 볼륨 등을 관리하는 역할을 한다. 
- **Docker 클라이언트** : docker 커맨드를 통해 한 개 이상의 데몬과 통신할 수 있으며 사용자가 Docker와 상호작용할 수 있는 가장 우선적인 방법이다.
- **Docker 레지스트리** : Docker 이미지 저장소. 기본적으로 Docker Hub라는 퍼블릭 레지스트리로 설정되어 있고, 프라이빗 레지스트리도 생성할 수 있다.

* * *

## Docker 설치하기:
### 레퍼지토리 등록:
```bash
# 어플리케이션 설치를 위한 사전작업
$ yum install -y yum-utils
$ yum-config-manager \
     --add-repo \
     https://download.docker.com/linux/centos/docker-ce.repo
```
```bash
# yum으로 docker 설치
$ yum install docker-ce docker-ce-cli containerd.io
```
<span style="color:#FA5858; font-size:12px">※ 필자는 20.10.17버전을 사용했다.</span>

### 폐쇄망 설치:
- 폐쇄망 서버에 Docker를 설치하는 경우 아래 URL 접속 시 CentOS 버전에 맞춰 폴더 접속한 후 **/x86_64/stable/Packages/**에서  필요한 RPM 다운받는다.
> * [Docker RPM 패키지 다운로드](https://download.docker.com/linux/centos/ "Docker RPM 패키지 다운로드")
```bash
# rpm으로 설치
$ rpm -Uvh containerd.io-1.6.9-3.1.el7.x86_64.rpm --nodeps
$ rpm -Uvh container-selinux-17.03.3.ce-1.el7.noarch.rpm --nodeps
$ rpm -Uvh docker-ce-cli-20.10.9-3.el7.x86_64.rpm --nodeps
$ rpm -Uvh docker-ce-20.10.9-3.el7.x86_64.rpm --nodeps
```

* * *

### docker 실행:
```bash
$ systemctl start docker
$ systemctl enable docker
```

<span style="color:#FA5858; font-size:12px">※ systemctl enable 명령어를 사용하는 이유 : 서버 부팅 시 자동으로 서비스 구동하기 위해 사용한다.</span>

* * *

### docker 구동확인:
- 아래 명령어로 구동이 되었는지 확인한다.
```bash
$ docker version
$ docker ps
```

[![텍스트](/assets/images/Linux/docker%20%EA%B5%AC%EB%8F%99%ED%99%95%EC%9D%B8.PNG)](/assets/images/Linux/docker%20%EA%B5%AC%EB%8F%99%ED%99%95%EC%9D%B8.PNG)

* * *

### 일반 유저 docker 실행 권한 부여:
- 필자는 root 유저로 로그인하여 설치부터 실행까지 수행하였으나, 일반 유저의 경우 install 할때 sudo를 쓰거나 su 명령어로 root 권한으로 변경해야한다. 
```bash
# -a : --append 사용자를 서브 그룹에 추가한다.
# -G : --groups 사용자를 추가할 그룹을 지정한다.
$ usermod -a -G docker 유저계정
```

* * *