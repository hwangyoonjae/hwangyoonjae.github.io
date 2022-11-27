---
title: "[Ansible] - Ansible Playbook 사용하기"
layout: post
date: 2022-11-25
image: /assets/images/Post/ansible.png
headerImage: true
tag:
- Ansible
- 자동화
- Playbook
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Ansible Playbook 사용하기:
### Playbook 구성:
- YAML 형식으로 작성되며, 문법은 리스트 및 해쉬로 구성되어있다.
- YAML은 tab을 지원하지 않는다.
- Playbook의 목적은 정해진 대상서버(hosts)에 정해진 순서의 작업들(tasks)들을 실행하기 때문에 ansible에서 지원하는 다양한 module을 통해 원하는 각 작업(task)을 수행한다.

* * *