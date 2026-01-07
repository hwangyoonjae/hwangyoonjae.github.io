---
layout: post
title: "Docker Container 모니터링"
date: 2022-12-04
categories: [컨테이너, Docker]
tags: [Docker, Container, 모니터링]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## 1. Docker Container 모니터링하기 :
- 아래 명령어 입력 시 컨테이너의 사용량을 확인 할 수 있습니다.

```bash
$ docker container stats 컨테이너 이름
```

![텍스트](/assets/img/post/docker/docker%20container%20%EB%A6%AC%EC%86%8C%EC%8A%A4%20%ED%99%95%EC%9D%B8.PNG)

- 각 부분별 설명은 아래와 같습니다.

```
‣ CONTAINER : 컨테이너 아이디
‣ NAME : 컨테이너 이름
‣ CPU% : CPU 사용률
‣ MEM USAGE / LIMIT : 메모리 사용량 / 컨테이너에서 사용 가능한 메모리
‣ MEM% : 메모리 사용량
‣ NET I/O : 네트워크 입출력
‣ BLOCK I/O : 차단 입출력
‣ PIDS : PID
```