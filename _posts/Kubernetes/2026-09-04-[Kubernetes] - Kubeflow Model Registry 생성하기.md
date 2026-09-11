---
layout: post
title: "Kubeflow Model Registry 생성하기"
date: 2026-09-04
categories: [Kubernetes]
tags: [MLOps, Kubeflow]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
mermaid: true
---

> Kubeflow Dashboard를 구축하여 MLOps 기능을 사용하고자 하나, ```Model Registry``` 메뉴 페이지에 대한 라우팅이 구성되어 있지 않아 추가 구성이 필요합니다.
{: .prompt-warning}

![kubeflow dashboard model registry 라우팅 미설정](/assets/img/post/kubernetes/kubeflow%20dashboard%20model%20registry%20라우팅%20미설정.png)

---

## 1. Model Resgitry는 무엇인가? :

- 학습이 끝난 모델을 등록하고, 버전과 메타데이터를 관리하는 모델 카탈로그입니다.

> Kubeflow Dashboard의 Model Registry는 학습 완료된 AI 모델을 이름, 버전, 저장 위치, 메타데이터와 함께 중앙에서 관리하는 모델 관리 시스템이라고 이해하면됩니다.
{: .prompt-tip}

---

## 2. Model Registry 설치하기 :

- 이전에 구축했던 Kubeflow 소스 구조를 그대로 사용하여 설치합니다.

---

### 2.1 namespace를 kubeflow-user로 변경하기 :

- 공식 overlay는 기본 예제 Profile namespace로 kubeflow-user-example-com을 사용하므로 현재 Kubeflow Profile 생성된 값으로 변경합니다.

```bash
$ cd /root/kubeflow/community-distribution

$ vi applications/hub/overlays/model-registry/kustomization.yaml
```
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: kubeflow-user # 해당부분 수정

resources:
- ../../upstream/overlays/postgres
- ../../upstream/options/istio
- ../../upstream/options/ui/overlays/istio
```

![namespace를 kubeflow-user로 변경](/assets/img/post/kubernetes/namespace를%20kubeflow-user로%20변경.png)

---

### 2.2 Model Registry YAML 파일 생성하기 :

- Model Registry YAML을 생성합니다.

```bash
# Knative service YAML 생성 
$ kustomize build applications/hub/overlays/model-registry \
  > ./kubeflow-rendered/21_model_registry.yaml
```

---

### 2.3 Model Registry Container Image 다운로드 :

- 위에서 생성한 Model Registry YAML을 통해 이미지 확인 후 다운받습니다.

```bash
docker pull postgres:16-alpine
docker pull ghcr.io/kubeflow/hub/server:v0.3.9
docker pull ghcr.io/kubeflow/hub/ui:v0.3.9

docker tag postgres:16-alpine                 harbor.test.com/kubeflow/postgres:16-alpine
docker tag ghcr.io/kubeflow/hub/server:v0.3.9 harbor.test.com/kubeflow/hub/server:v0.3.9
docker tag ghcr.io/kubeflow/hub/ui:v0.3.9     harbor.test.com/kubeflow/hub/ui:v0.3.9

docker push harbor.test.com/kubeflow/postgres:16-alpine
docker push harbor.test.com/kubeflow/hub/server:v0.3.9
docker push harbor.test.com/kubeflow/hub/ui:v0.3.9
```

---

### 2.4 Model Registry 생성하기 :

> 폐쇄망 환경에서 배포하는 경우 각 YAML의 지정된 이미지 경로를 harbor 주소로 변경하고, storageclass가 존재하는 경우 pvc 생성에 추가합니다.
{: .prompt-warning}

- 생성 전, Service FQDN를 기존에 생성한 네임스페이스로 수정합니다.

```bash
$ vi 21_model_registry.yaml

# 아래와 같이 vi 편집기에서 실행합니다.
:%s#kubeflow-user.example.com#kubeflow-user#g
```

---

- 위 과정에서 생성한 YAML 파일을 가지고 배포합니다.

```bash
$ kubectl apply -f 21_model_registry.yaml

# 상태 확인
$ kubectl get pod -n kubeflow-user
```

![Model Registry 생성 완료](/assets/img/post/kubernetes/Model%20Registry%20생성%20완료.png)

---

### 2.5 Model Registry 확인하기 :

- 위와 같이 정상 생성 후, Kubeflow Dashboard에 접속하여 Model Registry 페이지가 정상적으로 뜨는지 확인합니다.

![Model Registry 정상 확인](/assets/img/post/kubernetes/Model%20Registry%20정상%20확인.png)

---