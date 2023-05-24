---
title: "[Docker] - Docker 이미지 레이어 구조"
categories:
  - Docker
tags:
  - [Docker, Overlay2]

toc: true
toc_sticky: true

date: 2023-05-23
last_modified_at: 2023-05-23
---

## Docker Layer란?:
- 도커 이미지를 받으면 마치 여러개로 분리된 조각처럼 보이는게 분리된 데이터를 말한다.

* * *

## Docker home 디렉토리:
- Docker home 디렉토리는 아래 명령어로 확인 가능하다.
```bash
$ docker info | grep "Docker Root Dir"
```
[![텍스트](/assets/images/docker/docker%20home%20%EC%9C%84%EC%B9%98%20%ED%99%95%EC%9D%B8%20%EB%AA%85%EB%A0%B9%EC%96%B4.PNG)](/assets/images/docker/docker%20home%20%EC%9C%84%EC%B9%98%20%ED%99%95%EC%9D%B8%20%EB%AA%85%EB%A0%B9%EC%96%B4.PNG)

- systemctl 등의 systemd를 관리하는 툴을 이용하면 loaded되는 service 설정에 대한 path를 가져올 수 있는데 여기서 home path 변경이 가능하다.
[![텍스트](/assets/images/docker/docker%20service%20%EB%A1%9C%EB%93%9C%EB%90%98%EB%8A%94%20%EA%B2%BD%EB%A1%9C.PNG)](/assets/images/docker/docker%20service%20%EB%A1%9C%EB%93%9C%EB%90%98%EB%8A%94%20%EA%B2%BD%EB%A1%9C.PNG)

* * *

## Docker overlay2란?:
- 레이어 파일 시스템이라고 하며, Linux 배포판에 대해서 선호되는 스토리지 드라이버라고 한다.

* * *