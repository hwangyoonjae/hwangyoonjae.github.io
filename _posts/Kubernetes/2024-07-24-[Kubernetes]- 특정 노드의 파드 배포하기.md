---
layout: post
title: "특정 노드의 파드 배포하기"
date: 2024-07-24
categories: [컨테이너, Kubernetes]
tags: [Kubernetes, label, selector]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. label selector 이용하여 특정 노드에 파드 배포하기 :
### 1.1 label seclector 지정하기 :
- 노드의 라벨링을 지정합니다.

```bash
$ kubectl label nodes {node명} {label-key}={label-value}
```

* * *

- pod의 spec 부분에 nodeSelector를 설정합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-selector-pod
spec:
  containers:
  - name: my-container
    image: nginx
  ## 아래 부분 참고
  nodeSelector:
    label-key: label-value
```

* * *

### 1.2 Node의 label이 설정되었는지 확인하기 :
```bash
$ kubectl describe nodes/{node명}
```

* * *

### 1.3 label selector 삭제하기 :
- 라벨을 삭제하려면 라벨 키 뒤에 대시(-)를 붙여 명령어를 실행하면된다.

```bash
$ kubectl label node {node명} {label-key}-
```

* * *

### 1.4 label 삭제 확인하기 :
- 라벨이 성공적으로 삭제되었는지 확인하려면 다음 명령어로 노드의 라벨 목록을 조회합니다.

```bash
$ kubectl get nodes --show-labels
```

* * *

## 1.5 label selector의 한계 :
- 하나이상의 node를 선택할 수 없다.
- 다른 라벨이 지정된 node에 배포하고자 한다면 nodeSelector로는 한계가 있어 **NodeAffinity**를 사용합니다.

```yaml
$ apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - medium
            - large
```

> NodeAffinity를 사용하면 위와같이 medium과 large인 노드에 배포가능하다.
{: .prompt-tip }

* * *