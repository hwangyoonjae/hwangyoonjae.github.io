---
layout: post
title: "Kubernetes Node Taint & Tolerations"
date: 2024-07-24
categories: [컨테이너, Kubernetes] 
tags: [Kubernetes, taint, toleration]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## Taints와 Toleration? :
### Taint란? :
- 특정 노드에 추가되는 속성으로, 그 노드에 특정 Pod가 스케줄링되는 것을 제한하거나 방지한다.

### Toleration이란? :
- Pod의 속성으로, 특정 Taint를 무시하고 해당 노드에 스케줄링될 수 있도록 허용하는 것이다.

* * *

## Taints와 Tolerations 동작 원리 :
- **Taint**: 노드에 설정되며, 해당 노드에서 Pod의 스케줄링을 방해한다.
  - 노드에 kubectl taint nodes nodename key=value:NoSchedule로 Taint를 설정하면, 기본적으로 이 노드에는 어떠한 Pod도 스케줄링되지 않는다.
- **Toleration**: Pod에 설정되며, 특정 Taint를 무시하고 해당 노드에 스케줄링될 수 있도록 허용한다.
  - toleration의 effect, key, value가 일치하면, 해당 Taint를 가진 노드에도 Pod가 스케줄링될 수 있다.

* * *

## 노드의 taint 설정하기 :
- 노드의 taint 설정하는 방법은 아래와 같다.

```bash
$ kubectl taint node {nodename} {key}={value}:{option}
# kubectl taint node k8s-worker1 size=large:NoSchedule
```

> option : Taint에 대한 역할 제한이다. 일반적으로 NoSchedule, PreferNoschedule, NoExecute 3가지 설정이 존재한다.
{: .prompt-info }

| Effect 종류 | 개요 |
| :---------------- | :-------------------------------------------- |
| PreferNoSchedule | 가능한 한 스케줄링하지 않음 |
| NoSchedule | 스케줄하지 않음 (이미 스케줄링된 Pod는 그대로 유지) |
| NoExecute| 실행을 허가하지 않음 (이미 스케줄링된 Pod는 정지됨) |

- taint 해제 방법은 아래와 같다.

```bash
# 특정 taint 해제 방법
$ kubectl taint node k8s-worker1 {value}:{option}-

# 모든 taint 해제 방법
$ kubectl taint node k8s-worker1 {value}-
```

* * *

## 파드의 tolerations 설정하기 :

```yaml
# tolerations-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerations-pod
spec:
  containers:
    - name: tolerations-container
      image: harbor.com/nginx:1.25.3
      ports:
        - containerPort: 80
  tolerations:
    - key: "key1"
      operator: "Equal"
      value: "value1"
      effect: "NoSchedule"
    - key: "key2"
      operator: "Exists"
      effect: "NoExecute"
```

> tolerations에서 여러 개의 key 값을 작성하는 이유는?
> 파드가 노드의 다양한 taints를 무시(생성)하도록 허용하기 위해서다.
{: .prompt-tip}

```bash
$ kubectl create -f tolerations-pod.yaml
```

* * *