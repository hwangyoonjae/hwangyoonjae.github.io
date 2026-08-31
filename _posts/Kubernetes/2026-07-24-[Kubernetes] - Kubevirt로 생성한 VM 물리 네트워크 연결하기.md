---
layout: post
title: "Kubevirt로 생성한 VM 물리 네트워크 연결하기"
date: 2026-07-24
categories: [Kubernetes]
tags: [Kubernetes, Kubevirt]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. Multus + Bridge 이용하기:
### 1.1 Multus 설치하기 :

```bash
$ wget https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset-crio.yml
```

* * *

### 1.2 컨테이너 이미지 파일 준비하기 :

```bash
$ docker pull ghcr.io/k8snetworkplumbingwg/multus-cni:stable

$ docker save -o ghcr.io-k8snetworkplumbingwg-multus-cni-stable.tar ghcr.io/k8snetworkplumbingwg/multus-cni:stable

# harbor 서버에 컨테이너 이미지 파일 
$ docker load -i ghcr.io-k8snetworkplumbingwg-multus-cni-stable.tar

$ docker tag ghcr.io/k8snetworkplumbingwg/multus-cni:stable harbor.test.com/multus-cni:stable

$ docker push harbor.test.com/multus-cni:stable
```

* * *

### 1.3 Multus 배포하기 :

- 컨테이너 이미지 주소를 Harbor Repo 주소로 변경합니다.

```bash
$ vi multus-daemonset-crio.yml
```
```yaml
      containers:
        - name: kube-multus
          image: harbor.test.com/multus-cni:stable # 수정
          command: ["/thin_entrypoint"] # 수정
      initContainers:
        - name: install-multus-binary
          image: harbor.test.com/multus-cni:stable # 수정
```

* * *

- 변경 후, 배포합니다.

```bash
$ kubectl apply -f multus-daemonset-crio.yml

# 배포 후 정상 실행되었는지 확인
$ kubectl get po -n kube-system | grep multus
```

![multus pod 생성 확인 화면](/assets/img/post/kubernetes/multus%20pod%20생성%20확인%20화면.png)


* * *

- CRD도 생성되었는지 확인합니다.

```bash
$ kubectl get crd | grep network
```

![multus crd 생성 확인 화면](/assets/img/post/kubernetes/multus%20crd%20생성%20확인%20화면.png)

* * *

### 1.4 Multus가 CNI에 등록됐는지 확인하기 :

- 00-multus.conf가 생성되어 있어야 합니다.

```bash
$ ls /etc/cni/net.d
```

![multus cni 등록 확인](/assets/img/post/kubernetes/multus%20cni%20등록%20확인.png)

* * *

- 내용은 아래와 같습니다.

![multus cni 내용 확인](/assets/img/post/kubernetes/multus%20cni%20내용%20확인.png)

* * *

### 1.5 CRI-O CNI Plugin 경로 설정하기 :

- Multus DaemonSet을 배포하면 Worker Node에 Multus CNI 바이너리가 설치되어 먼저 Multus CNI 바이너리의 설치 위치를 확인합니다.

```bash
$ find /opt/cni /usr/libexec/cni /usr/lib/cni \
    -type f -name 'multus*' 2>/dev/null

# 아래와 같이 설치될 수 있습니다.
/opt/cni/bin/multus-shim
/usr/libexec/cni/multus
```

* * *

- CRI-O에서 현재 사용 중인 CNI Plugin 검색 경로를 확인합니다.

```bash
$ crio status config | grep -A10 -i network
```

![crio cni 플러그인 검색 경로 확인](/assets/img/post/kubernetes/crio%20cni%20플러그인%20검색%20경로%20확인.png)

> 00-multus.conf가 정상적으로 생성되어 있더라도 CRI-O의 plugin_dirs에 Multus 바이너리가 위치한 디렉터리가 포함되어 있지 않으면 Multus Secondary Network가 정상적으로 생성되지 않을 수 있기 때문에 Multus 바이너리가 /usr/libexec/cni/에 설치되어 있다면 해당 디렉터리도 CRI-O의 CNI Plugin 검색 경로에 추가해야 합니다.
{: .prompt-warning}

* * *

- CRI-O 설정 파일을 확인하여 다음 설정을 추가합니다.

```bash
$ vi /etc/crio/crio.conf.d/10-crio.conf

# 아래 내용을 추가합니다.
[crio.network]
network_dir = "/etc/cni/net.d/"
plugin_dirs = [
    "/opt/cni/bin/",
    "/usr/libexec/cni/",
]
```
```bash
# crio 서비스 재시작
$ systemctl restart crio
$ systemctl status crio
```

* * *

- 설정 변경 후 CRI-O가 정상적으로 설정을 읽는지 확인합니다.

```bash
$ crio status config | grep -A10 -i network
```

![crio cni 플러그인 검색 경로 추가 확인](/assets/img/post/kubernetes/crio%20cni%20플러그인%20검색%20경로%20추가%20확인.png)

* * *

## 2. Worker Node의 Bridge 생성하기 :
### 2.1 Worker Node IP 인터페이스 설정 백업하기 :

- 혹시 모를 상황에 대비하여 현재 네트워크 설정을 백업합니다.

```bash
# Connection Profile 백업
$ nmcli connection export ens192 /root/ens192.nmconnection

# 또는 설정 정보만 저장
$ nmcli connection show ens192 > /root/ens192.txt
```

* * *

### 2.2 Bridge 생성하기 :

- `br0` Bridge를 생성합니다.

```bash
$ nmcli connection add \
    type bridge \
    ifname br0 \
    con-name br0

# 생성 확인
$ nmcli connection show
```

![worker node bridge 설정하기](/assets/img/post/kubernetes/worker%20node%20bridge%20설정하기.png)

* * *

### 2.3 Bridge에 기존 Worker Node IP 설정하기 :

- 기존 Worker Node에서 사용 중인 IP를 Bridge(`br0`)에 설정합니다.

```bash
$ nmcli connection modify br0 \
    ipv4.method manual \
    ipv4.addresses 192.168.171.170/24 \
    ipv4.gateway 192.168.171.254 \
    ipv4.dns 192.168.171.1 \
    ipv6.method disabled
```

> IP 주소, Gateway, DNS는 환경에 맞게 변경하여 설정합니다.
{: .prompt-info}

* * *

### 2.4 Worker Node 인터페이스를 Bridge Slave로 변경하기 :

- 기존 `ens192` Connection을 삭제합니다.

```bash
# 기존 Connection 확인
$ nmcli connection show

# 기존 Connection 삭제
$ nmcli connection delete ens192
```

* * *

- 삭제 후 `ens192`를 Bridge Slave로 생성합니다.

```bash
$ nmcli connection add \
    type bridge-slave \
    ifname ens192 \
    master br0

# 생성 확인
$ nmcli connection show
```

* * *

- 예상 결과는 아래와 같습니다.

```text
NAME                   TYPE           DEVICE
br0                    bridge         br0
bridge-slave-ens192    ethernet       ens192
```

* * *

### 2.5 Bridge 활성화하기 :

- Bridge를 활성화합니다.

```bash
$ nmcli connection up br0
```

> `bridge-slave-ens192` Connection도 함께 활성화됩니다.
{: .prompt-tip}

* * *

- 필요한 경우 아래 명령으로 Slave를 직접 활성화할 수도 있습니다.

```bash
$ nmcli connection up bridge-slave-ens192
```

* * *

### 2.6 Bridge 구성 확인하기 : 

- Bridge가 정상적으로 구성되었는지 확인합니다.

```bash
# IP 확인
$ ip addr

# Bridge 연결 확인
$ bridge link

# Routing 확인
$ ip route
```

* * *

- 정상적으로 구성되었다면 다음과 같이 표시됩니다.

```text
br0
    inet 192.168.171.170/24

ens192
    master br0
```

```text
default via 192.168.171.254 dev br0
192.168.171.0/24 dev br0
```

> Worker Node의 IP가 `ens192`에서 `br0`로 이동하고, `ens192`는 Bridge Slave 상태가 되면 정상적으로 구성된 것입니다.
{: .prompt-tip}

* * *

## 3. NetworkAttachmentDefinition 생성하기 :
### 3.1 NetworkAttachmentDefinition 생성 :

- Multus를 통해 KubeVirt VM을 Worker Node의 br0에 연결하기 위해 NetworkAttachmentDefinition을 생성합니다.

```bash
$ vi br0-network.yaml
```
```yaml
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: br0-network
  namespace: default
spec:
  config: |
    {
      "cniVersion": "0.3.1",
      "type": "bridge",
      "bridge": "br0",
      "ipam": {}
    }
```

> ***cniversion***은 ***/etc/cni/net.d/00-multus.conf***파일에서 확인 가능합니다.
{: .prompt-tip}

* * *

- 적용하고, 확인합니다.

```bash
# 적용
$ kubectl apply -f br0-network.yaml
```
```bash
# 확인
$ kubectl get network-attachment-definitions -n default
```

![NetworkAttachmentDefinition 확인](/assets/img/post/kubernetes/NetworkAttachmentDefinition%20확인.png)

* * *

### 3.2 Multus + Bridge 연결 테스트하기 :

- 생성한 `NetworkAttachmentDefinition`이 정상적으로 동작하는지 확인하기 위해 테스트 Pod를 생성합니다.

> 테스트 Pod에는 `br0-network`를 Secondary Network로 연결합니다.
{: .prompt-info}

```bash
$ vi multus-br0-test.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multus-br0-test
  namespace: default
  annotations:
    k8s.v1.cni.cncf.io/networks: br0-network
spec:
  nodeSelector:
    kubernetes.io/hostname: worker03

  containers:
    - name: test
      image: harbor.test.com/library/busybox:latest
      command:
        - sleep
        - "3600"
```

* * *

- 테스트 Pod를 생성하고, 정상적으로 생성되었는지 확인합니다.

```bash
# 생성하기
$ kubectl apply -f multus-br0-test.yaml

# 확인하기
$ kubectl get pod multus-br0-test -n default -o wide
```

* * *

- Pod에 Multus Network가 요청되었는지 확인합니다.

```bash
$ kubectl get pod multus-br0-test -n default \
    -o jsonpath='{.metadata.annotations.k8s\.v1\.cni\.cncf\.io/networks}'

# 정상적으로 설정되어 있다면 아래와 같이 표시됩니다.
br0-network
```

* * *

- Multus가 실제로 Secondary Network를 생성했는지 확인합니다.

```bash
kubectl get pod multus-br0-test -n default \
    -o jsonpath='{.metadata.annotations.k8s\.v1\.cni\.cncf\.io/network-status}'
```

![Secondary Network를 생성 확인](/assets/img/post/kubernetes/Secondary%20Network를%20생성%20확인.png)

> network-status가 출력되지 않는 경우 Multus가 Secondary Network를 생성하지 못한 상태일 수 있습니다.
> 
> 이 경우 CRI-O의 CNI Plugin 경로와 Multus 바이너리 위치를 확인합니다.
{: .prompt-warning}

* * *

- Pod 내부에서 Network Interface를 확인합니다.

```bash
$ kubectl exec -n default multus-br0-test -- ip link
```

![multus + bridge network 연결 확인](/assets/img/post/kubernetes/multus%20+%20bridge%20network%20연결%20확인.png)

* * *

## 4. KubeVirt VM에 Physical Network 연결하기 :
### 4.1 KubeVirt VM 설정 적용하기 :

- KubeVirt VM의 interfaces와 networks에 생성한 br0-network를 연결합니다.

> 초기 테스트 단계에서는 기존 Pod Network와 Physical Network를 동시에 사용하는 구성이 편리합니다.
{: prompt-tip}

```yaml
spec:
  template:
    spec:
      domain:
        devices:
          interfaces:
            # Kubernetes 기본 Network
            - name: default
              masquerade: {}

            # Physical Network
            - name: physical-network
              bridge: {}

      networks:
        # Kubernetes 기본 Pod Network
        - name: default
          pod: {}

        # Multus Physical Network
        - name: physical-network
          multus:
            networkName: br0-network
```
```bash
$ kubectl apply -f rocky96-vm.yaml
```

* * *

- physical-network Interface는 Multus의 br0-network를 통해 Worker Node의 br0 Bridge에 연결됩니다.

```json
KubeVirt VM
   │
   ├─ default
   │    └─ Pod Network (Calico)
   │
   └─ physical-network
        └─ Multus
             └─ br0-network
                  └─ Worker Node br0
                       └─ ens192
                            └─ Physical Network
```

> interfaces와 networks에 정의한 name 값은 서로 동일해야 합니다.
{: .prompt-warning}

* * *

### 4.2 virt-launcher Pod Network 확인하기 :

- VM의 virt-launcher Pod를 확인합니다.

```bash
$ kubectl get pod -n default \
    -l kubevirt.io/domain=rocky96 \
    -o wide
```

* * *

- Multus Network가 정상적으로 연결되었는지 확인합니다.

```bash
$ POD=$(kubectl get pod -n default \
    -l kubevirt.io/domain=rocky96 \
    -o jsonpath='{.items[0].metadata.name}')

$ kubectl get pod ${POD} -n default \
    -o jsonpath='{.metadata.annotations.k8s\.v1\.cni\.cncf\.io/network-status}'
```

![Secondary Network 정보가 출력](/assets/img/post/kubernetes/Secondary%20Network%20정보가%20출력.png)

* * *

### 4.3 VM Physical Network Interface 확인하기 :

- VM Console에 접속하여 Network Interface를 확인합니다.

```bash
$ virtctl console rocky96 -n default

$ nmcli device status
```

![vm 내부 네트워크 인터페이스 확인](/assets/img/post/kubernetes/vm%20내부%20네트워크%20인터페이스%20확인.png)

* * *

- 각 Interface의 역할은 다음과 같습니다.

```json
eth0
 └─ Kubernetes 기본 Pod Network

eth1
 └─ Multus Physical Network
      └─ br0-network
           └─ Worker Node br0
                └─ Physical Network
```

> Network Interface 이름은 VM OS 환경에 따라 eth0, eth1, enp1s0, enp2s0 등으로 다를 수 있습니다. 반드시 nmcli device status 또는 ip link 명령어로 Physical Network Interface 이름을 확인합니다.
{: .prompt-warning}

* * *

### 4.4 VM에 Physical Network 고정 IP 설정하기 :

- Multus를 통해 연결된 Physical Network Interface에 Worker Node가 사용하는 물리 네트워크와 동일한 대역의 IP를 설정합니다.

```bash
$ nmcli connection add \
   type ethernet \
   ifname eth1 \
   con-name physical-network \
   ipv4.method manual \
   ipv4.addresses 192.168.171.179/24 \
   ipv4.gateway 192.168.171.254 \
   ipv4.never-default no \
   ipv4.route-metric 50 \
   ipv4.dns 192.168.171.1 \
   ipv6.method disabled \
   connection.autoconnect yes

# 생성한 Network Connection을 활성화
$ nmcli connection up physical-network
```

> 현재 VM은 Pod Network(eth0)와 Physical Network(eth1)를 동시에 사용하고 있으므로 테스트 단계에서는 ipv4.never-default yes를 설정합니다. 이를 통해 기존 Default Route는 eth0를 유지하고, 192.168.171.0/24 대역 통신만 eth1을 사용하도록 구성합니다.
{: .prompt-warning}

* * *

- 정상적으로 연결되었는지 확인합니다.

```bash
$ nmcli device status
```

![Physical Network Interface 생성하기](/assets/img/post/kubernetes/Physical%20Network%20Interface%20생성하기.png)

* * *

- Physical Network Interface에 IP가 정상적으로 설정되었는지 확인합니다.

```bash
$ ip addr show eth1
```

![Physical Network Interface IP 확인하기](/assets/img/post/kubernetes/Physical%20Network%20Interface%20IP%20확인하기.png)

* * *