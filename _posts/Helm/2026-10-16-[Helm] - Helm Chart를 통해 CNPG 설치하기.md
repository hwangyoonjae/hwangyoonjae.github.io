---
layout: post
title: "Helm Chart를 통해  CNPG 설치하기"
date: 2026-10-16
categories: [컨테이너, Helm]
tags: [Helm, Postgresql]
image: /assets/img/post-title/helm-wallpaper.jpg
---


## 1. CNPG 설치하기 :
### 1.1 CNPG Helm Chart 다운로드 :

```bash
$ helm repo add cnpg https://cloudnative-pg.github.io/charts
$ helm repo update
$ helm search repo cnpg
```

![cnpg helm chart 목록 조회](/assets/img/post/helm/cnpg%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull cnpg/cloudnative-pg --version 0.29.1

# 압축 파일 풀기
$ tar -zxvf cloudnative-pg-0.29.1.tgz
```

* * *

### 1.2 CNPG Container Image 다운로드 : 

```bash
$ docker pull ghcr.io/cloudnative-pg/cloudnative-pg:1.30.1

$ docker tag ghcr.io/cloudnative-pg/cloudnative-pg:1.30.1 harbor.test.com/cloudnative-pg/cloudnative-pg:1.30.1

$ docker push harbor.test.com/cloudnative-pg/cloudnative-pg:1.30.1
```

### 1.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
# cnpgd/values.yaml
image:
  repository: harbor.inno.com/cloudnative-pg/cloudnative-pg
  pullPolicy: IfNotPresent
  # -- Overrides the image tag whose default is the chart appVersion.
  tag: ""
```

* * *

### 1.4 CNPG Helm Chart 설치하기 :

```bash
# 네임스페이스 생성
$ kubectl create namespace cnpg-system

# cnpg/base 설치 진행
$ helm upgrade --install cnpg ./ -n cnpg-system
```

![cnpg Helm Chart 설치 완료 화면](/assets/img/post/helm/cnpg%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n cnpg-system
```

![cnpg helm chart 배포 후 확인](/assets/img/post/helm/cnpg%20helm%20chart%20배포%20후%20확인.png)

* * *

## 2. CNPG를 통해 PostSQL 설치하기 :
### 2.1 PostSQLContainer Image 다운로드 : 

```bash
$ docker pull ghcr.io/cloudnative-pg/postgresql:17.11

$ docker tag ghcr.io/cloudnative-pg/postgresql:17.11 harbor.test.com/cloudnative-pg/postgresql:17.11

$ docker push harbor.test.com/cloudnative-pg/postgresql:17.11
```

* * *

### 2.2 CNPG Cluster 생성하기 :

```bash
$ vi pg-cluster.yaml
```
```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pg-cluster
  namespace: cnpg-system

spec:
  instances: 3

  imageName: harbor.test.com/cloudnative-pg/postgresql:17.11

  storage:
    storageClass: nfs-client # storageclass 지정
    size: 10Gi
```
```bash
# 적용
$ kubectl apply -f pg-cluster.yaml
```

* * *

- Cluster 상태 확인 및 파드가 정상적으로 생성되었는지 확인합니다.

```bash
# cluster 상태 확인
$ kubectl get cluster -n cnpg-system

# cluster를 통해 자동 PostgreSQL 배포
$ kubectl get po -n cnpg-system -o wide
```

[cnpg를 이용한 postsql 배포 확인](/assets/img/post/helm/cnpg를%20이용한%20postsql%20배포%20확인.png)

* * *

- Cluster를 통해 자동 생성된 PVC도 확인합니다.

```bash
$ kubectl get pvc -n cnpg-system
```

[cnpg를 이용한 pvc 생성 확인](/assets/img/post/helm/cnpg를%20이용한%20pvc%20생성%20확인.png)

* * *

### 2.3 DB 계정 확인하기 :

- Secret을 통해서 DB 계정을 확인합니다.

```bash
$ kubectl get secret -n cnpg-system
```

[cnpg를 통해 db secret 생성 확인](/assets/img/post/helm/cnpg를%20통해%20db%20secret%20생성%20확인.png)

* * *