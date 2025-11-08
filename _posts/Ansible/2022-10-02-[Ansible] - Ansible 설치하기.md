---
layout: post
title: "Ansible 설치하기"
date: 2022-10-02
categories: [DevOps, Ansible]
tags: [Ansible, 자동화]
image: /assets/img/post-title/ansible-wallpaper.jpg
---

## 1. Ansible 설치하기 :
### 1.1 Ansible 패키지 설치하기 :
```bash
# RPEL yum 레포지토리 추가 및 추가목록 확인
$ yum install -y epel-release

# Ansible 설치 진행
$ yum install -y ansible 
```

### 1.2 Ansible 버전 확인하기 :
```bash
$ ansible --version
```
[![텍스트](/assets/img/post/Ansible/Ansible%20%EB%B2%84%EC%A0%84%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/Ansible/Ansible%20%EB%B2%84%EC%A0%84%ED%99%95%EC%9D%B8.PNG)<br>
<span style="color:#FA5858; font-size:12px">※ 필자는 2.9.27버전을 사용했다.</span>

* * *

### 1.3 SSH Key 생성하기 :
- Ansible은 SSH 접속을 기반으로 원격 서버들에게 명령을 전달한다.
- Controller 서버와 원격 서버간 SSH key가 공유되어야 하고, Controller 서버에서 모든 작업을 완료할 수 있다.

```bash
#RSA키 생성, 명령어 입력 후 나오는 것들 그냥 전부 엔터
$ ssh-keygen

# 생성한 key를 원격 서버에 복사
$ ssh-copy-id [원격서버계정ID]@[원격서버IP]
```
[![텍스트](/assets/img/post/Ansible/SSH%20Key%20%EC%83%9D%EC%84%B1%20%EB%B0%8F%20%EB%B3%B5%EC%82%AC.PNG)](/assets/img/post/Ansible/SSH%20Key%20%EC%83%9D%EC%84%B1%20%EB%B0%8F%20%EB%B3%B5%EC%82%AC.PNG)

- ssh key가 복사되었다면, controller 서버에서 원격 서버로 ssh 접속을 시도할 때, 비밀번호 입력없이 바로 접속되어야 한다.

```bash
$ ssh [원격서버계정ID]@[원격서버IP]
```
[![텍스트](/assets/img/post/Ansible/SSH%20%EC%A0%91%EC%86%8D%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/Ansible/SSH%20%EC%A0%91%EC%86%8D%ED%99%95%EC%9D%B8.PNG)

* * *

### 1.4 /etc/hosts 파일에 각각의 host와 ip 매핑하기 :

```bash
# /etc/hosts 파일에 각각의 host와 ip를 매핑 시켜주면 user@hostname 으로도 ssh 접근 가능
$ vi /etc/hosts
```
[![텍스트](/assets/img/post/Ansible/host%EC%99%80%20ip%20%EB%A7%A4%ED%95%91.PNG)](/assets/img/post/Ansible/host%EC%99%80%20ip%20%EB%A7%A4%ED%95%91.PNG)

```bash
#명령어를 입력하면 원격서버 host파일의 원격서버계정에서 sudo id를 입력한 결과 출력
ssh [원격서버계정ID]@[원격서버 host파일] sudo id
```
[![텍스트](/assets/img/post/Ansible/host%EC%99%80%20ip%20%EB%A7%A4%ED%95%91%20%EB%8F%99%EC%9E%91%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/Ansible/host%EC%99%80%20ip%20%EB%A7%A4%ED%95%91%20%EB%8F%99%EC%9E%91%20%ED%99%95%EC%9D%B8.PNG)

* * *