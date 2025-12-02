---
layout: post
title: "Pod로 설치한 Runner 등록하기"
date: 2025-11-27
categories: [컨테이너, Kubernetes, Runner] 
tags: [Kubernetes, Gitlab, Runner]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. RBAC 생성하기 :

- Runner 전용 RBAC를 Runner Pod가 존재하는 네임스페이스에 생성한다.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gitlab-runner
  namespace: test
--- 
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: gitlab-runner
  namespace: test
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/exec", "pods/attach", "pods/log", "secrets", "configmaps", "serviceaccounts"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
  - apiGroups: ["batch"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
--- 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: gitlab-runner-binding
  namespace: test
subjects:
  - kind: ServiceAccount
    name: gitlab-runner
    namespace: test
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: gitlab-runner
```

```bash
# RBAC 배포
$ kubectl apply -f gitlab-runner-rbac.yaml
```

* * *

## 2. Gitlab Runner 연동하기 :
### 2.1 Gitlab 프로젝트 Runner 생성하기 :

- 연동하고자 하는 프로젝트 화면에서 왼쪽 메뉴의 **Settings > CI/CD** 클릭하여 **Runners** 클릭 후 **New project runner** 버튼을 클릭한다.

![gitlab 프로젝트 runner 생성하기 1](/assets/img/post/kubernetes/gitlab%20프로젝트%20runner%20생성하기%201.png)

* * *

- 특정 태그를 통해 실행할 경우 태그명 입력하고, 태그 없이 실행할 경우에는 **Run untagged jobs** 체크하여 **Create runner** 버튼을 생성한다.

![gitlab 프로젝트 runner 생성하기 2](/assets/img/post/kubernetes/gitlab%20프로젝트%20runner%20생성하기%202.png)

* * *

- Runner 등록하는 명령을 복사하여 Runner 서버에 명령어를 입력한다.

![gitlab 프로젝트 runner 생성하기 3](/assets/img/post/kubernetes/gitlab%20프로젝트%20runner%20생성하기%203.png)

* * *

### 2.2 Runner 연동하기 :

```bash
# Runner Pod 안에 접속한다.
$ kubectl exec -it [gitlab-runner-pod명] -n [네임스페이스] -- /bin/bash

$ gitlab-runner register  --url https://gitlab.test.com  --token glrt-FMgTy3LsVme7VxSyRmtp
```

![gitlab-runner 등록](/assets/img/post/kubernetes/gitlab-runner%20등록.png)

> 필자는 공인인증서를 사용하였기에 별도에 CA 파일 지정을 하지 않았으나, ***self-signfed*** 인증서 사용 시 ***--tls-ca-file*** 옵션 추가하거나 ***--tls-skip-verify*** 옵션을 사용하여 TLS 검증 무시하면됩니다.
{: .prompt-info}

* * *

#### 2.2.1 연동 후 에러 발생 :

> getting Kubernetes config: invalid configuration: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
{: .prompt-warning}

![gitlab runner 등록 후 CI 에러 발생 1](/assets/img/post/kubernetes/gitlab%20runner%20등록%20후%20CI%20에러%20발생%201.png)

* * *

- 현재 Runner에는 **executor = "kubernetes"**까지만 등록된 상태라 Kubernetes 접속 정보를 모르기 때문에 ***config.toml*** 또는 환경변수로 따로 지정해줘야한다.

```toml
[[runners]]
  name = "gitlab-runner-k8s"
  url = "https://gitlab.test.com"
  token = "glrt-13kUJ2xER19AAvBFad5A"
  executor = "kubernetes"

  [runners.kubernetes]
    privileged = true
    namespace = "gitlab-runner"
    service_account = "gitlab-runner-sa"
```

```bash
# config.toml 수정 후, gitlab runner 서버 재시작
$ gitlab-runner restart
```

![runner kubernetes 접속 정보 config의 저장](/assets/img/post/kubernetes/runner%20kubernetes%20접속%20정보%20config의%20저장.png)

> Runner Config의 service_account만 추가해도 동작하는 이유?
>
> GitLab Runner가 “Kubernetes 안에서 실행 중인 Pod”일 경우, 쿠버네티스 클라이언트가 자동으로 in-cluster config를 사용하기 때문이다.
{: .prompt-tip}

* * *