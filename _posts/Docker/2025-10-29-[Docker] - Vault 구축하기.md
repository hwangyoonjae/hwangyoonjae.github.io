---
layout: post
title: "Vault 구축하기"
date: 2025-10-29
categories: [컨테이너, Docker]
tags: [Docker, Vault]
image: /assets/img/post-title/docker_wallpaper.jpg
---

## 1. Vault란? :
- ashiCorp에 의해서 개발된 크로스플랫폼 패스워드 및 인증 관리 시스템입니다. 공개되면 안되는 비밀번호, API 키, 토큰 등을 저장하고 관리합니다.

* * *

## 2. Vault 설치하기 :
### 2.1 Vault 이미지 다운로드 :

```bash
$ docker pull hashicorp/vault:{version}
```

![vault 이미지 다운로드](/assets/img/post/docker/vault%20이미지%20다운로드.png)

> docker에서 vault와 hashicorp/vault의 차이
> 'vault'와 'hashicorp/vault'는 모두 HashiCorp Vault의 Docker 이미지를 나타낸다.
> 'hashicorp/vault'는 보통 최신 기능과 업데이트를 더 자주 제공하므로, 최신 Vault 기능을 사용하려면 'hashicorp/vault'를 사용하는 것이 좋다.
{: .prompt-info}

* * *

### 2.3 Vault Docker Compose 생성하기 :

- vault 디렉토리 구조는 아래와 같습니다.

```bash
/vault
├─ docker-compose.yml
├─ config/
│  └─ vault.hcl
└─ data/            
```

* * *

- docker-compose.yaml 파일을 수정합니다.

```yaml
version: "3.8"

services:
  vault:
    image: [vault_container_image_name]:[vault_container_image_tag]
    container_name: vault
    restart: unless-stopped
    cap_add:
      - IPC_LOCK                    # 메모리 스왑 방지(권장)
    ulimits:
      memlock: -1
    environment:
      VAULT_LOCAL_CONFIG: |         # 기본값: config/vault.hcl을 사용하므로 없어도 됨
        {}
    volumes:
      - ./config:/vault/config:Z
      - ./data:/vault/data:Z
      - ./logs:/vault/logs:Z
    ports:
      - "8200:8200"                 # API
      - "8201:8201"                 # Cluster RPC(추후 HA 확장 시)
    command: vault server -config=/vault/config/vault.hcl
```

* * *

- config/vault.hcl 작성합니다.

```bash
ui = true

# 실습/내부망 전용: TLS 비활성 (프로덕션은 listener.tcp.tls_cert_file/ tls_key_file 설정 필수)
listener "tcp" {
  address     = "0.0.0.0:8200"
  cluster_address = "0.0.0.0:8201"
  #tls_disable = 1

  # 기존 tls_disable=1 은 주석 처리 또는 삭제
  tls_cert_file   = "/vault/certs/fullchain.crt"
  tls_key_file    = "/vault/certs/server.key"

storage "raft" {
  path    = "/vault/data"
  node_id = "vault-node-1"
}

api_addr     = "http://[vault_domain]:8200"
cluster_addr = "http://[vault_domain]:8201"

# (선택) 서버 로그 파일 출력
# disable_mlock = true # 컨테이너에 cap IPC_LOCK 추가했으므로 보통 불필요
log_level = "info"
```

* * *

- Docker Compose 파일을 통하여 Vault 컨테이너를 설치합니다.

```bash
$ docker compose up -d

# vault 컨테이너 로그 확인
$ docker logs -f vault
```

![vault 컨테이너 설치](/assets/img/post/docker/vault%20컨테이너%20설치.png)

* * *

## 3. Vault UI 접속하기 :

- 브라우저에서 Vault UI 접속합니다.

![vault 초기 화면](/assets/img/post/docker/vault%20초기%20화면.png)

| 항목 | 설명 |
|------|------|
| **Join an existing Raft cluster** | 이미 초기화된 Vault 노드가 존재하고, 현재 노드는 그 클러스터에 두 번째 이상 멤버로 합류할 때 사용 |
| **Create a new Raft cluster** | 현재 이 Vault가 클러스터의 첫 번째 노드(Leader)로서 새 Raft 저장소를 초기화할 때 사용 (대부분의 단일 서버 또는 첫 구성 시 선택해야 함) |

> “Create a new Raft cluster”를 선택합니다.
{: .prompt-info}

* * *

- Vault는 기동 시마다 자동으로 잠긴(sealed) 상태이고, 운영 중에도 서버가 재시작되면 반드시 언실(unseal)해야 합니다.

![vault sealed 화면](/assets/img/post/docker/vault%20sealed%20화면.png)

| 항목 | 설명 | 예시 | 권장 값 |
|------|------|------|------|
|**Key shares**|Vault의 마스터 키를 몇 개로 분할할지 (총 분할 수)|5로 설정하면 총 5개의 Unseal Key 생성|5|
|**Key threshold**|Vault를 언실(Unseal)하려면 그 중 몇 개가 필요한지|3으로 설정하면 5개 중 3개를 입력해야 언실됨|3|

> 즉, 5개 중 3개만 모여도 Vault를 열 수 있게 되는 구조로, Shamir’s Secret Sharing 알고리즘 기반입니다.
> (보안 + 복구 편의성을 동시에 확보)
{: .prompt-info}

* * *

- Vault는 항상 기동 시 자동으로 “Sealed(잠김)” 상태로 시작하기 때문에, 잠금해제를 진행해야하는 화면입니다.

![vault unseal 화면](/assets/img/post/docker/vault%20unseal%20화면.png)

| 항목 | 설명 |
|------|------|
| **Unseal Key** | Portion 입력칸 위에서 받은 5개의 Unseal Key 중에서 하나를 입력하는 곳입니다.
| **Unseal** | 버튼 입력 후 클릭하면, Vault가 이 키 조각을 기억하고 몇 개가 더 필요한지 알려준다. 예를 들어 threshold가 3이면, “1 of 3 unseal keys provided” 식으로 표시됩니다.

* * *

- Vault가 초기화 후 로그인 할 수 있는 관리자 토큰과 Unseal Key 조각들을 확인합니다.

![vault 초기화 완료 화면](/assets/img/post/docker/vault%20초기화%20완료%20화면.png)

| 항목 | 설명 |
|------|------|
| **Initial Root Token** | Vault에 로그인할 수 있는 “최고 관리자 토큰”입니다. UI나 CLI에서 인증 시 반드시 필요하다. (예: vault login <root-token>) |
| **Key 1 ~ Key 5**	| Vault의 마스터 키를 분할(Shamir Secret Sharing)한 Unseal Key 조각들입니다. Vault가 재시작되거나 sealed 상태일 때, 이 중 일부(threshold 수 만큼)를 입력해야 Vault를 “해제(Unseal)”할 수 있습니다.

> Initial Root Token이랑 Key 1~3 값을 복사하고, [Continue to Unseal] 버튼을 클릭하면 다음 단계로 이동합니다.
{: .prompt-info}

* * *

### 3.1 Vault 로그인 하기 :

- 위 단계에서 복사한 **Initial Root Token** 값을 입력하여 로그인 합니다.

![vault 로그인 화면](/assets/img/post/docker/vault%20로그인%20화면.png)
![vault 로그인 후 화면](/assets/img/post/docker/vault%20로그인%20후%20화면.png)

* * *
