---
layout: post
title: "[Kubernetes]- Kubernetes PV,PVC 생성하기"
date: 2024-09-18
categories: Kubernetes
tags: [Kubernetes, PV, PVC]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## PV(PersistentVolume)란? :
- ***PersistentVolume의 약자***로, 관리자가 프로비저닝하거나, Storage class를 사용해서 동적으로 프로비저닝한 클러스터의 스토리지이다.

* * *

## PVC(PersistentVolumeClaim)란? :
- ***PersistentVolumeClaim의 약자***로, ***사용자가 PV에 사용하고 싶은 용량은 얼마인지?, 읽기/쓰기는 어떤 모드로 설정하고 싶은지?***를 요청이며 리소스에 대한 클레임 검사 역할을 한다.

* * *

## PV, PVC 생성하기 :
> NAS 스토리지가 아닌 NFS 서버로 구성하여 테스트 하였습니다.
{: .prompt-warning}

### NFS 서버 세팅하기 :
> NFS 서버에서 진행하면됩니다.
{: .prompt-warning}

- NFS 관련 패키지 설치

```bash
# Ubuntu/Debian 계열
$ apt install nfs-kernel-server

# Centos/RHEL 계열
$ yum install nfs-utils

# NFS와 관련된 서비스 시작 및 자동 시작 설정
$ systemctl start rpcbind
$ systemctl start nfs-server
$ systemctl enable rpcbind
$ systemctl enable nfs-server
```

- 공유 디렉터리 생성

```bash
# /k8s-nas 폴더로 이동
cd /k8s-nas
# /k8s-nas 폴더에 `share`폴더 생성
mkdir share
# `share`폴더의 권한을 전체허용으로 변경
chmod 777 share
```

- NFS 서버 공유 설정

```bash
$ vi /etc/exports
# 아래 내용과 같이 작성
/k8s-nas/share 192.168.1.0/24(rw,sync,no_subtree_check)
```

- NFS 설정 적용 및 확인

```bash
# 설정 적용
$ exportfs -ra

# 설정 확인
$ exportfs -v
```

- 테스트 파일 생성하기
```bash
$ echo 'hello Persistent Volume!!' >> /k8s-nas/share/index.html
```

* * *

### NFS 클라이언드 세팅하기 :
> NFS 클라이언트에서 진행하면됩니다.
{: .prompt-warning}

- NFS 관련 패키지 설치

```bash
# Ubuntu/Debian 계열
$ apt install nfs-kernel-server

# Centos/RHEL 계열
$ yum install nfs-utils

# NFS와 관련된 서비스 시작 및 자동 시작 설정
$ systemctl start rpcbind
$ systemctl enable rpcbind
```

- NFS 서버의 공유 상태 확인
```bash
$ showmount -e Server_IP
```

- 클라이언트에서 NFS 마운트
```bash
$ mount -t nfs server_IP:/path/to/share /path/to/local_mount_point
```

* * *

### PV 생성하기 :
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
  labels:
    type: nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: ${nfs-server-ip} # nfs서버로 사용할 ip
    path: /k8s-nas/share
```

* * *

### PVC 생성하기 :
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  storageClassName: "" # 공백으로 설정시 자동으로 default 잡아줍니다.
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 4Gi
  selector:
    matchExpressions:
      - key: type
        operator: In
        values:
          - nfs
```

* * *

### nginx.yaml 생성하기 :
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nfs-nginx
spec:
  containers:
    - name: nfs-nginx
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - name: nginx-storage
          mountPath: "/usr/share/nginx/html"
  volumes:
    - name: nginx-storage
      persistentVolumeClaim:
        claimName: pvc
```

* * *