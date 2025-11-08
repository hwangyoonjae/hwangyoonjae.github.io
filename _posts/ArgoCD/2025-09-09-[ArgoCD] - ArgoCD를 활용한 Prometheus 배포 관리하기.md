---
layout: post
title: "ArgoCD를 활용한 Prometheus 배포 관리하기"
date: 2025-09-09
categories: [DevOps, ArgoCD]
tags: [ArgoCD, Prometheus]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

## 1. Prometheus 배포 파일 구조:

- Gitlab Repo의 저장한 파일 경로는 아래와 같다.

```bash
.
├── grafana-ingress.yaml
├── grafana-pvc.yaml
├── ingress.yaml
├── release
│   ├── alertmanager-alertmanager.yaml
│   ├── alertmanager-networkPolicy.yaml
│   ├── alertmanager-podDisruptionBudget.yaml
│   ├── alertmanager-prometheusRule.yaml
│   ├── alertmanager-secret.yaml
│   ├── alertmanager-serviceAccount.yaml
│   ├── alertmanager-serviceMonitor.yaml
│   ├── alertmanager-service.yaml
│   ├── blackboxExporter-clusterRoleBinding.yaml
│   ├── blackboxExporter-clusterRole.yaml
│   ├── blackboxExporter-configuration.yaml
│   ├── blackboxExporter-deployment.yaml
│   ├── blackboxExporter-networkPolicy.yaml
│   ├── blackboxExporter-serviceAccount.yaml
│   ├── blackboxExporter-serviceMonitor.yaml
│   ├── blackboxExporter-service.yaml
│   ├── grafana-config.yaml
│   ├── grafana-dashboardDatasources.yaml
│   ├── grafana-dashboardDefinitions.yaml
│   ├── grafana-dashboardSources.yaml
│   ├── grafana-deployment.yaml
│   ├── grafana-networkPolicy.yaml
│   ├── grafana-prometheusRule.yaml
│   ├── grafana-serviceAccount.yaml
│   ├── grafana-serviceMonitor.yaml
│   ├── grafana-service.yaml
│   ├── kubePrometheus-prometheusRule.yaml
│   ├── kubernetesControlPlane-prometheusRule.yaml
│   ├── kubernetesControlPlane-serviceMonitorApiserver.yaml
│   ├── kubernetesControlPlane-serviceMonitorCoreDNS.yaml
│   ├── kubernetesControlPlane-serviceMonitorKubeControllerManager.yaml
│   ├── kubernetesControlPlane-serviceMonitorKubelet.yaml
│   ├── kubernetesControlPlane-serviceMonitorKubeScheduler.yaml
│   ├── kubeStateMetrics-clusterRoleBinding.yaml
│   ├── kubeStateMetrics-clusterRole.yaml
│   ├── kubeStateMetrics-deployment.yaml
│   ├── kubeStateMetrics-networkPolicy.yaml
│   ├── kubeStateMetrics-prometheusRule.yaml
│   ├── kubeStateMetrics-serviceAccount.yaml
│   ├── kubeStateMetrics-serviceMonitor.yaml
│   ├── kubeStateMetrics-service.yaml
│   ├── nodeExporter-clusterRoleBinding.yaml
│   ├── nodeExporter-clusterRole.yaml
│   ├── nodeExporter-daemonset.yaml
│   ├── nodeExporter-networkPolicy.yaml
│   ├── nodeExporter-prometheusRule.yaml
│   ├── nodeExporter-serviceAccount.yaml
│   ├── nodeExporter-serviceMonitor.yaml
│   ├── nodeExporter-service.yaml
│   ├── prometheusAdapter-apiService.yaml
│   ├── prometheusAdapter-clusterRoleAggregatedMetricsReader.yaml
│   ├── prometheusAdapter-clusterRoleBindingDelegator.yaml
│   ├── prometheusAdapter-clusterRoleBinding.yaml
│   ├── prometheusAdapter-clusterRoleServerResources.yaml
│   ├── prometheusAdapter-clusterRole.yaml
│   ├── prometheusAdapter-configMap.yaml
│   ├── prometheusAdapter-deployment.yaml
│   ├── prometheusAdapter-networkPolicy.yaml
│   ├── prometheusAdapter-podDisruptionBudget.yaml
│   ├── prometheusAdapter-roleBindingAuthReader.yaml
│   ├── prometheusAdapter-serviceAccount.yaml
│   ├── prometheusAdapter-serviceMonitor.yaml
│   ├── prometheusAdapter-service.yaml
│   ├── prometheus-clusterRoleBinding.yaml
│   ├── prometheus-clusterRole.yaml
│   ├── prometheus-networkPolicy.yaml
│   ├── prometheusOperator-clusterRoleBinding.yaml
│   ├── prometheusOperator-clusterRole.yaml
│   ├── prometheusOperator-deployment.yaml
│   ├── prometheusOperator-networkPolicy.yaml
│   ├── prometheusOperator-prometheusRule.yaml
│   ├── prometheusOperator-serviceAccount.yaml
│   ├── prometheusOperator-serviceMonitor.yaml
│   ├── prometheusOperator-service.yaml
│   ├── prometheus-podDisruptionBudget.yaml
│   ├── prometheus-prometheusRule.yaml
│   ├── prometheus-prometheus.yaml
│   ├── prometheus-roleBindingConfig.yaml
│   ├── prometheus-roleBindingSpecificNamespaces.yaml
│   ├── prometheus-roleConfig.yaml
│   ├── prometheus-roleSpecificNamespaces.yaml
│   ├── prometheus-serviceAccount.yaml
│   ├── prometheus-serviceMonitor.yaml
│   └── prometheus-service.yaml
└── setup
    ├── 0alertmanagerConfigCustomResourceDefinition.yaml
    ├── 0alertmanagerCustomResourceDefinition.yaml
    ├── 0podmonitorCustomResourceDefinition.yaml
    ├── 0probeCustomResourceDefinition.yaml
    ├── 0prometheusagentCustomResourceDefinition.yaml
    ├── 0prometheusCustomResourceDefinition.yaml
    ├── 0prometheusruleCustomResourceDefinition.yaml
    ├── 0scrapeconfigCustomResourceDefinition.yaml
    ├── 0servicemonitorCustomResourceDefinition.yaml
    ├── 0thanosrulerCustomResourceDefinition.yaml
    └── namespace.yaml
```

* * *

## 2. Argocd Application 생성하기 :

- 아래와 같이 Argocd의 Application을 생성한다.

```yaml
project: default
source:
  repoURL: 'https://gitlab.test.com:8443/test/test.git'
  path: kubernetes-oss/prometheus
  targetRevision: HEAD
  directory:
    recurse: true
    jsonnet: {}
    include: '{setup/**,release/**}'
destination:
  server: 'https://kubernetes.default.svc'
  namespace: monitoring
syncPolicy:
  syncOptions:
    - Prune=false   # Git에 없는 리소스 삭제 금지 (입양 시 안전)
    - CreateNamespace=false # 네임스페이스는 수동으로 관리
    - ServerSideApply=true  # 서버 사이드 적용, 충돌 최소화
    - RespectIgnoreDifferences=true # 무시 규칙을 Sync에도 반영
```

* * *

- 아래 그림과 같이 Argocd를 통해서 Prometheus 배포 관리가 가능하다.

![argocd prometheus 애플리케이션 생성](/assets/img/post/ArgoCD/argocd%20prometheus%20애플리케이션%20생성.png)

* * *