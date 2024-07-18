---
layout: post
title: "[Docker] - Docker 이미지 레이어 구조"
date: 2023-05-25
categories: Docker
tags: [Docker, Overlay2]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## Docker Layer란?:
- 도커 이미지를 받으면 마치 여러개로 분리된 조각처럼 보이는게 분리된 데이터를 말한다.
- 기존 이미지에 추가적인 파일이 필요할 때 다시 다운로드 받는 방법이 아닌 해당 파일을 추가한다.
[![docker 이미지와 압축파일 비교](/assets/img/post/docker/docker%20%EC%9D%B4%EB%AF%B8%EC%A7%80%EC%99%80%20%EC%95%95%EC%B6%95%ED%8C%8C%EC%9D%BC%20%EA%B5%AC%EC%84%B1%20%EB%B9%84%EA%B5%90.PNG)](/assets/img/post/docker/docker%20%EC%9D%B4%EB%AF%B8%EC%A7%80%EC%99%80%20%EC%95%95%EC%B6%95%ED%8C%8C%EC%9D%BC%20%EA%B5%AC%EC%84%B1%20%EB%B9%84%EA%B5%90.PNG)

* * *

## Docker home 디렉토리:
- Docker home 디렉토리는 아래 명령어로 확인 가능하다.
```bash
$ docker info | grep "Docker Root Dir"
```
[![텍스트](/assets/img/post/docker/docker%20home%20%EC%9C%84%EC%B9%98%20%ED%99%95%EC%9D%B8%20%EB%AA%85%EB%A0%B9%EC%96%B4.PNG)](/assets/img/post/docker/docker%20home%20%EC%9C%84%EC%B9%98%20%ED%99%95%EC%9D%B8%20%EB%AA%85%EB%A0%B9%EC%96%B4.PNG)

- systemctl 등의 systemd를 관리하는 툴을 이용하면 loaded되는 service 설정에 대한 path를 가져올 수 있는데 여기서 home path 변경이 가능하다.
[![텍스트](/assets/img/post/docker/docker%20service%20%EB%A1%9C%EB%93%9C%EB%90%98%EB%8A%94%20%EA%B2%BD%EB%A1%9C.PNG)](/assets/img/post/docker/docker%20service%20%EB%A1%9C%EB%93%9C%EB%90%98%EB%8A%94%20%EA%B2%BD%EB%A1%9C.PNG)

* * *

## Docker Image Layer 정보 확인하기:
- 도커 이미지 레이어 정보는 **/var/lib/docker/image/overlay2/layerdb/sha256/**에 위치한다.
- 해당 경로에는 유니온 마운트를 구성하는 low layer, upper layer의 hash data들이 모두 저장되어있다.
```bash
$ docker inspect 이미지명
```
[![docker 이미지 레이어 정보 확인](/assets/img/post/docker/docker%20%EC%9D%B4%EB%AF%B8%EC%A7%80%20%EB%A0%88%EC%9D%B4%EC%96%B4%20%EC%A0%95%EB%B3%B4%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/docker/docker%20%EC%9D%B4%EB%AF%B8%EC%A7%80%20%EB%A0%88%EC%9D%B4%EC%96%B4%20%EC%A0%95%EB%B3%B4%20%ED%99%95%EC%9D%B8.PNG)

* * *

### Docker overlay2란?:
- 레이어 파일 시스템이라고 하며, Linux 배포판에 대해서 선호되는 스토리지 드라이버라고 한다.

* * *

## Docker Layer Data 위치 확인하기:
- 실제 layer 정보를 가지고 있는 폴더는 **/var/lib/docker/overlay2/**에 위치한다.
[![docker 레이어 데이터 위치 확인](/assets/img/post/docker/docker%20%EB%A0%88%EC%9D%B4%EC%96%B4%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9C%84%EC%B9%98%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/docker/docker%20%EB%A0%88%EC%9D%B4%EC%96%B4%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9C%84%EC%B9%98%20%ED%99%95%EC%9D%B8.PNG)
- 위 그림과 같이 레이어 메타정보에서 **cache-id; echo** 명령어로 레이어 데이터 위치를 알 수 있다.

* * *