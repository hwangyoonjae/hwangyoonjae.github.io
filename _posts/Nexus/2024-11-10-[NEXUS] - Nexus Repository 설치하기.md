---
layout: post
title: "Nexus Repository 설치하기"
date: 2024-11-10
categories: [DevOps, Nexus]
tags: [Nexus, Docker, Repo]
image: /assets/img/post-title/nexus-wallpaper.jpg
---

## 1. Nexus Repository란? :
-  개발 프로젝트에서 필요한 패키지, 라이브러리, 아티팩트(예: JAR 파일, NPM 패키지, Docker 이미지 등)를 저장, 관리, 배포하는 중앙 저장소 역할을 하는 툴입니다.

* * *

## 2. Nexus Repository 구축하기 :
### 2.1 docker compose 파일 통하여 Nexus 설치하기 :
- docker를 통해 Nexus Repository를 구축합니다.

```yaml
services:
  nexus3:
    container_name: "nexus3"
    image: sonatype/nexus3
    environment:
      - TZ=Asia/Seoul
    restart: always
    ports:
      - 8443:8443
      - 5000:5000
    volumes:
      - ./nexus-default.properties:/opt/sonatype/nexus/etc/nexus-default.properties
      - ./ssl:/opt/sonatype/nexus/etc/ssl
      - /data/volume/nexus/data:/nexus-data
```

> 컨테이너 외부의 호스트 디렉토리 /data/volume/nexus/data UID 200 권한 부여하기
>
> Nexus Repository Manager는 컨테이너 내부에서 기본적으로 사용자 ID (UID) 200을 사용해 실행된다.
> 
> 이 UID는 Nexus 애플리케이션을 실행하는 사용자에 해당하며, 이 사용자만이 /nexus-data 디렉토리에 접근하여 파일을 읽고 쓸 수 있습니다.
{: .prompt-tip}

```bash
$ podman-compose up -d
# or
$ docker-compose up -d
```

![Nexus 컨테이너 실행 화면](/assets/img/post/Nexus/Nexus%20컨테이너%20실행%20화면.png)

* * *

### 2.2 Nexus Repository 대시보드 화면 접속하기 :
- 위와 같이 컨테이너 구축 후 **IP주소:8443** 주소로 대시보드 화면에 접속합니다.

![Nexus 대시보드 화면](/assets/img/post/Nexus/Nexus%20대시보드%20화면.png)

* * *

### 2.3 Nexus Repository 대시보드 로그인 하기 :
- nexus 를 처음 실행하면 Admin 계정으로 로그인이 필요하다.
  - ID: admin
  - Password: nexus 설치 디렉토리의 admin.password 파일에 최초 비밀번호가 저장되어 있습니다.

- admin 비밀번호 확인 방법은 다음과 같습니다.

```bash
$ podman exec -it nexus3 /bin/bash
```

```bash
bash-4.4$ ls
cd nexus-data
cat admin.password
```

![Nexus admin 계정 초기 패스워드 확인](/assets/img/post/Nexus/Nexus%20admin%20계정%20초기%20패스워드%20확인.png)

- 로그인을 진행합니다.

![Nexus 로그인 진행 1](/assets/img/post/Nexus/Nexus%20로그인%20진행%201.png)

![Nexus 로그인 진행 2](/assets/img/post/Nexus/Nexus%20로그인%20진행%202.png)

![Nexus 로그인 진행 3](/assets/img/post/Nexus/Nexus%20로그인%20진행%203.png)

![Nexus 로그인 진행 4](/assets/img/post/Nexus/Nexus%20로그인%20진행%204.png)

![Nexus 로그인 진행 5](/assets/img/post/Nexus/Nexus%20로그인%20진행%205.png)

![Nexus 로그인 진행 후 화면](/assets/img/post/Nexus/Nexus%20로그인%20진행%20후%20화면.png)

* * *