---
layout: post
title: "Helm Chart를 통해 Prometheus 설치하기"
date: 2026-08-13
categories: [컨테이너, Helm]
tags: [Helm, Prometheus, Grafana]
image: /assets/img/post-title/helm-wallpaper.jpg
---

> 필자는 Prometheus 최신버전으로 구축하고자 재구축 진행하였습니다.
{: .prompt-info}

## 1. Prometheus 설치하기 :
### 1.1 Prometheus Helm Chart 다운로드 :

```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
$ helm search repo prometheus-community/kube-prometheus-stack --versions | head
```

![Prometheus helm chart 목록 조회](/assets/img/post/helm/Prometheus%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull prometheus-community/kube-prometheus-stack \
  --version 88.3.0

# 압축 파일 풀기
$ tar -zxvf kube-prometheus-stack-88.3.0.tgz
```

* * *

### 1.2 Prometheus Container Image 다운로드 : 

```bash
$ docker pull docker.io/busybox:1.38.0
$ docker pull quay.io/prometheus/alertmanager:v0.33.1
$ docker pull quay.io/prometheus-operator/admission-webhook:v0.93.0
$ docker pull ghcr.io/jkroepke/kube-webhook-certgen:1.8.5
$ docker pull quay.io/prometheus-operator/prometheus-operator:v0.93.0
$ docker pull quay.io/prometheus-operator/prometheus-config-reloader:v0.93.0
$ docker pull quay.io/prometheus/prometheus:v3.13.2-distroless
$ docker pull quay.io/thanos/thanos:v0.42.4
$ docker pull registry.k8s.io/kubectl:v1.35.3
$ docker pull quay.io/kiwigrid/k8s-sidecar:2.10.1
$ docker pull docker.io/grafana/grafana:13.1.3
$ docker pull docker.io/bats/bats/1.14.0
$ docker pull quay.io/prometheus/node-exporter:v1.12.1-distroless
$ docker pull quay.io/brancz/kube-rbac-proxy:v0.22.1
$ docker pull registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.19.1

$ docker tag docker.io/library/busybox:1.38.0                                 harbor.test.com/prometheus/library/busybox:1.38.0
$ docker tag quay.io/prometheus/alertmanager:v0.33.1                          harbor.test.com/prometheus/alertmanager:v0.33.1
$ docker tag quay.io/prometheus-operator/admission-webhook:v0.93.0            harbor.test.com/prometheus/prometheus-operator/admission-webhook:v0.93.0
$ docker tag ghcr.io/jkroepke/kube-webhook-certgen:1.8.5                      harbor.test.com/prometheus/jkroepke/kube-webhook-certgen:1.8.5
$ docker tag quay.io/prometheus-operator/prometheus-operator:v0.93.0          harbor.test.com/prometheus/prometheus-operator/prometheus-operator:v0.93.0
$ docker tag quay.io/prometheus-operator/prometheus-config-reloader:v0.93.0   harbor.test.com/prometheus/prometheus-operator/prometheus-config-reloader:v0.93.0
$ docker tag quay.io/prometheus/prometheus:v3.13.2-distroless                 harbor.test.com/prometheus/prometheus/prometheus:v3.13.2-distroless
$ docker tag quay.io/thanos/thanos:v0.42.4                                    harbor.test.com/prometheus/thanos/thanos:v0.42.4
$ docker tag registry.k8s.io/kubectl:v1.35.3                                  harbor.test.com/prometheus/kubectl:v1.35.3
$ docker tag quay.io/kiwigrid/k8s-sidecar:2.10.1                              harbor.test.com/prometheus/kiwigrid/k8s-sidecar:2.10.1
$ docker tag docker.io/grafana/grafana:13.1.3                                 harbor.test.com/prometheus/grafana/grafana:13.1.3
$ docker tag docker.io/bats/bats/1.14.0                                       harbor.test.com/prometheus/bats/bats/1.14.0
$ docker tag quay.io/prometheus/node-exporter:v1.12.1-distroless              harbor.test.com/prometheus/prometheus/node-exporter:v1.12.1-distroless
$ docker tag quay.io/brancz/kube-rbac-proxy:v0.22.1                           harbor.test.com/prometheus/brancz/kube-rbac-proxy:v0.22.1
$ docker tag registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.19.1    harbor.test.com/prometheus/kube-state-metrics/kube-state-metrics:v2.19.1

$ docker push harbor.test.com/prometheus/library/busybox:1.38.0
$ docker push harbor.test.com/prometheus/alertmanager:v0.33.1
$ docker push harbor.test.com/prometheus/prometheus-operator/admission-webhook:v0.93.0
$ docker push harbor.test.com/prometheus/jkroepke/kube-webhook-certgen:1.8.5
$ docker push harbor.test.com/prometheus/prometheus-operator/prometheus-operator:v0.93.0
$ docker push harbor.test.com/prometheus/prometheus-operator/prometheus-config-reloader:v0.93.0
$ docker push harbor.test.com/prometheus/prometheus/prometheus:v3.13.2-distroless
$ docker push harbor.test.com/prometheus/thanos/thanos:v0.42.4
$ docker push harbor.test.com/prometheus/kubectl:v1.35.3
$ docker push harbor.test.com/prometheus/kiwigrid/k8s-sidecar:2.10.1
$ docker push harbor.test.com/prometheus/grafana/grafana:13.1.3
$ docker push harbor.test.com/prometheus/bats/bats:1.14.0
$ docker push harbor.test.com/prometheus/prometheus/node-exporter:v1.12.1-distroless
$ docker push harbor.test.com/prometheus/brancz/kube-rbac-proxy:v0.22.1
$ docker push harbor.test.com/prometheus/kube-state-metrics/kube-state-metrics:v2.19.1
```

* * *

### 1.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
global:
  imageRegistry: "harbor.test.com/prometheus"

# Grafana Ingress 지정할 경우
grafana:
  ingress:
    enabled: true
    ingressClassName: nginx
    # hosts:
    #   - grafana.domain.com
    hosts:
      - grafana.test.com
    path: 
      - /
    pathType: Prefix
    tls: []
    # - secretName: grafana-general-tls
    #   hosts:
    #   - grafana.example.com
   
   # pvc 사용할 경우
  persistence:
    enabled: true
    type: sts
    storageClassName: "nfs-client"
    accessModes:
      - ReadWriteOnce
    size: 20Gi
    finalizers:
      - kubernetes.io/pvc-protection


# Prometheus Ingress 지정할 경우
prometheus:
  ingress:
    enabled: true
    ingressClassName: "nginx"
    # hosts:
    #   - prometheus.domain.com
    hosts:
      - prometheus.test.com
    paths: 
      - /
    pathType: Prefix
    tls: []
      # - secretName: prometheus-general-tls
      #   hosts:
      #     - prometheus.example.com
```

* * *

### 1.4 Prometheus Helm Chart 설치하기 :

```bash
# 네임스페이스 생성
$ kubectl create namespace monitoring

# 설치 진행
$ helm upgrade --install prometheus . -n monitoring
```

![Prometheus Helm Chart 설치 완료 화면](/assets/img/post/helm/Prometheus%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n monitoring
```

![Prometheus helm chart 배포 후 확인](/assets/img/post/helm/Prometheus%20helm%20chart%20배포%20후%20확인.png)

* * *

## 2. Prometheus UI 및 Grafana Dashboard 접속하기 :

- 아래 명령어를 통해 Ingress 주소를 확인하여 페이지에 접속합니다.

```bash
$ kubectl get ingress -n monitoring
```
![Prometheus & Grafana Ingress 주소 확인](/assets/img/post/helm/Prometheus%20&%20Grafana%20Ingress%20주소%20확인.png)

* * *

### 2.1 Prometheus UI 접속하기 :

![Prometheus UI 접속 화면](/assets/img/post/helm/Prometheus%20UI%20접속%20화면.png)

### 2.2 Grafana Dashboard 접속하기 :

![Grafana Dashboard 접속 화면](/assets/img/post/helm/Grafana%20Dashboard%20접속%20화면.png)

* * *