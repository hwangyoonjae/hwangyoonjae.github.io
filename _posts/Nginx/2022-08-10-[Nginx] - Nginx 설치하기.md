---
layout: post
title: "[Nginx] - Nginx 설치하기"
date: 2022-08-10
categories: Nginx
tags: [Web, Nginx]
image: /assets/img/post-title/nginx-wallpaper.jpg
---

## Nginx란? :
- 클라이언트로부터 요청을 받았을 때 요청에 맞는 정적 파일을 응답해주는 HTTP Web Server로 활용되기도 하고, Reverse Proxy Server로 활용하여 WAS 서버의 부하를 줄일 수 있는 로드 밸런서로 활용되기도 한다.

* * *

## Nginx 설치하기 :

### Yum 통해서 설치하기 :
- yum 저장소에는 nginx가 없기 때문에 외부저장소를 추가해야 한다.
```bash
# yum 외부 저장소 추가 :
$ vi /etc/yum.repos.d/nginx.repo
```

- **/etc/yum.repos.d**경로에 **nginx.repo** 파일을 추가하고 내용은 다음과 같이 작성한다.
```bash
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

- nginx 설치 진행
```bash
# yum으로 nginx 설치 :
$ yum install -y nginx
```

<span style="color:#FA5858; font-size:12px">※ 필자는 1.22.0버전을 사용했다.</span>

* * *

### 폐쇄망 설치:
- 폐쇄망 서버에 Nginx를 설치하는 경우 아래 URL 접속 시 CentOS 버전에 맞춰 폴더 접속한 후 **/Linux버전/x86_64/RPMS/**에서  필요한 RPM 다운받는다.
> * [Nginx RPM 패키지 다운로드](https://nginx.org/packages/centos/ "Nginx RPM 패키지 다운로드")

```bash
# rpm으로 설치
$ rpm -Uvh nginx-1.20.0-1.el7.ngx.x86_64.rpm --nodeps
```

* * *

### 방화벽 포트 개방 :
```bash
# 방화벽 포트 개방
$ firewall-cmd --permanent --zone=public --add-port=8089/tcp

# 방화벽 재시작
$ firewall-cmd --reload

# 방화벽 포트 목록 확인
$ firewall-cmd --list-all
```

* * *

### Nginx 포트 설정 :
- 방금 방화벽 개방한 포트로 변경한다.
```bash
$ vi /etc/nginx/conf.d/default.conf
```
[![텍스트](/assets/img/post/Linux/Nginx%20%ED%8F%AC%ED%8A%B8%20%EB%B3%80%EA%B2%BD.PNG)](/assets/img/post/Linux/Nginx%20%ED%8F%AC%ED%8A%B8%20%EB%B3%80%EA%B2%BD.PNG)
- 밑줄 친 부분을 변경한다.
```javascript
Ex) Listen 80 -> Listen 8089
```

* * *

### Nginx 실행 :
```bash
$ systemctl start nginx
$ systemctl enable nginx
```

<span style="color:#FA5858; font-size:12px">※ systemctl enable 명령어를 사용하는 이유 : 서버 부팅 시 자동으로 서비스 구동하기 위해 사용한다.</span>

* * *

### Nginx 실행화면 :
- 위와 같은 방법 진행 후 실행하면 아래와 같은 화면이 나오면서 웹 페이지 정상 구동된다.
[![텍스트](/assets/img/post/Linux/Nginx%20%EC%84%B1%EA%B3%B5%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/Linux/Nginx%20%EC%84%B1%EA%B3%B5%ED%99%94%EB%A9%B4.PNG)

* * *