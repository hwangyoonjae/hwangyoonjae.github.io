---
layout: post
title: "Nginx config 정리"
date: 2022-09-10
categories: Nginx
tags: [Web, Nginx, config]
image: /assets/img/post-title/nginx-wallpaper.jpg
---

## Configuration File 구조 분석하기:
- Nginx는 여러 모듈들로 구성되며, 이러한 모듈들은 configuration파일에 있는 directives에 의해 제어된다.
- Nginx의 기본 설정 파일은 **nginx.conf**이고, 초기 설치 시 설정 값은 아래와 같이 구성된다.<br>
[![텍스트](/assets/img/post/Linux/nginx.conf%20%EC%B4%88%EA%B8%B0%EC%84%A4%EC%B9%98.PNG)](/assets/img/post/Linux/nginx.conf%20%EC%B4%88%EA%B8%B0%EC%84%A4%EC%B9%98.PNG)

### Core 모듈 설정:
- 설정 파일 최상단에 위치하며 nginx의 프로세스 관리, 보안과 같은 기본적인 동작 방식을 정의한다.

  - **user** : nginx의 worker process가 실행되는 user권한이다.
  ```bash
  #사용문법
  user [user];
  ```

  - **worker_processes** : worker process의 수를 의미한다.
  ```bash
  #사용문법
  worker_processes [number] or [auto];
  ```

  - **error_log** : 로그파일 경로와 남기고자 하는 심각도 레벨이다.

  - **pid** : nginx main process(=master process)의 pid 저장 파일경로이다.

### event 블록:
- 네트워크의 동작방법과 관련된 설정값을 가진다.

  - **worker_connections** : 한개의 worker process가 동시에 오픈할 수 있는 최대 연결 갯수이며, client 와의 연결만 포함하는게 아니고, 모든 connection을 포함한다.
  ```bash
  #사용문법
  worker_connections [number];
  ```

### http 블록:
- server, location의 루트 블록이며, http블록 안에 적어도 하나의 server 블록을 선언할 수 있고, server 블록 안에서 한 개 이상의 location 블럭을 삽입할 수 있다.

  - **include** : http 블록에 가져올 context 파일 경로이다.

  - **default_type** : mimetype 중에 기본값으로 사용할 값을 설정한다.
  
  - **log_format** : 로그포멧 이름과 형식을 지정하고, 가상호스트 설정시 로그파일 뒤에 로그포멧 이름을 지정하면 해당 포맷대로 로그가 쌓인다.

  - **sendfile** : on으로 설정시 read/write시 하드디스크 io를 일으키지 않고 커널 내부에서 파일을 복사하여 속도 향상된다.

  - **keepalive_timeout** : 서버에 접속시 클라이언트와 커넥션을 열린채로 유지하는 시간이다.

  - **access_log** : access 로그를 저장할 파일을 지정한다.
  ```bash
  #사용문법
  access_log logs/access.log [context];
  ```

### server 블록:
- 한 서버(한 개의 ip)에서 각각의 도메인에 따라서 서로 다른 페이지를 서비스한다.
  ```bash
  #사용문법
  server {
      server_name  minimilab-zzang.com;
      root /var/www/minimilab-zzang.com
  }

  server {
      server_name  minimilab.com
      root /var/www/minimilab.com
  }
  ```

#### server / location 블록:
- location directive를 이용하여 nginx는 요청 URI에 따라 다른 서버로 트래픽을 전송할 수 있다.
  ```bash
  #사용문법
  location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
  ```
- location / 의 요청이 오면 /usr/share/nginx/html에서 정적인(static)페이지를 보여준다.

* * *