---
layout: post
title: "Helm Chart를 통해 Ceph CSI 설치하기"
date: 2026-05-27
categories: [컨테이너, Helm] 
tags: [Helm, Ceph, CSI]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. Ceph 정보 확인하기 :
- Ceph 서버에서 정보를 확인합니다.

```bash
$ ceph fsid
$ ceph mon dump
```

![ceph 정보 확인](/assets/img/post/helm/ceph%20정보%20확인.png)

* * *

- Ceph pool 명령어를 통해서 kubernetes를 생성합니다.

```bash
# 현재 pool 목록 확인
$ ceph osd pool ls

# kubernetes pool 생성
$ ceph osd pool create kubernetes 32

# RBD pool 초기화
$ rbd pool init kubernetes
```

* * *

## 2. 설치하기 :
### 2.1 Ceph CSI Helm Chart 다운로드 :

```bash
$ helm repo add ceph-csi https://ceph.github.io/csi-charts
$ helm repo update
$ helm search repo ceph-csi
```

![ceph csi helm chart 목록 조회](/assets/img/post/helm/ceph%20csi%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
$ helm pull ceph-csi/ceph-csi-rbd --version 3.17.0 --destination .
```

* * *

### 2.2 Ceph CSI Container Image 다운로드 : 

```bash
$ docker pull registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.16.0
$ docker pull quay.io/cephcsi/cephcsi:v3.17.0
$ docker pull registry.k8s.io/sig-storage/csi-provisioner:v6.2.0
$ docker pull registry.k8s.io/sig-storage/csi-attacher:v4.11.0
$ docker pull registry.k8s.io/sig-storage/csi-resizer:v2.1.0
$ docker pull registry.k8s.io/sig-storage/csi-snapshotter:v8.5.0

$ docker tag quay.io/cephcsi/cephcsi:v3.17.0                                harbor.test.com/cephcsi/cephcsi:v3.17.0
$ docker tag registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.16.0  harbor.test.com/sig-storage/csi-node-driver-registrar:v2.16.0
$ docker tag registry.k8s.io/sig-storage/csi-provisioner:v6.2.0             harbor.test.com/sig-storage/csi-provisioner:v6.2.0
$ docker tag registry.k8s.io/sig-storage/csi-attacher:v4.11.0               harbor.test.com/sig-storage/csi-attacher:v4.11.0
$ docker tag registry.k8s.io/sig-storage/csi-resizer:v2.1.0                 harbor.test.com/sig-storage/csi-resizer:v2.1.0
$ docker tag registry.k8s.io/sig-storage/csi-snapshotter:v8.5.0             harbor.test.com/sig-storage/csi-snapshotter:v8.5.0

$ docker push harbor.test.com/cephcsi/cephcsi:v3.17.0
$ docker push harbor.test.com/sig-storage/csi-node-driver-registrar:v2.16.0
$ docker push harbor.test.com/sig-storage/csi-provisioner:v6.2.0
$ docker push harbor.test.com/sig-storage/csi-attacher:v4.11.0
$ docker push harbor.test.com/sig-storage/csi-resizer:v2.1.0
$ docker push harbor.test.com/sig-storage/csi-snapshotter:v8.5.0
```

* * *

### 2.3 RBD용 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
csiConfig:
  - clusterID: "<CEPH_FSID>"
    monitors:
      - "192.168.0.11:6789"
      - "192.168.0.12:6789"
      - "192.168.0.13:6789"

storageClass:
  create: true
  name: ceph-rbd
  clusterID: "<CEPH_FSID>"
  pool: kubernetes
  imageFeatures: layering
  provisionerSecret: csi-rbd-secret
  controllerExpandSecret: csi-rbd-secret
  nodeStageSecret: csi-rbd-secret
  reclaimPolicy: Delete
  allowVolumeExpansion: true
  mountOptions: []

  registrar:
    image:
      repository: harbor.test.com/sig-storage/csi-node-driver-registrar
      tag: v2.16.0
      pullPolicy: IfNotPresent
    resources: {}

  plugin:
    image:
      repository: harbor.test.com/cephcsi/cephcsi
      tag: v3.17.0
      pullPolicy: IfNotPresent
    resources: {}

  provisioner:
    image:
      repository: harbor.test.com/sig-storage/csi-provisioner
      tag: v6.2.0
      pullPolicy: IfNotPresent
    resources: {}

  attacher:
    name: attacher
    enabled: true
    image:
      repository: harbor.test.com/sig-storage/csi-attacher
      tag: v4.11.0
      pullPolicy: IfNotPresent
    resources: {}

  resizer:
    name: resizer
    enabled: true
    image:
      repository: harbor.test.com/sig-storage/csi-resizer
      tag: v2.1.0
      pullPolicy: IfNotPresent
    resources: {}

  snapshotter:
    image:
      repository: harbor.test.com/sig-storage/csi-snapshotter
      tag: v8.5.0
      pullPolicy: IfNotPresent
    resources: {}
```

* * *

### 2.4 Secret 생성하기 :
- Ceph에서 CSI 계정을 생성합니다.

```bash
$ ceph auth get-or-create client.kubernetes \
    mon 'profile rbd' \
    osd 'profile rbd pool=kubernetes' \
    mgr 'profile rbd pool=kubernetes'
```

![ceph csi 계정 생성](/assets/img/post/helm/ceph%20csi%20계정%20생성.png)

* * *

- 해당 Key 값을 YAML 파일에 넣어 Secret Manifest 파일을 작성합니다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: csi-rbd-secret
  namespace: ceph-csi-rbd
stringData:
  userID: kubernetes
  userKey: <CLIENT_KEY>
```

* * *

- Namespace를 생성하고, Secret을 생성합니다.

```bash
$ kubectl create namespace ceph-csi-rbd
$ kubectl apply -f ceph-secret.yaml
```

![ceph csi secret 생성](/assets/img/post/helm/ceph%20csi%20secret%20생성.png)

* * *

### 2.5 Ceph CSI Helm Chart 설치하기 :

```bash
# 이미 생성했다면 해당 작업은 생략
$ kubectl create namespace ceph-csi-rbd

# 설치 진행
$ helm upgrade --install ceph-csi-rbd ./ -n ceph-csi-rbd -f values.yaml
```

![Ceph CSI Helm Chart 설치 완료 화면](/assets/img/post/helm/Ceph%20CSI%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n ceph-csi-rbd
```

![Ceph CSI helm chart 배포 후 확인](/assets/img/post/helm/Ceph%20CSI%20helm%20chart%20배포%20후%20확인.png)

* * *

- StorageClass도 생성되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get sc -n ceph-csi-rbd
```

![Ceph CSI storageclass 확인](/assets/img/post/helm/Ceph%20CSI%20storageclass%20확인.png)

* * *

### 2.6 PVC 생성 테스트 하기:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc-test
  namespace: ceph-csi-rbd
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: csi-rbd-sc
  resources:
    requests:
      storage: 1Gi
```

```bash
$ kubectl apply -f pvc-test.yaml
$ kubectl get pvc
```

![ceph csi pvc 생성하기](/assets/img/post/helm/ceph%20csi%20pvc%20생성하기.png)

* * *