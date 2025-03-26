---
layout: post
title: "[Kubernetes]- Kubespray로 Kubernetes 설치하기"
date: 2024-08-30
categories: Kubernetes
tags: [Kubernetes, Kubespray]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## Kubespray란? :
- 상용 서비스에 적합한 보안성과 고가용성이 있는 쿠버네티스 클러스터를 배포하는 오픈 소스 프로젝트이다.

* * *

## Kubespray 설치하기 :
### SSH 키 생성 및 복사하기 :
- 로커 서버의 SSH 키를 생성하여 원격 서버에 안전하게 연결하기 위해 공개 키 암호화를 사용할 수 있도록 공개 키와 비공개 키 쌍을 생성한다.

```bash
$ ssh-keygen
```

- 로컬 서버에서 생성한 SSH 공개 키를 원격 서버에 복사한다.

```bash
$ ssh-copy-id [계정]@[원격IP]
```

* * *

### kubespray Git 저장소 클론하기 :

```bash
$ git clone -b v2.24.0 https://github.com/kubernetes-sigs/kubespray.git
```

[![kubespray Git 저장소 클론](/assets/img/post/kubespray/kubespray%20Git%20저장소%20클론.png)](/assets/img/post/kubespray/kubespray%20Git%20저장소%20클론.png)

* * *

### requirements.txt 파일에서 의존성 확인 및 설치하기 :
- ansible과 필요한 도구들을 같이 설치한다.

```bash
$ vi requirements.txt
```

[![requirements.txt 파일에서 의존성 확인](/assets/img/post/kubespray/requirements.txt%20파일에서%20의존성%20확인.png)](/assets/img/post/kubespray/requirements.txt%20파일에서%20의존성%20확인.png)

```bash
$ pip3 install -r requirements.txt

# 오프라인 환경에서 설치해야하는 경우에는 아래 명령어를 통해 설치파일을 다운로드 받는다.
$ pip3 download -r requirements.txt

# 다운받은 설치파일들은 아래 명령어를 통해 설치한다.
$ pip3 install --no-index --find-links=[폴더 경로로] *.whl
```

[![requirements.txt 파일에서 의존성 설치](/assets/img/post/kubespray/requirements.txt%20파일에서%20의존성%20설치.png)](/assets/img/post/kubespray/requirements.txt%20파일에서%20의존성%20설치.png)

* * *