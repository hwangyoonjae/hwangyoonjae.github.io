---
layout: post
title: "Helm Chart를 통해 Gitlab Runner 설치하기"
date: 2025-12-17
categories: [컨테이너, Helm] 
tags: [Helm, Gitlab, Runner]
image: /assets/img/post-title/helm-wallpaper.jpg
---

> 필자는 Omnibus 단일 Pod로 구성하여 설치하였습니다.
{: .prompt-info}

## 1. Gitlab Runner 네임스페이스 생성 및 컨테이너 이미지 준비하기 :

* * *

- gitlab 네임스페이스를 생성한다.

```bash
$ kubectl create namespace gitlab-runner
```

* * *

- Gitlab Runner 컨테이너 이미지를 다운받고, 이미지 레포지토리(HARBOR)의 Push한다.

```bash
$ docker pull gitlab/gitlab-runner:alpine-v17.5.5

$ docker tag docker pull gitlab/gitlab-runner:alpine-v17.5.5 [HARBOR_DOMAIN]/gitlab/gitlab-runner:alpine-v17.5.5
$ docker push [HARBOR_DOMAIN]/gitlab/gitlab-runner:alpine-v17.5.5
```

```bash
# 
$ docker pull gitlab/gitlab-runner-helper:x86_64-v17.5.5

$ docker tag docker pull gitlab/gitlab-runner-helper:x86_64-v17.5.5 [HARBOR_DOMAIN]/gitlab/gitlab-runner-helper:x86_64-v17.5.5
$ docker push [HARBOR_DOMAIN]/gitlab/gitlab-runner-helper:x86_64-v17.5.5
```

> Gitlab Runner Helper 이미지가 Runner 버전과 맞아야하는 이유?
>
> Runner ↔ helper는 내부 프로토콜로 통신하여, 버전 불일치 시 ***Job 멈춤, artifact 업로드 실패, Pod 생성은 되는데 Job 실패***하므로 반드시 같은 mirror 버전을 사용해야한다.
{: .prompt-tip}

* * *

## 2. Gitlab Runner Helm Chart 생성하기 :
## 2.1 Gitlab Runner Helm Chart 구조 :
```bash
[root@yjhwang gitlab-runer]# tree
.
├─ Chart.yaml
├─ values.yaml
└─ templates/
   ├─ serviceaccount.yaml
   ├─ rbac.yaml
   ├─ secret.yaml
   ├─ pvc.yaml
   └─ deployment.yaml
```

* * *

## 2.2 values.yaml 생성하기 :

```yaml
gitlab:
  url: "https://[GITLAB_DOMAIN]"
  # 프로젝트/그룹/인스턴스 Registration Token (권장: Secret로 넣고 values에는 비워두기)
  registrationToken: ""

runner:
  name: "omnibus-k8s-runner"
  tags: "k8s,omnibus"
  locked: "false"
  runUntagged: "true"
  executor: "kubernetes"

image:
  repository: "[HARBOR_DOMAIN]/gitlab/gitlab-runner"
  tag: "alpine-v17.5.5"
  pullPolicy: IfNotPresent
  imagePullSecrets: []   # ["harbor-regcred"] 같은 시크릿 이름

rbac:
  create: true

serviceAccount:
  create: true
  name: gitlab-runner

certs:
  # Omnibus GitLab이 사설 인증서면 CA 번들 필요
  enabled: true
  caCrt: ""   # 여기에 PEM 내용 넣거나, 별도 시크릿으로 만들어서 mount하도록 변경 가능

persistence:
  enabled: true
  accessModes:
    - ReadWriteOnce
  size: 1Gi
  storageClassName: "nfs-client" # nfs-provisioner storage class 입력
```

* * *

## 2.3 Chart.yaml 생성하기 :

```yaml
apiVersion: v2
name: gitlab-runner
description: GitLab Runner for Omnibus GitLab (Kubernetes Executor)
type: application
version: 0.1.0
appVersion: "17.5.5"
```

* * *

## 2.4 templates 생성하기 :
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab-runner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitlab-runner
  template:
    metadata:
      labels:
        app: gitlab-runner
    spec:
      serviceAccountName: {{ "{{" }} .Values.serviceAccount.name {{ "}}" }}
      {{ "{{" }}- if .Values.image.imagePullSecrets {{ "}}" }}
      imagePullSecrets:
      {{ "{{" }}- range .Values.image.imagePullSecrets {{ "}}" }}
        - name: {{ "{{" }} . {{ "}}" }}
      {{ "{{" }}- end {{ "}}" }}
      {{ "{{" }}- end {{ "}}" }}
      containers:
        - name: runner
          image: "{{ "{{" }} .Values.image.repository {{ "}}" }}:{{ "{{" }} .Values.image.tag {{ "}}" }}"
          imagePullPolicy: {{ "{{" }} .Values.image.pullPolicy {{ "}}" }}
          volumeMounts:
            - name: runner-config
              mountPath: /etc/gitlab-runner
            {{ "{{" }}- if .Values.certs.enabled {{ "}}" }}
            - name: gitlab-ca
              mountPath: /etc/gitlab-runner/certs
            {{ "{{" }}- end {{ "}}" }}
          command: ["gitlab-runner"]
          args:
            - "run"
            - "--working-directory=/home/gitlab-runner"
      volumes:
        - name: runner-config
          persistentVolumeClaim:
            claimName: gitlab-runner-pvc
        {{ "{{" }}- if .Values.certs.enabled {{ "}}" }}
        - name: gitlab-ca
          secret:
            secretName: gitlab-runner-secret
            items:
              - key: ca.crt
                path: ca.crt
        {{ "{{" }}- end {{ "}}" }}
```

```yaml
# pvc.yaml
{{ "{{" }}- if .Values.persistence.enabled {{ "}}" }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitlab-runner-pvc
spec:
  accessModes:
{{ "{{" }}- range .Values.persistence.accessModes {{ "}}" }}
    - {{ "{{" }} . {{ "}}" }}
{{ "{{" }}- end {{ "}}" }}
  resources:
    requests:
      storage: {{ "{{" }} .Values.persistence.size {{ "}}" }}
  {{ "{{" }}- if .Values.persistence.storageClassName {{ "}}" }}
  storageClassName: {{ "{{" }} .Values.persistence.storageClassName {{ "}}" }}
  {{ "{{" }}- end {{ "}}" }}
{{ "{{" }}- end {{ "}}" }}
```

```yaml
# rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: gitlab-runner
rules:
  - apiGroups: [""]
    resources:
      - pods
      - pods/exec
      - pods/log
      - services
      - configmaps
      - secrets
      - persistentvolumeclaims
    verbs: ["get","list","watch","create","update","delete"]
  - apiGroups: ["apps"]
    resources:
      - deployments
    verbs: ["get","list","watch","create","update","delete"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: gitlab-runner
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: gitlab-runner
subjects:
  - kind: ServiceAccount
    name: {{ "{{" }} .Values.serviceAccount.name {{ "}}" }}
    namespace: {{ "{{" }} .Release.Namespace {{ "}}" }}
```

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: gitlab-runner-secret
type: Opaque
stringData:
  {{ "{{" }}- if .Values.certs.enabled {{ "}}" }}
  ca.crt: |
{{ "{{" }} .Values.certs.caCrt | indent 4 {{ "}}" }}
  {{ "{{" }}- end {{ "}}" }}
```

```yaml
# serviceaccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ "{{" }} .Values.serviceAccount.name {{ "}}" }}
```

* * *

## 2.5 Gitlab-Runner 설치하기 :

- 위 과정에서 생성한 Helm Chart를 이용하여 Gitlab-Runner 설치를 진행한다.

```bash
$ helm install gitlab-runner ./ -n gitlab-runner
```

![gitlab-runner helm chart 배포](/assets/img/post/helm/gitlab-runner%20helm%20chart%20배포.png)

* * *

- 설치 후 리소스들이 정상적으로 되었는지 확인한다.

```bash
# gitlab 리소스 전체 확인
$ kubectl get all -n gitlab-runner

# gitlab pvc 확인
$ kubectl get pvc -n gitlab-runner
```

![gitlab helm chart 배포 후 리소스 확인](/assets/img/post/helm/gitlab-runner%20helm%20chart%20배포%20후%20리소스%20확인.png)

* * *

## 2.6 Gitlab-Runner 등록하기 :

- Gitlab에 Runner 등록을 진행한다.

![gitlab-runner helm chart 배포 후 Runner 등록](/assets/img/post/helm/gitlab-runner%20helm%20chart%20배포%20후%20Runner%20등록.png)
![gitlab-runner helm chart 배포 후 Runner 등록 확인](/assets/img/post/helm/gitlab-runner%20helm%20chart%20배포%20후%20Runner%20등록%20확인.png)

* * *