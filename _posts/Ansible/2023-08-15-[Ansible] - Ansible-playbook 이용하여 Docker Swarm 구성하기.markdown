---
title: "[Ansible] - Ansible-playbook 이용하여 Docker Swarm 구성하기"
categories:
  - Ansible
tags:
  - [Ansible, Docker, Swarm]

toc: true
toc_sticky: true

date: 2023-08-15
last_modified_at: 2023-08-15
---

## Manager 노드 구성하기:
- Ansible Playbook에서 docker swarm init 명령을 사용하여 토큰을 생성하고 이를 파일에 저장한다.

```bash
# 파일명은 docker_manager_setup.yml
---
- name: Set Up Docker Swarm Manager
  hosts: swarm_manager
  tasks:
    - name: Initialize Swarm
      command: docker swarm init --advertise-addr IP주소
      register: swarm_init_result

    - name: Save Join Token
      # Manager Node에서 생성한 토큰값을 텍스트 파일에 저장
      shell: echo "{{ swarm_init_result.stdout_lines[1] }}" > ./token.txt
```

- 위와 같이 파일 생성 후 ansible-playbook 명령을 실행한다.
```bash
$ ansible-playbook -i host.ini docker_manager_setup.yml
```

* * *