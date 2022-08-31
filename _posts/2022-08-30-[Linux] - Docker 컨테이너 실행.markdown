---
title: "[Linux] - Docker 컨테이너 실행"
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

## Docker 컨테이너 실행하기
### 도커 컨테이너 (Docker Container) 생성하기: 
- run 명령어를 사용하면 컨테이너 ID가 출력된다.
```bash
$ docker container run [옵션] [이미지이름 or 이미지ID] [실행할 파일]
또는
$ docker run [옵션] [이미지이름 or 이미지ID] [실행할 파일]
```
[![텍스트](/assets/images/Linux/docker%20container%20ID%20%EC%B6%9C%EB%A0%A5%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/Linux/docker%20container%20ID%20%EC%B6%9C%EB%A0%A5%20%ED%99%94%EB%A9%B4.PNG)

#### 옵션
- **-d** 옵션
    - 실행 화면을 보지 않고 백그라운드로 컨테이너를 실행시킨다는 옵션

- **-p** 옵션
    - <호스트포트>:<컨테이너포트><br>
    <span style="color:#FA5858; font-size:12px">※ 호스트 포트는 중복되면 안된다.</span>

- **-it** 옵션
    - 컨테이너쪽 Shell에 들어가서 명령 실행할 수 있는 입력이 되도록 한다.

- **-t** 옵션
    - 터미널과 비슷한 환경을 조성해준다.

- **-v** 옵션
    - 볼륨 마운트 이용 시 사용한다.

- **--name** 옵션
    - 컨테이너의 이름을 지정한다.<br>
    <span style="color:#FA5858; font-size:12px">※ --name "이름" 옵션과 붙여서 사용한다.</span>

- **--rm** 옵션
    - 컨테이너를 stop 시키는 동시에 삭제까지 한번에 실행한다.

```bash
# 예시
$ docker container run -d -t -p 9000:8080 nginx --name nginx_server
```

* * *

### 도커 컨테이너 (Docker Container) 이름 바꾸기: 
- 컨테이너를 실행시킬 때 이름을 지정해서 만드는 경우가 많은데 이름을 지정하면 나중에 컨테이너에서 명령, 실행, 삭제, 중지 등을 실행할 때 지정해준 이름을 사용할 수 있다.

- docker 컨테이너 이름을 바꾸기 위해서는 docker rename 명령어를 사용하면 된다. 구체적으로는 docker rename [이전 container  이름] [새 container 이름] 형태로 실행한다.
```bash
$ docker rename <old_name> <new_name>
```

#### 도커 이미지 (Docker Image)의 이름(tag)을 바꾸려면?
- docker image의 이름(tag)은 그저 docker image hash의 alias이기 때문에 docker tag 명령어로 새로운 이름(tag)을 생성하고 기존의 이름(tag)을 삭제하면 된다.
```bash
$ docker image tag <이전 tag> <새 tag>
$ docker rmi <이전 tag>
```

* * *

### 실행중인 도커 컨테이너 (Docker Container) 목록 확인: 
```bash
$ docker container ls
또는
$ docker ps

# 종료된 컨테이너까지 모두 목록 확인
$ docker ps -a
```

* * *

### 실행중인 도커 컨테이너 (Docker Container) 명령어 전달:
- docker exec를 통해 명령 실행이 가능하다.
```bash
$ docker exec -it <container-id or name> <명령어> bash
```
<span style="color:#FA5858; font-size:12px">※ -it는 터미널과 컨테이너가 지속적으로 연결되도록 하는 옵션이다.</span>

- 위와 같은 명령어 실행 시 아래와 같이 진행된다.
[![텍스트](/assets/images/Linux/docker%20container%20%EB%AA%85%EB%A0%B9%EC%96%B4%20%EC%A0%84%EB%8B%AC%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/Linux/docker%20container%20%EB%AA%85%EB%A0%B9%EC%96%B4%20%EC%A0%84%EB%8B%AC%20%ED%99%94%EB%A9%B4.PNG)

#### 도커 컨테이너에서 명령어 실행하려면?
- 필자는 nginx 설정파일 변경을 위해 vi 명령어를 사용할려 했으나 아래와 같이 오류가 표시된다.
```bash
bash: vi: command not found
```

- 아래와 같이 명령어 사용을 위해 컨테이너 內 설치를 진행한다.
```bash
$ apt-get update
$ apt-get install vi
```

* * *