---
layout: post
title: "Helm Chart를 통해 Traefik Ingress 설치하기"
date: 2026-08-11
categories: [컨테이너, Helm]
tags: [Helm, Ingress, Gateway]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. Traefik Ingress란? :

- Kubernetes의 Ingress 리소스를 감시하면서 외부에서 등러오는 HTTP/HTTPS 요청을 라우팅 규칙에 따라 적절한 Service(Pod)로 전달해주는 Ingress Controller입니다.

* * *

## 2. Traefik Ingress 설치하기 :
### 2.1 Traefik Ingress Helm Chart 다운로드 :

```bash
$ helm repo add traefik https://traefik.github.io/charts
$ helm repo update
$ helm search repo traefik/traefik --versions | head
```

![traefik helm chart 목록 조회](/assets/img/post/helm/traefik%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull traefik/traefik \
  --version 41.2.0

# 압축 파일 풀기
$ tar -zxvf traefik-41.2.0.tgz
```

* * *

### 2.2 Traefik Ingress Container Image 다운로드 : 

```bash
$ docker pull docker.io/traefik:v3.7.10

$ docker tag docker.io/library/traefik:v3.7.10     harbor.test.com/traefik/traefik:v3.7.10

$ docker push harbor.test.com/traefik/traefik:v3.7.10
```

* * *

### 2.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
image:
  registry: harbor.test.com
  repository: traefik/traefik 
  tag: v3.7.10
  digest:
  pullPolicy: IfNotPresent

api:
  dashboard: true
  insecure: false

ingressRoute:
  dashboard:
    # Dashboard용 IngressRoute 생성
    enabled: true
    matchRule: PathPrefix(`/dashboard`) || PathPrefix(`/api`)
    entryPoints:
      - web
    middlewares:
      - name: traefik-dashboard-auth

extraObjects:
  - apiVersion: v1
    kind: Secret
    metadata:
      name: traefik-dashboard-auth-secret
    type: kubernetes.io/basic-auth
    stringData:
      username: admin
      password: "qwe1212!Q"

  - apiVersion: traefik.io/v1alpha1
    kind: Middleware
    metadata:
      name: traefik-dashboard-auth
    spec:
      basicAuth:
        secret: traefik-dashboard-auth-secret
```

* * *

### 2.4 Traefik Ingress Helm Chart 설치하기 :

```bash
# 네임스페이스 생성
$ kubectl create namespace traefik

# 설치 진행
$ helm upgrade --install traefik . -n traefik
```

![Traefik Ingress Helm Chart 설치 완료 화면](/assets/img/post/helm/Traefik%20Ingress%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n traefik
```

![Traefik Ingress helm chart 배포 후 확인](/assets/img/post/helm/Traefik%20Ingress%20helm%20chart%20배포%20후%20확인.png)

* * *

## 3. Treafik 대시보드 확인하기 : 
### 3.1 Dashboard IngressRoute 확인하기 :

```bash
$ kubectl get ingressroute -n traefik

# 상세확인
$ kubectl get ingressroute \
    -n traefik \
    traefik-dashboard \
    -o yaml
```

![Dashboard IngressRoute 확인](/assets/img/post/helm/Dashboard%20IngressRoute%20확인.png)

* * *

### 3.2 BasicAuth Middleware 확인하기 :

```bash
$ kubectl get middleware -n traefik

# 상세확인
$ kubectl get middleware \
    -n traefik \
    traefik-dashboard-auth \
    -o yaml
```

![BasicAuth Middleware 확인](/assets/img/post/helm/BasicAuth%20Middleware%20확인.png)

* * *

### 3.3 인증 Secret 확인하기 :

```bash
$ kubectl get secret \
    -n traefik \
    traefik-dashboard-auth-secret
```

![인증 시크릿 확인](/assets/img/post/helm/인증%20시크릿%20확인.png)

* * *

```bash
# 계정 확인
$ kubectl get secret \
    -n traefik \
    traefik-dashboard-auth-secret \
    -o jsonpath='{.data.username}' | base64 -d
  echo

# 패스워드 확인
$ kubectl get secret \
    -n traefik \
    traefik-dashboard-auth-secret \
    -o jsonpath='{.data.password}' | base64 -d
  echo
```

![인증 시크릿 계정 패스워드 확인](/assets/img/post/helm/인증%20시크릿%20계정%20패스워드%20확인.png)

* * *

### 3.4 Traefik Service 포트 확인하기 :

```bash
$ kubectl get svc -n traefik
```

![Traefik Service 포트 확인](/assets/img/post/helm/Traefik%20Service%20포트%20확인.png)

* * *

### 3.5 대시보드 접속하기 :

- Traefik Service가 NodePort를 통해 외부에 노출되어 있으므로, 다음과 같이 Kubernetes Node IP와 Traefik의 HTTP NodePort를 조합하여 접속합니다.

> Traefik Dashboard의 기본 접근 경로는 /dashboard/이며, URL 마지막의 /까지 포함하여 접속합니다.
>
> Ex. http://<NODE_IP>:<TRAEFIK_HTTP_NODEPORT>/dashboard/
{: .prompt-tip}

![Traefik 대시보드 초기화면](/assets/img/post/helm/Traefik%20대시보드%20초기화면.png)

* * *

- Basic Authentication 로그인 창이 뜨는데, values.yaml의 Secret에 설정한 계정 정보를 입력합니다.

```bash
Username : admin
Password : 설정한 비밀번호
```

![Traefik Dashboard 로그인 후 화면](/assets/img/post/helm/Traefik%20Dashboard%20로그인%20후%20화면.png)

* * *