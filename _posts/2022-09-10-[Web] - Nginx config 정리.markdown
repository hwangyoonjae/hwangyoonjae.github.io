---
title: "[Web] - Nginx config 정리"
layout: post
date: 2022-09-10
image: /assets/images/Post/nginx.png
headerImage: true
tag:
- WEB
- Nginx
- conf
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Configuration File 구조 분석하기
- Nginx는 여러 모듈들로 구성되며, 이러한 모듈들은 configuration파일에 있는 directives에 의해 제어된다.
- Nginx의 기본 설정 파일은 **nginx.conf**이고, 초기 설치 시 설정 값은 아래와 같이 구성된다.<br>
[![텍스트](/assets/images/Linux/nginx.conf%20%EC%B4%88%EA%B8%B0%EC%84%A4%EC%B9%98.PNG)](/assets/images/Linux/nginx.conf%20%EC%B4%88%EA%B8%B0%EC%84%A4%EC%B9%98.PNG)

- **user** : nginx의 worker process가 실행되는 user권한이다.

- **worker_processes** : worker process의 수를 의미한다.

- **error_log** : 로그파일 경로와 남기고자 하는 심각도 레벨이다.

- **pid** : nginx main process(=master process)의 pid 저장 파일경로이다.

- **events**
  - **worker_connections** : 한개의 worker process가 동시에 오픈할 수 있는 최대 연결 갯수이며, client 와의 연결만 포함하는게 아니고, 모든 connection을 포함한다.

- **http**
  - **include** : http 블록에 가져올 context 파일 경로이다.

  - **default_type** : mimetype 중에 기본값으로 사용할 값을 설정한다.
  
  - **log_format** : 로그포멧 이름과 형식을 지정하고, 가상호스트 설정시 로그파일 뒤에 로그포멧 이름을 지정하면 해당 포멧대로 로그가 쌓인다.

  - **sendfile** : on으로 설정시 read/write시 하드디스크 io를 일으키지 않고 커널 내부에서 파일을 복사하여 속도 향상된다.

  - **keepalive_timeout** : 서버에 접속시 클라이언트와 커넥션을 열린채로 유지하는 시간이다.

* * *