---
layout: post
title: "Helm Chart를 통해 Rook Ceph 클러스터 Kubernetes 배포하기"
date: 2026-05-28
categories: [컨테이너, Helm]
tags: [Helm, CSI, Rook, Ceph]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. Rook-Ceph Kubernetes 배포 방법 이해하기 :
### 1.1 Rook이란? :
- Kubernetes 위에서 Ceph 스토리지를 쉽게 운영할 수 있도록 해주는 Operator입니다.
- 보통 Kubernetes 환경에서는 **블록 스토리지 (RBD), 파일 스토리지 (CephFS), 오브젝트 스토리지 (S3)**를 사용하며, 구조적으로는 아래와 같습니다.

```bash
Kubernetes
 └─ Rook Operator
      └─ Ceph Cluster
           ├─ MON
           ├─ MGR
           ├─ OSD
           └─ MDS/RGW
```

* * *

### 1.2 Rook Ceph 준비사항 :
- 필자는 Rook-Ceph를 구성하기 위한 환경은 아래와 같이 구성 준비하였습니다.

| 구분 | 항목 | 권장/설정 내용 | 비고 |
|------|------|------|------|
| Kubernetes 환경 | Kubernetes 버전 | 1.28 이상 | 최신 안정 버전 권장 |
|  | Worker Node 수 | 최소 3대 | Ceph 고가용성(HA) 구성 권장 |
|  | 시간 동기화 | NTP 또는 Chrony 설정 | 모든 노드 시간 동일 필수 |
| 스토리지 | 추가 디스크 | 각 Worker Node별 1개 이상 | OSD용 디스크 |
|  | 디스크 예시 | `/dev/sdb` | OS 디스크와 분리 권장 |
|  | 디스크 상태 | 미사용(빈 디스크) | 파티션 및 파일시스템 제거 필요 |
| 필수 패키지 | lvm2 설치 | `sudo dnf install -y lvm2` | 모든 노드 수행 |
| 사전 확인 | 디스크 확인 명령어 | `lsblk` | 디스크 인식 여부 확인 |
|  | 확인 결과 예시 | `sdb 50G` | Ceph에서 사용할 디스크 확인 |

* * *

### 1.3 Rook Ceph 클러스터란? : 
- Rook을 사용하여 Kubernetes 환경에 배포된 Ceph 스토리지 클러스터를 의미합니다.
- Rok Ceph 클러스터는 다음과 같은 구성요소로 이루어져있습니다.

| 구성요소 | 설명 | 주요 역할 |
|----------|------|----------|
| Ceph MON (Monitor) | 클러스터 상태를 유지하고 모니터링하는 데몬 | 클러스터 맵 관리, OSD/MGR 상태 확인, Quorum 유지 |
| Ceph MGR (Manager) | 클러스터 상태 정보를 수집하고 관리 기능을 제공하는 데몬 | Dashboard 제공, 성능 모니터링, 메트릭 수집 |
| Ceph OSD (Object Storage Daemon) | 실제 데이터를 저장하고 처리하는 데몬 | 데이터 저장, 복제(Replication), 복구(Recovery) |
| Ceph MDS (Metadata Server) | CephFS의 메타데이터를 관리하는 데몬 | 디렉토리 구조, 파일 권한, 파일명 관리 |
| Rook Operator | Ceph 클러스터를 Kubernetes에서 관리하는 Operator | Ceph 배포, 확장, 업그레이드, 장애 복구 자동화 |
| Ceph CSI (RBD) | Kubernetes와 Ceph RBD를 연동하는 CSI 드라이버 | PVC 생성 및 Block Storage 제공 (RWO) |
| Ceph CSI (CephFS) | Kubernetes와 CephFS를 연동하는 CSI 드라이버 | PVC 생성 및 Shared File Storage 제공 (RWX) |

* * *

## 2. 설치하기 :
### 2.1 Rook Ceph Helm Chart 다운로드 :
```bash
$ helm repo add rook-release https://charts.rook.io/release
$ helm repo update
$ helm search repo rook-release --versions | head
```

![rook ceph helm chart 목록 조회](/assets/img/post/helm/rook%20ceph%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull rook-release/rook-ceph --version v1.19.6 --destination .

# 압축 파일 풀기
$ tar -zxvf rook-ceph-v1.19.6.tgz
```

* * *

### 2.2 Rook Ceph Container Image 다운로드 : 

```bash
$ docker pull docker.io/rook/ceph:v1.19.6
$ docker pull quay.io/cephcsi/cephcsi:v3.16.2
$ docker pull registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.16.0
$ docker pull registry.k8s.io/sig-storage/csi-provisioner:v6.1.1
$ docker pull registry.k8s.io/sig-storage/csi-snapshotter:v8.5.0
$ docker pull registry.k8s.io/sig-storage/csi-attacher:v4.11.0
$ docker pull registry.k8s.io/sig-storage/csi-resizer:v2.1.0
$ docker pull quay.io/cephcsi/ceph-csi-operator:v0.6.0
$ docker pull quay.io/csiaddons/k8s-sidecar:v0.14.0

$ docker tag docker.io/rook/ceph:v1.19.6                                   harbor.test.com/rook/ceph:v1.19.6
$ docker tag quay.io/cephcsi/cephcsi:v3.16.2                               harbor.test.com/cephcsi/cephcsi:v3.16.2
$ docker tag registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.16.0 harbor.test.com/sig-storage/csi-node-driver-registrar:v2.16.0
$ docker tag registry.k8s.io/sig-storage/csi-provisioner:v6.1.1            harbor.test.com/sig-storage/csi-provisioner:v6.1.1
$ docker tag registry.k8s.io/sig-storage/csi-snapshotter:v8.5.0            harbor.test.com/sig-storage/csi-snapshotter:v8.5.0
$ docker tag registry.k8s.io/sig-storage/csi-attacher:v4.11.0              harbor.test.com/sig-storage/csi-attacher:v4.11.0
$ docker tag registry.k8s.io/sig-storage/csi-resizer:v2.1.0                harbor.test.com/sig-storage/csi-resizer:v2.1.0
$ docker tag quay.io/cephcsi/ceph-csi-operator:v0.6.0                      harbor.test.com/cephcsi/ceph-csi-operator:v0.6.0
$ docker tag quay.io/csiaddons/k8s-sidecar:v0.14.0                         harbor.test.com/csiaddons/k8s-sidecar:v0.14.0

$ docker push harbor.test.com/rook/ceph:v1.19.6
$ docker push harbor.test.com/cephcsi/cephcsi:v3.16.2
$ docker push harbor.test.com/sig-storage/csi-node-driver-registrar:v2.16.0
$ docker push harbor.test.com/sig-storage/csi-provisioner:v6.1.1
$ docker push harbor.test.com/sig-storage/csi-snapshotter:v8.5.0
$ docker push harbor.test.com/sig-storage/csi-attacher:v4.11.0
$ docker push harbor.test.com/sig-storage/csi-resizer:v2.1.0
$ docker push harbor.test.com/cephcsi/ceph-csi-operator:v0.6.0 
$ docker push harbor.test.com/csiaddons/k8s-sidecar:v0.14.0
```

* * *

### 2.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
image:
  repository: harbor.test.com/rook/ceph
  tag: v1.19.6
  pullPolicy: IfNotPresent

csi:
  cephcsi:
    repository: harbor.test.com/cephcsi/cephcsi
    tag: v3.16.2

  registrar:
    repository: harbor.test.com/sig-storage/csi-node-driver-registrar
    tag: v2.16.0

  provisioner:
    repository: harbor.test.com/sig-storage/csi-provisioner
    tag: v6.1.1

  snapshotter:
    repository: harbor.test.com/sig-storage/csi-snapshotter
    tag: v8.5.0

  attacher:
    repository: harbor.test.com/sig-storage/csi-attacher
    tag: v4.11.0

  resizer:
    repository: harbor.test.com/sig-storage/csi-resizer
    tag: v2.1.0

  csiAddons:
    enabled: true

ceph-csi-operator:
  nameOverride: ceph-csi
  fullnameOverride: ceph-csi
  controllerManager:
    manager:
      env:
        csiServiceAccountPrefix: "ceph-csi-"
      image:
        repository: harbor.test.com/cephcsi/ceph-csi-operator
        tag: v0.6.0
```

* * *

### 2.4 Rook Ceph Helm Chart 설치하기 :

```bash
# 이미 생성했다면 해당 작업은 생략
$ kubectl create namespace rook-ceph

# 디렉토리 접근
$ cd rook-ceph

# 설치 진행
$ helm upgrade --install rook-ceph ./ -n rook-ceph
```

![rook ceph Helm Chart 설치 완료 화면](/assets/img/post/helm/rook%20ceph%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get pods -n rook-ceph
```

![rook ceph helm chart 배포 후 확인](/assets/img/post/helm/rook%20ceph%20helm%20chart%20배포%20후%20확인.png)

* * *

## 3. Rook Ceph Cluster 생성하기 :
### 3.1 Rook Ceph Cluster Helm Chart 다운로드 :
```bash
$ helm repo add rook-release https://charts.rook.io/release
$ helm repo update
$ helm search repo rook-release
```

![rook ceph cluster helm chart 목록 조회](/assets/img/post/helm/rook%20ceph%20cluster%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull rook-release/rook-ceph-cluster --version v1.19.6 --destination .

# 압축 파일 풀기
$ tar -zxvf rook-ceph-cluster-v1.19.6.tgz
```

* * *

### 3.2 Rook Ceph Cluster Container Image 다운로드 : 

```bash
$ docker pull quay.io/ceph/ceph:v19.2.3

$ docker tag quay.io/ceph/ceph:v19.2.3   harbor.test.com/ceph/ceph:v19.2.3

$ docker push harbor.test.com/ceph/ceph:v19.2.3
```

* * *

### 3.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
# -- Namespace of the main rook operator
operatorNamespace: rook-ceph

cephImage:
  repository: harbor.test.com/ceph/ceph
  tag: v19.2.3
  allowUnsupported: false

toolbox:
  enabled: true
  image: harbor.test.com/ceph/ceph:v19.2.3

cephClusterSpec:
  dataDirHostPath: /var/lib/rook

  mon:
    count: 3
    allowMultiplePerNode: false

  mgr:
    count: 2

  dashboard:
    enabled: true
    ssl: false

  storage:
    useAllNodes: true
    useAllDevices: true
    nodes:
      - name: k8s-worker1
        devices:
          - name: sdb
      - name: k8s-worker2
        devices:
          - name: sdb
      - name: k8s-worker3
        devices:
          - name: sdb
```

* * *

### 3.4 Rook Ceph Cluster Helm Chart 설치하기 :

```bash
# 이미 생성했다면 해당 작업은 생략
$ kubectl create namespace rook-ceph

# 디렉토리 접근
$ cd rook-ceph-cluster

# 설치 진행
$ helm upgrade --install rook-ceph-cluster ./ -n rook-ceph
```

![rook ceph cluster Helm Chart 설치 완료 화면](/assets/img/post/helm/rook%20ceph%20cluster%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get pods -n rook-ceph
```

![rook ceph cluster helm chart 배포 후 확인](/assets/img/post/helm/rook%20ceph%20cluster%20helm%20chart%20배포%20후%20확인.png)

* * *

## 4. PVC 생성하기 :
### 4.1 StorageClass 확인하기:

```bash
$ kubectl get storageclass
```

![rook ceph storageclass 목록 화면](/assets/img/post/helm/rook%20ceph%20storageclass%20목록%20화면.png)

* * *

### 4.2 RBD(Block Storage) 사용하여 PVC 생성하기 :

- PVC 생성 시, RBD(Block Storage) 사용하여 생성합니다.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ceph-rbd-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ceph-block
  resources:
    requests:
      storage: 5Gi
```

```bash
# pvc-rbd 생성
$ kubectl apply -f pvc-rbd.yaml -n rook-ceph
```

* * *

- pvc가 정상적으로 Bound 되었는지 확인합니다.

```bash
$ kubectl get pvc -n rook-ceph
```

![rook ceph rbd pvc 볼륨 생성 확인](/assets/img/post/helm/rook%20ceph%20rbd%20pvc%20볼륨%20생성%20확인.png)

* * *

### 4.3 CephFS(File Storage) 사용하여 PVC 생성하기 :

- 여러 파드에서 동시에 접근하려면 CephFS(File Storage) 사용하여 생성합니다.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cephfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ceph-filesystem
  resources:
    requests:
      storage: 5Gi
```

```bash
$ kubectl apply -f pvc-cephfs.yaml -n rook-ceph
```

* * *

- pvc가 정상적으로 Bound 되었는지 확인합니다.

```bash
$ kubectl get pvc -n rook-ceph
```

![rook ceph cephfs pvc 볼륨 생성 확인](/assets/img/post/helm/rook%20ceph%20cephfs%20pvc%20볼륨%20생성%20확인.png)

* * *