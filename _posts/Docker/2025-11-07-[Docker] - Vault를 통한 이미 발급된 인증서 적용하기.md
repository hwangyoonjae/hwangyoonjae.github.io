---
layout: post
title: "Vault를 통한 이미 발급된 인증서 적용하기"
date: 2025-11-07
categories: [컨테이너, Docker]
tags: [Docker, vault]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## 1. Vault에 HTTPS 인증서 사용하기 :
### 1.1 Vault 이미 발급된 인증서 볼륨 마운트하기 :

- Vault 설정 파일의 인증서 경로 작성한다.

```bash
$ vi config/vault.hcl

#===========================
listener "tcp" {
  address     = "0.0.0.0:8200"
  cluster_address = "0.0.0.0:8201"
  #tls_disable = 1 

  # 기존 tls_disable=1은 주석 처리 또는 삭제
  tls_cert_file   = "/vault/certs/server.cert"
  tls_key_file    = "/vault/certs/server.key"
```

* * *

- Vault 컨테이너 기동 시 인증서 마운트 경로 수정 후 컨테이너 재시작한다.

```bash
$ vi docker-compose.yaml

#===========================
services:
    volumes:
      - ./certs:/vault/certs:Z # volume 마운트 경로 추가 작성
```

```bash
$ docker compose down -v

$ docker compose up -d
```

* * *

## 2. Vault에 인증서 저장 및 ESO 접근 정책 구성하기 :
### 2.1 Vault KV v2 시크릿 엔진 활성화 + 값 넣기 :

```bash
# 환경변수 등록
export VAULT_ADDR="https://vault.inno.com:8200"
export VAULT_CACERT="/vault/certs/fullchain.crt"

# (최초 1회) KV v2 마운트
vault secrets enable -path=secret kv-v2

# 배포용 fullchain 생성 (이미 했다면 생략)
cat server.crt server_chain.crt > fullchain.crt

# 와일드카드 인증서 저장
vault kv put secret/tls/wildcard \
  tls.crt=@fullchain.crt \
  tls.key=@server.key

# 확인
vault kv get secret/tls/wildcard
```

* * *

### 2.2 Vault 읽기 전용 정책 생성하고 적용하기 :

```bash
cat > /vault/config/policy-wildcard-tls.hcl <<'EOF'
path "secret/data/tls/wildcard" {
  capabilities = ["read"]
}
EOF

# 적용
vault policy write policy-wildcard-tls policy-wildcard-tls.hcl
```

* * *

### 2.3 Kubernetes Auth 설정하기 :

```bash
# Kubernetes 인증 방식(Kubernetes Auth Method)”을 사용하도록 활성화
vault auth enable kubernetes

# 쿠버네티스 API 서버 주소/CA로 설정 (환경 맞게 치환)
vault write auth/kubernetes/config \
  kubernetes_host="https://<K8S_API_HOST>:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  disable_iss_validation=true \
  disable_local_ca_jwt=true
```

* * *

### 2.4 Vault가 클러스터 외부 (Docker / VM) 에서 동작한다면 :

- 인증서는 API 서버 인증서(/etc/kubernetes/pki/apiserver.crt)를 서명한 루트로 Vault 서버가 실행 중인 곳으로 옮기고, 경로를 지정한다.

```bash
# Master Node에서 진행
cat /etc/kubernetes/pki/ca.crt
```

```bash
vault write auth/kubernetes/config \
  kubernetes_host="https://<API_SERVER_IP>:6443" \
  kubernetes_ca_cert=@/vault/config/k8s-ca.crt \
  disable_local_ca_jwt=true \
  token_reviewer_jwt=@/vault/config/reviewer.jwt
```

> kubernetes_ca_cert의 값은 **/etc/kubernetes/pki/ca.crt** 값이다.
{: .prompt-info}

* * *

- **token_reviewer_jwt** 값은 아래 명령어를 통해 확인한다.

```bash
# --duration 옵션을 통해 JWT의 유효기간을 지정할 수 있다.
$ kubectl -n kube-system create token vault-auth --duration=720h > vault-auth.jwt
```
> token_reviewer_jwt의 값은 Vault가 Kubernetes API에 TokenReview 요청을 보낼 때 사용할 **리뷰용 ServiceAccount JWT**이다
{: .prompt-info}

![vault k8s 인증서 적용](/assets/img/post/docker/vault%20k8s%20인증서%20적용.png)

* * *

### 2.5 ESO용 Role 생성하기 :

- ESO가 사용할 ServiceAccount와 Namespace에 바인딩한다.

```bash
vault write auth/kubernetes/role/eso-wildcard-role \
  bound_service_account_names="external-secrets" \
  bound_service_account_namespaces="external-secrets" \
  audience="https://kubernetes.default.svc.cluster.local" \
  token_policies="eso-wildcard-policy" \
  ttl="24h"
```

![ESO용 Role 생성하기](/assets/img/post/docker/ESO용%20Role%20생성하기.png)

* * *

## 3. 인증서 변경하기 :
### 3.1 변경할 인증서를 Vault/ESO에 바로 적용하기 :

- 인증서가 만료되어 변경해야하는 경우 아래와 같이 진행한다.

```bash
# vault 로그인 진행
vault login [Root Token]

# Vault에 등록
vault kv put secret/tls/wildcard \
  tls.crt=@fullchain.crt \
  tls.key=@server.key
```

* * *

### 3.2 서버 인증서가 전체 변경된 경우 :

- ClusterSecretStore 안의 caBundle 값 또는 ESO의 CA 신뢰 설정을 변경된 인증서 값으로 적용해야한다.

```bash
# Base64 인코딩 값 확인
$ cat rootCA.crt | base64 | tr -d '\n'
```

![rootCA 인증서 base64 인코딩](/assets/img/post/docker/rootCA%20인증서%20base64%20인코딩.png)

* * *

- ClusterSecretStore의 caBundle 값을 변경한다.

```yaml
      path: "secret"
      version: "v2"
      caBundle: "[여기에 새 base64 값]"
```

* * *

- ESO 정상 동작 확인한다.

```bash
$ kubectl get clustersecretstore
```

![ESO 정상 동작 확인](/assets/img/post/docker/ESO%20정상%20동작%20확인.png)

* * *