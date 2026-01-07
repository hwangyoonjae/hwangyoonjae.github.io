---
layout: post
title: "Docker Jenkins 설치하기"
date: 2022-11-18
categories: [DevOps, Jenkins]
tags: [Jenkins, Java, CI, Docker]
image: /assets/img/post-title/jenkins-wallpaper.jpg
---

## 1. Docker에 Jenkins 설치하기 :
- Jenkins 설치는 해봤으니 Docker 통해서 Jenkins를 설치해본다.

* * *

## 2. Docker 설치하기 :
### 2.1 레퍼지토리 등록하기:
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
> 필자는 20.10.17버전을 사용했습니다.
{: .promt-info}

* * *

### 2.2 폐쇄망 설치하기:
- 폐쇄망 서버에 Docker를 설치하는 경우 아래 URL 접속 시 CentOS 버전에 맞춰 폴더 접속한 후 **/x86_64/stable/Packages/**에서  필요한 RPM 다운받습니다.
> * [Docker RPM 패키지 다운로드](https://download.docker.com/linux/centos/ "Docker RPM 패키지 다운로드")

```bash
# rpm으로 설치
$ rpm -Uvh containerd.io-1.6.9-3.1.el7.x86_64.rpm --nodeps
$ rpm -Uvh docker-ce-cli-20.10.9-3.el7.x86_64.rpm --nodeps
$ rpm -Uvh docker-ce-20.10.9-3.el7.x86_64.rpm --nodeps
```

* * *

### 2.3 Docker 실행하기 :
```bash
$ systemctl start docker
$ systemctl enable docker
```

* * *

### 2.4 Docker Jenkins 이미지 다운로드 :
- Jenkins 관련 이미지를 검색합니다.

```bash
$ docker search jenkins
```
![텍스트](/assets/img/post/docker/docker%20image%20%EA%B2%80%EC%83%89.PNG)

* * *

- Jenkins 관련 이미지를 확인하여 다운받습니다.

```bash
$ docker pull jenkins/jenkins
```
![텍스트](/assets/img/post/docker/docker%20jenkins%20%EC%9D%B4%EB%AF%B8%EC%A7%80%20%EC%84%A4%EC%B9%98.PNG)

- Jenkins 이미지 설치 후 확인합니다.

```bash
$ docker images
```
![텍스트](/assets/img/post/docker/docker%20jenkins%20%EC%9D%B4%EB%AF%B8%EC%A7%80%20%ED%99%95%EC%9D%B8.PNG)

* * *

## 3. Docker로 Jenkins 실행하기 :
```bash
docker run -d -p 9000:8080 -v /jenkins:/var/jenkins_home --name jenkins_server -u root jenkins/jenkins:lts
```
- **/jenkins:/var/jenkins_home**부분을 고려하여 container 내부의 디렉토리를 매핑합니다.

![텍스트](/assets/img/post/docker/docker%20jenkins%20%EC%8B%A4%ED%96%89%ED%99%94%EB%A9%B4.PNG)

- 로그 중간의 16진수로 이루어진 문자열은 별도로 저장하여 Jenkins 실행 시 관리자 패스워드 입력하는 부분에 붙혀넣기합니다.

* * *

### 3.1 Jenkins 비밀번호 입력하기 :
- Jenkins 실행 시 아래와 같이 실행됩니다.

![텍스트](/assets/img/post/docker/docker%20jenkins%20%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8.PNG

* * *

- **docker logs**명령어 통해서 jenkins 관리자 비밀번호를 찾는다.

```bash
$ docker logs [컨테이너ID]
```
![텍스트](/assets/img/post/docker/docker%20jenkins%20%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8%20%EC%B0%BE%EA%B8%B0.PNG)

* * *

- 비밀번호 입력 후, Install suggested plugins를 클릭해 기초 플러그인을 설치합니다.

![텍스트](/assets/img/post/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%ED%94%8C%EB%9F%AC%EA%B7%B8%EC%9D%B8%20%EC%84%A4%EC%B9%98.PNG)

- 계정 생성 후 메인 페이지 접속을 확인합니다.

![텍스트](/assets/img/post/docker/docker%20jenkins%20%EB%A9%94%EC%9D%B8%ED%99%94%EB%A9%B4.PNG)

* * *