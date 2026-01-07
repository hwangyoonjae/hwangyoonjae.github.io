---
layout: post
title: "Helm Chart를 통해 Jenkins 설치하기"
date: 2026-01-05
categories: [컨테이너, Helm] 
tags: [Helm, Jenkins]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. Jenkins 설치하기 :
### 1.1 Jenkins helm chart 다운로드 :

```bash
$ helm repo add jenkins https://charts.jenkins.io
$ helm repo update
$ helm repo list
```

![helm chart 목록 조회](/assets/img/post/helm/helm%20chart%20목록%20조회.png)#변경해야함

* * *

```bash
$ helm pull jenkins/jenkins
```

![jenkins helm chart 다운로드](/assets/img/post/helm/jenkins%20helm%20chart%20다운로드.png)

* * *

### 1.2 Jenkins container image 다운로드 :

- helm chart를 통해 설치될 Application version대로 이미지를 다운로드합니다.

```bash
$ docker pull jenkins/jenkins:2.544-jdk25
```

![jenkins 이미지 다운로드](/assets/img/post/helm/jenkins%20이미지%20다운로드.png)

* * *

### 1.3 Helm을 통해 Jenkins 설치하기 :

- Jenkins 설치를 위해 네임스페이스를 생성합니다.

```bash
$ kubectl create namespace jenkins
```

* * *

- values.yaml 파일에 값을 수정합니다.

```bash
$ vi values.yaml
```
{% raw %}
```yaml
# values.yaml
controller:
  # -- Used for label app.kubernetes.io/component
  componentName: "jenkins-controller"
  image:
    # -- Controller image registry
    registry: "[HARBOR_DOMAIN]"
    # -- Controller image repository
    repository: "jenkins/jenkins"

    # -- Controller image tag override; i.e., tag: "2.440.1-jdk21"
    tag: "2.544-jdk25"

  ingress:
    enabled: true
    apiVersion: networking.k8s.io/v1
    ingressClassName: nginx
    hostName: jenkins.test.com

    # annotations:
    #   nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    #   nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    #   nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"

    tls:
      - secretName: jenkins-tls      # <-- TLS Secret (미리 존재해야 함)
        hosts:
          - jenkins.test.com

  admin:
    # user: "admin"
    password: "[ADMIN_PASSWORD]"

persistence:
  enabled: true
  # -- Storage class for the PVC
  storageClass: "nfs-client"
  # -- The PVC access mode
  accessMode: "ReadWriteMany"
  # -- The size of the PVC
  size: "20Gi"
```
{% endraw %}

* * *

- 쿠버네티스에 Jenkins 애플리케이션을 설치합니다.

```bash
$ helm install jenkins ./jenkins -n jenkins -f values.yaml
```

![helm chart를 jenkins 통한 배포](/assets/img/post/helm/helm%20chart를%20jenkins%20통한%20배포.png)

* * *

- 쿠버네티스 클러스터의 정상적으로 설치되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n jenkins

# jenkins에 설치된 워크로드 목록 조회
$ kubectl get all -n jenkins
```

![jenkins workload 목록 확인](/assets/img/post/helm/jenkins%20workload%20목록%20확인.png)

* * *