---
title: "[Linux] - Ansible 설치하기"
layout: post
date: 2022-10-02
image: /assets/images/Post/ansible.png
headerImage: true
tag:
- Ansible
- 자동화
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Ansible 설치하기:
### Ansible 패키지 설치하기:
```bash
# RPEL yum 레포지토리 추가 및 추가목록 확인
$ yum install -y epel-release
$ yum repolist

# Ansible 설치 진행
$ yum install -y ansible 
```

### Ansible 버전 확인하기:
```bash
$ ansible --version
```
[![텍스트](/assets/images/Ansible/Ansible%20%EB%B2%84%EC%A0%84%ED%99%95%EC%9D%B8.PNG)](/assets/images/Ansible/Ansible%20%EB%B2%84%EC%A0%84%ED%99%95%EC%9D%B8.PNG)<br>
<span style="color:#FA5858; font-size:12px">※ 필자는 2.9.27버전을 사용했다.</span>

* * *