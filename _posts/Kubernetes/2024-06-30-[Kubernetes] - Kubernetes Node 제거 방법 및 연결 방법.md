---
layout: post
title: "Kubernetes Node 제거 방법 및 연결 방법"
date: 2024-06-30
categories: [컨테이너, Kubernetes]
tags: [Kubernetes, MasterNode, WorkerNode]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. 노드 드레이닝(Node Draining) :
- 마스터/워커 노드를 삭제하기 전에 해당 노드에서 실행 중인 파드를 다른 노드로 이동해야합니다.
``` bash
$ kubectl drain <Master/Worker Node명> --ignore-daemonsets --delete-emptydir-data
```

>Master/Worker Node 삭제 시 왜 드레이닝(Draining)을 해야할까?
>
>👉 kubernetes 클러스터의 안정성과 데이터 손실을 방지하기 위함입니다.
>
>1. **서비스 연속성 유지** :
>- 노드를 드레인하면 해당 노드에서 실행 중인 파드가 다른 노드로 재배치되므로, 서비스 중단 없이 마스터 노드를 안전하게 삭제할 수 있습니다.
>
>2. **안전한 노드 제거** :
>- 드레인 과정을 통해 노드가 클러스터에서 제거되기 전에 파드가 안전하게 이동되므로, 데이터 손실이나 서비스 중단을 방지할 수 있습니다.
>
>3. **클러스터 안정성** :
> - 마스터 노드는 클러스터의 중요한 구성 요소 중 하나이므로, 이를 안전하게 제거하기 위해서는 모든 파드가 다른 노드로 이동되어야 클러스터의 안정성을 유지할 수 있습니다.
{: .prompt-tip }

> kubectl drain 명령어의 목적?
>1. 파드 이동 :
>`kubectl drain <node>` 명령어는 해당 노드에서 실행 중인 모든 파드를 안전하게 종료시키고 다른 노드로 이동시키며, 이렇게 하면 삭제하려는 노드에서 실행 중인 서비스의 가용성을 유지할 수 있습니다.
>
>2. 데이터 손실 방지 :
>`--delete-emptydir-data` 옵션은 EmptyDir 볼륨을 사용하는 파드의 데이터를 삭제해서 이를 통해 빈 디렉터리 볼륨을 사용하는 파드가 안전하게 종료되고 다른 노드로 이동할 수 있습니다.
>
>3. 데몬셋 무시 :
>`--ignore-daemonsets` 옵션은 데몬셋으로 실행 중인 파드를 무시하고 드레인 작업을 수행하고, 데몬셋 파드는 각 노드에서 반드시 하나씩 실행되어야 하기 때문에, 이를 무시하고 다른 파드만 이동시킨다.
{: .prompt-info }

* * *   

## 2. 마스터/워커 노드를 클러스터에서 삭제 :
- 마스터/워커 노드를 Kubernetes 클러스터에서 삭제합니다.
```bash
$ kubectl delete node <Master Node명>
```

* * *

## 3. 마스터/워커 노드에서 데이터 삭제 :
- etcd-pod에 접속합니다.
```bash
$ kubectl exec -it etcd-노드명 -n kube-system sh
```

- etcd 목록을 확인합니다.
```bash
$ etcdctl --cacert="/etc/kubernetes/pki/etcd/ca.crt" --cert="/etc/kubernetes/pki/etcd/server.crt" --key="/etc/kubernetes/pki/etcd/server.key" member list
```

- etcd pod의 접속하여 기존의 join된 마스터 노드의 정보를 삭제합니다.
```bash
$ etcdctl --cacert="/etc/kubernetes/pki/etcd/ca.crt" --cert="/etc/kubernetes/pki/etcd/server.crt" --key="/etc/kubernetes/pki/etcd/server.key" member remove <1열의ID값>
```

* * *

## 4. 마스터/워커 노드 추가 :
- kubeadm command를 통해 Master node join에 필요한 certificate key를 받습니다.

> 클러스터를 최초 구성할 때 사용한 kubeadm-config.yaml 파일이 필요하다.
{: .prompt-warning }

```bash
$ kubeadm init phase upload-certs --upload-certs --config=config.yaml
# 아래와 같이 출력됩니다.
#[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
#[upload-certs] Using certificate key:
#9371b8ad6368f4de7e8901788f619d59b99a9ae2d34ea8707e6ec475b554d143 <-- 복사
```

- 출력된 certificate key를 사용하여 join command를 생성합니다.

> { certificate key } 에 위에서 출력된 key를 넣어 실행합니다.
{: .prompt-warning }

```bash
$ kubeadm token create --certificate-key {certificate key} --print-join-command
```

* * *

## 5. 워커 노드 추가 :
> 마스터 노드에서 작업을 진행해야합니다.
{: .prompt-warning }

- 출력된 certificate key를 사용하여 join command를 생성합니다.
```bash
$ kubeadm token create --print-join-command
------아래 예시와 같이 출력됩니다.------
kubeadm join cluster.hwabul-saas.com:6443 --token ofr8mg.ng5426tamhp57b5h --discovery-token-ca-cert-hash sha256:a140170a070e861d88373
```