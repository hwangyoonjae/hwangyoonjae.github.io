---
layout: post
title: "[Kubernetes]- k9s란"
date: 2024-07-30
categories: Kubernetes Concept
tags: [Kubernetes, k9s]
image: /assets/img/post-title/k9s-wallpaper.jpg
---

## k9s란? :
- Kubernetes 클러스터를 터미널에서 사용하기 위한 CLI 도구이다.
- k9s를 사용하면 kubectl과 함께 사용할 수 있어서 필요에 따라 kubectl과 k9s를 자유롭게 전환하여 사용할 수 있다.

* * *

## k9s 설치하기 :

```bash
# 파일 다운로드
$ wget https://github.com/derailed/k9s/releases/download/v0.32.5/k9s_Linux_amd64.tar.gz

# 압축파일 해제
$ tar -zxvf k9s_Linux_amd64.tar.gz

# 실행파일 이동
$ mv k9s /usr/local/bin/

# 설치정보 확인
$ k9s info
```
![k9s 정보 확인](/assets/img/post/kubernetes/k9s%20정보%20확인.png)

* * *

## k9s 메인화면 :
```bash
$ k9s
```

- 위 명령어 입력시 아래와 같이 전체 pod 정보를 확인할 수 있다.
![k9s 메인화면](/assets/img/post/kubernetes/k9s%20메인화면.png)

- 메인화면에서 단축키 **?**를 입력하면 help 화면으로 이동되어, 단축키 사용법에 대해서 알 수 있다.
![k9s help 화면](/assets/img/post/kubernetes/k9s%20help%20화면.png)

* * *

## CLI 인수 :
```bash
# 사용 가능한 모든 CLI 옵션 나열
$ k9s help

# K9s 런타임(로그, 구성 등)에 대한 정보를 가져옴
$ k9s info

# 지정된 네임스페이스에서 K9를 실행
$ k9s -n mycoolns

# K9s를 실행하고 pod 명령을 통해 pod view로 시작
$ k9s -c pod

# 기본값이 아닌 KubeConfig 컨텍스트에서 K9 시작
$ k9s --context coolCtx

# 모든 수정 명령이 비활성화된 상태에서 K9s 읽기 전용 모드로 시작
$ k9s --readonly

```

* * *