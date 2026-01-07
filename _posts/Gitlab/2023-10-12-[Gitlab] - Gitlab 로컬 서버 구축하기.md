---
layout: post
title: "Gitlab 로컬 서버 구축하기"
date: 2023-10-12
categories: [DevOps, Gitlab]
tags: [Git, Gitlab]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## GitLab을 로컬 서버의 구축하고 싶었던 계기:
- 고객사 서버 구축 및 패치 할 때, 버전 관리가 안되는거 같아 사내의 Git 저장소와 CI/CD 기능을 사용하고 싶어 구축하게되었다.

* * *

## GitLab이란?:
- 웹 기반 협업 및 DevOps 플랫폼으로, 소프트웨어 개발 및 버전 관리를 지원하는 강력한 도구이다.

* * *

## GitLab 기능과 역할:
- **Git 리포지토리 호스팅**: GitLab은 Git 리포지토리를 호스팅하고 관리하는 데 사용됩니다. 개발자 및 팀은 소스 코드를 업로드하고 협업하며 버전 관리를 수행할 수 있습니다.

- **이슈 및 프로젝트 관리**: 프로젝트 관리 및 이슈 추적을 위한 도구를 제공하고, 개발자와 팀은 작업 항목을 추적하고 우선순위를 관리할 수 있으며 프로젝트의 진행 상황을 시각화할 수 있습니다.

- **CI/CD (지속적 통합 및 지속적 전달)**: CI/CD 파이프라인을 설정하고 자동화하는 데 사용된다.

* * *

## GitLab 구축하기:
### GitLab의 필요한 종속성 설치 및 구성하기:
```bash
$ apt-get update
$ apt-get install -y curl openssh-server ca-certificates tzdata perl
```
<span style="color:#FA5858; font-size:12px">※ 필자는 Ubuntu 20.04.5 LTS 버전으로 구성하였다.</span>

* * *

### GitLab에서 알림 메일 보내는 Postfix 설치하기:
```bash
$ apt-get install -y postfix
```

* * *

### GitLab에서 패키지 저장소 추가하고 설치하기:
```bash
$ curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```
```bash
# 설치하기
$ apt install gitlab-ee
```

* * *

## GitLab 로그인하기:
- 브라우저의 https://IP주소로 입력하면 로그인 페이지가 나온다.
[![GitLab 로그인페이지](/assets/img/post/Gitlab/gitlab%20로그인페이지.png)](/assets/img/post/Gitlab/gitlab%20로그인페이지.png)

* * *

### root 비밀번호 확인하기:
```bash
$ cat /etc/gitlab/initial_root_password
```
[![root 비밀번호 확인](/assets/img/post/Gitlab/gitlab%20root%20패스워드%20확인.png)](/assets/img/post/Gitlab/gitlab%20root%20패스워드%20확인.png)

* * *

### root 비밀번호 변경하기:
```bash
$ gitlab-rails console -e production
```
```bash
$ irb(main):001:0> user = User.where(id: 1).first
  => #<User id:1 @root>
$ irb(main):002:0> user.password = '패스워드'
  => "패스워드"
$ irb(main):003:0> user.password_confirmation = '패스워드'
  => "패스워드"
$ irb(main):004:0> user.save
  => true
$ irb(main):005:0> exit
```
[![root 비밀번호 변경](/assets/img/post/Gitlab/gitlab%20root%20패스워드%20변경.png)](/assets/img/post/Gitlab/gitlab%20root%20패스워드%20변경.png)

* * *

## GitLab 메인페이지 확인하기:
- 로그인 후 GitLab 메인페이지 접속이 가능하다.
[![GitLab 메인페이지](/assets/img/post/Gitlab/gitlab%20메인페이지.png)](/assets/img/post/Gitlab/gitlab%20메인페이지.png)

* * *

## GitLab 서비스 구동하기:
- Gitlab 적용 및 재시작 해야하는 경우 아래 명령어를 입력합니다.
```bash
$ gitlab-ctl reconfigure
$ gitlab-ctl restart
```

* * *