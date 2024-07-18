---
layout: post
title: "[Docker] - Docker Network Bridge 이해하기"
date: 2023-06-07
categories: Docker
tags: [Docker, Network]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## Docker Network 개요:
- Docker Network는 Docker container 간 Network 연결을 할 수 있으며 Docker container와 Docker container가 아닌 워크로드에 연결할 수 있다.
- Docker 호스트는 Docker를 이용해 플랫폼에 구애받지 않고 Docker container들을 관리, 연결할 수 있다.

* * *

## Docker network Driver:
- Docker는 여러 Network driver들은 기본으로 제공되며, Network driver는 아래와 같다. 
```html
Bridge, Host, Overlay, Macvlan, None, Network plugins
```

* * *

## Docker network Driver Bridge:
- Docker를 설치하게 되면 자동으로 Host machine의 Network interface에 Docker0라는 Virtual interface가 생성된다.
```html
[Docker0의 특징]
• Gateway는 자동으로 172.17.0.1로 설정 되며 16 bit netmask(255.255.0.0)로 설정된다.
• 이 ip는 DHCP를 통해 할당 받는 것은 아니며, docker 내부 로직에 의해 자동 할당 받는 것이다.
• docker0 는 일반적인 interface가 아니며, virtual ethernet bridge 이다.
```
[![docker network 흐름도](/assets/img/post/docker/docker%20network%20%ED%9D%90%EB%A6%84%EB%8F%84.PNG)](/assets/img/post/docker/docker%20network%20%ED%9D%90%EB%A6%84%EB%8F%84.PNG)

* * *

### Docker network Driver Bridge 확인하기:
- Host machine의 Docker network를 확인하기 위해 다음과 같은 명령어를 입력한다.
```bash
$ docker network ls
```
[![docker network 명령어](/assets/img/post/docker/docker%20network%20%ED%99%95%EC%9D%B8%20%EB%AA%85%EB%A0%B9%EC%96%B4.PNG)](/assets/img/post/docker/docker%20network%20%ED%99%95%EC%9D%B8%20%EB%AA%85%EB%A0%B9%EC%96%B4.PNG)

- Default로 생성된 bridge에 대한 정보를 확인하기 위해 다음과 같은 명령어를 입력한다. 
```bash
$ docker network inspect bridge
```
[![docker network bridge 기본값](/assets/img/post/docker/docker%20network%20bridge%20%EA%B8%B0%EB%B3%B8%EA%B0%92.PNG)](/assets/img/post/docker/docker%20network%20bridge%20%EA%B8%B0%EB%B3%B8%EA%B0%92.PNG)

* * *