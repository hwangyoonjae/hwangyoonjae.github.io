---
layout: post
title: "Docker 이미지 다운로드"
date: 2022-08-25
categories: [컨테이너, Docker]
tags: [Docker, Image, Container]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## 1. 도커 이미지 (Docker Image)란 무엇인가? :
- "이미지 = 설정파일" 즉, 가상화에 WebServer, WAS, DB 등 설치하여 구성해야하는데 도커에서 이미지는 다운받아서 하나의 컨테이너만 만들면 그 만든 컨테이너 하나로 여러 개의 도커를 생성할 수 있습니다.

* * *

### 1.1 도커 이미지 (Docker Image) 특징 :
- 이미지는 컨테이너 실행에 필요한 파일과 설정값 등을 포함하고 있는 것으로 변하지 않는다.
- 이미지에 데이터가 쓰이는 것은 아니라 Immutable(불변적)입니다.
- 이미지를 다운로드할 때 개인이 git에 올려둔 것을 사용해도 되고, 이미지 경로(url)를 안 적으면 docker.org에 있는 서버로 연결됩니다.
- 최상위 layer에만 읽기 쓰기가 가능하다.

* * *

## 2. 도커 컨테이너 이미지 실습 :
### 2.1 도커 컨테이너 이미지 찾기 :
- 이미지는 Docker Hub 사이트에서 찾아볼 수 있고, 서버에서 명령어로 확인 가능하다.
> * [Docker Hub 바로가기](https://hub.docker.com/search?image_filter=official&q= "Docker Hub")

![텍스트](/assets/img/post/docker/docker%20hub%20%ED%8E%98%EC%9D%B4%EC%A7%80%20%ED%99%94%EB%A9%B4.PNG)

```bash
# 서버에서 도커 이미지 확인 방법
$ docker search [이미지명]
```
![텍스트](/assets/img/post/docker/docker%20image%20%EC%84%9C%EB%B2%84%EC%97%90%EC%84%9C%20%EA%B2%80%EC%83%89%20%ED%99%94%EB%A9%B4.PNG)

### 2.2 도커 컨테이너 이미지 다운로드 :
- 이미지 찾기를 통해서 다운로드할 이미지 이름과 필요한 버전을 입력하여 다운로드합니다.

```bash
$ docker pull [이미지명]
```
![텍스트](/assets/img/post/docker/docker%20image%20%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C%20%ED%99%94%EB%A9%B4.PNG)

* * *

### 2.3 도커 이미지 목록 보기 :
- 다운받은 이미지 목록을 확인합니다.

```bash
$ docker images
```
[![텍스트](/assets/img/post/docker/docker%20image%20%EB%AA%A9%EB%A1%9D%20%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/docker/docker%20image%20%EB%AA%A9%EB%A1%9D%20%ED%99%94%EB%A9%B4.PNG)

* * *

### 2.4 도커 이미지 압축하기 :
- 내부망(인터넷 접근 불가)에서 사용할 이미지를 압축합니다.

```bash
$ docker save <image명> > <image명>.tar
```
![텍스트](/assets/img/post/docker/docker%20image%20%EC%95%95%EC%B6%95%20%EC%A7%84%ED%96%89%20%ED%99%94%EB%A9%B4.PNG)

* * *

### 2.5 도커 이미지 삭제하기 :
- **"docker rmi"**명령어를 사용하여 도커 이미지를 삭제합니다.

> rmi는 "remove image"의 줄임말입니다.
{: .prompt-info}

```bash
$ docker rmi [이미지명]
```
![텍스트](/assets/img/post/docker/docker%20image%20%EC%A0%95%EC%83%81%EC%82%AD%EC%A0%9C.PNG)
  
* * *

### 2.6 이미지 강제 삭제하기 :
- 사용중인 이미지에 대해서는 아래와 같은 에러가 출력되며 삭제가 안됩니다.
```
Error response from daemon: conflict: unable to remove repository reference "nginx" (must force) - container 7d603f62cb2a is using its referenced image 2b7d6430f78d
```

- 강제로 삭제하는 -f 옵션을 넣어 강제 삭제합니다.

```bash
$ docker rmi -f [이미지명]
```
![텍스트](/assets/img/post/docker/docker%20image%20%EC%99%84%EC%A0%84%EC%82%AD%EC%A0%9C.PNG)
* * *