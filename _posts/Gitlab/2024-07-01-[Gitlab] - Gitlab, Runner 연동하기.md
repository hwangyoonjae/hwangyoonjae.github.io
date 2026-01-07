---
layout: post
title: "Gitlab, Runner 연동하기"
date: 2024.07.01
categories: [DevOps, Gitlab]
tags: [Git, Gitlab, Runner]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## Gitlab 프로젝트 Runner 설정하기 :
- 프로젝트 클릭 -> Settings -> CI/CD -> Runner -> Expand 버튼 클릭
[![gitlab 프로젝트 runner 설정](/assets/img/post/Gitlab/gitlab%20프로젝트%20runner%20설정.png)](/assets/img/post/Gitlab/gitlab%20프로젝트%20runner%20설정.png)

* * *

## Specific runners에 있는 URL과 Token값을 확인하기 :
- Gitlab-Runner 연동 시 사용할 URL과 Token 값을 확인합니다.
[![gitlab runner 설정 토큰값 확인](/assets/img/post/Gitlab/gitlab%20runner%20설정%20토큰값%20확인.png)](/assets/img/post/Gitlab/gitlab%20runner%20설정%20토큰값%20확인.png)

* * *

## Gitlab-Runner 컨테이너 접속하기 :

```bash
$ docker exec -it {gitlab-runner 컨테이너명} /bin/bash
```

* * *

## Gitlab-Runner 대화식으로 설정하기 :

```bash
$ gitlab-runner register
```

```bash
Enter the GitLab instance URL (for example, https://gitlab.com/):
# Gitlab 주소 입력 (https://gitlab.inno.com:8443/)

Enter the registration token:
# Gitlab token 입력 (GR1348941czTqnjZPQmkHaRd5eU_9)

Enter a description for the runner:
[gitlab-runner]: # {Enter}
Enter tags for the runner (comma-separated):  # {Enter}

Enter optional maintenance note for the runner:

Registering runner... succeeded           runner=GR1348941czTqnjZP

Enter an executor: custom, docker, docker-ssh, ssh, docker-autoscaler, docker+machine, docker-ssh+machine, instance, docker-windows, parallels, shell, virtualbox, kubernetes:
# docker (GitLab Runner가 빌드 작업을 실행할 방식 선택)
Enter the default Docker image (for example, ruby:2.7):
# docker-latest (기본적으로 사용할 Docker 이미지를 지정)
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!

Configuration (with the authentication token) was saved in "/etc/gitlab-runner/config.toml"

```

> Gitlab 주소를 도메인으로 지정해서 연결해야하는 경우
>- /etc/hosts의 **IP주소 도메인명** 형태로 저장합니다.
{: .prompt-warning }

- 아래와 같이 컨이이너 내부의 /etc/hosts 파일에 등록합니다.
```bash
$ echo "192.168.150.140 gitlab.inno.com" >> /etc/hosts
```

* * *

## Gitlab 페이지 들어가서 정상적으로 등록되었는지 확인하기 :
[![gitlab-runner 등록 확인](/assets/img/post/Gitlab/gitlab-runner%20등록%20확인.png)](/assets/img/post/Gitlab/gitlab-runner%20등록%20확인.png)

---
