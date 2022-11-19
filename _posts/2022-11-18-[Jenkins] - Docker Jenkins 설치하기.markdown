---
title: "[Jenkins] - Docker Jenkins 설치하기"
layout: post
date: 2022-11-18
image: /assets/images/Post/jenkins.png
headerImage: true
tag:
- Jenkins
- Java
- CI
- Docker
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Docker에 Jenkins 설치하기:
- Jenkins 설치는 해봤으니 Docker 통해서 Jenkins를 설치해본다.

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
$ rpm -Uvh docker-ce-cli-20.10.9-3.el7.x86_64.rpm --nodeps
$ rpm -Uvh docker-ce-20.10.9-3.el7.x86_64.rpm --nodeps
```

* * *

### Docker 실행:
```bash
$ systemctl start docker
$ systemctl enable docker
```

## Docker Jenkins 이미지 다운로드:
- Jenkins 관련 이미지를 검색한다.
```bash
$ docker search jenkins
```
[![텍스트](/assets/images/Linux/docker%20image%20%EA%B2%80%EC%83%89.PNG)](/assets/images/Linux/docker%20image%20%EA%B2%80%EC%83%89.PNG)

- Jenkins 관련 이미지를 확인하여 다운받는다.
```bash
$ docker pull jenkins/jenkins
```
[![텍스트](/assets/images/Linux/docker%20jenkins%20%EC%9D%B4%EB%AF%B8%EC%A7%80%20%EC%84%A4%EC%B9%98.PNG)](/assets/images/Linux/docker%20jenkins%20%EC%9D%B4%EB%AF%B8%EC%A7%80%20%EC%84%A4%EC%B9%98.PNG)

- Jenkins 이미지 설치 후 확인한다.
```bash
$ docker images
```
[![텍스트](/assets/images/Linux/docker%20jenkins%20%EC%9D%B4%EB%AF%B8%EC%A7%80%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/Linux/docker%20jenkins%20%EC%9D%B4%EB%AF%B8%EC%A7%80%20%ED%99%95%EC%9D%B8.PNG)

* * *