---
layout: post
title: "Kubeflow Pipelines 생성하기"
date: 2026-09-11
categories: [Kubernetes]
tags: [MLOps, Kubeflow]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
mermaid: true
---

> Kubeflow Dashboard를 구축하여 MLOps 기능을 사용하고자 하나, ```Pipelines``` 메뉴 페이지에 대한 라우팅이 구성되어 있지 않아 추가 구성이 필요합니다.
{: .prompt-warning}

![kubeflow dashboard Pipelines 라우팅 미설정](/assets/img/post/kubernetes/kubeflow%20dashboard%20Pipelines%20라우팅%20미설정.png)

---

## 1. Pipelines는 무엇인가? :

- 머신러닝 작업 여러 단계를 하나의 워크플로우로 정의하고 Kubernetes 위에서 자동 실행하는 기능입니다.

> Kubeflow Dashboard의 Pipelines는 Pipeline을 DAG 형태로 정의하고 실행 순서, 입력·출력, 조건, 병렬 처리 등을 관리합니다.
{: .prompt-tip}

---

## 2. Pipelines 설치하기 :

- 이전에 구축했던 Kubeflow 소스 구조를 그대로 사용하여 설치합니다.

---

### 2.1 Pipelines YAML 파일 생성하기 :

- Pipelines YAML을 생성합니다.

```bash
$ cd /root/kubeflow/community-distribution
# Knative service YAML 생성 
$ kustomize build applications/pipeline/upstream/env/cert-manager/platform-agnostic-multi-user \
> ./kubeflow-rendered/22_pipelines.yaml
```

---

### 2.2 Pipelines Container Image 다운로드 :

- 위에서 생성한 Pipelines YAML을 통해 이미지 확인 후 다운받습니다.

```bash
docker pull ghcr.io/kubeflow/kfp-cache-server:2.16.1
docker pull docker.io/alpine/k8s:1.32.3
docker pull ghcr.io/kubeflow/kfp-metadata-envoy:2.16.1
docker pull gcr.io/tfx-oss-public/ml_metadata_store_server:1.14.0
docker pull ghcr.io/kubeflow/kfp-metadata-writer:2.16.1
docker pull ghcr.io/kubeflow/kfp-api-server:2.16.1
docker pull ghcr.io/kubeflow/kfp-persistence-agent:2.16.1
docker pull ghcr.io/kubeflow/kfp-scheduled-workflow-controller:2.16.1
docker pull ghcr.io/kubeflow/kfp-frontend:2.16.1
docker pull ghcr.io/kubeflow/kfp-viewer-crd-controller:2.16.1
docker pull ghcr.io/kubeflow/kfp-visualization-server:2.16.1
docker pull mysql:8.4
docker pull chrislusf/seaweedfs:4.00
docker pull quay.io/argoproj/workflow-controller:v3.7.3
docker pull ghcr.io/metacontroller/metacontroller:v4.11.22

docker tag ghcr.io/kubeflow/kfp-cache-server:2.16.1                   harbor.test.com/kubeflow/kubeflow/kfp-cache-server:2.16.1
docker tag docker.io/alpine/k8s:1.32.3                                harbor.test.com/kubeflow/alpine/k8s:1.32.3
docker tag ghcr.io/kubeflow/kfp-metadata-envoy:2.16.1                 harbor.test.com/kubeflow/kubeflow/kfp-metadata-envoy:2.16.1
docker tag gcr.io/tfx-oss-public/ml_metadata_store_server:1.14.0      harbor.test.com/kubeflow/tfx-oss-public/ml_metadata_store_server:1.14.0
docker tag ghcr.io/kubeflow/kfp-metadata-writer:2.16.1                harbor.test.com/kubeflow/kubeflow/kfp-metadata-writer:2.16.1
docker tag ghcr.io/kubeflow/kfp-api-server:2.16.1                     harbor.test.com/kubeflow/kubeflow/kfp-api-server:2.16.1
docker tag ghcr.io/kubeflow/kfp-persistence-agent:2.16.1              harbor.test.com/kubeflow/kubeflow/kfp-persistence-agent:2.16.1
docker tag ghcr.io/kubeflow/kfp-scheduled-workflow-controller:2.16.1  harbor.test.com/kubeflow/kubeflow/kfp-scheduled-workflow-controller:2.16.1
docker tag ghcr.io/kubeflow/kfp-frontend:2.16.1                       harbor.test.com/kubeflow/kubeflow/kfp-frontend:2.16.1
docker tag ghcr.io/kubeflow/kfp-viewer-crd-controller:2.16.1          harbor.test.com/kubeflow/kubeflow/kfp-viewer-crd-controller:2.16.1
docker tag ghcr.io/kubeflow/kfp-visualization-server:2.16.1           harbor.test.com/kubeflow/kubeflow/kfp-visualization-server:2.16.1
docker tag mysql:8.4                                                  harbor.test.com/kubeflow/mysql:8.4
docker tag chrislusf/seaweedfs:4.00                                   harbor.test.com/kubeflow/chrislusf/seaweedfs:4.00
docker tag quay.io/argoproj/workflow-controller:v3.7.3                harbor.test.com/kubeflow/argoproj/workflow-controller:v3.7.3
docker tag ghcr.io/metacontroller/metacontroller:v4.11.22             harbor.test.com/kubeflow/metacontroller/metacontroller:v4.11.22

docker push harbor.test.com/kubeflow/kubeflow/kfp-cache-server:2.16.1
docker push harbor.test.com/kubeflow/alpine/k8s:1.32.3
docker push harbor.test.com/kubeflow/kubeflow/kfp-metadata-envoy:2.16.1
docker push harbor.test.com/kubeflow/tfx-oss-public/ml_metadata_store_server:1.14.0
docker push harbor.test.com/kubeflow/kubeflow/kfp-metadata-writer:2.16.1
docker push harbor.test.com/kubeflow/kubeflow/kfp-api-server:2.16.1
docker push harbor.test.com/kubeflow/kubeflow/kfp-persistence-agent:2.16.1
docker push harbor.test.com/kubeflow/kubeflow/kfp-scheduled-workflow-controller:2.16.1
docker push harbor.test.com/kubeflow/kubeflow/kfp-frontend:2.16.1
docker push harbor.test.com/kubeflow/kubeflow/kfp-viewer-crd-controller:2.16.1
docker push harbor.test.com/kubeflow/kubeflow/kfp-visualization-server:2.16.1
docker push harbor.test.com/kubeflow/mysql:8.4
docker push harbor.test.com/kubeflow/chrislusf/seaweedfs:4.00
docker push harbor.test.com/kubeflow/argoproj/workflow-controller:v3.7.3
docker push harbor.test.com/kubeflow/metacontroller/metacontroller:v4.11.22
```

---

### 2.3 Pipelines 생성하기 :

> 폐쇄망 환경에서 배포하는 경우 각 YAML의 지정된 이미지 경로를 harbor 주소로 변경하고, storageclass가 존재하는 경우 pvc 생성에 추가합니다.
{: .prompt-warning}

- 위 과정에서 생성한 YAML 파일을 가지고 배포합니다.

```bash
$ kubectl apply -f 22_pipelines.yaml

# 상태 확인
$ kubectl get pod -n kubeflow | grep -E \
"pipeline|workflow|metadata|mysql|minio|seaweed"
```

![Pipelines 생성 완료](/assets/img/post/kubernetes/Pipelines%20생성%20완료.png)

---

### 2.4 Pipelines 확인하기 :

- 위와 같이 정상 생성 후, Kubeflow Dashboard에 접속하여 Pipelines 페이지가 정상적으로 뜨는지 확인합니다.

![Pipelines 정상 확인](/assets/img/post/kubernetes/Pipelines%20정상%20확인.png)

---

