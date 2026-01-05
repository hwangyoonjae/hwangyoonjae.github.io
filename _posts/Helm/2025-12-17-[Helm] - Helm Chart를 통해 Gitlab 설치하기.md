---
layout: post
title: "Helm Chart를 통해 Gitlab 설치하기"
date: 2025-12-17
categories: [컨테이너, Helm] 
tags: [Helm, Gitlab, Runner]
image: /assets/img/post-title/helm-wallpaper.jpg
---

> 필자는 Omnibus 단일 Pod로 구성하여 설치하였습니다.
{: .prompt-info}

* * *

## 1. Gitlab 네임스페이스 생성 및 컨테이너 이미지 준비하기 :

- gitlab 네임스페이스를 생성한다.

```bash
$ kubectl create namespace gitlab
```

* * *

- Gitlab 컨테이너 이미지를 다운받고, 이미지 레포지토리(HARBOR)의 Push한다.

```bash
$ docker pull gitlab/gitlab-ce:18.6.2-ce.0

$ docker tag gitlab/gitlab-ce:18.6.2-ce.0 [HARBOR_DOMAIN]/gitlab/gitlab-ce:18.6.2-ce.0
$ docker push [HARBOR_DOMAIN]/gitlab/gitlab-ce:18.6.2-ce.0
```

* * *

## 2. Gitlab Helm Chart 생성하기 :
## 2.1 Gitlab Helm Chart 구조 :
```bash
[root@yjhwang gitlab]# tree
.
├── values.yaml
├── Chart.yaml
├── templates
│   ├── configmap.yaml
│   ├── ingress.yaml
│   ├── secret.yaml
│   ├── service.yaml
│   ├── statefulset.yaml
```

* * *

## 2.2 values.yaml 생성하기 :

```yaml
image:
  repository: [HARBOR_DOMAIN]/gitlab/gitlab-ce
  tag: "18.6.2-ce.0"
  pullPolicy: IfNotPresent
  imagePullSecrets:
    - harbor-registry-secret

host: [GITLAB_DOMAIN]

service:
  type: ClusterIP
  httpPort: 80
  sshPort: 22

ingress:
  enabled: true
  className: nginx
  tls:
    enabled: true
    secretName: gitlab-ingress-tls
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"

persistence:
  storageClassName: nfs-client   # NFS provisioner SC
  accessModes:
    - ReadWriteMany
  configSize: 5Gi
  dataSize: 200Gi
  logsSize: 50Gi

gitlab:
  externalUrl: "https://[GITLAB_DOMAIN]"
  timeZone: "Asia/Seoul"
  sshPort: 22
  omnibusNginxHttps: false       # TLS는 Ingress에서 종료

  rootPassword: "[ROOT_PASSWORD]"
  rootPasswordSecret:
    name: gitlab-root-secret
    key: password

podSecurityContext:
  fsGroup: 1000                  # NFS 권한 이슈 예방
```

* * *

## 2.3 Chart.yaml 생성하기 :

```yaml
apiVersion: v2
name: gitlab-omnibus
description: GitLab Omnibus single-pod deployment (on-prem)
type: application
version: 0.1.0
appVersion: "18.6.2-ce.0"
```

* * *

## 2.4 templates 생성하기 :

{% raw %}
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  gitlab.rb: |
    external_url "{{ .Values.gitlab.externalUrl }}"
    gitlab_rails['time_zone'] = '{{ .Values.gitlab.timeZone }}'
    gitlab_rails['gitlab_shell_ssh_port'] = {{ .Values.gitlab.sshPort }}

    # Ingress에서 TLS 종료하는 구조면 내부는 HTTP만
    nginx['listen_https'] = {{ ternary "true" "false" .Values.gitlab.omnibusNginxHttps }}
    nginx['listen_port'] = 80

    # root password는 환경변수에서 읽음
    gitlab_rails['initial_root_password'] = ENV['GITLAB_ROOT_PASSWORD']
```
{% endraw %}

{% raw %}
```yaml
# ingress.yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}
  annotations:
{{- range $k, $v := .Values.ingress.annotations }}
    {{ $k }}: {{ $v | quote }}
{{- end }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  tls:
    - hosts:
        - {{ .Values.host }}
      secretName: {{ .Values.ingress.tls.secretName }}
  rules:
    - host: {{ .Values.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ .Release.Name }}
                port:
                  number: {{ .Values.service.httpPort }}
{{- end }}
```
{% endraw %}

{% raw %}
```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-root
type: Opaque
stringData:
  password: {{ .Values.gitlab.rootPassword | quote }}
```
{% endraw %}

{% raw %}
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
    - name: http
      port: {{ .Values.service.httpPort }}
      targetPort: 80
    - name: ssh
      port: {{ .Values.service.sshPort }}
      targetPort: 22
```
{% endraw %}

{% raw %}
```yaml
# statefulSet.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: {{ .Release.Name }}
spec:
  serviceName: {{ .Release.Name }}
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      securityContext:
        fsGroup: {{ .Values.podSecurityContext.fsGroup }}
      imagePullSecrets:
      {{- range .Values.image.imagePullSecrets }}
        - name: {{ . }}
      {{- end }}
      containers:
        - name: gitlab
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          env:
            - name: GITLAB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ .Release.Name }}-root
                  key: password
            - name: GITLAB_OMNIBUS_CONFIG
              value: |
                external_url '{{ .Values.gitlab.externalUrl }}'
                # Ingress/외부 LB에서 TLS 종료(HTTPS)하는 경우 보통 아래처럼:
                nginx['listen_port'] = 80
                nginx['listen_https'] = false
          ports:
            - containerPort: 80
            - containerPort: 22
          volumeMounts:
            - name: config
              mountPath: /etc/gitlab
            - name: data
              mountPath: /var/opt/gitlab
            - name: logs
              mountPath: /var/log/gitlab
  volumeClaimTemplates:
    - metadata:
        name: config
      spec:
        accessModes: {{ toYaml .Values.persistence.accessModes | nindent 8 }}
        storageClassName: {{ .Values.persistence.storageClassName }}
        resources:
          requests:
            storage: {{ .Values.persistence.configSize }}
    - metadata:
        name: data
      spec:
        accessModes: {{ toYaml .Values.persistence.accessModes | nindent 8 }}
        storageClassName: {{ .Values.persistence.storageClassName }}
        resources:
          requests:
            storage: {{ .Values.persistence.dataSize }}
    - metadata:
        name: logs
      spec:
        accessModes: {{ toYaml .Values.persistence.accessModes | nindent 8 }}
        storageClassName: {{ .Values.persistence.storageClassName }}
        resources:
          requests:
            storage: {{ .Values.persistence.logsSize }}
```
{% endraw %}

* * *

## 2.5 Gitlab 설치하기 :

- 위 과정에서 생성한 Helm Chart를 이용하여 Gitlab 설치를 진행한다.

```bash
$ helm install gitlab ./ -n gitlab 
```

![gitlab helm chart 배포](/assets/img/post/helm/gitlab%20helm%20chart%20배포.png)

* * *

- 설치 후 리소스들이 정상적으로 되었는지 확인한다.

```bash
# gitlab 리소스 전체 확인
$ kubectl get all -n gitlab

# gitlab pvc 확인
$ kubectl get pvc -n gitlab
```

![gitlab helm chart 배포 후 리소스 확인](/assets/img/post/helm/gitlab%20helm%20chart%20배포%20후%20리소스%20확인.png)

* * *

- 설정한 Ingress 주소로 접속되는지 확인한다.

![gitlab helm chart 배포 후 페이지 접속](/assets/img/post/helm/gitlab%20helm%20chart%20배포%20후%20페이지%20접속.png)

* * *