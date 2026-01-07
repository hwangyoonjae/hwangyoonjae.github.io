---
layout: post
title: "Ansible란"
date: 2022-08-24
categories: [DevOps, Ansible]
tags: [Ansible, 자동화]
image: /assets/img/post-title/ansible-wallpaper.jpg
---

## 1. Ansible(앤서블)란? :
- 인프라 관리를 코드 기반으로 자동화하는 도구입니다.
- 코드 스크립트를 실행하기만 하면 코드가 알아서 다 해주게된다.

* * *

### 1.1 Ansible(앤서블) 특징 :
1. **멱등성 지원**: 
    - 멱등성 : 여러번 실행해도 같은 결과 값이 나오는 성질
    - Ansible에서의 멱등성 : 결과의 상태값이 다르더라도 결국에 결과는 동일하게 나오게 하는 성질
2. **Modular** 
    - 많은 모듈 지원 
    - Shell에 의존하지 않고 Ansible에서 지원하는 Module로 구성관리에 용이하다.
3. **YAML 형식 지원** 
    - 기존의 Shell Scripts보다 간편하게 구성 가능하다.
4. **대형 워크로드에 용이** 
    - 많은 서버에 구성이 필요한 경우 Shell Scripts보다 신속하게 처리 가능하다.
    
* * *

## 2. Ansible(앤서블)의 3가지 요소 :
- 앤서블은 크게 3가지인 인벤토리, 플레이북, 모듈로 이루어져있습니다.

* * *

### 2.1 인벤토리(inventory) :
- 앤서블에 의해 제어될 대상을 정의합니다.
- 일반적으로 hosts.ini 파일에 정의해 사용 하며, 여러 서버들의 SSH접근 iP, 포트, 리눅스 사용자 와 같은 접속 정보를 아래와 같이 정의합니다.

```bash
[webserver]

web1 ansible_host = aaa.app.host

web2 ansible_host = bbb.app.host

[db]

db1 ansible_host = aaa.db1.host

db2 ansible_host = bbb.db2.host
```

* * *

### 2.2 플레이북(Playbook) :
- 인벤토리 파일에서 정의한 대상들이 무엇을 수행할 것인지 정의하는 역할을하며, yaml 포맷으로 설정합니다.
- 단독으로 사용되는 것이 아닌 인벤터리와 플레이북의 조합으로 아래와 같이 사용합니다.

```bash
- name: ngins install
  hosts: all
  become: true
  tasks:
    - name: ngix package install
    yum:
       name: nginx
       state: installed
```

* * *

### 2.3 모듈(Module) :

- 플레이북에서 task가 어떻게 수행될지를 정의하는 요소입니다.
Python Code를 호출하여 실행하기 때문에 Python이 필수적으로 필요하며, 실제로 앤서블을 설치해보면 다양한 모듈이 같이 설치되는 것을 볼 수가 있습니다.

* * *