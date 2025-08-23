---
layout: post
title: "Kubernetes Jenkins 설치하기"
date: 2025-06-26
categories: [컨테이너, Kubernetes] 
tags: [Kubernetes, Jenkins]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## Jenkins 설치하기:
### Namespace 생성하기:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: jenkins
```

```bash
$ kubectl create -f namespace.yaml
```

* * *

### ServiceAccount 생성하기:

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-admin
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin
  namespace: jenkins
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-admin
subjects:
- kind: ServiceAccount
  name: jenkins-admin
  namespace: jenkins
```

```bash
$ kubectl create -f serviceaccount.yaml
```

* * *

### PVC 생성하기:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pv-claim
  namespace: jenkins
spec:
  storageClassName: nfs-client # NFS 사용하는 경우
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

```bash
$ kubectl create -f pvc.yaml
```

* * *

### Deployment 생성하기:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins-server
  template:
    metadata:
      labels:
        app: jenkins-server
    spec:
      securityContext:
            fsGroup: 1000
            runAsUser: 1000
      serviceAccountName: jenkins-admin
      containers:
        - name: jenkins
          image: harbor.inno.com/jenkins/jenkins:latest
          resources:
            limits:
              memory: "2Gi"
              cpu: "1000m"
            requests:
              memory: "500Mi"
              cpu: "500m"
          ports:
            - name: httpport
              containerPort: 8080
            - name: jnlpport
              containerPort: 50000
          livenessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 90
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 5
          readinessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          volumeMounts:
            - name: jenkins-data
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkins-data
          persistentVolumeClaim:
              claimName: jenkins-pv-claim
```

```bash
$ kubectl create -f deployment.yaml
```

* * *

### Service 생성하기:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: jenkins
  annotations:
      prometheus.io/scrape: 'true'
      prometheus.io/path:   /
      prometheus.io/port:   '8080'
spec:
  selector:
    app: jenkins-server
  ports:
    - port: 8080
      targetPort: 8080
```

```bash
$ kubectl create -f service.yaml
```

* * *

### Ingress 생성하기:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: jenkins-ingress
  namespace: jenkins
spec:
  ingressClassName: nginx
  rules:
  - host: {Domain 주소}
    http:
      paths:
      - backend:
          service:
            name: jenkins-service
            port:
              number: 8080
        path: /
        pathType: Prefix
```

```bash
$ kubectl create -f ingress.yaml
```

* * *

## Jenkins 세팅하기:
### Ingress를 통해서 Jenkins 접속하기:
- Ingress 구성 시 작성한 Domain 주소로 접속합니다.

> DNS 서버가 없을 경우 PC /etc/hosts의 등록해줘야 접속 가능합니다.
{: .prompt-warning}

- 접속 시 아래와 같이 패스워드 입력 화면이 나옵니다.

![Jenkins 패스워드 입력](/assets/img/post/kubernetes/Jenkins%20패스워드%20입력.png)

* * *

### Administor Password 입력하기:

- Jenkins POD의 접속하여 Administor 패스워드를 확인하여 입력한다.

```bash
$ kubectl exec {jenkins pod명} -n jenkins -it /bin/bash
$ cat /var/jenkins_home/secrets/initialAdminPassword
```

![Jenkins administor password 확인방법](/assets/img/post/kubernetes/Jenkins%20administor%20password%20확인방법.png)

* * *

### Plugin 설치하기:

- 필자는 인터넷 연결이 안된 상태에서 설치 진행하여 skip 하였습니다.

![plugin 설치](/assets/img/post/kubernetes/plugin%20설치.png)

* * *

### Jenkins 계정 생성하기:

![jenkins 계정 생성](/assets/img/post/kubernetes/jenkins%20계정%20생성.png)

* * *

### Jenkins 접속 주소 지정하기:

![jenkins 접속 주소 지정](/assets/img/post/kubernetes/jenkins%20접속%20주소%20지정.png)

* * *

### Jenkins 웹 페이지 확인하기:

![Jenkins 설정 완료](/assets/img/post/kubernetes/Jenkins%20설정%20완료.png)
![Jenkins 로그인 후 화면](/assets/img/post/kubernetes/Jenkins%20로그인%20후%20화면.png)

* * *