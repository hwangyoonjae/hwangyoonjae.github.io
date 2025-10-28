---
layout: post
title: "Helm Chart를 통해 cert-manager 설치하기"
date: 2025-10-27
categories: [컨테이너, Helm] 
tags: [Kubernetes, Jenkins]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## cert-manager 설치하기:
### cert-manager helm chart 다운로드:

```bash
$ helm repo add jetstack https://charts.jetstack.io
$ helm repo update
$ helm repo list
```

![helm chart 목록 조회](/assets/img/post/helm/helm%20chart%20목록%20조회.png)

* * *

```bash
# 버전 (예: 1.18.3)
$ helm pull jetstack/cert-manager --version 1.18.3
```

![cert manager helm chart 다운로드](/assets/img/post/helm/cert%20manager%20helm%20chart%20다운로드.png)

* * *

### cert-manager container image 다운로드:

- helm chart를 통해 설치될 Application verion대로 이미지를 다운로그한다.

```bash
$ docker pull quay.io/jetstack/cert-manager-controller:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-webhook:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-cainjector:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-acmesolver:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-startupapicheck:v1.18.3
```

![cert-manager 이미지 다운로드](/assets/img/post/helm/cert-manager%20이미지%20다운로드.png)

* * *

### Helm을 통해 Cert-Manager 설치하기:

- cert-manager 설치를 위해 네임스페이스를 생성한다.

```bash
$ kubectl create namespace cert-manager
```

* * *

- values.yaml 파일에 값을 수정한다.

```bash
$ vi values.yaml
```
```yaml
# values.yaml
installCRDs: true

global:
  leaderElection:
    namespace: {{ cert_manager_namespace }}
    name: {{ cert_manager_name }}

replicaCount: {{ cert_manager_replica_count }}

resources:
  requests:
    cpu: "{{ cert_manager_cpu_request }}"
    memory: "{{ cert_manager_memory_request }}"
  limits:
    cpu: "{{ cert_manager_cpu_limit }}"
    memory: "{{ cert_manager_memory_limit }}"

image:
  registry: {{ cert_manager_image_registry }}
  repository: {{ cert_manager_image_namespace }}/cert-manager-controller
  tag: {{ cert_manager_image_version }}
  pullPolicy: {{ cert_manager_image_pull_policy }}

webhook:
  image:
    registry: {{ cert_manager_image_registry }}
    repository: {{ cert_manager_image_namespace }}/cert-manager-webhook
    tag: {{ cert_manager_image_version }}
    pullPolicy: {{ cert_manager_image_pull_policy }}

cainjector:
  image:
    registry: {{ cert_manager_image_registry }}
    repository: {{ cert_manager_image_namespace }}/cert-manager-cainjector
    tag: {{ cert_manager_image_version }}
    pullPolicy: {{ cert_manager_image_pull_policy }}

startupapicheck:
  image:
    registry: {{ cert_manager_image_registry }}
    repository: {{ cert_manager_image_namespace }}/cert-manager-startupapicheck
    tag: {{ cert_manager_image_version }}
    pullPolicy: {{ cert_manager_image_pull_policy }}
```

* * *

- 쿠버네티스에 cert-manager 애플리케이션을 설치한다.

```bash
$ helm install cert-manager ./ \
> -n cert-manager \
> --wait --timeout 10m \
> -f values.yaml
```

![helm chart를 cert-manager 통한 배포](/assets/img/post/helm/helm%20chart를%20cert-manager%20통한%20배포.png)

* * *

- 쿠버네티스 클러스터의 정상적으로 설치되었는지 확인한다.

```bash
# pod 목록 조회
$ kubectl get po -n cert-manager

# cert-manager에 설치된 워크로드 목록 조회
$ kubectl get all -n cert-manager
```

![cert-manager workload 목록 확인](/assets/img/post/helm/cert-manager%20workload%20목록%20확인.png)

* * *

## cert-manager CRDs(CustomResourceDefinitions) 설치하기 :
### CRDs란? :

- 쿠버네티스에 새로운 리소스 타입(API 오브젝트)을 추가하기 위한 확장 메커니즘이다.

> 쿠버네티스 기본에는 없는 리소스를 “추가로 등록”해서 kubectl get certificate, kubectl get issuer 같은 명령을 사용할 수 있게 만들어주는 정의 파일이다.
{: .prompt-info}

* * *

### cert-manager가 추가하는 CRD 종류 :

- TLS 인증서 자동화를 위해 아래와 같은 커스텀 리소스(Custom Resources) 를 정의하고 사용한다.

| CRD 리소스 | 역할 설명 |
|-------------|------------|
| **Certificate** (`certificates.cert-manager.io`) | 실제 발급받고 싶은 인증서 정보 (DNS, Secret 이름 등) |
| **Issuer** (`issuers.cert-manager.io`) | 특정 네임스페이스 전용 인증서 발급자 |
| **ClusterIssuer** (`clusterissuers.cert-manager.io`) | 클러스터 전체에서 사용할 수 있는 인증서 발급자 |
| **CertificateRequest** (`certificaterequests.cert-manager.io`) | cert-manager가 실제 발급 요청을 Vault나 CA에 전달할 때 사용하는 내부 요청 리소스 |
| **Order** (`orders.acme.cert-manager.io`) | ACME 프로토콜(Let’s Encrypt)에서 인증서 발급 과정을 추적하기 위한 내부 리소스 |
| **Challenge** (`challenges.acme.cert-manager.io`) | ACME 인증(HTTP-01, DNS-01 등) 과정을 나타내는 내부 리소스 |

> CRDs가 없으면 cert-manager는 아무런 커스텀 리소스를 인식하지 못해서 **kubectl apply -f certificate.yaml** 같은 명령이 오류가 발생한다.
{: .prompt-tip}

* * *

### CRDs 설치하기 :

- 차트만 오프라인으로 가져오면 CRD는 자동 설치되지 않기 때문에, Repo의 CRDs 디렉터리에서 직접 적용한다.

```bash
# (온라인 PC에서) 필요한 CRD들만 폴더째로 수집
$ git clone --depth=1 https://github.com/cert-manager/cert-manager.git

# CRDs 리소스 정의
$ cd cert-manager/crds
$ kubectl apply -f cert-manager.io_clusterissuers.yaml
$ kubectl apply -f cert-manager.io_issuers.yaml
$ kubectl apply -f cert-manager.io_certificates.yaml
$ kubectl apply -f cert-manager.io_certificaterequests.yaml
$ kubectl apply -f acme.cert-manager.io_orders.yaml
$ kubectl apply -f acme.cert-manager.io_challenges.yaml
```

![CRD 리소스 배포](/assets/img/post/helm/CRD%20리소스%20배포.png)

* * *

- CRDs 리소스 배포 확인한다.

```bash
$ kubectl get crd | grep cert-manager.io
```

![CRDs 리소스 배포 확인](/assets/img/post/helm/CRDs%20리소스%20배포%20확인.png)

* * *

### Vault Token을 Secret으로 만들기 :

- tokenSecretRef가 가리키는 Secret을 cert-manager 네임스페이스의 생성한다.

```bash
$ kubectl -n cert-manager create secret generic cert-manager-vault-auth \
  --from-literal=token='<VAULT_TOKEN>'
```

![Vault 토큰을 Secret 생성](/assets/img/post/helm/Vault%20토큰을%20Secret%20생성.png)

> ClusterIssuer는 클러스터 스코프이지만, cert-manager는 자기 네임스페이스의 Secret만 읽는다.
{: .prompt-warning}

* * *

### ClusterIssuer 생성 파일 작성 및 적용하기 :

```yaml
# vault-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: vault-issuer
spec:
  vault:
    server: http://vault.test.com:8200
    path: pki/sign/secloudit-certificates
    auth:
      tokenSecretRef:
        name: cert-manager-vault-auth
        key: token
```

```bash
$ kubectl apply -f vault-issuer.yaml
$ kubectl describe clusterissuer vault-issuer
```

![vault-issuer 배포](/assets/img/post/helm/vault-issuer%20배포.png)

* * *