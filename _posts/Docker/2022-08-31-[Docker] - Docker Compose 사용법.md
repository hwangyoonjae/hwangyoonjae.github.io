---
layout: post
title: "Docker 컴포즈 사용법"
date: 2022-08-31
categories: [컨테이너, Docker] 
tags: [Docker, YAML]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## 1. 도커 컴포즈(Docker Compose)란? :
- 복수 개의 컨테이너가 유기적으로 묶여서 하나의 도커 애플리케이션으로 동작할 수 있도록 구성하는 도구이며, 복수 개의 컨테이너 생성 및 실행을 자동화하고 관리하는 기능입니다.

* * *

## 2. 도커 컴포즈(Docker Compose) 기본 사용법 :
- docker-compose.yml 파일을 작성하여 실행할 수 있고, YAML(야멜) 형식으로 작성해야 합니다.

* * *

### 2.1 YAML 문법 :
- 들여쓰기와 함게 key와 value를 중심으로 작성되고, 기본적으로 숫자형, 문자형, Boolean 자료형을 지원합니다.

- **기본 문법**
    - **#** : 해당 라인을 주석처리합니다.
    - **---** : 문서의 시작을 알린다.(옵션)
    - **...** : 문서의 끝을 알린다.(옵션)
    - **key** : 딕셔너리 자료형의 키 값과 동일 개념입니다.
    - **value** : 딕셔너리 자료형의 벨류 값과 동일 개념입니다.

* * *

## 3. 도커 컴포즈(Docker Compose) 사용법 이해하기 :
### 3.1 docker-compose.yml 예시 :
- 기본적으로 docker-compose.yml은 version, services, volumes, networks의 카테고리로 작성되지만, 주로 version과 services가 많이 사용됩니다.

![텍스트](/assets/img/post/docker/docker%20compose%20%EC%9E%91%EC%84%B1%20%EC%98%88%EC%8B%9C.PNG)

- **문법**
    - **version** : docker compose의 파일 포맷 버전을 지정합니다.<br>
    <span style="color:#FA5858; font-size:12px">※ 기본적으로 버전 3을 사용하는 것이 일반적입니다.</span>

    - **services** : 한개 또는 여러 개의 docker container를 설정합니다.

    - **image** : docker container의 이름을 정의합니다.<br>
    <span style="color:#FA5858; font-size:12px">※ Docker Hub에 있는 이미지를 사용하여 docker container를 작성할 경우 image를 설정할 수 있습니다.</span>

    - **restart** : docker container가 다운되었을 경우, 항상 재시작하는 설정입니다.

    - **volumes** : 로컬에서 작업 할 폴더의 위치와 컨테이너의 위치를 연결시켜준다.<br>
    <span style="color:#D8D8D8; font-size:12px">left{로컬스토리지}:right{컨테이너내부}</span><br>
    <span style="color:#FA5858; font-size:12px">※ 여러 개의 volume을 지정할 수 있고, 리스트처럼 작성하면 됩니다.</span>

    - **environment** : dockerfile의 ENV 옵션과 동일한 역할을 합니다.<br>
     <span style="color:#FA5858; font-size:12px">※ env_file 옵션으로 환경변수 값이 들어있는 파일을 읽을 수도 있습니다. (패스워드 등의 보안을 위한 방법)</span>
    ```bash
    # env_file 포맷
    $ cat mysql.env
    MYSQL_ROOT_PASSWORD=dgkcoding
    MYSQL_DATABASE=dgkdb
    ```

    - **ports** : docker run 명령의 -p 옵션과 동일한 역할을 합니다.<br>
    <span style="color:#D8D8D8; font-size:12px">호스트 머신의 포트번호: 컨테이너의 포트번호</span><br>
    <span style="color:#FA5858; font-size:12px">※ 포트번호를 입력할 때에는 반드시 쌍따옴표 안에 작성해야 합니다. (YAML 문법에서 숫자:숫자 는 시간으로 해석하기 때문입니다.)</span>

    - **networks** : docker container 간의 네트워크 분리를 위해 추가로 설정을 합니다.

    - **build** : docker image를 Dockerfile 기반으로 작성 시 사용합니다.<br>
    <span style="color:#FA5858; font-size:12px">※ context는 Dockerfile이 있는 디렉토리를 의미하며 dockerfile은 Dockerfile의 파일명을 의미합니다.</span>

    - **links** : 특정 docker container 내부에서 다른 docker container에 접속하고 싶을 때 사용합니다.

    - **container_name** : docker-compose.yml 파일로 docker container를 만들 때, 해당 컨테이너 이름을 설정할 수 있습니다.

    - **depends_on** : 여러 개의 docker container를  Docker Compose로 실행할 경우, 각 컨테이너가 실행되는 시점이 다르기 때문에 특정 컨테이너가 시작하자마자 바로 다른 컨테이너에 접속하면 경우에 따라서는 접속할 수 없는 상황이 발생합니다.

    - **deploy** : 도커 스웜에서 사용되는 여러 옵션들을 정의합니다.

* * *

## 4.1 Docker Compose 설치하기 :
- curl 명령어를 통해 docker-compose를 설치합니다.

```bash
$ curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

* * *

- 도커 컴포즈 파일을 실행하기 다운로드 경로에 권한을 부여합니다.

```bash
$ chmod +x /usr/local/bin/docker-compose
```

* * *

- docker-compose 버전을 확인합니다.

```bash
# docker compose 버전 확인
$ docker-compose -version
```

![docker compose 설치하는 과정](/assets/img/post/docker/docker%20compose%20설치하는%20과정.PNG)