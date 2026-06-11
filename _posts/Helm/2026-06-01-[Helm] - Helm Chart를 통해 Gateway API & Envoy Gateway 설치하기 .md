---
layout: post
title: "Helm Chart를 통해 Gateway API & Envoy Gateway 설치하기"
date: 2026-06-01
categories: [컨테이너, Helm]
tags: [Helm, Ingress, Gateway]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. Envoy Gateway란? :
- Kubernetes 클러스터를 웹 브라우저에서 쉽게 관리할 수 있도록 제공하는 Kubernetes 전용 Web UI 대시보드입니다.

* * *

## 2. Envoy Gateway 설치하기 :
### 2.1 Envovy Gateway Helm Chart 다운로드 :

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull oci://docker.io/envoyproxy/gateway-helm \
  --version v1.8.0

# 압축 파일 풀기
$ tar -zxvf gateway-helm-v1.8.0.tgz
```

* * *

### 2.2 Envovy Gateway Container Image 다운로드 : 

```bash
$ docker pull docker.io/envoyproxy/gateway:v1.8.0
$ docker pull docker.io/envoyproxy/ratelimit:ff287602
$ docker pull docker.io/envoyproxy/envoy:v1.36.0

$ docker tag docker.io/envoyproxy/gateway:v1.8.0      harbor.test.com/envoyproxy/gateway:v1.8.0
$ docker tag docker.io/envoyproxy/ratelimit:ff287602  harbor.test.com/envoyproxy/ratelimit:ff287602
$ docker tag docker.io/envoyproxy/envoy:v1.36.0       harbor.test.com/envoyproxy/envoy:v1.36.0

$ docker push harbor.test.com/envoyproxy/gateway:v1.8.0
$ docker push harbor.test.com/envoyproxy/ratelimit:ff287602
$ docker push harbor.test.com/envoyproxy/envoy:v1.36.0
```

* * *

### 2.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
global:
  images:
    envoyGateway:
      image: harbor.test.com/envoyproxy/gateway:v1.8.0
      pullPolicy: IfNotPresent
      pullSecrets: []
    ratelimit:
      image: "harbor.test.com/envoyproxy/ratelimit:ff287602"
      pullPolicy: IfNotPresent
      pullSecrets: []
    envoyProxy:
      image: "harbor.test.com/envoyproxy/envoy:v1.36.0"
      pullPolicy: IfNotPresent
      pullSecrets: []
```

* * *

### 2.4 Envoy Gateway Helm Chart 설치하기 :

```bash
# 네임스페이스 생성
$ kubectl create namespace envoy-gateway

# 설치 진행
$ helm upgrade --install envoy-gateway . -n envoy-gateway
```

![Envoy Gateway Helm Chart 설치 완료 화면](/assets/img/post/helm/Envoy%20Gateway%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n envoy-gateway
```

![Envoy Gateway helm chart 배포 후 확인](/assets/img/post/helm/Envoy%20Gateway%20helm%20chart%20배포%20후%20확인.png)

* * *

## 3. GatewayClass 및 Gateway 생성하기 :
### 3.1 GatewayClass 생성하기 : 

- Envoy Gateway Helm Chart를 설치하면 Envoy Gateway Controller는 정상적으로 배포되지만, GatewayClass 리소스는 자동으로 생성되지 않을 수 있어 Gateway API를 사용하기 위해서는 GatewayClass를 직접 생성해야 합니다.

```yaml
apiVersion: gateway.networking.k8s.io/v1 
kind: GatewayClass 
metadata: 
  name: envoy-gateway
  namespace: envoy-gateway
spec: 
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```

```yaml
# Envoy Proxy Service를 NodePort로 고정한 경우
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy-gateway
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
  parametersRef:
    group: gateway.envoyproxy.io
    kind: EnvoyProxy
    name: envoy-proxy-config
    namespace: envoy-gateway
```

* * *

- GatewayClass를 생성합니다.

```bash
$ kubectl apply -f envoy-gatewayclass.yaml
```

* * *

- 정상적으로 생성되었는지 확인합니다.

```bash
$ kubectl get gatewayclass
```

![Evovy gateway GatewayClass 생성 화면](/assets/img/post/helm/Evovy%20gateway%20GatewayClass%20생성%20화면.png)

* * *

### 3.2 TLS Secret 생성하기 :

- HTTPS Listener를 사용하기 위해서는 인증서 Secret이 필요하며, Secret은 Gateway가 생성되는 네임스페이스에 존재해야 합니다.

```bash
$ kubectl create secret tls wildcard-tls \
--cert=fullchain.crt \ 
--key=server.key \
-n envoy-gateway
```

* * *

- 정상적으로 생성되었는지 확인합니다.

```bash
$ kubectl get secret -n envoy-gateway
```

* * *

### 3.3 (선택) EnvoyProxy 생성하기 :

> EnvoyProxy 생성 시점
>
> Envoy Gateway는 Gateway 리소스를 생성하면 실제 트래픽을 처리하는 Envoy Proxy Deployment와 Service를 자동으로 생성합니다.
> 
> 기본 설정으로 생성되는 Envoy Proxy Service는 LoadBalancer 타입이거나 임의의 NodePort를 사용할 수 있지만, HAProxy에서 특정 NodePort로 트래픽을 전달하도록 구성한 경우, Envoy Proxy Service의 NodePort를 고정해야 합니다.
> 
> 따라서 Envoy Proxy Service를 커스터마이징하기 위한 EnvoyProxy 리소스는 Gateway 리소스를 생성하기 전에 먼저 생성하는 것이 좋습니다.
{: .prompt-warning}

```yaml
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: EnvoyProxy
metadata:
  name: envoy-proxy-config
  namespace: envoy-gateway
spec:
  provider:
    type: Kubernetes
    kubernetes:
      envoyService:
        type: NodePort
        patch:
          type: StrategicMerge
          value:
            spec:
              ports:
                - name: http-80
                  port: 1080
                  nodePort: 31080
                  protocol: TCP
                - name: https-443
                  port: 1443
                  nodePort: 31443
                  protocol: TCP
```

* * *

- EnvoyProxy를 생성합니다.

```bash
$ kubectl apply -f envoyproxy.yaml
```

* * *

- 정상적으로 생성되었는지 확인합니다.

```bash
$ kubectl get EnvoyProxy -n envoy-gateway
```

![Evovy gateway EnvoyProxy 생성 화면](/assets/img/post/helm/Evovy%20gateway%20EnvoyProxy%20생성%20화면.png)

* * *

### 3.4 HTTP/HTTPS Gateway 생성하기 :

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: envoy-gateway
  namespace: envoy-gateway
spec:
  gatewayClassName: envoy-gateway
  listeners:
    - name: http
      protocol: HTTP
      port: 1080
      allowedRoutes:
        namespaces:
          from: All
    - name: https
      protocol: HTTPS
      port: 1443
      tls:
        mode: Terminate
        certificateRefs:
          - kind: Secret
            name: wildcard-tls
      allowedRoutes:
        namespaces:
          from: All
```

* * *

- envoy-gateway를 생성합니다.

```bash
$ kubectl apply -f envoy-gateway.yaml
```

* * *

- 정상적으로 생성되었는지 확인합니다.

```bash
$ kubectl get Gateway -n envoy-gateway
```

* * *

### 3.5 HTTPRoute 생성하기 :

```yaml
# 서비스별 HTTPRoute 생성
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: argocd-route
  namespace: argocd
spec:
  parentRefs:
    - name: envoy-gateway
      namespace: envoy-gateway
  hostnames:
    - argocd.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: argocd-server
          port: 443
```

```yaml
# HTTP → HTTPS 리다이렉트가 필요한 경우
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: argocd-http-redirect
  namespace: argocd
spec:
  parentRefs:
    - name: envoy-gateway
      namespace: envoy-gateway
      sectionName: http
  hostnames:
    - argocd.example.com
  rules:
    - filters:
        - type: RequestRedirect
          requestRedirect:
            scheme: https
            statusCode: 301
```

* * *

- httproute를 생성합니다.

```bash
$ kubectl apply -f httproute-argocd.yaml
```

* * *

- 정상적으로 생성되었는지 확인합니다.

```bash
$ kubectl get httproute -n argocd
```

![Evovy gateway httproute 생성 화면](/assets/img/post/helm/Evovy%20gateway%20httproute%20생성%20화면.png)

* * *