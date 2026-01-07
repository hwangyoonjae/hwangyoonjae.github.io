---
layout: post
title: "Docker Gitlab 설치하기"
date: 2023-10-25
categories: [DevOps, Gitlab]
tags: [Git, Gitlab, Docker]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## Docker에 Gitlab 설치하기:
- Gitlab 설치는 해봤으니 Docker 통해서 Gitlab 설치해본다.

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
<span style="color:#FA5858; font-size:12px">※ 필자는 20.10.17버전을 사용했습니다.</span>

* * *

### 폐쇄망 설치:
- 폐쇄망 서버에 Docker를 설치하는 경우 아래 URL 접속 시 CentOS 버전에 맞춰 폴더 접속한 후 **/x86_64/stable/Packages/**에서  필요한 RPM 다운받습니다.
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

* * *

### Docker Gitlab 이미지 다운로드:
- Gitlab 관련 이미지를 검색합니다.
```bash
$ docker search gitlab
```
[![gitlab docker image 목록](/assets/img/post/Gitlab/gitlab%20docker%20image%20목록.png)](/assets/img/post/Gitlab/gitlab%20docker%20image%20목록.png)

- Gitlab 관련 이미지를 확인하여 다운받습니다.
```bash
$ docker pull gitlab/gitlab-ee
```
[![gitlab docker image 다운](/assets/img/post/Gitlab/gitlab%20docker%20image%20다운.png)](/assets/img/post/Gitlab/gitlab%20docker%20image%20다운.png)

- Gitlab 이미지 설치 후 확인합니다.
```bash
$ docker images
```
[![gitlab docker image 설치 확인](/assets/img/post/Gitlab/gitlab%20docker%20image%20설치%20확인.png)](/assets/img/post/Gitlab/gitlab%20docker%20image%20설치%20확인.png)

* * *

## Docker로 Gitlab 실행하기:
```bash
$ docker run --detach \
>   --hostname gitlab.joeunins.com \
>   --publish 9001:80 --publish 9002:443 --publish 9003:22 \
>   --name gitlab --restart always \
>   --volume $GITLAB_HOME/config:/etc/gitlab \
>   --volume $GITLAB_HOME/logs:/var/log/gitlab \
>   --volume $GITLAB_HOME/data:/var/opt/gitlab \
>   --env GITLAB_ROOT_NAME=new_root_username \
>   --env GITLAB_ROOT_PASSWORD=new_root_password \
>   gitlab/gitlab-ee:latest
```
[![gitlab docker 서비스 실행](/assets/img/post/Gitlab/gitlab%20docker%20서비스%20실행.png)](/assets/img/post/Gitlab/gitlab%20docker%20서비스%20실행.png)

* * *

### Docker로 Gitlab 실행 로그 확인하기:
- 아래 명령어를 통해서 Docker로 Gitlab 실행 로그를 확인할 수 있습니다.
```bash
$ docekr logs -f gitlab
```

* * *