---
layout: post
title: "[ArgoCD] - Gitops (ArgoCD, Gitlab) 연동하기"
date: 2024-07-05
categories: ArgoCD
tags: [ArgoCD, Gitlab, Gitops]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

## ArgoCD Architecture :
![argocd 아키텍처](/assets/img/post/ArgoCD/argocd%20아키텍처.png)

* * *

## 1. Gitlab 구성하기 :
- root path에 deployment.yaml 파일과 service.yaml 파일을 생성하여 gitlab repository의 commit하여 저장한다.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hyj-test
  labels:
    app: hyj-test
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hyj-test
  template:
    metadata:
      labels:
        app: hyj-test
    spec:
      containers:
      - name: hyj-test
        image: harbor.com/hyj-test/my-docker-image:latest
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hyj-test
spec:
  selector:
    app: hyj-test
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  type: NodePort
```

* * *

## 2. 배포할 Application 생성하기 :
![argocd 어플리케이션 생성 화면 1](/assets/img/post/ArgoCD/argocd%20어플리케이션%20생성%20화면%201.png)
- Application Name : 배포할 어플리케이션 이름 (sample-argocd-test)
- Project : ArgoCD 내 어플리케이션 그룹 (default)
- SYNC POLICY : (Manual, Automatic) Git 변동사항을 자동으로 반영할 것인지, 수동으로 반영할 것인지 선택 (Manual)

![argocd 어플리케이션 생성 화면 2](/assets/img/post/ArgoCD/argocd%20어플리케이션%20생성%20화면%202.png)
- Repository URL : Git 저장소 URL 입력 Ex.) https://gitlab.com:/root/agocd.git
- Revision : Git의 어떤 Revision을 바라 볼 것인지 지정 (HEAD, main, tag 등) (main)
- Path : Repository 내 변경사항을 관리할 파일이 위치한 경로 지정 (. 일 경우 root) (.)

![argocd 어플리케이션 생성 화면 3](/assets/img/post/ArgoCD/argocd%20어플리케이션%20생성%20화면%203.png)
- Cluster URL : 배포 대상 Kubernetes Cluster 지정 (https://kubernetes.default.svc)
- Namespace : 배포 할 Kubernetes Cluster 내 Namespace 지정 (default)
- DIRECTORY RECURSE : Path 하위 경로 디렉토리의 변동사항 체크 확인 여부

위와 같이 입력 후 CREATE 버튼을 클릭한다.

* * *

## 3. 연동 중 에러 발생사항 :
### 10.96.0.10:53: no such host 에러 발생한 경우

> Unable to create application: application spec for argo-test is invalid: InvalidSpecError: repository not accessible: repositories not accessible: &Repository{Repo: "https://gitlab.com:8443/root/argocd.git", Type: "", Name: "", Project: ""}: repo client error while testing repository: rpc error: code = Unknown desc = error testing repository connectivity: Get "https://gitlab.com:8443/root/argocd.git/info/refs?service=git-upload-pack": dial tcp: lookup gitlab.com on 10.96.0.10:53: no such host
{: .prompt-warning }

### 조치 방법 :
- coredns configmap의 hosts 플러그인 추가

```bash
# configmap 내용 수정
$ kubectl -n kube-system edit configmap coredns
```

```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
# 아래 hosts 부분 추가
        hosts {
            {Gitlab IP 주소} gitlab.com
            fallthrough
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153 
        forward . /etc/resolv.conf 
        cache 30 
        loop reload 
        loadbalance 
    }
```

```bash
# coredns 시작
$ kubectl -n kube-system rollout restart deployment coredns
```

* * *

### x509: certificate signed by unknown authority 에러 발생한 경우

> Unable to create application: application spec for argo-test is invalid: InvalidSpecError: repository not accessible: repositories not accessible: &Repository{Repo: "https://gitlab.com:8443/root/argocd.git", Type: "", Name: "", Project: ""}: repo client error while testing repository: rpc error: code = Unknown desc = error testing repository connectivity: Get "https://gitlab.com:8443/root/argocd.git/info/refs?service=git-upload-pack": x509: certificate signed by unknown authority
{: .prompt-warning }

### 조치 방법 :
- argocd-tls-certs-cm.yaml 파일 생성

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-tls-certs-cm
  namespace: argocd
data:
  gitlab.inno.com: |
    -----BEGIN CERTIFICATE-----
    (여기에 인코딩되지 않은 gitlab.crt 내용 입력)
    -----END CERTIFICATE-----
```

```bash
# configmap 적용
$ kubectl apply -f argocd-tls-certs-cm.yaml
```

```bash
# ArgoCD 재시작
$ kubectl rollout restart deployment argocd-server -n argocd
$ kubectl rollout restart deployment argocd-repo-server -n argocd
```

- argocd-cm ConfigMap에도 GitLab 인증서가 설정되어 있는지 확인하여 해당 ConfigMap을 편집하여 인증서가 제대로 설정되어 있는지 확인한다.

```bash
$ kubectl -n argocd edit configmap argocd-cm
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ConfigMap","metadata":{"annotations":{},"labels":{"app.kubernetes.io/name":"argocd-cm","app.kubernetes.io/part-of":"argocd"},"name":"argocd-cm","namespace":"argocd"}}
  creationTimestamp: "2024-07-05T03:11:54Z"
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
  name: argocd-cm
  namespace: argocd
  resourceVersion: "1638126"
  uid: c37c66c0-3b3f-4e14-bd35-1b69bb9b79cb
# 아래 내용 추가 작성
data:
  application.instanceLabelKey: argocd.argoproj.io/instance
  repository.credentials: |
    - url: https://gitlab.com:8443
      usernameSecret:
        name: gitlab-credentials
        key: username
      passwordSecret:
        name: gitlab-credentials
        key: password
  repository.hosts: |
    - url: https://gitlab.com:8443
      host: |
        {gitlab IP 주소} gitlab.com
  url: https://argo.com
```

* * *
 
### "gitlab-credentials" not found 에러 발생한 경우

>Unable to create application: error while validating and normalizing app: error validating the repo: secret "gitlab-credentials" not found
{: .prompt-warning }

### 조치 방법 :
- Gitlab 자격 증명 시크릿 생성
```bash
$ kubectl -n argocd create secret generic gitlab-credentials --from-literal=username=<your-username> --from-literal=password=<your-personal-access-token>
```

- ArgoCD를 재시작하여 설정 반영
```bash
$ kubectl rollout restart deployment argocd-server -n argocd
$ kubectl rollout restart deployment argocd-repo-server -n argocd
```

* * *

## 4. ArgoCD Application 생성하기 :
- NEW APP으로 추가된 APPS는 SYNC를 수행하기 전 상태로 OutOfSync 상태의 App으로 되고, 하단의 SYNC 버튼을 클릭한 후 SYNCHRONIZE를 클릭한다.
![[Pasted image 20240705124811.png]]
![[Pasted image 20240705140227.png]]

- 연동 후 정상적으로 pod 구성되면 아래 화면과 같이 나온다.
![[Pasted image 20240705140557.png]]

- 구성된 pod의 대한 Application 배포를 확인한다.
![[Pasted image 20240705141439.png]]

* * *

## 5. ArgoCD 자동 배포 적용하기 :
- gitlab의 이벤트 발생 시 변경사항 확인하여 자동으로 배포되도록 설정한다.

```bash
$ kubectl edit application {Application name} -n argocd
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  creationTimestamp: "2024-07-08T03:50:25Z"
  generation: 592
  name: argo-test
  namespace: argocd
  resourceVersion: "857561"
  uid: 181b1c62-87c1-4541-b49b-8d182a9361a9
spec:
  destination:
    namespace: hyj-test
    server: https://kubernetes.default.svc
  project: default
  source:
    path: .
    repoURL: https://gitlab.com:8443/root/argocd.git
    targetRevision: HEAD
# 아래 내용을 추가한다.
  syncPolicy: # 애플리케이션 동기화 정책 설명
    automated: # 애플리케이션을 자동으로 동기화하도록 설정
      prune: true # 더 이상 원격 저장소의 존재하지 않는 리소스 클러스터에서 삭제
      selfHeal: true # 원격 저장소의 구성과 일치하지 않을 때 자동으로 수정
```

* * *