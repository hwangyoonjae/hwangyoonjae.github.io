---
layout: post
title: " Kubernetes Dashboard 설치하기"
date: 2024-02-13
categories: [컨테이너, Kubernetes]
tags: [Kubernetes, Dashboard]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. Kubernetes Dashboard란? :
- 웹 기반 쿠버네티스 유저 인터페이스로, 대시보드를 통해 컨테이너화 된 애플리케이션을 쿠버네티스 클러스터에 배포할 수 있고, 컨테이너화 된 애플리케이션을 트러블슈팅할 수 있으며, 클러스터 리소스들을 관리할 수 있습니다.

* * *

## 2. Kubernetes 설치 전 준비사항 :
- Kubernetes 클러스터 구축을 해야합니다.
- Kubernetes 설치를 해야하는 경우 아래 링크 참조하면된다.
> * [Kubernetes 설치방법](https://hwangyoonjae.github.io/kubernetes/Kubernetes-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0(%EB%8F%84%EC%BB%A4-X)/ "Kubernetes 설치방법")

* * *

## 3. Kubernetes Dashboard 설치하기 :
### 3.1 Dashboard yaml 파일 다운로드 및 배포하기 :
```bash
$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
$ kubectl apply -f  recommended.yaml
```

* * *

### 3.2 Kubernetes Dashboard 서비스 목록 확인하기 :
```bash
$ kubectl get services -n kubernetes-dashboard
```
![Kubernetes dashboard 서비스 목록](/assets/img/post/kubernetes/Kubernetes%20dashboard%20서비스%20목록.png)

* * *

### 3.3 NodePort를 이용하여 외부에서 접속가능 하도록 설정하기 :
- ***type:ClusterIP***->***type:NodePort***로 변경합니다.

```bash
$ kubectl edit services kubernetes-dashboard -n kubernetes-dashboard
```
![Kubernetes dashboard type 변경](/assets/img/post/kubernetes/Kubernetes%20dashboard%20type%20변경.png)

* * *

### 3.4 Kubernetes Dashboard 포트 적용된 서비스 목록 확인하기 :
- 외부에서 접속가능하도록 포트 변경 후, 서비스 목록에서 ***443:{NodePort}*** 표기되는지 확인합니다.

![Kubernetes dashboard 서비스 포트 지정 후 목록](/assets/img/post/kubernetes/Kubernetes%20dashboard%20서비스%20포트%20지정%20후%20목록.png)

* * *

### 3.5 Kubernetes Dashboard 접속하기 :
- ***https://{k8s-masterIP}:{NodePort}***로 접속합니다.

![Kubernetes Dashboard 토큰 인증 화면](/assets/img/post/kubernetes/Kubernetes%20Dashboard%20토큰%20인증%20화면.png)

* * *

## 4. Kubernetes Dashboard Token 생성하기 :
- Kubernetes Dashboard 접속 시 토큰 입력하라는 화면이 나오므로, 아래 내용을 참고하여 Token을 생성합니다.
- 사용자 계정 생성하는 yaml를 만든다.

```bash
$ vi dashboard-service-account.yaml
```
```yaml
# 아래 내용 입력
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```
```bash
# yaml 파일 적용
$ kubectl create -f dashboard-service-account.yaml
```

* * *

- 클러스터 자원에 대한 접근 권한을 부여하는 yaml를 만든다.

```bash
$ vi dashboard-service-rolebinding.yaml
```
```yaml
# 아래 내용 입력
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
# -과 kind 사이에 공백하나 들어갑니다.
-kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
```bash
# yaml 파일 적용
$ kubectl create -f dashboard-service-rolebinding.yaml
```

* * *

- Token을 생성합니다.

```bash
$ kubectl -n kubernetes-dashboard create token admin-user
```
![Kubernetes Dashboard 토큰 생성](/assets/img/post/kubernetes/Kubernetes%20Dashboard%20토큰%20생성.png)

* * *

- Token 사용 기간을 무제한으로 사용하기 위해 ***- --token-ttl=0*** 내용을 추가합니다.

```bash
$ kubectl edit -n kubernetes-dashboard deployments.apps kubernetes-dashboard
```
![Kubernetes Dashboard 토큰 사용기간 무제한](/assets/img/post/kubernetes/Kubernetes%20Dashboard%20토큰%20사용기간%20무제한.png)

* * *

## 5. Kubernetes Dashboard 메인화면 :
- 토큰 입력 후 접속 시 ***아래와 같이 표시할 데이터가 없습니다.***라고 나오는데,

![Kubernetes Dashboard 초기화면](/assets/img/post/kubernetes/Kubernetes%20Dashboard%20초기화면.png)

* * *

- 대쉬보드 왼쪽 상단의 'Default'를 '모든 네임스페이스'로 변경하면 아래 그림과 같이 목록을 확인 할 수 있습니다.

![Kubernetes Dashboard 모든네임스페이스 클릭 후 화면](/assets/img/post/kubernetes/Kubernetes%20Dashboard%20모든네임스페이스%20클릭%20후%20화면.png)

* * *