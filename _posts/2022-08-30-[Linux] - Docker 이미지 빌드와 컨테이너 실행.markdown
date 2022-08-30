---
title: "[Linux] - Docker 이미지 빌드와 컨테이너 실행"
layout: post
date: 2022-08-30
image: /assets/images/Post/docker.png
headerImage: true
tag:
- Docker
- Image
- Container
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Docker 이미지 빌드와 컨테이너 실행하기
### 도커 이미지 (Docker Image) 빌드하기:
- "이미지 = 설정파일" 즉, 가상화에 WebServer, WAS, DB 등 설치하여 구성해야하는데 도커에서 이미지는 다운받아서 하나의 컨테이너만 만들면 그 만든 컨테이너 하나로 여러 개의 도커를 생성할 수 있다.

* * *

### 도커 컨테이너 (Docker Container) 생성하기: 
- run 명령어를 사용하면 컨테이너 ID가 출력된다.
```bash
$ docker container run
또는
$ docker run
```

[![텍스트](/assets/images/Linux/docker%20container%20ID%20%EC%B6%9C%EB%A0%A5%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/Linux/docker%20container%20ID%20%EC%B6%9C%EB%A0%A5%20%ED%99%94%EB%A9%B4.PNG)

* * *