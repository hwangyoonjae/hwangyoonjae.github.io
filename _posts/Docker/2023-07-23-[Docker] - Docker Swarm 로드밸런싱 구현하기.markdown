---
title: "[Docker] - Docker Swarm 로드밸런싱 구현하기"
categories:
  - Docker
tags:
  - [Docker, Load Balancing]

toc: true
toc_sticky: true

date: 2023-07-23
last_modified_at: 2023-07-23
---

## Docker Swarm 로드밸런싱 구현하기:
- 아래 그림과 같이 Docker container로 Swarm으로 결합하여 로드밸런싱을 구현해보려한다.

[![docker swarm 로드밸런싱 구현하기](/assets/images/docker/docker%20swarm%20로드밸런싱%20구현하기.PNG)](/assets/images/docker/docker%20swarm%20로드밸런싱%20구현하기.PNG)

* * *

## Docker Swarm 로드밸런싱 구현을 위한 준비:
- Docker Swarm 로드밸런싱 구현을 위해서 몇가지 준비사항이 있다.

> * [Docker 설치하기](https://hwangyoonjae.github.io/docker/Docker-Docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/ "Docker 설치하기")

> * [Docker Swarm 구성하기](https://hwangyoonjae.github.io/docker/Docker-Docker-Swarm-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0/ "Docker Swarm 구성하기")

* * *