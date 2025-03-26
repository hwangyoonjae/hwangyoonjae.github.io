---
layout: post
title: "[Gitlab] - Gitlab-Runner Harbor 이미지 저장하기"
date: 2024.07.12
categories: Gitlab
tags: [Git, Gitlab, Runner, CI, CD]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## 1. .gitlab.yaml 생성하기:
```yaml
# 빌드와 배포 단계 정의
stages:
  - build
  - deploy

# Harbor 관련 변수들 설정
variables:
  HARBOR_REGISTRY_URL: harbor.inno.com
  HARBOR_USER: admin
  HARBOR_PASSWORD: qwe1212!Q
  HARBOR_PROJECT: hyj-test

# 기본 설정으로 Docker를 실행할 수 있도록 설정
default:
  # 각 job 실행 전에 Docker 환경을 확인하기 위해 `docker info` 명령을 실행
  before_script:
    - docker info

build_image:
  stage: build
  script:
    - docker build -t my-docker-image .
    - echo $HARBOR_PASSWORD | docker login $HARBOR_REGISTRY_URL -u $HARBOR_USER --password-stdin
    - docker tag my-docker-image:latest $HARBOR_REGISTRY_URL/$HARBOR_PROJECT/my-docker-image:latest
    - docker push $HARBOR_REGISTRY_URL/$HARBOR_PROJECT/my-docker-image:latest
  after_script:
    - docker logout $HARBOR_REGISTRY_URL

# Harbor에 배포된 것을 환경으로 표시
deploy_to_harbor:
  stage: deploy
  script:
    - echo "Deployment to Harbor completed."
  environment:
    name: production
```

* * *

## 2. Dockerfile 생성하기 :
```bash
FROM nginx:latest 

WORKDIR /etc/nginx
RUN mv nginx.conf nginx.conf.orig
COPY nginx.conf nginx.conf
CMD ["nginx", "-g", "daemon off;"]

EXPOSE 80
```

* * *

## 3. Nginx.conf 생성하기 :
```config
# 기본적인 nginx 설정 파일 구조 예시
events {
    # 이벤트 관련 설정
    worker_connections  1024;
}

http {
    # HTTP 서버 관련 설정
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
  
    # 로그 포맷 등 다양한 설정 가능
    # 서버 블록 설정
    server {
        # 서버 설정 시작
        listen       80;  # 포트 설정
        server_name  localhost;  # 서버 도메인 설정
        location / {
            root   /usr/share/nginx/html;  # 루트 디렉토리 설정
            index  index.html index.htm;   # 인덱스 파일 설정
        }
        error_page   500 502 503 504  /50x.html;  # 에러 페이지 설정
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
        # 서버 설정 종료
    }
}
```

* * *

## 4. Index.html 생성하기 :
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello Nginx Docker</title>
</head>
<body>
    <h1>Hello Nginx Docker!</h1>
    <p>This is a simple HTML page served by nginx in a Docker container.</p>

</body>
</html>
```

* * *