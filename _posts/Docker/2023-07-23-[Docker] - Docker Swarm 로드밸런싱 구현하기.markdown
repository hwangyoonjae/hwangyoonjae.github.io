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
- 아래 그림과 같이 Docker container로 Swarm으로 결합하여 구현해보려한다.
[![docker swarm 로드밸런싱 구현하기](/assets/images/docker/docker%20swarm%20로드밸런싱%20구현하기.PNG)](/assets/images/docker/docker%20swarm%20로드밸런싱%20구현하기.PNG)

- Docker Mager에서는 로드밸런싱으로 구현해보려한다.
[![docker 로드밸런싱 구현하기](/assets/images/docker/docker%20로드밸런싱%20구현하기.PNG)](/assets/images/docker/docker%20로드밸런싱%20구현하기.PNG)

* * *

## Docker Swarm 로드밸런싱 구현을 위한 준비:
- Docker Swarm 로드밸런싱 구현을 위해서 몇가지 준비사항이 있다.

> * [Docker 설치하기](https://hwangyoonjae.github.io/docker/Docker-Docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/ "Docker 설치하기")

> * [Docker Swarm 구성하기](https://hwangyoonjae.github.io/docker/Docker-Docker-Swarm-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0/ "Docker Swarm 구성하기")

* * *

## Docker Compose를 이용하기:
- 도커 구성을 아래와 같이 파일 형태로 사용한다.
```bash
# 파일명은 docker-web.yml
version: "3.3"
# 서비스 구성
services:
  # Nginx
  nginx:
    image: nginx
    container_name: nginx
    restart: always
    ports:
      - 80:80
    volumes:
      - ./nginx/config/nginx.conf:/etc/nginx/nginx.conf
    environment:
      TZ: "Asia/Seoul"
  # Tomcat1
  tomcat1:
    image: tomcat
    container_name: tomcat1
    restart: always
    ports:
      - 8443:8080
    volumes:
      - ./tomcat1/webapps/:/usr/local/tomcat/webapps/
    environment:
      TZ: "Asia/Seoul"
  # Tomcat2
  tomcat1:
    image: tomcat
    container_name: tomcat1
    restart: always
    ports:
      - 8444:8080
    volumes:
      - ./tomcat1/webapps/:/usr/local/tomcat/webapps/
    environment:
      TZ: "Asia/Seoul"
  # mariadb
  mariadb:
    image: mariadb
    container_name: mariadb
    restart: always
    ports:
      - 3306:3306
    volumes:
      - ./mariadb/conf.d:/etc/mysql/conf.d
      - ./mariadb/data:/var/lib/mysql
    environment:
      MARIADB_DATABASE: dockerdb
      MARIADB_USER: test
      MARIADB_PASSWORD: 12345
      MARIADB_ROOT_PASSWORD: 12345
      TZ: "Asia/Seoul"
```

* * *