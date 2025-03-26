---
layout: post
title: "[Kubernetes] - Kubernetes Object Yaml 정의"
date: 2024-05-31
categories: Kubernetes Concept
tags: [Kubernetes, Yaml]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## Pod 생성 방법 :
```yaml
apiVersion: v1 # 리소스의 API 버전을 지정
kind: Pod # 리소스의 종류 지정
metadata:
  name: web # 파드의 이름
  labels:  # 파드에 적용할 레이블을 정의
      app: nginx
spec: # 파드의 사양을 정의 (컨테이너, 볼륨, 네트워크 설정 등이 포함)
  containers: # 파드 내에서 실행될 컨테이너 목록을 정의
    - name: {컨테이너명}
      image: {이미지주소/이미지명:tag} # 컨테이너 이미지 경로
      port: 80 # 컨테이너 포트
```

* * *

## Replicaset 생성 방법 :
```yaml
apiVersion: apps/v1 # 리소스의 API 버전을 지정
kind: ReplicaSet # 리소스의 종류 지정
metadata:
  name: nginx-replicaset # Replicaset의 이름  
# [2] ReplicaSet 스펙
spec:
  selector: # Pod 템플릿의 검색 조건
    matchLabels:
      app: nginx-replicaset # 파드를 묶는 라벨 이름
  replicas: 3 # Pod의 수 
  # [3] Pod 템플릿
  template:
    metadata:
      labels:
        app: nginx-replicaset # 생성될 파드에 부여할 레이블
    # [4] Pod 스펙
    spec:
      containers:
            - name: {컨테이너명}
              image: {이미지주소/이미지명:tag} # 컨테이너 이미지 경로
              ports:
		         - containerPort: 80 # 컨테이너 포트
```

* * *

## Deployment 생성 방법 :
```yaml
apiVersion: apps/v1 # 리소스의 API 버전을 지정
kind: Deployment # 리소스의 종류 지정
metadata: # 리소스에 대한 메타데이터
  name: {디플로이먼트 이름}
  namespace: {네임스페이스명} # 디플로이먼트를 생성할 네임스페이스
# [2] Deployment 스펙
spec:
  selector: # 디플로이먼트가 관리할 파드를 선택하는 데 사용되는 라벨 셀렉터
    matchLabels:
      app: {디플로이먼트와 파드를 묶는 라벨 이름}
   replicas: 3 # 파드 복제 수
   # [3] Pod 템플릿
  template:
    metadata:
      labels:
        app: {디플로이먼트와 파드를 묶는 라벨 이름}
    # [4] Pod 스펙
    spec:
      volumes: # 사용할 PVC 정의
      - name: data
        persistentVolumeClaim:
          claimName: {PVC 이름} # 구성한 PVC를 참조하도록 지정
      containers: # 컨테이너 목록 정의
        - name: {컨테이너명}
          image: {이미지주소/이미지명:tag} # 컨테이너 이미지 경로
          ports:
	        - containerPort: 80 # 컨테이너 포트
	  volumeMounts: # 볼륨을 컨테이너 내에 마운트
          - name: data
             mountPath: {nfs 공유한 폴더 경로}
```

* * *

## Service 생성 방법 :
```yaml
apiVersion: v1 # 리소스의 API 버전을 지정
kind: Service # 리소스의 종류 지정
metadata:
  name: secloudit-sample
  namespace: {namespace 이름}
spec:
  type: NodePort # 서비스 타입 지정
  selector:
    sample-app: {디플로이먼트와 파드를 묶는 라벨 이름}
  ports:
    - protocol: TCP
      nodePort: {외부 접속 시 사용할 포트} # 노드의 {}번 포트로 들어오면
      port: 80 # 서비스의 80번 포트로 전달하고
      targetPort: 80 # Pod의 80번 포트로 전달 30001(Node) - 80(Service) - 80(Pod)
```

* * *

## PV 생성 방법 :
```yaml
apiVersion: v1 # 리소스의 API 버전을 지정
kind: PersistentVolume # 리소스의 종류 지정
metadata:
  name: nfs-storage # pv 이름
  labels: # PV에 적용할 레이블을 지정
    type: nfs # PV가 NFS 타입의 스토리지를 사용한다
#[2] PV의 스펙을 정의
spec:
  capacity:
    storage: 10Gi # PV의 스토리지 용량 지정
  accessModes: ["ReadWriteMany"] # PV의 접근 모드를 지정
  nfs: # NFS 서버에 대한 정보를 지정
    server: [NFS_SERVER_IP] # NFS 서버
    path: /mnt/shared # NFS 서버 경로
```

* * *

## PVC 생성 방법 :
```yaml
apiVersion: v1 # 리소스의 API 버전을 지정
kind: PersistentVolumeClaim # 리소스의 종류 지정
metadata:
  name: nfs-storage-claim # pvc 이름
spec:
  storageClassName: "" # 빈 문자열을 줍니다(중요)
  accessModes: ["ReadWriteMany"] # PV의 accessModes와 동일한 옵션을 사용해야 bound 가능
  resources:
    requests: # 사용을 원하는 볼륨의 요구조건을 명시
      storage: 4Gi # PV의 용량보다 크면 안됨
  selector:
    matchExpressions: # PVC가 바인딩될 PV를 선택하는 조건 지정
      - key: type
        operator: In # 라벨의 값이 values 리스트에 포함되는지 확인
        values:
          - nfs # PV의 label 중 'type'이 'nfs'인 PV를 선택하도록 설정
```

* * *

## Ingress 생성 방법 :
```yaml
apiVersion: networking.k8s.io/v1 # 리소스의 API 버전을 지정
kind: Ingress # 리소스의 종류 지정
metadata:
  name: color-web # ingress 이름
  annotations: # NGINX Ingress Controller에 특정 동작을 지시하는 주석
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx # 사용할 Ingress 클래스의 이름
  defaultBackend:
    service:
      name: blue-svc
      port:
        number: 80
  rules:
    - host: example.com # 접근할 호스트명
      http: # 트래픽을 처리하는 규칙 정의
        paths:
        - pathType: Prefix
          path: /blue # blue 경로로 들어오는 서비스로 라우팅
          backend:
            service:
              name: blue-svc
              port:
                number: 80
        - pathType: Prefix
          path: /green # green 경로로 들어오는 서비스로 라우팅
          backend:
            service:
              name: green-svc
              port:
                number: 80
    - host: color.com # 접근할 호스트명
      http: # 트래픽을 처리하는 규칙 정의
        paths:
        - pathType: Prefix
          path: /
          backend:
            service:
              name: yellow-svc
              port:
                number: 80
```

* * *