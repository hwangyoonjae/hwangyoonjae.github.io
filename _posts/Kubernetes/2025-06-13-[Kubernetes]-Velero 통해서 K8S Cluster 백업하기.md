---
layout: post
title: "Velero 통해서 K8S Cluster 백업하기"
date: 2025-06-13
categories: Kubernetes 
tags: [Kubernetes, Velero, Backup]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## MinIO 설치하기:
```bash
# minio-user 계정 생성
$ sudo mkdir -p /opt/minio/data
$ sudo useradd -r minio-user
$ sudo chown -R minio-user:minio-user /opt/minio

# MinIO 바이너리 다운로드
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/

# systemd 서비스 파일 생성
sudo tee /etc/systemd/system/minio.service > /dev/null <<EOF
[Unit]
Description=MinIO
After=network.target

[Service]
User=minio-user
Group=minio-user
ExecStart=/usr/local/bin/minio server /opt/minio/data --console-address ":9001"
Restart=always
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

> bucket 경로는 “/opt/minio/data” 폴더 안에 생성됩니다.
{: .prompt-info }

```bash
# 서비스 시작
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable --now minio
```

* * *

## MinIO 설정하기:
- 브라우저 접속 (http://{MinIO_IP}:9001)

> ID : minioadmin / PW : minioadmin 
{: .prompt-info }

![MinIO 초기화면](/assets/img/post/kubernetes/MinIO%20초기화면.png)

* * *

- 로그인 후 초기 화면

![MinIO 로그인 후 초기 화면](/assets/img/post/kubernetes/MinIO%20로그인%20후%20초기%20화면.png)

* * *

- Busket 생성하기

![MinIO Busket 생성](/assets/img/post/kubernetes/MinIO%20Busket%20생성.png)
![MinIO Busket 생성 후 화면](/assets/img/post/kubernetes/MinIO%20Busket%20생성%20후%20화면.png)

* * *

- 사용자 생성하기

> MinIO가 "standalone server" 모드로 실행 중이면서 IAM 기능이 비활성화되어 있기 때문에 mc 클라이언트 도구를 사용하여 계성 생성한다.
{: .prompt-warning }

```bash
# mc 설치
$ wget https://dl.min.io/client/mc/release/linux-amd64/mc
$ chmod +x mc
$ mv mc /usr/local/bin/

# MinIO 서버 등록
$ mc alias set minio http://<MINIO_IP>:9000 minioadmin minioadmin

# 사용자 생성
$ mc admin user add minio velero velero123

# 사용자에 정책 적용
$ mc admin policy attach minio readwrite --user velero

# 버킷 생성 (백업 저장소)
$ mc mb minio/<버킷명>   # 위 작업에서 진행하였으므로, 저장소 이름을 다르게 지정할 경우 사용

# 버킷 내용 삭제 (필수)
$ mc rm --recursive --force minio/버킷명

# 버킷 삭제
$ mc rb minio/<버킷명>
```

![사용자 생성 과정](/assets/img/post/kubernetes/사용자%20생성%20과정.png)

* * *

## Velero 설치하기:

- Velero CLI 설치하기

```bash
$ VELERO_VERSION=v1.13.0
$ wget https://github.com/vmware-tanzu/velero/releases/download/${VELERO_VERSION}/velero-${VELERO_VERSION}-linux-amd64.tar.gz
$ tar -xvf velero-${VELERO_VERSION}-linux-amd64.tar.gz
$ sudo mv velero-${VELERO_VERSION}-linux-amd64/velero /usr/local/bin/
```

* * *

- 인증파일 설치하기

```bash
$ vi credentials-velero
======================================
[default]
aws_access_key_id=velero
aws_secret_access_key=velero123
```

* * *

- Velero 설치하기

```bash
$ velero install \
  --provider aws \
  --plugins {Harbor Domain}/velero/velero-plugin-for-aws:v1.8.0 \
  --bucket velero \
  --secret-file ./credentials-velero \
  --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://<LB_SERVER_IP>:9000
```

> LB 서버 아이피 주소 입력 시 kubectl 명령어 사용이 가능해야합니다.
{: .prompt-warning }

* * *

- Velero 삭제하기

```bash
$ velero uninstall
```

![velero 삭제](/assets/img/post/kubernetes/velero%20삭제.png)

* * *

## Velero 백업하기:
- 클러스터 전체 백업하기

```bash
$ velero backup create cluster-backup --wait
```

> ***`cluster-backup:`*** 백업 이름 (원하는 이름으로 변경 가능)
> 
> ***`--wait:`*** 백업이 완료될 때까지 기다림
{: .prompt-example }

* * *

- 백업 성공 여부 확인

```bash
$ velero backup get
```

* * *

- 특정 네임스페이스만 백업하기

```bash
$ velero backup create partial-backup \
  --include-resources persistentvolumes,persistentvolumeclaims,pods \
  --wait
```

> ***`partial-backup`:*** 백업 이름 (원하는 이름으로 변경 가능)
> 
> ***`persistentvolumes,persistentvolumeclaims,pods`:*** 리소스 목록(컴마로 구분)
> 
> ***`--wait`:*** 백업이 완료될 때까지 기다림
{: .prompt-example }

* * *

- 백업 상세 정보 확인

```bash
$ velero backup describe cluster-backup --details
```

* * *

- 백업 로그 보기

```bash
$ velero backup logs cluster-backup
```

* * *

- 백업 파일 MinIO 웹 콘솔에서 확인

![백업 파일 MinIO 웹 콘솔에서 확인](/assets/img/post/kubernetes/백업%20파일%20MinIO%20웹%20콘솔에서%20확인.png)

## Velero 백업 주기 설정하기:
- 주기적인 백업 설정 방법

```bash
# 매일 오전 3시에 전체 백업
velero schedule create daily-backup \
  --schedule="0 3 * * *" \
  --include-namespaces=* \
  --ttl 168h
```

- 각 옵션 설명

|옵션|설명|
|---|---|
|daily-backup|스케줄 이름|
|--schedule="0 3 * * *"|Cron 형식|
|--include-namespaces=*|전체 네임스페이스 백업|
|--ttl 168h|백업 보관 기간|

> Velero에서 일 단위를 공식 지원하지 않아 백업 보관 기간은 반드시 시간 단위로 설정해야한다.
{:prompt-warning}

* * *

- 스케줄 확인 및 관리 명령어

```bash
# 등록된 스케줄 확인
$ velero schedule get

# 상세 정보 확인
$ velero schedule describe <NAME>

# 스케줄 삭제
$ velero schedule delete <NAME>
```