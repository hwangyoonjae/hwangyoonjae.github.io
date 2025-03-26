---
layout: post
title: "[Gitlab] - Gitlab, Runner 연동 시 질문"
date: 2024.07.11
categories: Gitlab 
tags: [Git, Gitlab, Runner]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## 1. GitLab Runner가 사용할 실행기를 선택하기 :
- CI/CD 작업을 실행할 환경을 정의

>질문 내용
>Enter an executor: docker-autoscaler, docker-windows, ssh, virtualbox, parallels, shell, docker+machine, docker-ssh+machine, instance, custom, docker, docker-ssh, kubernetes:
{: .prompt-warning }

- 선택한 실행기는 GitLab Runner가 CI/CD 파이프라인을 실행할 환경을 결정한다.

>실행기 유형
>- **docker-autoscaler**: Docker 환경을 자동으로 확장하여 실행한다.
>- **docker-windows**: Windows 기반 Docker 컨테이너에서 실행한다.
>- **ssh**: 원격 서버에서 SSH를 통해 실행한다.
>- **virtualbox**: VirtualBox VM에서 실행한다.
>- **parallels**: Parallels VM에서 실행한다.
>- **shell**: 로컬 셸 환경에서 실행한다.
>- **docker+machine**: Docker와 Docker Machine을 함께 사용하여 실행한다.
>- **docker-ssh+machine**: Docker, SSH, Docker Machine을 함께 사용하여 실행한다.
>- **instance**: 특정 인스턴스에서 실행한다.
>- **custom**: 사용자 정의 실행기에서 실행한다.
>- **docker**: Docker 컨테이너에서 실행한다.
>- **docker-ssh**: Docker 컨테이너에서 SSH를 통해 실행한다.
>- **kubernetes**: Kubernetes 클러스터에서 실행한다.
{: .prompt-tip }

---
## 2. Docker 실행기를 선택한 경우, CI/CD 작업을 실행할 기본 Docker 이미지 지정하기 :
- 파이프라인에서 사용할 언어 또는 도구가 포함된 이미지를 지정

>질문 내용
>Enter the default Docker image (for example, ruby:2.7):
{: .prompt-warning }

- 이 이미지는 .gitlab-ci.yml 파일에서 특정 이미지가 지정되지 않은 경우 사용된다.
- 예를 들어, 빌드, 테스트, 배포 작업에서 사용할 도구가 포함된 이미지를 지정하면, 해당 이미지가 기본적으로 사용한다.

---