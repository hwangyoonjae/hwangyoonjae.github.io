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

- Vault 설정 파일의 인증서 경로 작성합니다.

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

- Vault 컨테이너 기동 시 인증서 마운트 경로 수정 후 컨테이너 재시작합니다.

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
export VAULT_ADDR="https://127.0.0.1:8200"
export VAULT_CACERT="/vault/certs/fullchain.crt"

# 와일드카드 인증서 저장
vault kv put secret/certs/ssl \
  certificate="$(cat server.crt)" \
  private_key="$(cat server.key)" \
  chain="$(cat fullchain.crt)"

# 확인
vault kv get secret/tls/wildcard
```

* * *

### 2.2 Vault 읽기 전용 정책 생성하고 적용하기 :

```bash
# ESO용 Vault Policy 생성
vault policy write eso-ingress-tls-policy - <<EOF
path "secret/data/certs/ssl" {
  capabilities = ["read"]
}

path "secret/metadata/certs/ssl" {
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
vault write auth/kubernetes/role/eso-ingress-tls \
  bound_service_account_names=external-secrets \
  bound_service_account_namespaces=external-secrets \
  policies=eso-ingress-tls-policy \
  ttl=24h
```

* * *

## 3. 인증서 변경하기 :
### 3.1 변경할 인증서를 Vault/ESO에 바로 적용하기 :

- 인증서가 만료되어 변경해야하는 경우 아래와 같이 진행합니다.

```bash
# vault 로그인 진행
vault login [Root Token]

# Vault에 등록
vault kv put secret/certs/ssl \
  certificate="$(cat server.crt)" \
  private_key="$(cat server.key)" \
  chain="$(cat fullchain.crt)"
```

* * *

### 3.2 서버 인증서가 전체 변경된 경우 :

- ClusterSecretStore 안의 caBundle 값 또는 ESO의 CA 신뢰 설정을 변경된 인증서 값으로 적용해야합니다.

```bash
# Base64 인코딩 값 확인
$ cat rootCA.crt | base64 | tr -d '\n'
```

![rootCA 인증서 base64 인코딩](/assets/img/post/docker/rootCA%20인증서%20base64%20인코딩.png)

* * *

- ClusterSecretStore의 caBundle 값을 변경합니다.

```yaml
      path: "secret"
      version: "v2"
      caBundle: "[여기에 새 base64 값]"
```

* * *

- ESO 정상 동작 확인합니다.

```bash
$ kubectl get clustersecretstore
```

![ESO 정상 동작 확인](/assets/img/post/docker/ESO%20정상%20동작%20확인.png)

* * *