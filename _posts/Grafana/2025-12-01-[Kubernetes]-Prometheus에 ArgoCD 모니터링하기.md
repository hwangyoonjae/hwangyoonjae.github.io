---
layout: post
title: "Prometheus에 ArgoCD 모니터링하기"
date: 2025-12-01
categories: [DevOps, Grafana] 
tags: [Kubernetes, Prometheus, Grafana]
image: /assets/img/post-title/grafana-wallpaper.jpg
---

## 1. ArgoCD Metric 옵션 활성화하기 :

> 필자는 ArgoCD를 Helm Chart 통해서 설치했다.
{: .prompt-info}

- ArgoCD Chart의 values.yaml 에서 컨트롤러 metrics를 켠다.

```yaml
controllers:
  metrics:
    enabled: true
```

![argocd metric 활성화](/assets/img/post/grafana/argocd%20metric%20활성화.png)

* * *

## 2. ArgoCD ServiceMonitor 생성하기 :

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-application-controller-metrics
  namespace: [monitoring_namaspace]  # ← Prometheus NS
spec:
  namespaceSelector:
    matchNames:
      - argocd
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-metrics
      app.kubernetes.io/component: application-controller
  endpoints:
    - port: http-metrics
      path: /metrics
      scheme: http
      interval: 30s
```

* * *

## 3. Prometheus RBAC 열어주기 :

- Prometheus ServiceAccount가 ArgoCD Namespace의 Service/Endpoints/Pods를 볼 권한(RBAC)을 생성한다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prometheus-k8s
  namespace: argocd
rules:
  - apiGroups: [""]
    resources:
      - services
      - endpoints
      - pods
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prometheus-k8s
  namespace: argocd
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-k8s
subjects:
  - kind: ServiceAccount
    name: prometheus-k8s                # ← Prometheus SA 이름
    namespace: [monitoring_namaspace]   # ← Prometheus NS
```

```bash
$ kubectl apply -f prometheus-argocd-rbac.yaml
```

* * *

## 4. ArgoCD 대시보드 생성하기 :

- Prometheus UI -> Status -> Targets에서 확인한다.

![argocd prometheus target 확인](/assets/img/post/grafana/argocd%20prometheus%20target%20확인.png)

* * *

- Grafana 홈페이지에서 제공하는 대시보드 다운받아서 사용한다.
> * [Grafana ArrgoCD 대시보드 다운로드](https://grafana.com/grafana/dashboards/?search=argocd "Grafana ArrgoCD 대시보드 다운로드")

![argocd 모니터링 대시보드](/assets/img/post/grafana/argocd%20모니터링%20대시보드.png)

* * *