---
layout: post
title: "etcd 백업하기"
date: 2024-07-17
categories: [컨테이너, Kubernetes]
tags: [Kubernetes, etcd]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. etcd-master-endpoint 확인하기 :
```bash
# etcd API 버전 3을 사용하도록 설정
$ ETCDCTL_API=3;
etcdctl endpoint status --write-out=table \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key
```

>API 버전 3을 사용하도록 지정할까?
>
>API v3는 여러 면에서 개선된 기능을 제공하며, 특히 대규모 시스템이나 높은 트랜잭션 빈도 환경에서 더 나은 성능과 안정성을 제공합니다.
>따라서 새로운 프로젝트나 시스템에서는 v3 API를 사용하는 것이 권장한다고 합니다.
{: .prompt-warning }

* * *

## 2. etcd 백업하기 :
```bash
$ ETCDCTL_API=3;
etcdctl --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/server.crt \
        --key=/etc/kubernetes/pki/etcd/server.key \
          snapshot save /opt/snapshot-pre-boot.db;

# 아래와 같이 스냅샷 캡처 과정 출력
{"level":"info","ts":1721179099.5287964,"caller":"snapshot/v3_snapshot.go:68","msg":"created temporary db file","path":"/opt/snapshot-pre-boot.db.part"}
{"level":"info","ts":1721179099.5375545,"logger":"client","caller":"v3/maintenance.go:211","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1721179099.5376866,"caller":"snapshot/v3_snapshot.go:76","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
{"level":"info","ts":1721179099.9228318,"logger":"client","caller":"v3/maintenance.go:219","msg":"completed snapshot read; closing"}
{"level":"info","ts":1721179100.2069778,"caller":"snapshot/v3_snapshot.go:91","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"37 MB","took":"now"}
{"level":"info","ts":1721179100.2070692,"caller":"snapshot/v3_snapshot.go:100","msg":"saved","path":"/opt/snapshot-pre-boot.db"}
Snapshot saved at /opt/snapshot-pre-boot.db
```

## 3. 백업 파일 확인하기 :
```bash
$ ls -al /opt
# -rw-------   1 root root 36724768 Jul 17 10:18 snapshot-pre-boot.db
```

* * *

## 4. etcd snapshot restore하기 :
```bash
$ ETCDCTL_API=3;
etcdctl --data-dir /var/lib/etcd-from-backup \
     snapshot restore /opt/snapshot-pre-boot.db;
```

* * *

## 5. etcd.yaml 수정 :
- etcd 스냅샷을  /var/lib/etcd-from-backup 경로로 복원했으므로 볼륨에 대한 호스트 경로를 변경해야합니다.

```bash
$ vi /etc/kubernetes/manifests/etcd.yaml
```
```yaml
volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
```

* * *

## 6. etcd 정상 동작 확인 :
```bash
$ docker ps -a | grep etcd -> up 상태 확인
```

---