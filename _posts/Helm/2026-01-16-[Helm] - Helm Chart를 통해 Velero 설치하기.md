---
layout: post
title: "Helm Chart를 통해 Velero 설치하기"
date: 2026-01-16
categories: [컨테이너, Helm] 
tags: [Helm, Velero]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 설치하기 :
### 1.1 Velero credentials Secret 생성하기 :

- ***aws_access_key_id / aws_secret_access_key***는 MinIO에 만든 velero 유저로 입력합니다.

```yaml
# velero-credentials.yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloud-credentials
  namespace: velero
type: Opaque
stringData:
  cloud: |
    [default]
    aws_access_key_id=velero
    aws_secret_access_key=velero123!
```
```bash
$ kubectl apply -f velero-credentials.yaml
```

* * *

### 4.2 BackupStorageLocation 생성하기 :

```yaml

```

| 항목 | 값 | 의미 |
|:---:|:---:|:---|
| provider | aws | MinIO도 AWS SDK 사용 |
| bucket | velero | MinIO에 만든 버킷 |
| prefix | backups | 버킷 내부 경로 |
| s3Url | http://minio.minio.svc.cluster.local:9000 | MinIO 서비스 주소 |
| s3ForcePathStyle | "true" | MinIO 필수 옵션 |
| region | minio | 아무 문자열이나 가능 |
| credential | cloud-credentials | 위 Secret 연결 |

> MinIO 서비스 주소는 Ingress 주소가 아닌 ClusterIP(Service) 주소를 작성합니다.
{: .prompt-warning}

* * *

### 1.2 Velero Helm Chart 다운로드 :

```bash
$ helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts
$ helm repo update
$ helm search repo vmware-tanzu --versions | head
```

![velero helm chart 목록 조회](/assets/img/post/helm/velero%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
$ helm pull vmware-tanzu/velero --version 11.3.2 --destination .
```

* * *

### 1.3 Velero Container Image 다운로드 : 

```bash

```

* * *

### 1.4 values.yaml 수정하기 :

- 위 가정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
image:
  repository: harbor.test.com/velero/velero
  tag: v1.17.1

initContainers:
  - name: velero-plugin-for-aws
    image: harbor.test.com/velero/velero-plugin-for-aws:v1.13.1
    imagePullPolicy: IfNotPresent
    volumeMounts:
      - mountPath: /target
        name: plugins

configuration:
  backupStorageLocation:
  - name: minio
    provider: aws
    bucket: velero
    caCert:
    prefix: backups
    default: true
    validationFrequency:
    accessMode: ReadWrite
    credential:
      name: cloud-credentials
      key: cloud
    config:
     region: minio
     s3ForcePathStyle: "true"
     s3Url:  http://minio.minio.svc.cluster.local:9000

deployNodeAgent: true

snapshotsEnabled: false
```

* * *

### 1.5 Velero Helm Chart 설치하기 :

```bash
# argocd가 설치된 상태에서 진행하는 경우 생성 필요없습니다.
$ kubectl create namespace velero

# 설치 진행
$ helm upgrade --install velero ./ -n velero
```

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n velero
```

![velero helm chart 배포 후 확인](/assets/img/post/helm/velero%20helm%20chart%20배포%20후%20확인.png)

* * *