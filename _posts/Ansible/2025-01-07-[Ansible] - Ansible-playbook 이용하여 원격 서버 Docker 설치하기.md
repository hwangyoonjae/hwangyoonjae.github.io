---
layout: post
title: "Ansible-playbook 이용하여 원격 서버 Docker 설치하기"
date: 2025-01-07
categories: [DevOps, Ansible]
tags: [Ansible, Docker]
image: /assets/img/post-title/ansible-wallpaper.jpg
---

## 1. Ansible 설치 파일 준비하기 :
> 폐쇄망에서 설치하기 위해 인터넷이 되는 서버로부터 RPM 파일을 다운로드 받는다.
{: .prompt-warning}

![Ansible 설치파일 준비](/assets/img/post/Ansible/Ansible%20설치파일%20준비.png)

* * *

## 2. Docker 설치하는 Ansible-Playbook 생성하기 :

- 아래와 같이 Ansible-Playbook 생성하여 원격서버들로부터 Docker 설치를 진행합니다.

```yaml
---
- name: Install Docker and related dependencies on remote servers
  hosts: all
  become: yes
  vars:
    docker_install_file: "[설치파일 경로]"
    docker_version: "27.1.2-1"
    containerd_version: "1.7.20-3.1"
    docker_compose_version: "2.29.1-1"
    rpm_files:
      - "containerd.io-{{ containerd_version }}.el9.x86_64.rpm"
      - "container-selinux-2.229.0-1.el9.noarch.rpm"
      - "docker-buildx-plugin-0.16.2-1.el9.x86_64.rpm"
      - "docker-ce-{{ docker_version }}.el9.x86_64.rpm"
      - "docker-ce-cli-{{ docker_version }}.el9.x86_64.rpm"
      - "docker-ce-rootless-extras-{{ docker_version }}.el9.x86_64.rpm"
      - "docker-compose-plugin-{{ docker_compose_version }}.el9.x86_64.rpm"
      - "fuse3-3.10.2-8.el9.x86_64.rpm"
      - "fuse3-libs-3.10.2-8.el9.x86_64.rpm"
      - "fuse-common-3.10.2-8.el9.x86_64.rpm"
      - "fuse-overlayfs-1.13-1.el9.x86_64.rpm"
      - "libslirp-4.4.0-7.el9.x86_64.rpm"
      - "slirp4netns-1.2.3-1.el9.x86_64.rpm"
      - "tar-1.34-6.el9_1.x86_64.rpm"

  tasks:
    - name: Copy all Docker RPM files to the remote server
      copy:
        src: "{{ docker_install_file }}/{{ item }}"
        dest: "/tmp/{{ item }}"
      with_items: "{{ rpm_files }}"

    - name: Install Docker and dependencies using RPM packages
      command: "rpm -Uvh /tmp/{{ item }} --nodeps"
      with_items: "{{ rpm_files }}"
      register: install_result
      failed_when: "'is already installed' not in install_result.stderr and install_result.rc != 0"

    - name: Enable Docker service
      command: systemctl enable docker
      notify:
        - Start Docker service

    - name: Start Docker service
      command: systemctl start docker
      notify:
        - Start Docker service

    - name: Remove RPM files after installation
      file:
        path: "/tmp/{{ item }}"
        state: absent
      with_items: "{{ rpm_files }}"

  handlers:
    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes
```

* * *

- Ansible hosts파일의 배포할 원격서버 IP주소를 입력합니다.

```bash
$ vi /etc/ansible/host
```

```bash
[docker_install]
192.168.1.2
192.168.1.3
```

* * *

- 배포를 진행합니다.

```bash
$ ansible-playbook -i /etc/ansible/hosts docker-install.yml --limit docker_install -vvv
```

* * *