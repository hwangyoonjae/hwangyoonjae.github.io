---
layout: post
title: "Nexus Repository Proxy 생성 및 사용방법"
date: 2026-06-30
categories: [DevOps, Nexus]
tags: [Nexus, Repo, DMZ]
image: /assets/img/post-title/nexus-wallpaper.jpg
---

## 1. Nexus 구축하기 :

> 필자는 DMZ망, OA망에 각각 Nexus 1대씩 구성하였으며, 망별 구성환경은 아래와 같습니다.
>
> DMZ망 : Docker
>
> OA망 : Kubernetes(Pod)
{: .prompt-info}

* * *

### 1.1 DMZ망에 Nexus 구성하기 :

```bash
$ vi docker-conmpose.yaml

# 아래 내용 입력
services:
  nexus3:
    container_name: "nexus3"
    image: harbor.test.com/nexus/nexus3:3.93.2
    environment:
      - TZ=Asia/Seoul
    res
    ports:
      - 8081:8081
    volumes:
      - ./nexus-default.properties:/opt/sonatype/nexus/etc/nexus-default.properties
      - ./ssl:/opt/sonatype/nexus/etc/ssl
      - ./nexus-data:/nexus-data
```

```bash
# nexus-default.properties

application-host=0.0.0.0
application-port=8081
## https로 구성할 경우 아래 내용으로 입력
#application-port-ssl=8443
#ssl.etc=/opt/sonatype/nexus/etc/ssl

nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
## https로 구성할 경우 아래 내용으로 입력
#nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-https.xml,${jetty.etc}/jetty-requestlog.xml
nexus-context-path=/${NEXUS_CONTEXT}

# Nexus section
nexus-edition=nexus-pro-edition
nexus-features=\
 nexus-pro-feature

nexus.hazelcast.discovery.isEnabled=true
```

* * *

- 위와 같이 YAML 파일 생성 후 배포합니다.

```bash
$ docker compose up -d
```

* * *

### 1.2 OA망에 Nexus 구성하기 :

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nexus-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: nfs-client
  resources:
    requests:
      storage: 10Gi
```

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus
  labels:
    app.kubernetes.io/name: nexus
    app.kubernetes.io/instance: nexus
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: nexus
      app.kubernetes.io/instance: nexus
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nexus
        app.kubernetes.io/instance: nexus
    spec:
      initContainers:
      containers:
        - name: nexus
          image: "harbor.test.com/library/sonatype/nexus3:3.93.2"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8081
          volumeMounts:
            - name: nexus-data
              mountPath: /nexus-data
          resources:
            requests:
              memory: 1Gi
              cpu: 500m
            limits:
              memory: 2Gi
              cpu: 1000m
      volumes:
        - name: nexus-data
          persistentVolumeClaim:
            claimName: nexus-pvc
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nexus
  labels:
    app.kubernetes.io/name: nexus
    app.kubernetes.io/instance: nexus
spec:
  selector:
    app.kubernetes.io/name: nexus
    app.kubernetes.io/instance: nexus
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
  type: ClusterIP
```

* * *

- 위와 같이 YAML 파일 생성 후 배포합니다.

```bash
$ kubectl apply -f pvc.yaml
$ kubectl apply -f deployment.yaml
$ kubectl apply -f service.yaml
```

* * *

> Kubernetes로 Nexus 배포 후 Repository 생성 시 SSRF 보안 정책으로 생성이 불가할 경우 아래와 같이 설정 부탁드립니다.
{: .prompt-warning}

```bash
curl -k -u admin:[PASSWORD] \
-X PUT \
-H "Content-Type: application/json" \
https://[NEXUS_URL:PORT]/service/rest/v1/security/ssrf-protection \
-d '{
  "enabled": false,
  "allowPrivateNetworks": false
}'
```

* * *

### 1.3 OA망 Nexus의 Repository 생성하기 :

- DMZ Nexus에는 외부 저장소를 바라보는 Proxy가 있어야하며 생성합니다.

```bash
Name: maven-dmz-proxy
Type: proxy
Format: maven2
Remote Storage URL: http://DMZ_NEXUS_URL/repository/maven-public/
```
![dmz-repo-proxy 생성](/assets/img/post/kubernetes/dmz-repo-proxy%20생성.png)

* * *

- 그리고 OA Nexus의 Group Repository를 구성합니다.

```bash
maven-public
Type: group
Format: maven2
Member:
  - maven-releases
  - maven-snapshots
  - maven-dmz-proxy # 해당 부분 추가
```
![oa망 group repo 생성](/assets/img/post/kubernetes/oa망%20group%20repo%20생성.png)

* * *

## 2. Nexus 라이브러리 Proxy 연동 테스트 :

> OA망 Nexus에서 요청한 라이브러리가 존재하지 않을 경우, DMZ망 Nexus를 통해 외부 저장소(Maven Central)에서 다운로드하고, 이후 OA망 및 DMZ망 Nexus에 정상적으로 캐시되는지 확인합니다.
{: .prompt-tip}

- OA Nexus의 maven-public Repository를 통해 라이브러리를 요청합니다.

```bash
$ curl -O http://nexus.test.com/repository/maven-public/org/springframework/spring-core/6.2.0/spring-core-6.2.0.jar
```

* * *

- 라이브러리 요청 시 아래와 같은 순서로 동작하는지 확인합니다.

```text
사용자
   │
   ▼
OA Nexus
   │ (라이브러리 없음)
   ▼
DMZ Nexus
   │ (라이브러리 없음)
   ▼
Maven Central
   │
   ▼
DMZ Nexus 캐시 저장
   │
   ▼
OA Nexus 캐시 저장
   │
   ▼
사용자에게 라이브러리 제공
```

* * *

- 다운로드 완료 후 다음 저장소에 라이브러리가 생성되었는지 확인합니다.

- **OA Nexus**
  - `maven-dmz-proxy`

- **DMZ Nexus**
  - `maven-central`

![dmz,oa망 라이브러리 다운로드](/assets/img/post/kubernetes/dmz,oa망%20라이브러리%20다운로드.png)

* * *