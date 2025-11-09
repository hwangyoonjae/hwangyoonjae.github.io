---
layout: post
title: "Kubernetes 클러스터 Grafana 모니터링"
date: 2025-03-31
categories: [컨테이너, Kubernetes] 
tags: [Kubernetes, Prometheus, Grafana]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. Grafana Ingress 생성하기 :

```bash
$ vi grafana-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-monitoring
  namespace: <Namespace 입력> # 이 부분에 실제 네임스페이스
spec:
  ingressClassName: nginx
  rules:
  - host: <Domain 주소> # 도메인 주소를 입력
    http:
      paths:
      - backend:
          service:
            name: <서비스명 입력> # 실제 서비스 이름을 입력
            port:
              number: 3000 # 서비스 포트
        path: /
        pathType: Prefix
```

```bash
$ kubectl apply -f grafana-ingress.yaml
```

---

## 2. Coredns의 정적 호스트 매핑하기 :
- configmap에 IP, 도메인 주소를 입력한다.

```bash
$ kubectl edit configmap coredns -n kube-system
```

```bash
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        hosts {
            <ip주소> <domain주소> # 해당 부분 추가
            fallthrough
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }

# 수정 후
:wq
```

---

## 3. Prometheus 데이터 소스 URL 입력하기 :
- 햄버거 버튼 > Connections > Your Connections 클릭한다.

![prometheus 데이터 소스URL 입력 1](/assets/img/post/kubernetes/prometheus%20데이터%20소스URL%20입력%201.png)

* * *

- Add new data source 버튼 클릭한다.

![prometheus 데이터 소스URL 입력 2](/assets/img/post/kubernetes/prometheus%20데이터%20소스URL%20입력%202.png)

* * *

- Prometheus 클릭한다.

![prometheus 데이터 소스URL 입력 3](/assets/img/post/kubernetes/prometheus%20데이터%20소스URL%20입력%203.png)

* * *

- 이름, URL 주소 입력 후, 스크롤 내려서 아래 “Save & test“ 버튼 클릭한다.

> Prometheus의 도메인 주소는 환경에 맞게 입력 필요합니다.
{: .prompt-warning}

![prometheus 데이터 소스URL 입력 4](/assets/img/post/kubernetes/prometheus%20데이터%20소스URL%20입력%204.png)
![prometheus 데이터 소스URL 입력 5](/assets/img/post/kubernetes/prometheus%20데이터%20소스URL%20입력%205.png)

---

## 4. Grafana 모니터링 확인하기 :
- 햄버거 버튼 > Dashboards 클릭한다.

![Grafana 모니터링 확인하기 1](/assets/img/post/kubernetes/Grafana%20모니터링%20확인하기%201.png)

* * *

- Default 클릭하여 아래 대시보드를 클릭하여 필요한 대시보드를 띄웁니다.

> 다른 대시보드를 원할 경우 인터넷에 검색하여 찾아서 json 파일 적용하면됩니다.
{: .prompt-warning}

![Grafana 모니터링 확인하기 2](/assets/img/post/kubernetes/Grafana%20모니터링%20확인하기%202.png)

* * *

- 필자는 아래 주소를 통해 Grafana Dashboard 오픈소스를 사용했다.
> * [Grafana Dashboard 다운로드](https://grafana.com/grafana/dashboards/15661-k8s-dashboard-en-20250125/ "Grafana Dashboard 다운로드")

![grafana 대시보드 화면](/assets/img/post/kubernetes/grafana%20대시보드%20화면.png)

---