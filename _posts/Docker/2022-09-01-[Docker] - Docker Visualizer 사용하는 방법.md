---
layout: post
title: "Docker Visualizer 사용하는 방법"
date: 2022-09-01
categories: [컨테이너, Docker] 
tags: [Docker, YAML, Visualizer]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## 1. Docker Visualizer이란? :
- Swarm mode에서 노드와 컨테이너의 분포상태를 시각적으로 볼 수 있게해주는 도구이다.

* * *

## 2. Docker Visualizer 사용하기 :
### 2.1 visualizer.yml 파일 만들기 :
- 아래와 같이 visualizer.yml 파일을 생성합니다.

![텍스트](/assets/img/post/docker/docker%20visualizer%20%ED%8C%8C%EC%9D%BC.PNG)

* * *

### 2.2 Visualizer 배포하기 :
- 아래와 같이 **docker stack** 명령어 사용하여 진행합니다.

```bash
$ docker stack deploy -c <yaml-file> <stack-name>
```

![텍스트](/assets//images/docker/docker%20visualizer%20%EC%83%9D%EC%84%B1.PNG)

* * *

### 2.3 Visualizer 서비스 확인하기 :
```bash
$ docker service ls
```
![텍스트](/assets/img/post/docker/docker%20visualizer%20%EB%AA%A9%EB%A1%9D%20%ED%99%95%EC%9D%B8.PNG)

* * *

## 3. 웹브라우저 접속하기 :
- localhost:9000 또는 서버IP:9000로 접속합니다.

![텍스트](/assets/img/post/docker/docker%20visualizer%20%EC%9B%B9%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%20%EC%A0%91%EC%86%8D%ED%99%94%EB%A9%B4.PNG)

* * *