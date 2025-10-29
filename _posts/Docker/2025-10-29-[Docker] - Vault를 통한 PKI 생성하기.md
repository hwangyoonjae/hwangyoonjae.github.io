---
layout: post
title: "Vault를 통한 PKI 생성하기"
date: 2025-10-29
categories: [컨테이너, Docker]
tags: [Docker, vault]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## Vault 설치하기 :
- 해당 작업을 위해 사전에 vault가 구축되어있어야 한다.
> * [vault 설치하기](https://hwangyoonjae.github.io/posts/Docker-Vault-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0/ "vault 설치하기")

* * *

## PKI 생성하기 :
### Vault Container 접속하기 :

```bash
$ docker exec -it vault sh

vault login [Root Token]
Error authenticating: error looking up token: Get "https://127.0.0.1:8200/v1/auth/token/lookup-self": http: server gave HTTP response to HTTPS client
export vault_ADDR='http://127.0.0.1:8200'
vault login [Root Token]
```

![vault CLI 로그인 진행](/assets/img/post/docker/vault%20CLI%20로그인%20진행.png)

> vault 로그인 진행 시 Error 발생하는 이유
> vault 서버는 HTTP로 열려 있는데, vault CLI는 HTTPS로 접속하려고 해서 생긴 문제다.
{: .prompt-warning}

* * *

### PKI 엔진 활성화하기 :

- vault는 처음에 PKI 기능이 꺼져있어 활성화해야한다.

```bash
vault secrets enable pki
vault secrets tune -max-lease-ttl=87600h pki
```

![vault PKI 엔진 활성화](/assets/img/post/docker/vault%20PKI%20엔진%20활성화.png)

* * *

### Root CA 생성하기 :

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

> 실제 운영에서는 pki/root 대신 pki_int를 만들어 중간 CA(Intermediate CA) 로 구성하고, 조직의 루트 CA로부터 서명받는 구조로 쓴다.
{: .prompt-tip}

* * *

### CA URL 구성하기 :

- 외부에서 인증서 체인과 CRL을 가져갈 수 있게 URL을 등록한다.

```bash
vault write pki/config/urls \
  issuing_certificates="http://[vault_Domain]:8200/v1/pki/ca" \
  crl_distribution_points="http://[vault_Domain]:8200/v1/pki/crl"
```

![vault CA URL 구성](/assets/img/post/docker/vault%20CA%20URL%20구성.png)

* * *

### 인증서 발급용 Role 생성하기 :

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

### cert-manager용 Policy 생성하기 :

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

### cert-manager용 Token 발급하기 :

- cert-manager가 Vault API를 호출할 때 사용할 토큰을 발급한다.

```bash
vault token create -policy=pki-sign -ttl=17520h -format=json | grep '"client_token"' | cut -d'"' -f4
```

> 위 명령어를 통해서 발급된 TOKEN 값을 복사한다.
{: .prompt-tip}

![vault cert-manager용 Token 발급](/assets/img/post/docker/vault%20cert-manager용%20Token%20발급.png)

* * *

### Kubernetes에 Vault Token 등록하기 :

- cert-manager가 있는 namespace(cert-manager)에 Secret 생성한다.

```bash
$ kubectl -n cert-manager create secret generic cert-manager-vault-auth \
  --from-literal=token='[vault_cert_manager_token]'
```

![vault Kubernetes에 Vault Token 등록](/assets/img/post/docker/vault%20Kubernetes에%20Vault%20Token%20등록.png)

* * *

### ClusterIssuer 생성하기 :

- cert-manager가 Vault를 사용해서 인증서를 발급할 수 있도록 설정한다.

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: vault-issuer
spec:
  vault:
    server: http://vault.test.com:8200
    path: pki/sign/test-certificates
    auth:
      tokenSecretRef:
        name: cert-manager-vault-auth
        key: token
```

![vault ClusterIssuer 생성](/assets/img/post/docker/vault%20ClusterIssuer%20생성.png)

* * *

- cert-manager가 vault를 통해 인증서를 발급하도록 지시하는 Certificate 리소스 생성 시 아래와 같이 알림(Warning) 발생한다.

```html
# 단순히 “기본값이 바뀌었어요” 라는 안내 메시지
Warning: spec.privateKey.rotationPolicy: In cert-manager >= v1.18.0,
the default value changed from `Never` to `Always`.
```

- cert-manager가 인증서를 갱신할 때(예: 만료가 다가오면 새로 발급할 때), 기존 private key를 계속 쓸지, 새로 만들지를 결정하는 설정이 있다.

```yaml
spec:
  privateKey:
    rotationPolicy: <값>
```

| 값 | 의미 |
|------|------|
| **Never** | 인증서가 갱신될 때 기존 private key를 그대로 사용함 |
| **Always** | 인증서가 갱신될 때 새 private key를 새로 생성함 |

> cert-manager 1.18에서 바뀐 점
> 기존(1.17 이하)은 기본이 Never라 한 번 생성된 private key는 계속 재사용했다.
> 하지만 1.18부터는 기본이 Always로 바뀌면서 인증서가 갱신될 때마다 새 key를 자동으로 만들어 준다.
{: .prompt-info}

* * *