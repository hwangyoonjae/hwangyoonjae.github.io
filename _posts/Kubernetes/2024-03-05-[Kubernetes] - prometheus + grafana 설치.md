---
layout: post
title: "[Kubernetes] - prometheus + grafana 설치"
date: 2024-03-05
categories: Kubernetes 설치
tags: [Kubernetes, Prometheus, Grafana]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## prometheus + grafana 설치 준비하기:
### Helm 설치하기:
- prometheus와 grafana를 설치하려면 Helm을 설치해야한다. 
> * [Helm 설치방법](https://hwangyoonjae.github.io/kubernetes/Kubernetes-Helm%EC%9D%B4%EB%9E%80/ "Helm 설치방법")

- Helm 설치 후 Helm repo 추가 및 업데이트한다.
```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
```

- kube-prometheus-stack을 다운로드한다.
```bash
$ helm pull prometheus-community/kube-prometheus-stack
$ tar -zxvf kube-prometheus-stack-56.21.1.tgz
$ cd kube-prometheus-stack/
```

- grafana의 접속할 관리자 계정 패스워드를 설정한다.
```bash
$ vi values.yaml
#==============================
#패스워드 변경
adminPassword: [관리자패스워드]
```
[![grafana 패스워드 입력](/assets/img/post/kubernetes/grafana%20패스워드%20입력.png)](/assets/img/post/kubernetes/grafana%20패스워드%20입력.png)

- monitoring namespace 생성한다.
```bash
$ kubectl create namespace monitoring
```

* * *

## prometheus + grafana 설치하기:
- prometheus 설치한다.
```bash
$ helm install prometheus . -n monitoring -f values.yaml
```
[![prometheus 설치 완료](/assets/img/post/kubernetes/prometheus%20설치%20완료.png)](/assets/img/post/kubernetes/prometheus%20설치%20완료.png)

- kubectl 명령어를 통해서 정상설치되었는지 확인한다.
```bash
$ kubectl get pod -n monitoring 
```
[![prometheus,grafana pod 확인](/assets/img/post/kubernetes/prometheus,grafana%20pod%20확인.png)](/assets/img/post/kubernetes/prometheus,grafana%20pod%20확인.png)

- prometheus, grafana 서비스도 확인한다.
```bash
$ kubectl get svc -n monitoring 
```
[![prometheus,grafana service 확인](/assets/img/post/kubernetes/prometheus,grafana%20service%20확인.png)](/assets/img/post/kubernetes/prometheus,grafana%20service%20확인.png)

* * *

## grafana 외부 접근 설정하기:
- 외부에서 접근을 해서 웹브라우저로 접속하기 위해 ***type: nodePort***로 수정해야한다.
```bash
$ kubectl edit service -n monitoring prometheus-grafana
```
[![grafana 외부 접속 포트 설정](/assets/img/post/kubernetes/grafana%20외부%20접속%20포트%20설정.png)](/assets/img/post/kubernetes/grafana%20외부%20접속%20포트%20설정.png)

* * *

## prometheus 외부 접근 설정하기:
- 외부에서 접근을 해서 웹브라우저로 접속하기 위해 ***type: nodePort***로 수정해야한다.
```bash
$ kubectl edit service -n monitoring prometheus-kube-prometheus-prometheus
```
[![prometheus 외부 접속 포트 설정](/assets/img/post/kubernetes/prometheus%20외부%20접속%20포트%20설정.png)](/assets/img/post/kubernetes/prometheus%20외부%20접속%20포트%20설정.png)

* * *
