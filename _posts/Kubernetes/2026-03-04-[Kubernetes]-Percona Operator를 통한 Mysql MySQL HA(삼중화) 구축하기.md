---
layout: post
title: "Percona Operator를 통한 MySQL HA(삼중화) 구축하기"
date: 2026-03-04
categories: [컨테이너, Kubernetes] 
tags: [Kubernetes, Percona, Operator]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. Percona Container Image 다운로드 :

```bash
# 이미지 다운로드
$ docker pull percona/percona-server-mysql-operator:1.0.0
$ docker pull percona/percona-server:8.4.6-6.1
$ docker pull percona/haproxy:2.8.15
$ docker pull percona/percona-mysql-router:8.4.6
$ docker pull percona/percona-orchestrator:3.2.6-18
$ docker pull percona/pmm-client:3.4.1
$ docker pull percona/percona-xtrabackup:8.4.0-4.1
$ docker pull percona/percona-toolkit:3.7.0-2

# 이미지 태그 변경
$ docker tag percona/haproxy:2.8.15 harbor.test.com/library/percona-haproxy:2.8.15
$ docker tag percona/percona-mysql-router:8.4.6 harbor.test.com/library/percona-percona-mysql-router:8.4.6
$ docker tag percona/percona-orchestrator:3.2.6-18 harbor.test.com/library/percona-percona-orchestrator:3.2.6-18
$ docker tag percona/percona-server:8.4.6-6.1 harbor.test.com/library/percona-percona-server:8.4.6-6.1
$ docker tag percona/percona-server-mysql-operator:1.0.0 harbor.test.com/library/percona-percona-server-mysql-operator:1.0.0
$ docker tag percona/percona-toolkit:3.7.0-2 harbor.test.com/library/percona-percona-toolkit:3.7.0-2
$ docker tag percona/percona-xtrabackup:8.4.0-4.1 harbor.test.com/library/percona-percona-xtrabackup:8.4.0-4.1
$ docker tag percona/pmm-client:3.4.1 harbor.test.com/library/percona-pmm-client:3.4.1

# 이미지 레지스트리의 저장
$ docker push harbor.test.com/library/percona-haproxy:2.8.15
$ docker push harbor.test.com/library/percona-percona-mysql-router:8.4.6
$ docker push harbor.test.com/library/percona-percona-orchestrator:3.2.6-18
$ docker push harbor.test.com/library/percona-percona-server:8.4.6-6.1
$ docker push harbor.test.com/library/percona-percona-server-mysql-operator:1.0.0
$ docker push harbor.test.com/library/percona-percona-toolkit:3.7.0-2
$ docker push harbor.test.com/library/percona-percona-xtrabackup:8.4.0-4.1
$ docker push harbor.test.com/library/percona-pmm-client:3.4.1
```

![percona 컨테이너 이미지 다운로드](/assets/img/post/kubernetes/percona%20container%20이미지%20다운로드.png)

* * *

## 2. Percona 설치하기 :
### 2.1 Namespace 생성하기 :

```bash
$ kubectl create namespace pxc
```

### 2.2 Operator 생성하기 :

- 인터넷이 되는 환경에서 해당 파일을 다운받습니다.

```bash
$ wget https://raw.githubusercontent.com/percona/percona-xtradb-cluster-operator/v1.19.0/deploy/bundle.yaml
```

- Operator를 설치합니다.

```bash
$ kubectl apply --server-side -f bundle.yaml -n <namespace>
```

* * *

### 2.3 Percona 설치 파일 이미지 경로 변경하기 :

- 인터넷이 되는 환경에서 해당 파일을 다운받습니다.

```bash
$ wget https://raw.githubusercontent.com/percona/percona-xtradb-cluster-operator/v1.19.0/deploy/cr.yaml
```

* * *

- vi 편집기를 통해서 이미지 경로를 수정합니다.

```bash
$ vi cr.yaml
```

```yaml
  pxc:
    size: 3
    image: harbor.test.com/library/percona/percona-xtradb-cluster:8.4.7-7.1
      persistentVolumeClaim:
        storageClassName: nfs-client # storageclass 있는 경우 변경
#        accessModes: [ "ReadWriteOnce" ]
#        dataSource:
#          name: new-snapshot-test
#          kind: VolumeSnapshot
#          apiGroup: snapshot.storage.k8s.io
        resources:
          requests:
            storage: 6G

  haproxy:
    enabled: true
    size: 3
    image: harbor.test.com/library/percona/haproxy:2.8.17

  logcollector:
    enabled: true
    image: harbor.test.com/library/percona/fluentbit:4.0.1-1

  backup:
#    allowParallel: true
    image: harbor.test.com/library/percona/percona-xtrabackup:8.4.0-5.1
```

* * *

### 2.4 PCX 클러스터 생성하기 :

- PCX를 설치합니다.

```bash
$ kubectl apply -f cr.yaml -n <namespace>
```

* * *

- pxc가 정상 상태인지 확인합니다.

```bash
$ kubectl get pxc -n <namespace>
```
![percona pxc 생성](/assets/img/post/kubernetes/percona%20pxc%20생성.png)

- pod가 정상 상태인지 확인합니다.

```bash
$ kubectl get pod -n <namespace>
```
![percona pod 생성](/assets/img/post/kubernetes/percona%20pod%20생성.png)

- pvc도 정상 상태인지 확인합니다.

```bash
$ kubectl get pvc -n <namespace>
```
![percona pvc 볼륨 생성](/assets/img/post/kubernetes/percona%20pvc%20볼륨%20생성.png)

* * *

## 3. 기존 MySQL에서 데이터 옮기기 :
### 3.1 기존 MySQL에서 데이터 dump 파일 생성하기 :

```bash
$ kubectl exec -it <기존-mysql-pod> -- bash

bash-4.4# mysqldump -uroot -p \
> --all-databases \
> --single-transaction \
> --quick \
> --lock-tables=false \
> > dump.sql
```

![mysql 데이터 dump 파일 생성](/assets/img//post/kubernetes/mysql%20데이터%20dump%20파일%20생성.png)

* * *

### 3.2 데이터 파일 옮기기 :

- 기존 MySQL에서 가져온 데이터 파일을 PXC에 복사합니다.

```bash
# dump 파일 내보내기
$ kubectl cp mysql-0:dump.sql ./dump.sql -n console
# 생성한 pxc pod의 파일 보내기
$ kubectl cp ./dump.sql pxc/cluster1-pxc-0:/tmp/dump.sql -n pxc
```

* * *

### 3.3 데이터 파일 넣기 :

- 기존 MySQL의 저장된 데이터를 새로 구성한 PXC에 이관합니다.

```bash
$ kubectl exec -it -n pxc cluster1-pxc-0 -- bash

bash-5.1$ mysql -uroot -p < /tmp/dump.sql
```
 
> ERROR 1105 (HY000) at line 51: Percona-XtraDB-Cluster prohibits use of LOCK TABLE/FLUSH TABLE <table> WITH READ LOCK/FOR EXPORT with pxc_strict_mode = ENFORCING
>
> 위와 같이 에러가 발생할 경우 다음과 같이 명령어를 입력합니다.
{: .prompt-error}

```bash
# 명령어를 실행해서 잠시 끄기
mysql> SET GLOBAL pxc_strict_mode=PERMISSIVE;

# permissive 상태 확인
mysql> SHOW VARIABLES LIKE 'pxc_strict_mode';
```

![permissive 상태 확인](/assets/img/post/kubernetes/permissive%20상태%20확인.png)

* * *

- 다시 데이터 파일을 통해 PXC의 데이터가 정상적으로 이관되었는지 확인합니다.

![percona operator를 통한 데이터 이관](/assets/img/post/kubernetes/percona%20operator를%20통한%20데이터%20이관.png)

* * *