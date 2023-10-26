---
title: "[Docker] - Portainer 구축하기"
categories:
  - Docker
tags:
  - [Docker, Portainer]

toc: true
toc_sticky: true

date: 2023-10-26
last_modified_at: 2023-10-26
---

## Portainer란?:
- Docker Web 관리 Tool로 Docker와 관련된 컨테이너, 이미지, 볼륨, 네트워크 등을 web에서 관리할 수 있게 해준다.

* * *

## Portainer 구축하기:
### Docker Portainer 이미지 다운로드:
```bash
$ docker pull portainer/portainer-ce
```

### Docker Portainer 이미지 설치하기:
```bash
$ docker run --name portainer -p 9000:9000 -d --restart always -v /data/portainer:/data -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer-ce
```
[![portainer 이미지 설치](/assets/images/Portainer/portainer%20이미지%20설치.png)](/assets/images/Portainer/portainer%20이미지%20설치.png)

* * *

## Portainer 관리자 계정 설정하기:
- 설치 후 웹브라우저 Portainer 서버에 접속하여 괸리자 계정 패스워드를 설정한다.
[![potainer 계정 패스워드 설정](/assets/images//Portainer/potainer%20계정%20패스워드%20설정.png)](/assets/images//Portainer/potainer%20계정%20패스워드%20설정.png)

* * *

## Portainer 시작하기:
- 관리자 계정 생성 후 로그인하여 접속하면 Docker 환경을 볼 수 있다.
[![potainer 시작](/assets/images/Portainer/potainer%20시작.png)](/assets/images/Portainer/potainer%20시작.png)
[![potainer local 컨테이너 접속](/assets/images/Portainer/potainer%20local%20컨테이너%20접속.png)](/assets/images/Portainer/potainer%20local%20컨테이너%20접속.png)

* * *