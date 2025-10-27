---
layout: post
title: "Helm Chart를 통해 cert-manager 설치하기"
date: 2025-10-27
categories: [컨테이너, Kubernetes] 
tags: [Kubernetes, Jenkins]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## cert-manager 설치하기:
### cert-manager helm chart 다운로드:

```bash
$ helm repo add jetstack https://charts.jetstack.io
$ helm repo update
$ helm repo list
```

![helm chart 목록 조회](/assets/img/post/helm/helm%20chart%20목록%20조회.png)

* * *

```bash
# 버전 (예: 1.18.3)
$ helm pull jetstack/cert-manager --version 1.18.3
```

![cert manager helm chart 다운로드](/assets/img/post/helm/cert%20manager%20helm%20chart%20다운로드.png)

* * *

### cert-manager container image 다운로드:

- helm chart를 통해 설치될 Application verion대로 이미지를 다운로그한다.

```bash
$ docker pull quay.io/jetstack/cert-manager-controller:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-webhook:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-cainjector:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-acmesolver:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-startupapicheck:v1.18.3
```

![cert-manager 이미지 다운로드](/assets/img/post/helm/cert-manager%20이미지%20다운로드.png)

* * *

### Helm을 통해 Cert-Manager 설치하기:

- cert-manager 설치를 위해 네임스페이스를 생성한다.

```bash
$ kubectl create namespace cert-manager
```

* * *

- values.yaml 파일에 값을 수정한다.

```bash
$ vi values.yaml

# values.yaml
installCRDs: true

global:
  leaderElection:
    namespace: {{ cert_manager_namespace }}
    name: {{ cert_manager_name }}

replicaCount: {{ cert_manager_replica_count }}

resources:
  requests:
    cpu: "{{ cert_manager_cpu_request }}"
    memory: "{{ cert_manager_memory_request }}"
  limits:
    cpu: "{{ cert_manager_cpu_limit }}"
    memory: "{{ cert_manager_memory_limit }}"

image:
  registry: {{ cert_manager_image_registry }}
  repository: {{ cert_manager_image_namespace }}/cert-manager-controller
  tag: {{ cert_manager_image_version }}
  pullPolicy: {{ cert_manager_image_pull_policy }}

webhook:
  image:
    registry: {{ cert_manager_image_registry }}
    repository: {{ cert_manager_image_namespace }}/cert-manager-webhook
    tag: {{ cert_manager_image_version }}
    pullPolicy: {{ cert_manager_image_pull_policy }}

cainjector:
  image:
    registry: {{ cert_manager_image_registry }}
    repository: {{ cert_manager_image_namespace }}/cert-manager-cainjector
    tag: {{ cert_manager_image_version }}
    pullPolicy: {{ cert_manager_image_pull_policy }}

startupapicheck:
  image:
    registry: {{ cert_manager_image_registry }}
    repository: {{ cert_manager_image_namespace }}/cert-manager-startupapicheck
    tag: {{ cert_manager_image_version }}
    pullPolicy: {{ cert_manager_image_pull_policy }}
```

* * *

- 쿠버네티스에 cert-manager 애플리케이션을 설치한다.

```bash
$ helm install cert-manager ./ \
> -n cert-manager \
> --wait --timeout 10m \
> -f values.yaml
```

![helm chart를 cert-manager 통한 배포](/assets/img/post/helm/helm%20chart를%20cert-manager%20통한%20배포.png)

* * *

- 쿠버네티스 클러스터의 정상적으로 설치되었는지 확인한다.

```bash
# pod 목록 조회
$ kubectl get po -n cert-manager

# cert-manager에 설치된 워크로드 목록 조회
$ kubectl get all -n cert-manager
```

![cert-manager workload 목록 확인](/assets/img/post/helm/cert-manager%20workload%20목록%20확인.png)

* * *