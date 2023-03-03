---
title: "[Docker] - Docker 로그 확인하기"
categories:
  - Docker
tags:
  - [Docker, Logs]

toc: true
toc_sticky: true

date: 2023-01-09
last_modified_at: 2023-01-09
---

## Docker 관련 로그 확인하기:
### Docker Container 실행 로그 보기:
- 도커 컨테이너 실행에 대한 로그를 보고 싶으면 아래 명령어를 입력한다.
```bash
# -f를 추가하면 실시간으로 볼 수 있다.
$ docker log -f 컨테이너 이름
```
[![텍스트](/assets/images/docker/docker%20container%20%EB%A1%9C%EA%B7%B8%20%ED%99%95%EC%9D%B8%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/docker/docker%20container%20%EB%A1%9C%EA%B7%B8%20%ED%99%95%EC%9D%B8%20%ED%99%94%EB%A9%B4.PNG)

* * *