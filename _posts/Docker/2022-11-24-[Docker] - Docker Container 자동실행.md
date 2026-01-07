---
layout: post
title: "Docker Container 자동실행"
date: 2022-11-24
categories: [컨테이너, Docker]
tags: [Docker, Container]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## 1. Docker Container 자동실행 :
### 1.1 Container 확인하기 :
- Container ID를 확인합니다.

```bash
$ docker ps
```

![텍스트](/assets/img/post/docker/docker%20container%20%EB%AA%A9%EB%A1%9D%20%ED%99%95%EC%9D%B8.PNG)

* * *

### 1.2 Container 서비스 파일생성 :
- 서버에 등록할 서비스 파일을 생성합니다.

```bash
$ cd /etc/systemd/system
$ vi 서비스명.service
```
```html
# 아래 내용 추가
[Unit]
Wants=docker.service
After=docker.service

[Service]
RemainAfterExit=yes
ExecStart=/usr/bin/docker start 실행할 docker container 이름
ExecStop=/usr/bin/docker stop 실행할 docker container 이름

[Install]
WantedBy=multi-user.target
```

![텍스트](/assets/img/post/docker/docker%20container%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%ED%8C%8C%EC%9D%BC%20%EB%82%B4%EC%9A%A9.PNG)

- 생성한 서비스 파일을 등록합니다.

```bash
# 서비스 시작
$ systemctl start 서비스명
# 서비스 자동 실행 등록
$ systemctl enable 서비스명
```

![텍스트](/assets/img/post/docker/docker%20container%20%EC%9E%90%EB%8F%99%EC%8B%A4%ED%96%89%20%EB%93%B1%EB%A1%9D.PNG)

- 서비스 파일이 자동실행으로 등록되었는지 확인합니다.

```bash
$ systemctl list-unit-files | grep 서비스명
```
![텍스트](/assets/img/post/docker/docker%20container%20%EC%9E%90%EB%8F%99%EC%8B%A4%ED%96%89%20%ED%99%95%EC%9D%B8.PNG))

- 재부팅 후 서비스가 자동으로 실행되는지 확인합니다.

* * *