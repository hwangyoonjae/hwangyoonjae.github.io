---
layout: post
title: "Helm Chart를 통해 cert-manager 설치하기"
date: 2025-10-27
categories: [컨테이너, Helm] 
tags: [Helm, Cert-manager]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. cert-manager 설치하기 :
### 1.1 cert-manager helm chart 다운로드 :

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

### 1.2 cert-manager container image 다운로드 :

- helm chart를 통해 설치될 Application version대로 이미지를 다운로드합니다.

```bash
$ docker pull quay.io/jetstack/cert-manager-controller:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-webhook:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-cainjector:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-acmesolver:v1.18.3
$ docker pull quay.io/jetstack/cert-manager-startupapicheck:v1.18.3
```

![cert-manager 이미지 다운로드](/assets/img/post/helm/cert-manager%20이미지%20다운로드.png)

* * *

### 1.3 Helm을 통해 Cert-Manager 설치하기 :

- cert-manager 설치를 위해 네임스페이스를 생성합니다.

```bash
$ kubectl create namespace cert-manager
```

* * *

- values.yaml 파일에 값을 수정합니다.

```bash
$ vi values.yaml
```
{% raw %}
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
{% endraw %}

* * *

- 쿠버네티스에 cert-manager 애플리케이션을 설치합니다.

```bash
$ helm install cert-manager ./ \
> -n cert-manager \
> --wait --timeout 10m \
> -f values.yaml
```

![helm chart를 cert-manager 통한 배포](/assets/img/post/helm/helm%20chart를%20cert-manager%20통한%20배포.png)

* * *

- 쿠버네티스 클러스터의 정상적으로 설치되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n cert-manager

# cert-manager에 설치된 워크로드 목록 조회
$ kubectl get all -n cert-manager
```

![cert-manager workload 목록 확인](/assets/img/post/helm/cert-manager%20workload%20목록%20확인.png)

* * *

## 2. cert-manager CRDs(CustomResourceDefinitions) 설치하기 :
### 2.1 CRDs란? :

- 쿠버네티스에 새로운 리소스 타입(API 오브젝트)을 추가하기 위한 확장 메커니즘입니다.

> 쿠버네티스 기본에는 없는 리소스를 “추가로 등록”해서 kubectl get certificate, kubectl get issuer 같은 명령을 사용할 수 있게 만들어주는 정의 파일입니다.
{: .prompt-info}

* * *

### 2.2 cert-manager가 추가하는 CRD 종류 :

- TLS 인증서 자동화를 위해 아래와 같은 커스텀 리소스(Custom Resources) 를 정의하고 사용합니다.

| CRD 리소스 | 역할 설명 |
|-------------|------------|
| **Certificate** (`certificates.cert-manager.io`) | 실제 발급받고 싶은 인증서 정보 (DNS, Secret 이름 등) |
| **Issuer** (`issuers.cert-manager.io`) | 특정 네임스페이스 전용 인증서 발급자 |
| **ClusterIssuer** (`clusterissuers.cert-manager.io`) | 클러스터 전체에서 사용할 수 있는 인증서 발급자 |
| **CertificateRequest** (`certificaterequests.cert-manager.io`) | cert-manager가 실제 발급 요청을 Vault나 CA에 전달할 때 사용하는 내부 요청 리소스 |
| **Order** (`orders.acme.cert-manager.io`) | ACME 프로토콜(Let’s Encrypt)에서 인증서 발급 과정을 추적하기 위한 내부 리소스 |
| **Challenge** (`challenges.acme.cert-manager.io`) | ACME 인증(HTTP-01, DNS-01 등) 과정을 나타내는 내부 리소스 |

> CRDs가 없으면 cert-manager는 아무런 커스텀 리소스를 인식하지 못해서 **kubectl apply -f certificate.yaml** 같은 명령이 오류가 발생합니다.
{: .prompt-tip}

* * *

### 2.3 CRDs 설치하기 :

- 차트만 오프라인으로 가져오면 CRD는 자동 설치되지 않기 때문에, Repo의 CRDs 디렉터리에서 직접 적용합니다.

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

- CRDs 리소스 배포 확인합니다.

```bash
$ kubectl get crd | grep cert-manager.io
```

![CRDs 리소스 배포 확인](/assets/img/post/helm/CRDs%20리소스%20배포%20확인.png)

* * *

### 2.4 Vault Token을 Secret으로 만들기 :

- tokenSecretRef가 가리키는 Secret을 cert-manager 네임스페이스의 생성합니다.

```bash
$ kubectl -n cert-manager create secret generic cert-manager-vault-auth \
  --from-literal=token='<VAULT_TOKEN>'
```

![Vault 토큰을 Secret 생성](/assets/img/post/helm/Vault%20토큰을%20Secret%20생성.png)

> ClusterIssuer는 클러스터 스코프이지만, cert-manager는 자기 네임스페이스의 Secret만 읽는다.
{: .prompt-warning}

* * *

### 2.5 ClusterIssuer 생성 파일 작성 및 적용하기 :

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

### 2.6 인증서 요청서 생성해보기 :

- 필자는 ArgoCD에 인증서를 자동으로 발급받도록 Certificate 리소스를 생성했다.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: argocd-tls
  namespace: argocd
spec:
  secretName: argocd-server-tls
  duration: 720h        # 30일 예시
  renewBefore: 120h     # 만료 5일 전 갱신
  dnsNames:
    - argocd.test.com
  issuerRef: # 발급자 서명
    kind: ClusterIssuer
    name: vault-issuer   # 위에서 만든 ClusterIssuer 이름
```

> 각 Namespace마다 사용할 Secret을 만들어야 합니다.
{: .prompt-warning}

* * *

- cert-manager가 vault를 통해 인증서를 발급하도록 지시하는 Certificate 리소스 생성 시 아래와 같이 알림(Warning) 발생합니다.

```html
# 단순히 “기본값이 바뀌었어요” 라는 안내 메시지
Warning: spec.privateKey.rotationPolicy: In cert-manager >= v1.18.0,
the default value changed from `Never` to `Always`.
```
![cert-manager 생성 시 warning 발생](/assets/img/post/helm/cert-manager%20생성%20시%20warning%20발생.png)

- cert-manager가 인증서를 갱신할 때(예: 만료가 다가오면 새로 발급할 때), 기존 private key를 계속 쓸지, 새로 만들지를 결정하는 설정이 있습니다.

```yaml
spec:
  privateKey:
    rotationPolicy: <값>
```

| 값 | 의미 |
|:------:|:------:|
| **Never** | 인증서가 갱신될 때 기존 private key를 그대로 사용함 |
| **Always** | 인증서가 갱신될 때 새 private key를 새로 생성함 |

> cert-manager 1.18에서 바뀐 점
> 
> 기존(1.17 이하)은 기본이 Never라 한 번 생성된 private key는 계속 재사용했다.
> 하지만 1.18부터는 기본이 Always로 바뀌면서 인증서가 갱신될 때마다 새 key를 자동으로 만들어 준다.
{: .prompt-info}

* * *