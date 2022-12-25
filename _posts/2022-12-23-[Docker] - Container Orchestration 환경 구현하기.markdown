---
title: "[Docker] - Container Orchestration 환경 구현하기"
layout: post
date: 2022-12-23
image: /assets/images/Post/docker.png
headerImage: true
tag:
- Docker
- YAML
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Container Orchestration 환경 구현하기:
- docker-compose.yml 파일을 통해서 Docker Container에 관한 실행 옵션을 기재하여 Container Service를 실행시킨다.

[![텍스트](/assets/images/docker/docker%20Container%20Orchestration%20%ED%99%98%EA%B2%BD%20%EA%B5%AC%ED%98%84%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/docker/docker%20Container%20Orchestration%20%ED%99%98%EA%B2%BD%20%EA%B5%AC%ED%98%84%20%ED%99%94%EB%A9%B4.PNG)

* * *

### 시스템 환경:
- OS : CentOS Linux release 7.9.2009 (Core)
- Docker : 20.10.9

위와 같은 환경으로 docker Container Orchestration 환경을 구성한다.

* * *

## 설치 구성하기:
- 먼저 docker 설치는 아래 게시글 통해서 설치한다.
> * [Docker 설치방법](https://hwangyoonjae.github.io/Docker-Docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/ "Docker 설치방법")


- docker 스웜 구성은 아래 게시글 통해서 진행한다.
> * [Docker 스웜 구성하기](https://hwangyoonjae.github.io/Docker-Docker-Swarm-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0/ "Docker 스웜 구성하기")

* * *