---
layout: post
title: "Kubernetes에 MinIO 구성하기"
date: 2026-01-15
categories: [컨테이너, Kubernetes] 
tags: [Kubernetes, Velero, MinIO]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

> 필자는 MinIO를 Kubernetes Cluster의 구축해보고 싶어 진행하였고, 실제 운영환경 및 DR환경 구성 시에는 가장 안전한 방법인 systemd로 설치하였습니다.
{: .prompt-tip}

## 1. MinIO Container Image 다운로드 :

```bash
$ docker pull minio/minio:RELEASE.2025-09-07T16-13-09Z
$ docker tag minio/minio:RELEASE.2025-09-07T16-13-09Z harbor.test.com/minio/minio:RELEASE.2025-09-07T16-13-09Z
$ docker push harbor.test.com/minio/minio:RELEASE.2025-09-07T16-13-09Z
```

![minio 컨테이너 이미지 다운로드](/assets/img/post/kubernetes/minio%20컨테이너%20이미지%20다운로드.png)

* * *

## 2. MinIO 설치하기 :
### 2.1 Namespace 생성하기 :

```bash
$ kubectl create ns minio
```

### 2.2 PVC 생성하기 :

```yaml
# minio-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: minio-data
  namespace: minio
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Gi
  storageClassName: nfs-client   # nfs-provisioner 설치 시 스토리지 클래스 입력
```
```bash
$ kubectl apply -f minio-pvc.yaml
```

* * *

### 2.3 Deployment 생성하기 :

```yaml
# minio-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minio
  namespace: minio
spec:
  replicas: 1
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      labels:
        app: minio
    spec:
      containers:
        - name: minio
          image: minio/minio:RELEASE.2025-01-20T00-00-00Z
          args:
            - server
            - /data
            - --console-address
            - ":9001"
          envFrom:
            - secretRef:
                name: minio-root
          ports:
            - name: s3
              containerPort: 9000
            - name: console
              containerPort: 9001
          volumeMounts:
            - name: data
              mountPath: /data
          readinessProbe:
            httpGet:
              path: /minio/health/ready
              port: 9000
            initialDelaySeconds: 10
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /minio/health/live
              port: 9000
            initialDelaySeconds: 20
            periodSeconds: 20
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: minio-data
```
```bash
$ kubectl apply -f minio-deployment.yaml
```

* * *

### 2.4 Service 생성하기 :

```yaml
# minio-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: minio
spec:
  selector:
    app: minio
  ports:
    - name: s3
      port: 9000
      targetPort: 9000
    - name: console
      port: 9001
      targetPort: 9001
  type: ClusterIP
```
```bash
$ kubectl apply -f minio-svc.yaml
```

* * *

### 2.5 Ingress 생성하기 :

```yaml
# minio-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minio-console
  namespace: minio
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - minio-console.test.com
      secretName: minio-console-tls
  rules:
    - host: minio-console.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: minio
                port:
                  number: 9001
```
```bash
$ kubectl apply -f minio-ingress.yaml
```

* * *

## 3. Velero 전용 계정/버킷 준비하기 :
### 3.1 버킷 생성하기 :

- MinIO Pod 안에서 버킷 생성 시 root(admin)로 1회 생성하여 만들어주면 됩니다.

> MinIO 컨테이너에 MINIO_ROOT_USER / MINIO_ROOT_PASSWORD 환경변수 들어있다는 전제하에 진행하였으며, 위 과정에서 Secret 생성 시 환경변수는 저장됩니다.
{: .prompt-tip}

```bash
$ mc alias set local http://127.0.0.1:9000 "$MINIO_ROOT_USER" "$MINIO_ROOT_PASSWORD"
$ mc mb local/velero
```

![minio 버킷 생성](/assets/img/post/kubernetes/minio%20버킷%20생성.png)

* * *

### 3.2 Velero User 생성하기 :

```bash
$ mc admin user add local velero velero123!
```

* * *

```json
cat << 'EOF' > velero-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation",
        "s3:ListBucket",
        "s3:ListBucketMultipartUploads"
      ],
      "Resource": ["arn:aws:s3:::velero"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:AbortMultipartUpload",
        "s3:ListMultipartUploadParts"
      ],
      "Resource": ["arn:aws:s3:::velero/*"]
    }
  ]
}
EOF
```

* * *

### MinIO 정책 생성하기 :

```bash
$ mc admin policy create local velero-policy velero-policy.json
$ mc admin policy attach local velero-policy --user velero

# 이미 정책이 존재하여 에러가 발생하는 경우 아래와 같이 진행합니다.
$ mc admin policy remove local velero-policy
$ mc admin policy create local velero-policy velero-policy.json
```

* * *

## 4. Velero 설치하기 :

