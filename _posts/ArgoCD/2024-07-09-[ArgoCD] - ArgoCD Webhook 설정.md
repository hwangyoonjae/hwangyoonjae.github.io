---
layout: post
title: "ArgoCD Webhook 설정"
date: 2024-07-09
categories: [DevOps, ArgoCD]
tags: [ArgoCD, Gitlab, Webhook]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

## 1. ArgoCD 비밀키 구성하기 :
- Kubernetes 비밀키에서 `argocd-secret`1단계에서 구성한 Git 공급자의 웹훅 비밀을 사용하여 다음 키 중 하나를 구성한다.

![argocd 비밀키 구성](/assets/img/post/ArgoCD/argocd%20비밀키%20구성.png)

- argocd configmap의 비밀키 값을 명시한다.

```bash
$ kubectl edit configmap argocd-cm -n argocd
```

```yaml
apiVersion: v1
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
        192.168.1.1 gitlab.com
  url: https://argo.com
  # 아래 내용 추가
  webhook.gitlab.secret: {사용할 패스워드 입력}
```

* * *

## 2. Gitlab Webhook 설정 후 정상 동작 확인하기 :
- gitlab 프로젝트 > Settings > Webhooks 접속하여 필요한 값을 입력 후 저장한다.

> webhook 입력 내용
>- URL : argocd DNS 주소명 입력
>- Secrest token : 위에서 입력한 **webhook.gitlab.secret**의 값
>- Trigger는 "Push events" 체크
{: .prompt-tip }

![argocd webhook 이벤트 생성 화면](/assets/img/post/ArgoCD/argocd%20webhook%20이벤트%20생성%20화면.png)
![argocd webhook 이벤트 생성 클릭](/assets/img/post/ArgoCD/argocd%20webhook%20이벤트%20생성%20클릭.png)

* * *

## 3. 연동 중 에러 발생사항 :
### 3.1 Url is blocked: Requests to the local network are not allowed 에러 발생한 경우 :

>Url is blocked: Requests to the local network are not allowed
{: .prompt-warning }

* * *

### 조치 방법 :
- Admin Area > Settings > Network > Outbound requests 접속하여 **Allow requests to the local network from webhooks and integrations** 체크 후 저장

![argocd webhook 연동 에러 조치 방법 1](/assets/img/post/ArgoCD/argocd%20webhook%20연동%20에러%20조치%20방법%201.png)

* * *