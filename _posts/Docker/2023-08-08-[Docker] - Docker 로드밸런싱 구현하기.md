---
layout: post
title: "[Docker] - Docker 로드밸런싱 구현하기"
date: 2023-08-08
categories: Docker
tags: [Docker, Load Balancing]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## Docker 로드밸런싱 구현하기:
- Docker로 구성하여 Tomcat을 로드밸런싱으로 구현해보려한다.
[![docker 로드밸런싱 구현하기](/assets/img/post/docker/docker%20로드밸런싱%20구현하기.PNG)](/assets/img/post/docker/docker%20로드밸런싱%20구현하기.PNG)

* * *

## Docker 로드밸런싱 구현을 위한 준비:
- Docker Swarm 로드밸런싱 구현을 위해서 몇가지 준비사항이 있다.

> * [Docker 설치하기](https://hwangyoonjae.github.io/docker/Docker-Docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/ "Docker 설치하기")

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
      - 10001:8080
    volumes:
      - ./tomcat1/webapps/:/usr/local/tomcat/webapps/
    environment:
      TZ: "Asia/Seoul"
  # Tomcat2
  tomcat1:
    image: tomcat
    container_name: tomcat2
    restart: always
    ports:
      - 10002:8080
    volumes:
      - ./tomcat2/webapps/:/usr/local/tomcat/webapps/
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
      - ./mariadb/mysql-init-files/:/docker-entrypoint-initdb.d/
    environment:
      MARIADB_DATABASE: dockerdb
      MARIADB_USER: test
      MARIADB_PASSWORD: 12345
      MARIADB_ROOT_PASSWORD: 12345
      TZ: "Asia/Seoul"
```

* * *

## Docker Visualizer 통해 컨테이너 구성확인하기:
- docker-compose 파일로 구성한 컨테이너를  Visualizer 통해 확인한다.
[![docker 로드밸런싱 구성 visualizer로 확인](/assets/img/post/docker/docker%20로드밸런싱%20구성%20visualizer로%20확인.PNG)](/assets/img/post/docker/docker%20로드밸런싱%20구성%20visualizer로%20확인.PNG)

* * *

## Docker 로드밸런싱 확인하기:
- 웹페이지 접속하여 새로고침을 계속하면 ***tomcat1***, ***tomcat2***가 번갈아 가면서 보인다.
[![docker 로드밸런싱 tomcat1,2 화면](/assets/img/post/docker//docker%20로드밸런싱%20tomcat1,2%20화면.PNG)](/assets/img/post/docker//docker%20로드밸런싱%20tomcat1,2%20화면.PNG)

* * *