---
layout: post
title: "nfs-subdir-external-provisioner 사용하기"
date: 2025-04-07
categories: Kubernetes 
tags: [Kubernetes, nfs]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## nfs-provisioner 다운로드 :
- nfs-provisioner 구성 파일을 직접 다운로드 받거나 git 명령어를 통해서 다운받습니다.
> * [nfs-provisioner 다운로드](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/ "nfs-provisioner 다운로드")

```bash
$ git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner.git
```

---

## nfs-provisioner 설치하기 :
### RBAC 및 서비스 계정 생성하기 :
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

```bash
$ kubectl apply -f rbac.yaml
```

### Deployment 생성하기 :
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: <NFS_SERVER> # nfs-server 주소
            - name: NFS_PATH
              value: <NFS_PATH> # nfs-server 폴더 경로
      volumes:
        - name: nfs-client-root
          nfs:
            server: <NFS_SERVER> # nfs-server 주소
            path: <NFS_PATH> # nfs-server 폴더 경로
```

```bash
$ kubectl apply -f deployment.yaml
```

### StorageClass 생성하기 :
```bash
# storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: storage.nfs.k8s.io
parameters:
  archiveOnDelete: "true"
reclaimPolicy: Retain
volumeBindingMode: Immediate
```

```bash
$ kubectl apply -f storageclass.yaml
```

---

## PVC 생성하기 :

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: nfs-client
```

---

## Grafana Deployment에서 PVC 연결하기 :
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
      volumes:
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: grafana-pvc # pvc 이름
```

```bash
$ kubectl apply -f grafana-deployment.yaml
```

---

> Pod가 재시작되거나 삭제돼도 PVC에 저장된 데이터가 유지되고,
> NFS 서버의 특정 디렉토리 하위에 PVC 이름 기반으로 서브 디렉토리를 자동으로 생성하고, 거기에 데이터를 저장해준다.
{: .prompt-info}