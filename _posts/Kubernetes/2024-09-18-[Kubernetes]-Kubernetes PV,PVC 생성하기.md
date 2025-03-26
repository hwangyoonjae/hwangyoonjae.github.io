---
layout: post
title: "[Kubernetes]- Kubernetes PV,PVC 생성하기"
date: 2024-09-18
categories: Kubernetes Concept
tags: [Kubernetes, PV, PVC]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## PV(PersistentVolume)란? :
- ***PersistentVolume의 약자***로, 관리자가 프로비저닝하거나, Storage class를 사용해서 동적으로 프로비저닝한 클러스터의 스토리지이다.

* * *

## PVC(PersistentVolumeClaim)란? :
- ***PersistentVolumeClaim의 약자***로, ***사용자가 PV에 사용하고 싶은 용량은 얼마인지?, 읽기/쓰기는 어떤 모드로 설정하고 싶은지?***를 요청이며 리소스에 대한 클레임 검사 역할을 한다.

* * *

## PV, PVC 생명주기 :
![pv,pvc 생명주기](/assets/img/post/kubernetes/pv,pvc%20생명주기.png)

### Provisioning(프로비저닝) :
- PV를 만드는 단계를 프로비저닝(Provisioning)이라고 합니다. 프로비저닝 방법에는 두 가지가 있는데, PV를 미리 만들어 두고 사용하는 정적(static) 방법과 요청이 있을 때 마다 PV를 만드는 동적(dynamic) 방법이다.

#### 정적(static) 프로비저닝 :
- 사용할 수 있는 스토리지 용량에 제한이 있을 때 유용하다.
- 사용하도록 미리 만들어 둔 PV의 용량이 100GB라면 150GB를 사용하려는 요청들은 실패하고, 1TB 스토리지를 사용하더라도 미리 만들어 둔 PV 용량이 150GB 이상인 것이 없으면 요청이 실패한다.

#### 동적(dynamic) 프로비저닝 :
- 사용자가 원하는 용량만큼을 생성해서 사용할 수 있다.
- 정적 프로비저닝과 달리 필요하다면 한번에 200GB PV도 만들 수 있고, PVC는 동적 프로비저닝할 때 여러가지 스토리지 중 원하는 스토리지를 정의하는 스토리지 클래스(Storage Class)로 PV를 생성한다.

* * *

### Binding(바인딩) :
- 프로비저닝으로 만든 PV를 PVC와 연결하는 단계로,PVC에서 원하는 스토리지의 용량과 접근방법을 명시해서 요청하면 거기에 맞는 PV가 할당된다.
- PVC에서 원하는 PV가 없다면 요청은 실패하지만 PVC는 원하는 PV가 생길 때까지 대기하다가 바인딩한다.

* * *

### Using(사용) :
- PVC는 파드에 설정되고 파드는 PVC를 볼륨으로 인식해서 사용한다.
- 할당된 PVC는 파드를 유지하는 동안 계속 사용하며 시스템에서 임의로 삭제할 수 없다.

* * *

### Reclaiming(반환) :
- 사용이 끝난 PVC는 삭제되고 PVC를 사용하던 PV를 초기화(reclaim)하는 과정을 거친다.
- 초기화 정책에는 ***Retain, Delete, Recycle***이 있다.

> Retain : PV를 그대로 보존
> Delete : PV를 삭제하고 연결된 외부 스토리지 쪽의 볼륨도 삭제
> Recycle : PV의 데이터들을 삭제하고 다시 새로운 PVC에서 PV를 사용
{: .prompt-info}

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

```bash
$ vi nfs-pv.yaml
```

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

```bash
$ vi nfs-pvc.yaml
```

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

```bash
$ vi nfs-nginx.yaml
```

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
      image: harbor.com/nginx:1.25.3
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

### yaml 파일로 pod 생성하기 :
```bash
$ kubectl apply -f nfs-pv.yaml
$ kubectl apply -f nfs-pvc.yaml
$ kubectl apply -f nfs-nginx.yaml
```

* * *

### pod 확인하기 :
```bash
$ kubectl exec -it ${pod-name} -- bash

# 접속후 해당 파일 읽기
$ cat /usr/share/nginx/html/index.html

# hello Persistent Volume! 메세지 확인
```

![pv,pvc 테스트 nginx 웹서비스 확인](/assets/img/post/kubernetes/pv,pvc%20테스트%20nginx%20웹서비스%20확인.png)

* * *