---
layout: post
title: "Kubernetes Readiness 시연 테스트"
date: 2025-06-16
categories: [컨테이너, Kubernetes] 
tags: [Kubernetes, Readiness, Service]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## Readiness Probe가 설정된 Deployment 작성하기:

- "/ready" 경로로 HTTP 요청을 보내 확인하도록 구성

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: readiness-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: readiness-demo
  template:
    metadata:
      labels:
        app: readiness-demo
    spec:
      containers:
      - name: app
        image: <Harbor주소>/<프로젝트명>/<이미지명>:<태그>
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

> 해당 Pod는 /ready 경로가 없으므로, 초기 상태부터 Readiness에 실패하게 된다.
{: .prompt-warning}

* * *

## 배포 후 Pod 상태 확인하기:
```bash
$ kubectl get pods
```

![Readiness 실패](/assets/img/post/kubernetes/Readiness%20실패.png)

* * *

## 서비스에 연결되지 않는지 확인하기:

```bash
$ kubectl describe pod <pod-name>
```

> 출력 중 Readiness probe failed: 메시지가 보이면, probe가 작동 중이다.
{: .prompt-tip}

![Readiness 실패 로그](/assets/img/post/kubernetes/Readiness%20실패%20로그.png)

* * *

## 정상 상태로 변경하여 Ready 상태 확인하기:
### Nginx 경로 수정하기:
- Nginx의 경우 /ready 경로가 없기 때문에, 대신 /index.html을 readiness path로 바꾸면 통과한다.

```yaml
readinessProbe:
  httpGet:
    path: /index.html
    port: 80
```
```bash
$ kubectl apply -f readiness-demo.yaml
```

* * *

### Pod Ready 상태 확인하기:
- Ready 상태가 1/1이 된다.

```bash
$ kubectl get pods
```

![Readiness 성공](/assets/img/post/kubernetes/Readiness%20성공.png)

* * *