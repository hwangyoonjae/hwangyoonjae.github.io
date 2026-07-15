---
layout: post
title: "Kubevirt 구성하기"
date: 2026-07-09
categories: [Kubernetes]
tags: [Kubernetes, Kubevirt]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. KVM 확인하기 :
### 1.1 CPU에 VT-x(vmx) 노출 확인하기 :

```bash
$ egrep -c '(vmx|svm)' /proc/cpuinfo
## 아래처럼 표기됩니다.
# 48
```

* * *

### 1.2 CPU에서 Virtualization 지원 확인하기 :

```bash
$ lscpu | grep -i Virtualization
## 아래처럼 표기됩니다.
# Virtualization:        VT-x
# Virtualization type:   full
```

* * *

### 1.3 KVM 모듈 로드하기 :

```bash
modprobe kvm
modprobe kvm_intel
## 에러가 없으면 됩니다.
```

* * *

### 1.4 모듈 확인하기 :

```bash
$ lsmod | grep kvm

## 아래처럼 표기되어야합니다.
# kvm_intel     446464  0
# kvm           1404928  1 kvm_intel
```

* * *

### 1.5 /dev/kvm 생성 확인하기 :

```bash
$ ls -l /dev/kvm

## 아래처럼 표기되어야합니다.
# crw-rw-rw- 1 root kvm 10, 232 Jul  9 14:56 /dev/kvm
```

* * *

## 2. KubeVirt 설치하기 :
### 2.1 Kubevirt란? :
- 쿠버네티스 위에서 가상머신을 실행할 수 있게 해주는 가상화 확장 플랫폼이며, VM도 쿠버네티스 리소스처럼 YAML로 생성하고 관리할 수 있게 해주는 기술입니다.

* * *

### 2.2 KubeVirt 버전 선택하기 :

- 먼저 설치 yaml을 받아서 어떤 이미지가 필요한지 확인합니다.

```bash
$ export KV_VER=v1.8.4
$ curl -LO https://github.com/kubevirt/kubevirt/releases/download/${KV_VER}/kubevirt-operator.yaml
$ curl -LO https://github.com/kubevirt/kubevirt/releases/download/${KV_VER}/kubevirt-cr.yaml

# 이미지 목록 확인
$ grep image: kubevirt-*.yaml
```

* * *

### 2.3 KubeVirt 이미지 복사하기 :

- 위 과정에서 진행한 이미지 목록 확인 후 다운받아 Harbor 서버에 저장합니다.

```bash
$ docker pull quay.io/kubevirt/virt-operator:v1.8.4
$ docker pull quay.io/kubevirt/virt-api:v1.8.4
$ docker pull quay.io/kubevirt/virt-controller:v1.8.4
$ docker pull quay.io/kubevirt/virt-handler:v1.8.4
$ docker pull quay.io/kubevirt/virt-launcher:v1.8.4

$ docker save -o quay.io-kubevirt-virt-operator-v1.8.4.tar quay.io/kubevirt/virt-operator:v1.8.4
$ docker save -o quay.io-kubevirt-virt-api-v1.8.4.tar quay.io/kubevirt/virt-api:v1.8.4
$ docker save -o quay.io-kubevirt-virt-controller-v1.8.4.tar quay.io/kubevirt/virt-controller:v1.8.4
$ docker save -o quay.io-kubevirt-virt-handler-v1.8.4.tar quay.io/kubevirt/virt-handler:v1.8.4
$ docker save -o quay.io-kubevirt-virt-launcher-v1.8.4.tar quay.io/kubevirt/virt-launcher:v1.8.4

# 위에서 저장한 이미지를 Harbor 서버에 업로드 후
$ docker load -i quay.io-kubevirt-virt-operator-v1.8.4.tar
$ docker load -i quay.io-kubevirt-virt-api-v1.8.4.tar
$ docker load -i quay.io-kubevirt-virt-controller-v1.8.4.tar
$ docker load -i quay.io-kubevirt-virt-handler-v1.8.4.tar
$ docker load -i quay.io-kubevirt-virt-launcher-v1.8.4.tar

$ docker tag quay.io/kubevirt/virt-operator:v1.8.4 harbor.test.com/kubevirt/virt-operator:v1.8.4
$ docker tag quay.io/kubevirt/virt-api:v1.8.4.tar harbor.test.com/kubevirt/virt-api:v1.8.4
$ docker tag quay.io/kubevirt/virt-controller:v1.8.4.tar harbor.test.com/kubevirt/virt-controller:v1.8.4
$ docker tag quay.io/kubevirt/virt-handler:v1.8.4.tar harbor.test.com/kubevirt/virt-handler:v1.8.4
$ docker tag quay.io/kubevirt/virt-launcher-v1.8.4 harbor.test.com/kubevirt/virt-launcher:v1.8.4

$ docker push harbor.test.com/kubevirt/virt-operator:v1.8.4
$ docker push harbor.test.com/kubevirt/virt-api:v1.8.4
$ docker push harbor.test.com/kubevirt/virt-controller:v1.8.4
$ docker push harbor.test.com/kubevirt/virt-handler:v1.8.4
$ docker push harbor.test.com/kubevirt/virt-launcher:v1.8.4
```

* * *

### 2.4 KubeVirt Operator 생성하기 :

- 외부 이미지 대신 Harbor를 사용하도록 수정합니다.

```bash
$ vi kubevirt-operator.yaml

# 아래 내용 수정
image: harbor.test.com/kubevirt/virt-operator:v1.8.4
```

* * *

- 변경 후 설치합니다.

```bash
$ kubectl apply -f kubevirt-operator.yaml

# 생성 확인
$ kubectl get pods -n kubevirt
```

![kubevirt 설치 확인하기](/assets/img/post/kubernetes/kubevirt%20설치%20확인하기.png)

* * *

### 2.5 KubeVirt CR 생성하기 :

- virt-api / virt-controller / virt-handler / virt-launcher는 kubevirt-operator.yaml에 직접 안 보이고, virt-operator가 kubevirt-cr.yaml을 보고 생성하기 때문에 Harbor 주소를 설정해야합니다.

```bash
$ vi kubevirt-cr.yaml
```
```yaml
# 아래와 같이 수정합니다.
apiVersion: kubevirt.io/v1
kind: KubeVirt
metadata:
  name: kubevirt
  namespace: kubevirt
spec:
  imageRegistry: harbor.test.com/kubevirt # 추가
  imageTag: v1.8.4                        # 추가
  certificateRotateStrategy: {}
  configuration:
    developerConfiguration:
      featureGates: []
    imagePullPolicy: IfNotPresent
  customizeComponents: {}
  imagePullPolicy: IfNotPresent
  workloadUpdateStrategy: {}
```

```bash
$ kubectl apply -f kubevirt-cr.yaml

# 생성 확인
$ kubectl get kubevirt -n kubevirt
$ kubectl get pods -n kubevirt
```

![kubevirt 정상 설치 확인](/assets/img/post/kubernetes/kubevirt%20정상%20설치%20확인.png)

* * *

## 3. CDI(Containerized Data Importer) 설치하기 :
### 3.1 CDI란? :

* * *

### 3.2 CDI 버전 선택하기 :

- 먼저 설치 yaml을 받아서 어떤 이미지가 필요한지 확인합니다.

```bash
$ export CDI_VERSION=v1.65.0
$ curl -LO https://github.com/kubevirt/containerized-data-importer/releases/download/${CDI_VERSION}/cdi-operator.yaml
$ curl -LO https://github.com/kubevirt/containerized-data-importer/releases/download/${CDI_VERSION}/cdi-cr.yaml

# 이미지 목록 확인
$ grep image: cdi-*.yaml
```

* * *

### 3.3 KubeVirt 이미지 복사하기 :

- 위 과정에서 진행한 이미지 목록 확인 후 다운받아 Harbor 서버에 저장합니다.

```bash
$ docker pull quay.io/kubevirt/cdi-operator:v1.65.0
$ docker pull quay.io/kubevirt/cdi-controller:v1.65.0
$ docker pull quay.io/kubevirt/cdi-importer:v1.65.0
$ docker pull quay.io/kubevirt/cdi-cloner:v1.65.0
$ docker pull quay.io/kubevirt/cdi-importer:v1.65.0
$ docker pull quay.io/kubevirt/cdi-apiserver:v1.65.0
$ docker pull quay.io/kubevirt/cdi-uploadserver:v1.65.0
$ docker pull quay.io/kubevirt/cdi-uploadproxy:v1.65.0

$ docker save -o quay.io-kubevirt-cdi-operator-v1.65.0.tar quay.io/kubevirt/cdi-operator:v1.65.0
$ docker save -o quay.io-kubevirt-cdi-controller-v1.65.0.tar    quay.io/kubevirt/cdi-controller:v1.65.0
$ docker save -o quay.io-kubevirt-cdi-importer-v1.65.0.tar      quay.io/kubevirt/cdi-importer:v1.65.0
$ docker save -o quay.io-kubevirt-cdi-cloner-v1.65.0.tar        quay.io/kubevirt/cdi-cloner:v1.65.0
$ docker save -o quay.io-kubevirt-cdi-importer-v1.65.0.tar      quay.io/kubevirt/cdi-importer:v1.65.0
$ docker save -o quay.io-kubevirt-cdi-apiserver-v1.65.0.tar     quay.io/kubevirt/cdi-apiserver:v1.65.0
$ docker save -o quay.io-kubevirt-cdi-uploadserver-v1.65.0.tar  quay.io/kubevirt/cdi-uploadserver:v1.65.0
$ docker save -o quay.io-kubevirt-cdi-uploadproxy-v1.65.0.tar   quay.io/kubevirt/cdi-uploadproxy:v1.65.0

$ docker load -i quay.io-kubevirt-cdi-operator-v1.65.0.tar
$ docker load -i quay.io-kubevirt-cdi-controller-v1.65.0.tar  
$ docker load -i quay.io-kubevirt-cdi-importer-v1.65.0.tar    
$ docker load -i quay.io-kubevirt-cdi-cloner-v1.65.0.tar      
$ docker load -i quay.io-kubevirt-cdi-importer-v1.65.0.tar    
$ docker load -i quay.io-kubevirt-cdi-apiserver-v1.65.0.tar   
$ docker load -i quay.io-kubevirt-cdi-uploadserver-v1.65.0.tar
$ docker load -i quay.io-kubevirt-cdi-uploadproxy-v1.65.0.tar 

$ docker tag quay.io/kubevirt/cdi-operator:v1.65.0      harbor.test.com/kubevirt/cdi-operator:v1.65.0
$ docker tag quay.io/kubevirt/cdi-controller:v1.65.0    harbor.test.com/kubevirt/cdi-controller:v1.65.0  
$ docker tag quay.io/kubevirt/cdi-importer:v1.65.0      harbor.test.com/kubevirt/cdi-importer:v1.65.0
$ docker tag quay.io/kubevirt/cdi-cloner:v1.65.0        harbor.test.com/kubevirt/cdi-cloner:v1.65.0
$ docker tag quay.io/kubevirt/cdi-importer:v1.65.0      harbor.test.com/kubevirt/cdi-importer:v1.65.0
$ docker tag quay.io/kubevirt/cdi-apiserver:v1.65.0     harbor.test.com/kubevirt/cdi-apiserver:v1.65.0
$ docker tag quay.io/kubevirt/cdi-uploadserver:v1.65.0  harbor.test.com/kubevirt/cdi-uploadserver:v1.65.0
$ docker tag quay.io/kubevirt/cdi-uploadproxy:v1.65.0   harbor.test.com/kubevirt/cdi-uploadproxy:v1.65.0

$ docker push harbor.test.com/kubevirt/cdi-operator:v1.65.0
$ docker push harbor.test.com/kubevirt/cdi-controller:v1.65.0  
$ docker push harbor.test.com/kubevirt/cdi-importer:v1.65.0
$ docker push harbor.test.com/kubevirt/cdi-cloner:v1.65.0
$ docker push harbor.test.com/kubevirt/cdi-importer:v1.65.0
$ docker push harbor.test.com/kubevirt/cdi-apiserver:v1.65.0
$ docker push harbor.test.com/kubevirt/cdi-uploadserver:v1.65.0
$ docker push harbor.test.com/kubevirt/cdi-uploadproxy:v1.65.0
```

* * *

### 3.4 CDI Operator 생성하기 :

- 외부 이미지 대신 Harbor를 사용하도록 수정하고, 

```bash
$ vi cdi-operator.yaml

# 아래 내용 수정
      containers:
      - env:
        - name: CONTROLLER_IMAGE
          value: harbor.test.com/kubevirt/cdi-controller:v1.65.0
        - name: IMPORTER_IMAGE
          value: harbor.test.com/kubevirt/cdi-importer:v1.65.0
        - name: CLONER_IMAGE
          value: harbor.test.com/kubevirt/cdi-cloner:v1.65.0
        - name: OVIRT_POPULATOR_IMAGE
          value: harbor.test.com/kubevirt/cdi-importer:v1.65.0
        - name: APISERVER_IMAGE
          value: harbor.test.com/kubevirt/cdi-apiserver:v1.65.0
        - name: UPLOAD_SERVER_IMAGE
          value: harbor.test.com/kubevirt/cdi-uploadserver:v1.65.0
        - name: UPLOAD_PROXY_IMAGE
          value: harbor.test.com/kubevirt/cdi-uploadproxy:v1.65.0
        - name: MONITORING_NAMESPACE
        image: harbor.test.com/kubevirt/cdi-operator:v1.65.0
```

* * *

- 변경 후 설치합니다.

```bash
$ kubectl apply -f cdi-operator.yaml

# 생성 확인
$ kubectl get pods -n cdi
```

![cdi 설치 확인하기](/assets/img/post/kubernetes/cdi%20설치%20확인하기.png)

* * *

### 2.5 CDI CR 생성하기 :

```bash
$ kubectl apply -f cdi-cr.yaml

# 생성 확인
$ kubectl get cdi -n cdi
$ kubectl get pods -n cdi
```

![cdi 정상 설치 확인](/assets/img/post/kubernetes/cdi%20정상%20설치%20확인.png)

* * *

## 4. VM 생성하기 :
### 4.1 테스트 VM 이미지 준비하기 :

```bash
$ docker pull quay.io/kubevirt/cirros-container-disk-demo:20260709_7c2959d25a

$ docker save -o quay.io-kubevirt-cirros-container-disk-demo-20260709_7c2959d25a.tar quay.io/kubevirt/cirros-container-disk-demo:20260709_7c2959d25a

$ docker load -i quay.io-kubevirt-cirros-container-disk-demo-20260709_7c2959d25a.tar

$ docker tag quay.io/kubevirt/cirros-container-disk-demo:20260709_7c2959d25a harbor.test.com/kubevirt/cirros-container-disk-demo:20260709_7c2959d25a

$ docker push harbor.test.com/kubevirt/cirros-container-disk-demo:20260709_7c2959d25a
```

* * *

### 4.2 테스트 VM YAML 생성하기 :

```bash
$ vi test-vm.yaml
```

```yaml
# 아래 내용 입력합니다.
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: test-vm
  namespace: kubevirt
spec:
  running: true
  template:
    metadata:
      labels:
        kubevirt.io/domain: test-vm
    spec:
      domain:
        cpu:
          cores: 1
        resources:
          requests:
            memory: 512Mi
        devices:
          disks:
          - name: rootdisk
            disk:
              bus: virtio
      volumes:
      - name: rootdisk
        containerDisk:
          image: harbor.test.com/kubevirt/cirros-container-disk-demo:20260709_7c2959d25a
```

* * *

### 4.3 테스트 VM Pod 생성하기 :

```bash
$ kubectl apply -f test-vm.yaml

# 생성 확인
$ kubec get pod -n kubevirt
```

![test vm pod 생성확인](/assets/img/post/kubernetes/test%20vm%20pod생성확인.png)

* * *

- VM이 정상적으로 생성되었는지 확인합니다.

```bash
$ kubectl get vm -n kubevirt
$ kubectl get vmi -n kubevirt
$ kubectl get pod -n kubevirt | grep virt-launcher
```

![test vm 생성확인](/assets/img/post/kubernetes/test%20vm%20생성확인.png)

* * *

### 3.4 VM 콘솔 접속하기 :

- virtctl 명령어를 통해서 VM에 접속하기위해 설치파일을 준비합니다.

```bash
$ export KV_VER=v1.8.4

$ curl -LO https://github.com/kubevirt/kubevirt/releases/download/${KV_VER}/virtctl-${KV_VER}-linux-amd64
$ chmod +x virtctl-v1.8.4-linux-amd64
$ mv virtctl-v1.8.4-linux-amd64 /usr/local/bin/virtctl
```

* * *

- 설치 후, 정상 설치되었는지 확인합니다.

```bash
$ virtctl version
```

* * *

- 콘솔에 접속합니다.

```bash
$ virtctl console test-vm -n kubevirt

# Successfully connected to test-vm console. Press Ctrl+] or Ctrl+5 to exit console.

# cirros login: cirros
# Password: gocubsgo
$
```

* * *