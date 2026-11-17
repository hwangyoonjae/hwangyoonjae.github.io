---
layout: post
title: "Helm Chart를 통해 Falco 설치하기"
date: 2026-11-17
categories: [컨테이너, Helm]
tags: [Helm, Falco]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. Falco 설치하기 :
### 1.1 Falco란? :

- Kubernetes와 Linux 서버에서 발생하는 비정상적인 행위를 실시간으로 감지하는 오픈소스 런타임 보안 도구이며, 기본적으로 이상 행위를 탐지하고 경고하는 도구이며, 자동으로 Pod를 차단하거나 삭제하지는 않습니다.

---

### 1.2 설치 전 환경 확인하기 :

- Falco를 설치하기 전, Kubernetes 클러스터를 구성하는 노드의 운영체제(OS), 커널 버전(Kernel Version), 컨테이너 런타임(Container Runtime)을 확인합니다.

```bash
$ kubectl get nodes \
  -o custom-columns='NAME:.metadata.name,KERNEL:.status.nodeInfo.kernelVersion,OS:.status.nodeInfo.osImage,RUNTIME:.status.nodeInfo.containerRuntimeVersion'
```

> Falco의 Modern eBPF 드라이버가 현재 노드 환경에서 정상적으로 동작할 수 있는지 확인하기 위함입니다.
{: .prompt-info}

---

### 1.3 Falco Helm Chart 다운로드 :

```bash
$ helm repo add falcosecurity https://falcosecurity.github.io/charts
$ helm repo update
$ helm search repo falcosecurity/falco --versions
```

![falco helm chart 목록 조회](/assets/img/post/helm/falco%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull falcosecurity/falco --version 9.2.2

# 압축 파일 풀기
$ tar -zxvf falco-9.2.2.tgz
```

* * *

### 1.4 FALCO Container Image 다운로드 : 

```bash
$ docker pull docker.io/falcosecurity/falco:0.45.2
$ docker pull docker.io/falcosecurity/falcoctl:0.14.3
$ docker pull docker.io/falcosecurity/falco-driver-loader:0.45.2

$ docker tag docker.io/falcosecurity/falco:0.45.2                harbor.inno.com/falcosecurity/falco:0.45.2
$ docker tag docker.io/falcosecurity/falcoctl:0.14.3             harbor.inno.com/falcosecurity/falcoctl:0.14.3
$ docker tag docker.io/falcosecurity/falco-driver-loader:0.45.2  harbor.inno.com/falcosecurity/falco-driver-loader:0.45.2

$ docker push harbor.inno.com/falcosecurity/falco-driver-loader:0.45.2
$ docker push harbor.inno.com/falcosecurity/falcoctl:0.14.3
$ docker push harbor.inno.com/falcosecurity/falco:0.45.2
```

* * *

### 1.5 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
# falco/values.yaml
image:
  # -- The image pull policy.
  pullPolicy: IfNotPresent
  # -- The image registry to pull from.
  registry: harbor.test.com
  # -- The image repository to pull from
  repository: falcosecurity/falco
  # -- The image tag to pull. Overrides the image tag whose default is the chart appVersion.
  tag: ""

falco:
  json_output: true
  log_level: info

  stdout_output:
    enabled: true

  priority: debug

falcoctl:
  image:
    # -- The image pull policy.
    pullPolicy: IfNotPresent
    # -- The image registry to pull from.
    registry: harbor.test.com
    # -- The image repository to pull from.
    repository: falcosecurity/falcoctl
    # -- The image tag to pull.
    tag: "0.14.3"
  artifact:
    follow:
      enabled: false

driver:
  enabled: true
  kind: modern_ebpf
  loader:
    # -- Enable/disable the init container.
    enabled: true
    initContainer:
      image:
        # -- The image pull policy.
        pullPolicy: IfNotPresent
        # -- The image registry to pull from.
        registry: harbor.test.com
        # -- The image repository to pull from.
        repository: falcosecurity/falco-driver-loader
        #  -- Overrides the image tag whose default is the chart appVersion.
        tag: ""
```

* * *

### 1.4 Falco Helm Chart 설치하기 :

```bash
# 네임스페이스 생성
$ kubectl create namespace falco

# falco/base 설치 진행
$ helm upgrade --install falco ./ -n falco
```

![falco Helm Chart 설치 완료 화면](/assets/img/post/helm/falco%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n falco
```

![falco helm chart 배포 후 확인](/assets/img/post/helm/falco%20helm%20chart%20배포%20후%20확인.png)

* * *

---
