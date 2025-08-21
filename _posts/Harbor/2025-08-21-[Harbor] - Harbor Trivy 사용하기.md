---
layout: post
title: "Harbor Trivy 사용하기"
date: 2025-08-21
categories: Harbor
tags: [Harbor, Dockerhub, Image, Trivy]
image: /assets/img/post-title/harbor-wallpaper.jpg
---

## harbor.yml 수정하기:

```bash
$ vi harbor.yml
```

```yaml
trivy:
  ignore_unfixed: false        # 전체 리스크 가시화
  skip_update: true            # 외부 DB 다운로드 금지
  offline_scan: true           # 의존성 네트워크 조회 금지
  security_check: vuln         # 먼저 취약점만, 필요 시 'vuln,config,secret'
  insecure: false              # 검증 유지(사설 CA 신뢰 추가 권장)
```

![harbor.yml 수정](/assets/img/post/harbor/harbor.yml%20수정.png)

---

## Trivy 취약점 DB 다운로드:
### ORAS 릴리즈 설치하기:

```bash
$ curl -fL -o "oras_1.2.2_linux_amd64.tar.gz" \
  "https://github.com/oras-project/oras/releases/download/v1.2.2/oras_1.2.2_linux_amd64.tar.gz"

# 압축 해제 후 배치
$ sudo tar -xzf "oras_1.2.2_linux_amd64.tar.gz" oras
$ sudo install -m 0755 oras /usr/local/bin/oras

$ oras version
```

![oras 릴리즈 설치](/assets/img/post/harbor/oras%20릴리즈%20설치.png)

---

### 취약점 다운로드:

```bash
$ oras pull ghcr.io/aquasecurity/trivy-db:2
# -> db.tar.gz

$ oras pull ghcr.io/aquasecurity/trivy-java-db:1
# -> javadb.tar.gz

$ mkdir -p ./{db,java-db}

$ tar -xzf db.tar.gz     -C ./db
$ tar -xzf javadb.tar.gz -C ./java-db

$ ls -l ./db
# → trivy.db, metadata.json
$ ls -l ./java-db
# → trivy-java.db, metadata.json
```

![trivy 취약점 db 다운로드](/assets/img/post/harbor/trivy%20취약점%20db%20다운로드.png)

---

## Trivy 실행하기:
### Trivy 적용하기:

```bash
# trivy 취약점 db 경로 복사 및 권한 부여
$ cp -r db/ [Harbor_Volume_Path]
$ cp -r javadb/ [Harbor_Volume_Path]
$ chown -R 10000:10000 [Harbor_Volume_Path]
```

![trivy 취약점 db 볼륨 복사](/assets/img/post/harbor/trivy%20취약점%20db%20볼륨%20복사.png)


```bash
$ ./prepare --with-trivy

$ docker compose up -d
```

![trivy 설치](/assets/img/post/harbor/trivy%20설치.png)

---

### Trivy 적용 확인하기:

```bash
$ docker exec -it trivy-adapter ls -l /home/scanner/.cache/trivy/db /home/scanner/.cache/trivy/java-db
```