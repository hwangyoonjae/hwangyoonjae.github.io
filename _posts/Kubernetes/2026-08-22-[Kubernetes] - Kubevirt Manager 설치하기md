---
layout: post
title: "Kubevirt Manager 설치하기"
date: 2026-08-22
categories: [Kubernetes]
tags: [Kubernetes, Kubevirt]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. 사전 환경 확인하기 :
### 1.1 Kubevirt 설치하기 :
- Kubevirt를 설치해야 하므로 환경 구성이 안되어있을 경우, 아래 링크를 통해서 설치 진행하면됩니다.
> * [Kubevirt 구성하기](https://hwangyoonjae.github.io/posts/Kubernetes-Kubevirt-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0/ "Kubevirt 구성하기")

---

### 1.2 KubeVirt와 CDI 상태 확인하기 :

```bash
$ kubectl get kubevirt -n kubevirt
$ kubectl get pods -n kubevirt

$ kubectl get cdi
$ kubectl get pods -n cdi
```

![kubevir와 cdi 상태 확인](/assets/img/post/kubernetes/kubevir와%20cdi%20상태%20확인.png)

---

## 2. KubeVirt Manager 설치하기 :
### 2.1 KubeVirt Manager YAML 다운로드 :

```bash
$ wget https://github.com/kubevirt-manager/kubevirt-manager/releases/download/v1.5.4/bundled-v1.5.4.yaml
```

![Kubevirt Manager yaml 파일 다운로드](/assets/img/post/kubernetes/Kubevirt%20Manager%20yaml%20파일%20다운로드.png)

---

### 2.2 KubeVirt Manager 이미지 다운로드 :

```bash
$ docker pull kubevirtmanager/kubevirt-manager:1.5.4

$ docker tag kubevirtmanager/kubevirt-manager:1.5.4 harbor.test.com/kubevirtmanager/kubevirt-manager:1.5.4

$ docker push harbor.test.com/kubevirtmanager/kubevirt-manager:1.5.4
```

---

### 2.3 bundle.yaml 파일 수정하기 :

```bash
$ vi bundled-v1.5.4.yaml
```
```yaml
spec:
    spec:
      containers:
        - name: kubevirtmgr
          image: harbor.test.com/kubevirtmanager/kubevirt-manager:1.5.4 # 이미지 경로 맞추기
          imagePullPolicy: IfNotPresent
```

---

### 2.4 KubeVirt Manager 생성하기 :

```bash
# 생성
$ kubectl apply -f bundled-v1.5.4.yaml

# 확인
$ kubectl get all -n kubevirt-manager
```

![kubevirt manager 설치 확인](/assets/img/post/kubernetes/kubevirt%20manager%20설치%20확인.png)

---

### 2.5 KubeVirt Manager Ingress 생성하기 :

```bash
$ vi kubevirt-manager-ingress.yaml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubevirt-manager
  namespace: kubevirt-manager
spec:
  ingressClassName: nginx
  rules:
    - host: kubevirt.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kubevirt-manager
                port:
                  number: 8080
```
```bash
# 생성
$ kubectl apply -f kubevirt-manager-ingress.yaml
```

---

## 3. Kubevirt Manager Dashboard 접속하기 :

- 위 과정에서 생성한 Ingress의 host 주소로 접속합니다.

![Kubevirt Manager Dashboard 접속화면](/assets/img/post/kubernetes/Kubevirt%20Manager%20Dashboard%20접속화면.png)

---