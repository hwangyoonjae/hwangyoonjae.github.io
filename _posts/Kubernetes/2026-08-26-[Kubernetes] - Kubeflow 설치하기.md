---
layout: post
title: "Kubeflow 설치하기"
date: 2026-08-26
categories: [Kubernetes]
tags: [Kubernetes, Kubeflow]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

> Kubeflow 전체를 공식 Helm Chart 하나로 설치하는 방식은 아직 제공되지 않아 Dashboard + 인증 + Notebook에 필요한 최소 구성만 설치하고, 고객사에서는 GPU서버가 있어 그래픽카드 할당받은걸로 Notebook 테스트 진행했으니 필자의 테스트 환경에는 GPU 서버가 없어 기본 환경에서 진행하였습니다.
{: .prompt-tip}

## 1. Kubeflow 소스 준비하기 :
```bash
$ git clone \
    --branch 26.03.1 \
    https://github.com/kubeflow/community-distribution.git
$ cd community-distribution

# 확인
$ git describe --tags
```
![kubeflow 설치 파일 준비](/assets/img/post/kubernetes/kubeflow%20설치%20파일%20준비.png)

---

## 2. Kubeflow Kustomize 소스 컴포넌트별 분리하기 :

- 위 과정에서 다운받은 Kubeflow 공식 매니페스트 파일을 각 컴포넌트별 아래와 같이 YAML 형태로 분리합니다.

```bash
$ mkdir kubeflow-rendered

# 아래와 같이 분리 예정
kubeflow-rendered/
├── 01_namespace.yaml
├── 02_cert-manager.yaml
├── 03_istio.yaml
├── 04_oauth2-proxy.yaml
├── 05_dex.yaml
├── 06_kubeflow-istio-resources.yaml
├── 07_multitenancy.yaml
├── 08_dashboard.yaml
├── 09_notebook-controller.yaml
├── 10_jupyter-web-app.yaml
├── 11_pvcviewer.yaml
├── 12_profiles.yaml
├── 13_volumes-web-app.yaml
├── 14_tensorboard-controller.yaml
└── 15_tensorboard-web-app.yaml
```

---

### 2.1 kustomize 설치하기 :

- Kubeflow 공식 매니페스트는 기본적으로 Kustomize 기반이라 standalone kustomize가 있으면 작업하기 편합니다.

```bash
$ curl -LO \
  https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv5.8.1/kustomize_v5.8.1_linux_amd64.tar.gz

$ tar -xzf kustomize_v5.8.1_linux_amd64.tar.gz
$ mv kustomize /usr/local/bin/
$ chmod +x /usr/local/bin/kustomize

# 버전 확인
$ kustomize version
```
![kubeflow kustomize 명령어 설치](/assets/img/post/kubernetes/kubeflow%20kustomize%20명령어%20설치.png)

---

### 2.2 Namespace YAML 생성하기 :

```bash
$ kustomize build \
  common/kubeflow-namespace/base \
  > ./kubeflow-rendered/01_namespace.yaml
```

---

### 2.3 Istio YAML 생성하기 :

```bash
$ kustomize build common/istio/istio-install/overlays/oauth2-proxy \
  > ./kubeflow-rendered/02_istio.yaml
```

---

### 2.4 OAuth2 Proxy YAML 생성하기 :

```bash
$ kustomize build \
  common/oauth2-proxy/overlays/m2m-dex-only \
  > ./kubeflow-rendered/03_oauth2-proxy.yaml
```

> m2m-dex-only는 Kubeflow 기본 Dex 연동용 OAuth2 Proxy overlay다.
{: .prompt-info}

---

### 2.5 Dex YAML 생성하기 :

```bash
$ kustomize build \
  common/dex/overlays/oauth2-proxy \
  > ./kubeflow-rendered/04_dex.yaml
```

---

### 2.6 Kubeflow Istio Resources YAML 생성하기 :

```bash
$ kustomize build \
  common/istio/kubeflow-istio-resources/base \
  > ./kubeflow-rendered/05_kubeflow-istio-resources.yaml
```

---

### 2.7 Multi-tenancy YAML 생성하기 :

```bash
$ kustomize build \
  common/kubeflow-roles/base \
  > ./kubeflow-rendered/06_kubeflow-roles.yaml
```

---

### 2.8 Dashboard YAML 생성하기 :

```bash
$ kustomize build applications/dashboard/overlays/istio \
  > ./kubeflow-rendered/07_dashboard.yaml
```

---

### 2.9 Notebook Controller YAML 생성하기 :

```bash
$ kustomize build \
  applications/notebooks-v1/upstream/notebook-controller/overlays/kubeflow \
  > ./kubeflow-rendered/08_notebook-controller.yaml
```

---

### 2.10 Jupyter Web App YAML 생성하기 :

```bash
$ kustomize build \
  applications/notebooks-v1/upstream/jupyter-web-app/overlays/istio \
  > ./kubeflow-rendered/09_jupyter-web-app.yaml
```

---


### 2.11 PVC Viewer YAML 생성하기 :

```bash
$ kustomize build \
  applications/notebooks-v1/upstream/pvcviewer-controller/base \
  > ./kubeflow-rendered/10_pvcviewer.yaml
```

---

### 2.12 Volumes Web App YAML 생성하기 :

```bash
$ kustomize build \
  applications/notebooks-v1/upstream/volumes-web-app/overlays/istio \
  > ./kubeflow-rendered/11_volumes-web-app.yaml
```

---

### 2.13 TensorBoard Controller YAML 생성하기 :

```bash
$ kustomize build \
  applications/notebooks-v1/upstream/tensorboard-controller/overlays/kubeflow \
  > ./kubeflow-rendered/12_tensorboard-controller.yaml
```

---

### 2.14 TensorBoards Web App YAML 생성하기 :

```bash
$ kustomize build \
  applications/notebooks-v1/upstream/tensorboards-web-app/overlays/istio \
  > ./kubeflow-rendered/13_tensorboards-web-app.yaml
```

---

### 2.15 Profile YAML 생성하기 :

```bash
$ kustomize build \
  applications/dashboard/upstream/profile-controller/overlays/kubeflow \
  > ./kubeflow-rendered/14_profiles.yaml
```

---

## 3. Kubeflow 설치하기 :
### 3.1 Kubeflow Container Image 다운로드 :

```bash
$ docker pull quay.io/oauth2-proxy/oauth2-proxy:v7.15.2
$ docker pull ghcr.io/dexidp/dex:v2.45.1
$ docker pull ghcr.io/kubeflow/dashboard/dashboard:v2.0.0
$ docker pull ghcr.io/kubeflow/dashboard/poddefaults-webhook:v2.0.0
$ docker pull ghcr.io/kubeflow/dashboard/access-management:v2.0.0
$ docker pull ghcr.io/kubeflow/dashboard/profile-controller:v2.0.0
$ docker pull ghcr.io/kubeflow/notebooks/notebook-controller:v1.11.0
$ docker pull ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-scipy:v1.11.0
$ docker pull ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-pytorch-full:v1.11.0
$ docker pull ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-pytorch-cuda-full:v1.11.0
$ docker pull ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-tensorflow-full:v1.11.0
$ docker pull ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-tensorflow-cuda-full:v1.11.0
$ docker pull ghcr.io/kubeflow/notebooks/jupyter-web-app:v1.11.0
$ docker pull ghcr.io/kubeflow/notebooks/pvcviewer-controller:v1.11.0
$ docker pull quay.io/brancz/kube-rbac-proxy:v0.13.1
$ docker pull ghcr.io/kubeflow/notebooks/volumes-web-app:v1.11.0
$ docker pull ghcr.io/kubeflow/notebooks/tensorboard-controller:v1.11.0
$ docker pull quay.io/brancz/kube-rbac-proxy:v0.8.0
$ docker pull ghcr.io/kubeflow/dashboard/access-management:v2.0.0
$ docker pull ghcr.io/kubeflow/dashboard/profile-controller:v2.0.0
$ docker pull busybox:1.28
$ docker pull registry.istio.io/release/proxyv2:1.30.1
$ docker pull registry.istio.io/release/pilot:1.30.1
$ docker pull registry.istio.io/release/install-cni:1.30.1

$ docker tag quay.io/oauth2-proxy/oauth2-proxy:v7.15.2                                        harbor.test.com/oauth2-proxy/oauth2-proxy:v7.15.2
$ docker tag ghcr.io/dexidp/dex:v2.45.1                                                       harbor.test.com/dexidp/dex:v2.45.1
$ docker tag ghcr.io/kubeflow/dashboard/dashboard:v2.0.0                                      harbor.test.com/kubeflow/dashboard/dashboard:v2.0.0
$ docker tag ghcr.io/kubeflow/dashboard/poddefaults-webhook:v2.0.0                            harbor.test.com/kubeflow/dashboard/poddefaults-webhook:v2.0.0
$ docker tag ghcr.io/kubeflow/dashboard/access-management:v2.0.0                              harbor.test.com/kubeflow/dashboard/access-management:v2.0.0
$ docker tag ghcr.io/kubeflow/dashboard/profile-controller:v2.0.0                             harbor.test.com/kubeflow/dashboard/profile-controller:v2.0.0
$ docker tag ghcr.io/kubeflow/notebooks/notebook-controller:v1.11.0                           harbor.test.com/kubeflow/notebooks/notebook-controller:v1.11.0
$ docker tag ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-scipy:v1.11.0                 harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-scipy:v1.11.0
$ docker tag ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-pytorch-full:v1.11.0          harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-pytorch-full:v1.11.0
$ docker tag ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-pytorch-cuda-full:v1.11.0     harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-pytorch-cuda-full:v1.11.0
$ docker tag ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-tensorflow-full:v1.11.0       harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-tensorflow-full:v1.11.0
$ docker tag ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-tensorflow-cuda-full:v1.11.0  harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-tensorflow-cuda-full:v1.11.0
$ docker tag ghcr.io/kubeflow/notebooks/jupyter-web-app:v1.11.0                               harbor.test.com/kubeflow/notebooks/jupyter-web-app:v1.11.0
$ docker tag ghcr.io/kubeflow/notebooks/pvcviewer-controller:v1.11.0                          harbor.test.com/kubeflow/notebooks/pvcviewer-controller:v1.11.0
$ docker tag quay.io/brancz/kube-rbac-proxy:v0.13.1                                           harbor.test.com/brancz/kube-rbac-proxy:v0.13.1
$ docker tag ghcr.io/kubeflow/notebooks/volumes-web-app:v1.11.0                               harbor.test.com/kubeflow/notebooks/volumes-web-app:v1.11.0
$ docker tag ghcr.io/kubeflow/notebooks/tensorboard-controller:v1.11.0                        harbor.test.com/kubeflow/notebooks/tensorboard-controller:v1.11.0
$ docker tag quay.io/brancz/kube-rbac-proxy:v0.8.0                                            harbor.test.com/brancz/kube-rbac-proxy:v0.8.0
$ docker tag ghcr.io/kubeflow/dashboard/access-management:v2.0.0                              harbor.test.com/kubeflow/dashboard/access-management:v2.0.0
$ docker tag ghcr.io/kubeflow/dashboard/profile-controller:v2.0.0                             harbor.test.com/kubeflow/dashboard/profile-controller:v2.0.0
$ docker tag docker.io/library/busybox:1.28                                                   harbor.test.com/kubeflow/busybox:1.28
$ docker tag registry.istio.io/release/proxyv2:1.30.1                                         harbor.test.com/istio/proxyv2:1.30.1
$ docker tag registry.istio.io/release/pilot:1.30.1                                           harbor.test.com/istio/pilot:1.30.1
$ docker tag registry.istio.io/release/install-cni:1.30.1                                     harbor.test.com/istio/install-cni:1.30.1

$ docker push harbor.test.com/oauth2-proxy/oauth2-proxy:v7.15.2
$ docker push harbor.test.com/dexidp/dex:v2.45.1
$ docker push harbor.test.com/kubeflow/dashboard/dashboard:v2.0.0
$ docker push harbor.test.com/kubeflow/dashboard/poddefaults-webhook:v2.0.0
$ docker push harbor.test.com/kubeflow/dashboard/access-management:v2.0.0
$ docker push harbor.test.com/kubeflow/dashboard/profile-controller:v2.0.0
$ docker push harbor.test.com/kubeflow/notebooks/notebook-controller:v1.11.0
$ docker push harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-scipy:v1.11.0
$ docker push harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-pytorch-full:v1.11.0
$ docker push harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-pytorch-cuda-full:v1.11.0
$ docker push harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-tensorflow-full:v1.11.0
$ docker push harbor.test.com/kubeflow/kubeflow/notebook-servers/jupyter-tensorflow-cuda-full:v1.11.0
$ docker push harbor.test.com/kubeflow/notebooks/jupyter-web-app:v1.11.0
$ docker push harbor.test.com/kubeflow/notebooks/pvcviewer-controller:v1.11.0
$ docker push harbor.test.com/brancz/kube-rbac-proxy:v0.13.1
$ docker push harbor.test.com/kubeflow/notebooks/volumes-web-app:v1.11.0
$ docker push harbor.test.com/kubeflow/notebooks/tensorboard-controller:v1.11.0
$ docker push harbor.test.com/brancz/kube-rbac-proxy:v0.8.0
$ docker push harbor.test.com/kubeflow/dashboard/access-management:v2.0.0
$ docker push harbor.test.com/kubeflow/dashboard/profile-controller:v2.0.0
$ docker push harbor.test.com/kubeflow/busybox:1.28
$ docker push harbor.test.com/istio/proxyv2:1.30.1
$ docker push harbor.test.com/istio/pilot:1.30.1
$ docker push harbor.test.com/istio/install-cni:1.30.1
```

---

### 3.2 Kubeflow 컴포넌트별 설치하기 :

> 폐쇄망 환경에서 배포하는 경우 각 YAML의 지정된 이미지 경로를 harbor 주소로 변경합니다.
{: .prompt-warning}

- 위 과정에서 생성한 YAML 파일을 가지고 배포합니다.

```bash
# 배포
$ kubectl apply -f 01_namespace.yaml
$ kubectl apply -f 02_istio.yaml
$ kubectl apply -f 03_oauth2-proxy.yaml
$ kubectl apply -f 04_dex.yaml
$ kubectl apply -f 05_kubeflow-istio-resources.yaml
$ kubectl apply -f 06_kubeflow-roles.yaml
$ kubectl apply -f 07_dashboard.yaml
$ kubectl apply -f 08_notebook-controller.yaml
$ kubectl apply -f 09_jupyter-web-app.yaml
$ kubectl apply -f 10_pvcviewer.yaml
$ kubectl apply -f 11_volumes-web-app.yaml
$ kubectl apply -f 12_tensorboard-controller.yaml
$ kubectl apply -f 13_tensorboards-web-app.yaml
$ kubectl apply -f 14_profiles.yaml

# 상태 확인
$ kubectl get pod -n kubeflow
```

![kubeflow 설치 pod 목록 확인](/assets/img/post/kubernetes/kubeflow%20설치%20pod%20목록%20확인.png)

---