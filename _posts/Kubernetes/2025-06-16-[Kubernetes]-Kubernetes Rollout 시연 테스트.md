---
layout: post
title: "Kubernetes Rollout 시연 테스트"
date: 2025-06-16
categories: [컨테이너, Kubernetes] 
tags: [Kubernetes, Deployment, Rollout]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. 커스터마이징된 Nginx 이미지 만들기 :
### 1.1 ***custom-v1*** Nginx 이미지 생성하기 :
- Dockerfile 작성하기

```bash
# Dockerfile
FROM nginx:1.21
RUN echo "<h1>Welcome to nginx 1.21</h1>" > /usr/share/nginx/html/index.html
```

* * *

- 이미지 빌드 및 푸시

```bash
# 이미지 빌드 및 푸시 (Docker Hub 기준)
$ docker build -t <Harbor주소>/<프로젝트명>/nginx:custom-v1 .
$ docker push <Harbor주소>/<프로젝트명>/nginx:custom-v1
```
![nginx custom v1 이미지 빌드](/assets/img/post/kubernetes/nginx%20custom%20v1%20이미지%20빌드.png)

* * *

### 1.2 ***custom-v2*** Nginx 이미지 생성하기 :
- Dockerfile 작성하기

```bash
# Dockerfile
FROM nginx:1.21
RUN echo "<h1>Welcome to nginx 1.21 - V2</h1>" > /usr/share/nginx/html/index.html
```

* * *

- 이미지 빌드 및 푸시하기

```bash
# 이미지 빌드 및 푸시 (Docker Hub 기준)
$ docker build -t <Harbor주소>/<프로젝트명>/nginx:custom-v2 .
$ docker push <Harbor주소>/<프로젝트명>/nginx:custom-v2
```
![nginx custom v2 이미지 빌드](/assets/img/post/kubernetes/nginx%20custom%20v2%20이미지%20빌드.png)

* * *

## 2. Deployment 생성 및 배포하기 :
### 2.1 ***custom-v1*** deployment 배포하기 :

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-app
        image: <Harbor주소>/<프로젝트명>/nginx:custom-v1
        ports:
        - containerPort: 80
```

```bash
$ kubectl apply -f deployment.yaml
# 서비스 생성
$ kubectl expose deployment <서비스명> --type=NodePort --port=80
```

![nginx custom service 생성](/assets/img/post/kubernetes/nginx%20custom%20service%20생성.png)

* * *

## 3. 버전 업데이트 :

- 기존에 사용했던 이미지를 변경하여 업데이트한다.

```bash
$ kubectl set image deployment/<deployment명> demo-app=<Harbor주소>/<프로젝트명>/nginx:custom-v2
$ kubectl rollout status deployment demo-app
```

![nginx custom image 업데이트](/assets/img/post/kubernetes/nginx%20custom%20image%20업데이트.png)

* * *

## 4. 확인 테스트 :

```bash
# NodePort 확인
kubectl get svc <service명>

# 브라우저나 curl로 접근
curl http://<NodeIP>:<NodePort>
```

![nginx custom service 통신 확인](/assets/img/post/kubernetes/nginx%20custom%20service%20통신%20확인.png)

* * *

## 5. 롤백 테스트 :

```bash
$ kubectl rollout undo deployment <deployment명>
$ kubectl rollout status deployment <deployment명>
```

![nginx custom service 롤백](/assets/img/post/kubernetes/nginx%20custom%20service%20롤백.png)

* * *