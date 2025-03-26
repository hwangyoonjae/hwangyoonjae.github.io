---
layout: post
title: "[Docker] - Docker Stack이란"
date: 2022-09-01
categories: Docker Concept
tags: [Docker, YAML]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## 도커 스택(Docker Stack)이란?:
- 여러 개의 서비스로 구성된 어플리케이션을 관리하기 위한 도구이다.
- Swarm mode에서만 사용 가능하다.

* * *

### 도커 스택(Docker Stack)의 구조:
- Docker Compose와 비슷하지만 지원하는 옵션이나 내부 로직에서 차이가 있다.
- 기본 네트워크가 브릿지 네트워크로 생성되지만 **Docker Stack**은 Overlay 네트워크가 생성된다.

* * *

## 도커 스택(Docker Stack) 사용하기:
- 아래와 같이 docker-compose.yaml 파일로 서비스를 정의한다.
[![텍스트](/assets/img/post/docker/docker%20compose%20%ED%8C%8C%EC%9D%BC%20%EC%9E%85%EB%A0%A5.PNG)](/assets/img/post/docker/docker%20compose%20%ED%8C%8C%EC%9D%BC%20%EC%9E%85%EB%A0%A5.PNG)

- **docker stack deploy** 명령어로 서비스를 배포한다.
```bash
$ docker stack deploy -c <yaml-file> <stack-name>
```
[![텍스트](/assets/img/post/docker/docker%20stack%20deploy%EB%A1%9C%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%EB%B0%B0%ED%8F%AC%20%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/docker/docker%20stack%20deploy%EB%A1%9C%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%EB%B0%B0%ED%8F%AC%20%ED%99%94%EB%A9%B4.PNG)
- **-C** 옵션으로 배포할 서비스가 정의된 YAML 파일을 지정할 수 있다.

* * *

## 도커 서비스(Docker Service) 확인하기:
```bash
$ docker service ls
```
[![텍스트](/assets/img/post/docker/docker%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/docker/docker%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%ED%99%95%EC%9D%B8.PNG)

* * *