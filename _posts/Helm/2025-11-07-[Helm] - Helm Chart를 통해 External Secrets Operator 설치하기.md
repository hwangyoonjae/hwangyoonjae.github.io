---
layout: post
title: "Helm Chart를 통해 External Secrets Operator 설치하기"
date: 2025-11-07
categories: [컨테이너, Helm] 
tags: [Helm, Cert-manager]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. External Secrets Operator 이해하기 :
### 1.1 External Secrets Operator란? :

- 외부 비밀 저장소(Vault, AWS Secrets Manager, GCP Secret Manager 등)에 저장된 값을 자동으로 Kubernetes Secret으로 동기화해주는 오퍼레이터(Controller)이다.

![external secret operator 아키텍처](/assets/img/post/helm/external%20secret%20operator%20아키텍처.png)

* * *

### 1.2 External Secrets Operator가 왜 필요한가? :

- 외부 API를 통해 비밀 정보를 자동으로 동기화하여 Kubernetes Secrets을 사용할 수 있다는 점이다.
- 외부 시스템에서 Secret이 변경되면, Kubernetes에서 자동으로 이를 반영할 수 있어 운영 부담을 크게 줄일 수 있습니다.

* * *

### 1.3 Cert-manager와 ESO 비교하기 :

| 항목 | External Secrets Operator | cert-manager |
|:--:|:--|:--|
| **목적** | 외부 저장소의 비밀(Secrets, 인증서 등)을 K8s Secret으로 가져오기 | CA/Vault/ACME로부터 인증서를 발급받기 |
| **주요 대상** | Vault, AWS Secrets Manager, GCP Secret Manager 등 | Let's Encrypt, Vault PKI, Self-signed CA 등 |
| **관리 단위** | KV 키 단위(任意 데이터) | X.509 Certificate (자동 만료, 재발급) |
| **Secret 생성 방식** | 외부 값 복사(Sync) | 인증서 생성(Sign) |
| **갱신 방식** | refreshInterval (값 변경 감지) | RenewBefore (만료일 기반 자동 재발급) |
| **함께 쓰는 경우** | Vault가 인증서 저장소일 때, ESO로 앱에 배포 | Vault가 CA일 때, cert-manager가 자동 발급 |

> Vault가 PKI 엔진을 발급자(CA)로 쓸 땐 → cert-manager + vault-issuer
>
> Vault가 인증서 저장소(저장만)로 쓸 땐 → ESO(External Secrets Operator)
{: .prompt-info}

* * *

## 2. External Secrets Operator 설치하기 :
### 2.1 External Secrets Operator Helm Chart 다운로드 :

```bash
$ helm repo add external-secrets https://charts.external-secrets.io
$ helm repo update
$ helm repo list
```

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면됩니다.

```bash
$ helm pull external-secrets/external-secrets --version 0.20.3
```

* * *

### 2.2 External Secrets Operator Container Image 다운로드 :

```bash
$ docker pull ghcr.io/external-secrets/external-secrets:v0.20.3
```

* * *

### 2.3 External Secrets Operator Helm Chart 설치하기 :

```bash
# namespace 생성
$ kubectl create namespace external-secrets

# 설치 진행
$ helm install external-secrets ./ \
  -n external-secrets \
  --set installCRDs=true \
  --set rbac.create=true \
  --set serviceAccount.create=true
```

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n external-secrets

# crd 목록 조회
$ kubectl get crd | grep external-secrets.io

# api-resources 목록 조회
$ kubectl api-resources | grep -i secretstore
```

![extermal secret operator 목록 확인](/assets/img/post/helm/extermal%20secret%20operator%20목록%20확인.png)

* * *

## 3. Vault Secret 연동 구성 등록하기 :
### 3.1 ClusterSecretStore 설치하기 :

- External Secrets Operator에서 사용할 **Vault 연동 설정(ClusterSecretStore)**을 등록합니다.

```yaml
apiVersion: external-secrets.io/v1
kind: ClusterSecretStore
metadata:
  name: vault-wildcard
spec:
  provider:
    vault:
      server: "https://[Vault_Server_Domain]:8200"
      path: "secret"
      version: "v2"
      caBundle: [rootCA.crt_BASE64] # cat rootCA.crt | base64 | tr -d '\n' 명령어로 확인
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "eso-wildcard-role"
          serviceAccountRef:
            name: external-secrets        # ← 이 둘이
            namespace: external-secrets   # ← Vault Role과 일치해야 함
```

```bash
$ kubectl apply -f ClusterSecretStore.yaml
```

* * *

### 3.2 ClusterSecretStore 상태 확인하기 :

- Vault와의 인증 및 연결 검증을 확인합니다.

```bash
$ kubectl get clustersecretstore
```

![clustersecretstore 상태 확인](/assets/img/post/helm/clustersecretstore%20상태%20확인.png)

| 상태 | 의미 |
|:--:|:--|
| **True ✅** | ESO 컨트롤러가 Vault(또는 다른 외부 비밀 저장소)에 성공적으로 로그인하여 연결 테스트를 통과함 |
| **False ❌** | 인증 실패, 권한 문제, 네트워크 오류 등으로 외부 저장소 연결에 실패함 |
| **Unknown ⏳** | 아직 ESO 컨트롤러가 해당 리소스를 평가 중 (초기 생성 직후 잠깐 나올 수 있음) |


* * *

## 4. Vault의 K8s Auth 설정하기 :
### 4.1 “리뷰어 토큰”을 명시적으로 설정하기 :
- K8s 1.24+에서는 자동 생성되는 SA Secret이 없으므로, Vault에 줄 token_reviewer_jwt를 직접 발급해서 넣어줘야합니다.

```bash
# kube-system(또는 vault 네임스페이스)에 리뷰어 SA 생성
$ kubectl -n kube-system create serviceaccount vault-reviewer

# 방법 A: 내장 ClusterRole 사용
$ kubectl create clusterrolebinding vault-reviewer-binding \
  --clusterrole=system:auth-delegator \
  --serviceaccount=kube-system:vault-reviewer
```

* * *

### 4.2 리뷰어 토큰 발급하기 :

```bash
# 새 reviewer 토큰 발급(24h 유효)
$ kubectl -n kube-system create token vault-reviewer --duration=24h > reviewer.jwt
```

> 리뷰어 SA/RBAC를 지우면, 이전에 Vault에 저장된 token_reviewer_jwt가 더 이상 유효하지 않아 재생성 진행해야합니다.
{: .prompt-warning}

* * *

### 4.3 Vault에 K8s Auth 재설정하기 :

- ESO 파드의 SA 토큰 aud가 보통 "https://kubernetes.default.svc.cluster.local"이므로, Role의 token_audiences를 아예 제거(미지정)하거나, 정확히 그 값 맞춰야합니다.

```bash
# (선택1) audiences 미지정으로 단순화
vault write auth/kubernetes/role/eso-wildcard-role \
  bound_service_account_names=external-secrets \
  bound_service_account_namespaces=external-secrets \
  token_policies=policy-wildcard-tls \
  token_ttl=24h

# (선택2) aud를 명시해 고정
vault write auth/kubernetes/role/eso-wildcard-role \
  bound_service_account_names=external-secrets \
  bound_service_account_namespaces=external-secrets \
  bound_audiences="https://kubernetes.default.svc.cluster.local \
  token_policies=policy-wildcard-tls \
  token_ttl=24h"
```

* * *

### 4.4 ESO가 쓰는 SA/경로 최종 확인하기 :

- ESO Deployment의 serviceAccountName이 실제로 external-secrets인지 확인합니다.

```bash
$ kubectl -n external-secrets get deploy external-secrets \
  -o jsonpath='{.spec.template.spec.serviceAccountName}{"\n"}'
```

* * *

### 4.5 ClusterSecretStore 연결 검증 및 ESO 재시작하기 :

```bash
# ESO 컨트롤러 재시작(선택)
$ kubectl -n external-secrets rollout restart deploy/external-secrets

# 상태 확인
$ kubectl get clustersecretstore vault-wildcard -o jsonpath='{range .status.conditions[*]}{.type}={.status} {.reason} {.message}{"\n"}{end}'

# 컨트롤러 로그
$ kubectl -n external-secrets logs deploy/external-secrets --tail=200 | grep -i 'kubernetes/login' -n || true
```

* * *

## 5. ExternalSecret 생성하기 :
### 5.1 각 Namespace 별 ExternalSecret 생성하기 :

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: argocd-server-tls
  namespace: argocd
spec:
  refreshInterval: 1h
  secretStoreRef:
    kind: ClusterSecretStore  # ESO의 동기화 주기
    name: vault-wildcard
  target:
    name: argocd-server-tls   # Argo CD values.yaml의 server.certificate.secretName과 일치
    creationPolicy: Owner
    template:
      type: kubernetes.io/tls
  data:
    - secretKey: tls.crt
      remoteRef: { key: tls/wildcard, property: tls.crt }
    - secretKey: tls.key
      remoteRef: { key: tls/wildcard, property: tls.key }
```

* * *