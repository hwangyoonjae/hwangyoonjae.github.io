---
layout: post
title: "Ansible Playbook 사용하기"
date: 2022-11-25
categories: [DevOps, Ansible]
tags: [Ansible, 자동화, Playbook]
image: /assets/img/post-title/ansible-wallpaper.jpg
---

## 1. Ansible Playbook 사용하기 :
### 1.1 Playbook 구성 :
- YAML 형식으로 작성되며, 문법은 리스트 및 해쉬로 구성되어있습니다.
- YAML은 tab을 지원하지 않는다.
- Playbook의 목적은 정해진 대상서버(hosts)에 정해진 순서의 작업들(tasks)들을 실행하기 때문에 ansible에서 지원하는 다양한 module을 통해 원하는 각 작업(task)을 수행합니다.

* * *

### 1.2 Playbook 예시 :
[![텍스트](/assets/img/post/Ansible/Playbook%20%EC%98%88%EC%8B%9C.PNG)](/assets/img/post/Ansible/Playbook%20%EC%98%88%EC%8B%9C.PNG)
- Playbook을 통해 nginx를 설치하는 예시를 작성했고, 형식에 대한 의미는 아래와 같습니다.
  
```html
- name: Playbook 이름
- hosts: 연결할 host 이름
- become: 명령을 실행하는 사용자 계정의 권한을 승격
- tasks: 연결할 host에서 수행할 작업
```

- CentOS는 yum을 사용하고, Ubuntu는 apt를 사용하고, present=설치, absent=삭제, latest=저장소 내의 최신버전 설치를 의미합니다.
- service는 OS의 systemctl과 같은 명령어로 started=실행, stopped=중지, restarted=재실행 을 의미합니다.

### 1.3 연결할 host 이름 지정하기 :
```bash
$ vi /etc/ansible/hosts
```
[![텍스트](/assets/img/post/Ansible/host%20%EC%9D%B4%EB%A6%84%20%EC%A7%80%EC%A0%95.PNG)](/assets/img/post/Ansible/host%20%EC%9D%B4%EB%A6%84%20%EC%A7%80%EC%A0%95.PNG)

> 위 그림과 같이 ansible에 설정한 host이름을 지정한 서버IP별로 설치할 수 있습니다.
{: .prompt-tip}

* * *

### 1.4 Playbook 실행하기 :
```bash
$ ansible-playbook 파일명.yml -k
```
[![텍스트](/assets/img/post/Ansible/Playbook%20%EC%8B%A4%ED%96%89%EA%B2%B0%EA%B3%BC.PNG)](/assets/img/post/Ansible/Playbook%20%EC%8B%A4%ED%96%89%EA%B2%B0%EA%B3%BC.PNG)

> **-k** 옵션은 ansible 실행 시 ansible 호스트의 비밀번호를 물어볼 수 있도록 하는 옵션입니다.
{: .prompt-info}

* * *
