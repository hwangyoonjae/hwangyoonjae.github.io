---
layout: post
title: "Vault를 통한 PKI 생성하기"
date: 2025-10-29
categories: [컨테이너, Docker]
tags: [Docker, vault]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## 1. Vault 설치하기 :
- 해당 작업을 위해 사전에 vault가 구축되어있어야 한다.
> * [vault 설치하기](https://hwangyoonjae.github.io/posts/Docker-Vault-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0/ "vault 설치하기")

* * *

## 2. PKI 생성하기 :
### 2.1 Vault Container 접속하기 :

```bash
$ docker exec -it vault sh

vault login [Root Token]
Error authenticating: error looking up token: Get "https://127.0.0.1:8200/v1/auth/token/lookup-self": http: server gave HTTP response to HTTPS client
export VAULT_ADDR='http://127.0.0.1:8200'
vault login [Root Token]
```

![vault CLI 로그인 진행](/assets/img/post/docker/vault%20CLI%20로그인%20진행.png)

> vault 로그인 진행 시 Error 발생하는 이유
> 
> vault 서버는 HTTP로 열려 있는데, vault CLI는 HTTPS로 접속하려고 해서 생긴 문제다.
{: .prompt-warning}

* * *

### 2.2 PKI 엔진 활성화하기 :

- vault는 처음에 PKI 기능이 꺼져있어 활성화해야한다.

```bash
vault secrets enable pki
vault secrets tune -max-lease-ttl=87600h pki
```

![vault PKI 엔진 활성화](/assets/img/post/docker/vault%20PKI%20엔진%20활성화.png)

* * *

### 2.3 Root CA 생성하기 :

- 테스트용 Root CA를 vault 내부에서 직접 생성한다.

```bash
vault write pki/root/generate/internal \
  common_name="test.com Root CA" \
  organization="TEST" \
  ou="DevOps" \
  country="KR" \
  province="Seoul" \
  locality="Seoul" \
  postal_code="00000" \
  ttl=87600h
```

![vault Root CA 생성](/assets/img/post/docker/vault%20Root%20CA%20생성.png)

> 실제 운영에서는 pki/root 대신 pki_int를 만들어 중간 CA(Intermediate CA)로 구성하고, 조직의 루트 CA로부터 서명받는 구조로 쓴다.
{: .prompt-tip}

* * *

### 2.4 CA URL 구성하기 :

- 외부에서 인증서 체인과 CRL을 가져갈 수 있게 URL을 등록한다.

```bash
vault write pki/config/urls \
  issuing_certificates="http://[vault_Domain]:8200/v1/pki/ca" \
  crl_distribution_points="http://[vault_Domain]:8200/v1/pki/crl"
```

![vault CA URL 구성](/assets/img/post/docker/vault%20CA%20URL%20구성.png)

* * *

### 2.5 인증서 발급용 Role 생성하기 :

- 이 Role은 cert-manager나 사용자들이 CSR을 제출할 때 어떤 인증서를 발급할 수 있는지 정의한다.

```bash
vault write pki/roles/test-certificates \
  allowed_domains="test.com" \
  allow_subdomains=true \
  allow_bare_domains=false \
  allow_glob_domains=false \
  require_cn=false \
  use_csr_sans=true \
  key_type="rsa" \
  key_bits=2048 \
  server_flag=true \
  client_flag=false \
  key_usage="DigitalSignature,KeyEncipherment" \
  max_ttl="2160h"
```

![vault 인증서 발급용 Role 생성](/assets/img/post/docker/vault%20인증서%20발급용%20Role%20생성.png)

* * *

### 2.6 cert-manager용 Policy 생성하기 :

- cert-manager가 Vault에서 인증서를 서명받을 때 필요한 최소 권한 정책을 만들어준다.

```bash
cat > /vault/config/pki-sign.hcl <<'EOF'
path "pki/sign/secloudit-certificates" {
  capabilities = ["update"]
}
EOF

vault policy write pki-sign /vault/config/pki-sign.hcl
```

![vault cert-manager용 Policy 생성](/assets/img/post/docker/vault%20cert-manager용%20Policy%20생성.png)

* * *