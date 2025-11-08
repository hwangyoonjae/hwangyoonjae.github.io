---
layout: post
title: "Chaos Mesh 설치"
date: 2024-07-10
categories: [인프라, Chaos-Mesh]
tags: [chaos-mesh]
image: /assets/img/post-title/chaos-mesh-wallpaper.jpg
---


## 1. Helm 설치하기 :
- Chaos Mesh를 설치하기 위해 Helm 설치가 되어있어야한다.

```bash
# helm 버전 확인
$ helm version
```

- 아래와 같이 출력된다.

```bash
version.BuildInfo{Version:"v3.15.1", GitCommit:"e211f2aa62992bd72586b395de50979e31231829", GitTreeState:"clean", GoVersion:"go1.22.3"}
```

* * *

## 2. Chaos Mesh 설치하기 :
- 외부 네트워크 연결이 있는 컴퓨터에서 모든 Chaos Mesh 이미지와 저장소 압축 패키지를 다운로드해야한다.

```bash
# 버전 지정
$ export CHAOS_MESH_VERSION=v2.6.3

# 이미지 다운로드
$ docker pull ghcr.io/chaos-mesh/chaos-mesh:${CHAOS_MESH_VERSION}  
$ docker pull ghcr.io/chaos-mesh/chaos-daemon:${CHAOS_MESH_VERSION}  
$ docker pull ghcr.io/chaos-mesh/chaos-dashboard:${CHAOS_MESH_VERSION}

# 이미지 파일 형태로 저장
$ docker save ghcr.io/chaos-mesh/chaos-mesh:${CHAOS_MESH_VERSION} > image-chaos-mesh.tar  
$ docker save ghcr.io/chaos-mesh/chaos-daemon:${CHAOS_MESH_VERSION} > image-chaos-daemon.tar  
$ docker save ghcr.io/chaos-mesh/chaos-dashboard:${CHAOS_MESH_VERSION} > image-chaos-dashboard.tar

# chaos mesh zip 패키지 다운로드
$ curl -fsSL -o chaos-mesh.zip https://github.com/chaos-mesh/chaos-mesh/archive/refs/tags/v2.6.3.zip
```

- Private Registry의 이미지 push한다.

```bash
# 버전 지정 및 docker registry 주소 지정
$ export CHAOS_MESH_VERSION=v2.6.3; export DOCKER_REGISTRY=harbor.com

# 이미지 tag 지정
$ export CHAOS_MESH_IMAGE=$DOCKER_REGISTRY/chaos-mesh/chaos-mesh:${CHAOS_MESH_VERSION}  
$ export CHAOS_DAEMON_IMAGE=$DOCKER_REGISTRY/chaos-mesh/chaos-daemon:${CHAOS_MESH_VERSION}  
$ export CHAOS_DASHBOARD_IMAGE=$DOCKER_REGISTRY/chaos-mesh/chaos-dashboard:${CHAOS_MESH_VERSION}  
$ docker image tag ghcr.io/chaos-mesh/chaos-mesh:${CHAOS_MESH_VERSION} $CHAOS_MESH_IMAGE  
$ docker image tag ghcr.io/chaos-mesh/chaos-daemon:${CHAOS_MESH_VERSION} $CHAOS_DAEMON_IMAGE  
$ docker image tag ghcr.io/chaos-mesh/chaos-dashboard:${CHAOS_MESH_VERSION} $CHAOS_DASHBOARD_IMAGE

# docker registry의 image push
$ docker push $CHAOS_MESH_IMAGE  
$ docker push $CHAOS_DAEMON_IMAGE  
$ docker push $CHAOS_DASHBOARD_IMAGE
```

- Chaos Mesh zip 패키지 풀고, 설치를 진행한다.

```bash
# chaos mesh zip 패키지 풀기
$ unzip chaos-mesh.zip -d chaos-mesh && cd chaos-mesh

# chaos mesh namespace 생성
$ kubectl create ns chaos-mesh

# helm 통해서 설치 진행
$ helm install chaos-mesh helm/chaos-mesh -n=chaos-mesh --set images.registry=$DOCKER_REGISTRY
```

* * *

## 3. Chaos Mesh Dashboard 접속하기 :
- dashboard를 ingress 통해서 접속하기 위해 설정한다.

```yaml
# chaos-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-chaos-dashboard-under-subpath
  namespace: chaos-mesh
  annotations:
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/configuration-snippet: |
      sub_filter '<head>' '<head> <base href="/chaos-mesh/">';
spec:
  ingressClassName: nginx
  rules:
  - host: chaos-mesh.com
    http:
      paths:
      - backend:
          service:
            name: chaos-dashboard
            port:
              number: 2333
        path: /chaos-mesh/?(.*)
        pathType: Prefix
```

- 위와 같이 작성 후 ingress를 생성한다.

```bash
$ kubectl create -f chaos-ingress.yaml
```

- 설치 후 dashboard에 접속 후 **Click here to generate**를 클릭한다.

[![Token 입력 요청 화면](/assets/img/post/chaos-mesh/Token%20입력%20요청%20화면.png)](/assets/img/post/chaos-mesh/Token%20입력%20요청%20화면.png)

- Token 생성하는 방법 참고하여 RBAC권한을 생성

[![액세스 요청 화면](/assets/img/post/chaos-mesh/액세스%20요청%20화면.png)](/assets/img/post/chaos-mesh/액세스%20요청%20화면.png)

>RBAC란?
>
>사용자에게 리소스에 대한 액세스 권한 부여 여부를 결정하기 위해 역할과 권한을 정의하는 액세스 제어 메커니즘으로 역할은 사용자의 위치, 부서, 연공서열 또는 직무와 같은 특성을 기반으로 정의됩니다. 권한은 액세스(사용자가 볼 수 있는 것), 작업(사용자가 수행할 수 있는 것) 및 세션(사용자가 작업을 수행할 수 있는 시간)에 따라 할당된다.
{: .prompt-tip }

- 위 과정 진행 후 아래와 같이 Dashboard가 보인다.

[![대시보드 화면](/assets/img/post/chaos-mesh/대시보드%20화면.png)](/assets/img/post/chaos-mesh/대시보드%20화면.png)

* * *

## 4. 전체 Namespace를 관리하고 싶은 경우 :
- 특정 namespace가 아닌 전체 관리를 하고 싶은경우에는 admin token을 발급받는다.

```yaml
# serviceaccount.yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  namespace: chaos-mesh
  name: chaos-mesh-admin
```

```yaml
# clusterrole.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: chaos-mesh
  name: chaos-mesh-admin-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["chaos-mesh.org"]
  resources: [ "*" ]
  verbs: ["*"]
```

```yaml
### clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: chaos-mesh-admin-binding
subjects:
- kind: ServiceAccount
  name: chaos-mesh-admin
  namespace: chaos-mesh
roleRef:
  kind: ClusterRole
  name: chaos-mesh-admin-role
  apiGroup: rbac.authorization.k8s.io
```

* * *