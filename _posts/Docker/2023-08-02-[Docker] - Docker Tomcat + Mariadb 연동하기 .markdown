---
title: "[Docker] - Docker Tomcat + Mariadb 연동하기"
categories:
  - Docker
tags:
  - [Docker, Tomat, Mariadb]

toc: true
toc_sticky: true

date: 2023-08-02
last_modified_at: 2023-08-02
---

## Docker Tomcat 구성하기:
- Docker Tomcat 구성을 아래와 같이 파일 형태로 사용한다.
```bash
# 파일명은 docker-tomcat.yml
version: "3.3"
# 서비스 구성
services:
  tomcat1:
    image: tomcat
    container_name: tomcat1
    restart: always
    ports:
      - 10001:8080
    volumes:
      - ./tomcat1/webapps/:/usr/local/tomcat/webapps
    environment:
      TZ: "Asia/Seoul"
```

* * *