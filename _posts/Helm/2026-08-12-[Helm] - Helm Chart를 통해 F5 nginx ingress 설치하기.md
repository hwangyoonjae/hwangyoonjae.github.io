---
layout: post
title: "Helm Chart를 통해 F5 nginx ingress 설치하기"
date: 2026-08-12
categories: [컨테이너, Helm]
tags: [Helm, Ingress, Gateway]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. Ingress Nginx -> F5 NGINX Ingress Controller로 마이그레이션 :

> Kubernetes Community에서 관리하던 Ingress NGINX는 2026년 3월 24일부로 공식 지원이 종료되어, 이후 신규 버전 및 보안 패치가 제공되지 않습니다.
> 
> 이에 따라 운영 환경에서는 보안 및 향후 Kubernetes 버전 호환성을 위해 지속적으로 유지보수되는 Ingress Controller로의 전환이 필요합니다.
> 
> 이번 구성에서는 기존 Ingress 구조를 최대한 유지할 수 있는 F5 NGINX Ingress Controller를 대안으로 설치하여 전환하고자 합니다.
{: .promt-info}

* * *

## 2. F5 NGINX Ingress Controller 설치하기 :
### 2.1 F5 NGINX Ingress Controller Helm Chart 다운로드 :

```bash
$ helm repo add nginx-stable https://helm.nginx.com/stable
$ helm repo update
$ helm search repo nginx-stable/nginx-ingress --versions | head
```

![F5 Nginx helm chart 목록 조회](/assets/img/post/helm/F5%20Nginx%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull nginx-stable/nginx-ingress \
  --version 2.6.4

# 압축 파일 풀기
$ tar -zxvf nginx-ingress-2.6.4.tgz
```

* * *

### 2.2 F5 NGINX Ingress Container Image 다운로드 : 

```bash
$ docker pull nginx/nginx-ingress:5.5.4

$ docker tag docker.io/nginx/nginx-ingress:5.5.4     harbor.test.com/nginx/nginx-ingress:5.5.4

$ docker push harbor.test.com/nginx/nginx-ingress:5.5.4
```

* * *

### 2.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
controller:
  ingressClass:
    name: f5-nginx
    create: true
    setAsDefaultIngress: false

  service:
    type: NodePort
    externalTrafficPolicy: Local

    httpPort:
      enable: true
      port: 80
      targetPort: 80
      nodePort: 30080 # 없으면 추가
      name: "http"

    httpsPort:
      enable: true
      port: 443
      targetPort: 443
      nodePort: 30443 # 없으면 추가
      name: "https"

  enableCustomResources: true

  image:
    repository: harbor.test.com/nginx/nginx-ingress
```

* * *

### 2.4 F5 NGINX Ingress Helm Chart 설치하기 :

```bash
# 네임스페이스 생성
$ kubectl create namespace nginx-ingress

# 설치 진행
$ helm upgrade --install nginx-ingress . -n nginx-ingress
```

![F5 NGINX Ingress Helm Chart 설치 완료 화면](/assets/img/post/helm//F5%20NGINX%20Ingress%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n nginx-ingress
```

![F5 NGINX Ingress helm chart 배포 후 확인](/assets/img/post/helm/F5%20NGINX%20Ingress%20helm%20chart%20배포%20후%20확인.png)

* * *

### 3. 애플리케이션 배포 테스트하기 :

- 테스트를 위해 별도 Namespace를 하나 생성합니다.

```bash
$ kubectl create namespace f5-test
```

* * *

- traefik이 정말 Ingress Controller 역할을 하는지 확인하기 위해 deployment,service, ingress를 생성합니다.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  namespace: f5-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-test
  template:
    metadata:
      labels:
        app: nginx-test
    spec:
      containers:
        - name: nginx
          image: harbor.test.com/nginx/nginx:1.25.3
          ports:
            - containerPort: 80
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-test
  namespace: f5-test
spec:
  selector:
    app: nginx-test
  ports:
    - port: 80
      targetPort: 80
````

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-test
  namespace: f5-test
spec:
  ingressClassName: f5-nginx
  rules:
    - host: f5nginx.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-test
                port:
                  number: 80
```

* * *

- 생성 후, 배포합니다.

```bash
$ kubectl apply -f deployment.yaml
$ kubectl apply -f service.yaml
$ kubectl apply -f ingress.yaml
```

* * *

- 배포 후 Ingress의 설정한 host 주소를 브라우저를 통해 접속합니다.

![F5 Nginx Ingress 적용 후 애플리케이션 접속 화면](/assets/img/post/helm/f5%20ngnix%20Ingress%20적용%20후%20애플리케이션%20접속%20화면.png)

* * *