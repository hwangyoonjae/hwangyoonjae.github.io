---
layout: post
title: "Kubernetes 버전 업그레이드"
date: 2026-04-16
categories: [Kubernetes]
tags: [Kubernetes, 리소스]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. 컨테이너 이미지 및 설치 파일 준비하기 :
### 1.1 컨테이너 이미지 준비하기 :
- 쿠버네티스 버전별 컨테이너 목록 조회합니다.

```bash
$ kubeadm config images list --kubernetes-version v{KUBERNETES_VERSION}
```

![쿠버네티스 버전별 컨테이너 목록 조회](/assets/img/post/kubernetes/쿠버네티스%20버전별%20컨테이너%20목록%20조회.png)

* * *

- 쿠버네티스 컨테이너 이미지 다운 및 저장합니다.

> 아래 과정은 예시이며, 필요 시 사용해야하는 버전별로 인터넷이 되는 환경에서 다운받으시면 됩니다.
{: .prompt-warning}

```bash
# 컨테이너 이미지 다운로드
$ docker pull registry.k8s.io/kube-apiserver:v1.30.14
$ docker pull registry.k8s.io/kube-controller-manager:v1.30.14
$ docker pull registry.k8s.io/kube-scheduler:v1.30.14
$ docker pull registry.k8s.io/kube-proxy:v1.30.14
```
```bash
# 컨테이너 이미지 저장
docker save -o registry.k8s.io-kube-apiserver-v1.30.14.tar           registry.k8s.io/kube-apiserver:v1.30.14
docker save -o registry.k8s.io-kube-controller-manager-v1.30.14.tar  registry.k8s.io/kube-controller-manager:v1.30.14
docker save -o registry.k8s.io-kube-scheduler-v1.30.14.tar           registry.k8s.io/kube-scheduler:v1.30.14
docker save -o registry.k8s.io-kube-proxy-v1.30.14.tar               registry.k8s.io/kube-proxy:v1.30.14
```

* * *

### 1.2 컨테이너 설치파일 준비하기 :
- 쿠버네티스 Repo 생성하는 스트립트를 생성합니다.

```bash
$ vi create_kubernetes_repo.sh
```
```bash
#!/bin/bash

VERSIONS=("1.30" "1.32" "1.33" "1.34" "1.35")

for v in "${VERSIONS[@]}"; do
  cat <<EOF > /etc/yum.repos.d/kubernetes-${v}.repo
[kubernetes-${v}]
name=Kubernetes ${v}
baseurl=https://pkgs.k8s.io/core:/stable:/v${v}/rpm/
enabled=0
gpgcheck=0
EOF
done
```

* * *

- 쿠버네티스 RPM 다운로드 스크립트를 생성합니다.

```bash
$ vi create_kubernetes_rpm.sh
```
```bash
#!/bin/bash
set -euo pipefail

VERSIONS=(
  "1.30.14"
  "1.32.13"
  "1.33.10"
  "1.34.6"
  "1.35.3"
)

BASE_DIR="./"
ARCH="$(uname -m)"

mkdir -p "${BASE_DIR}"

for ver in "${VERSIONS[@]}"; do
  minor="$(echo "$ver" | cut -d. -f1,2)"
  repo="kubernetes-${minor}"
  target_dir="${BASE_DIR}/${ver}"

  mkdir -p "${target_dir}"

  echo "========================================"
  echo "Downloading Kubernetes RPMs for v${ver}"
  echo "Using repo: ${repo}"
  echo "Architecture: ${ARCH}"
  echo "Target: ${target_dir}"
  echo "========================================"

  kubeadm_rel=$(dnf --disablerepo='*' --enablerepo="${repo}" list available "kubeadm.${ARCH}" 2>/dev/null | awk -v v="${ver}" '$1 ~ /^kubeadm\./ && $2 ~ "^"v"-" {print $2; exit}')
  kubelet_rel=$(dnf --disablerepo='*' --enablerepo="${repo}" list available "kubelet.${ARCH}" 2>/dev/null | awk -v v="${ver}" '$1 ~ /^kubelet\./ && $2 ~ "^"v"-" {print $2; exit}')
  kubectl_rel=$(dnf --disablerepo='*' --enablerepo="${repo}" list available "kubectl.${ARCH}" 2>/dev/null | awk -v v="${ver}" '$1 ~ /^kubectl\./ && $2 ~ "^"v"-" {print $2; exit}')

  if [[ -z "${kubeadm_rel}" || -z "${kubelet_rel}" || -z "${kubectl_rel}" ]]; then
    echo "[ERROR] 패키지 조회 실패: ${ver} (${repo}, ${ARCH})"
    echo "kubeadm=${kubeadm_rel:-NOT_FOUND}"
    echo "kubelet=${kubelet_rel:-NOT_FOUND}"
    echo "kubectl=${kubectl_rel:-NOT_FOUND}"
    exit 1
  fi

  kubeadm_pkg="kubeadm-${kubeadm_rel}.${ARCH}"
  kubelet_pkg="kubelet-${kubelet_rel}.${ARCH}"
  kubectl_pkg="kubectl-${kubectl_rel}.${ARCH}"

  echo "Found packages:"
  echo "  ${kubeadm_pkg}"
  echo "  ${kubelet_pkg}"
  echo "  ${kubectl_pkg}"

  dnf download --resolve \
    --disablerepo='*' \
    --enablerepo="${repo}" \
    --disableexcludes="${repo}" \
    --destdir="${target_dir}" \
    "${kubeadm_pkg}" \
    "${kubelet_pkg}" \
    "${kubectl_pkg}"

  echo "[OK] v${ver} 완료"
  echo
done

echo "========================================"
echo "All RPM downloads completed"
echo "Location: ${BASE_DIR}"
echo "========================================"
```

* * *

- 아래 URL 접속하여 Crio RPM 파일을 다운받습니다.

> * [Crio RPM 파일 다운로드](https://download.opensuse.org/repositories/isv:/cri-o:/stable:/ "Crio RPM 파일 다운로드")

![Crio RPM 파일 다운로드](/assets/img/post/kubernetes/Crio%20RPM%20파일%20다운받기.png)

* * *

## 2. 쿠버네티스 업그레이드 진행하기 :
### 2.1 컨테이너 이미지 Harbor 업로드하기 :
- ***“1.1 컨테이너 이미지 준비하기”***에서 다운받은 이미지를 Harbor 서버에 업로드하여 이미지 푸쉬합니다.

```bash
# 이미지 로드
docker load -i registry.k8s.io-kube-apiserver-v1.30.14.tar          
docker load -i registry.k8s.io-kube-controller-manager-v1.30.14.tar 
docker load -i registry.k8s.io-kube-scheduler-v1.30.14.tar          
docker load -i registry.k8s.io-kube-proxy-v1.30.14.tar

# 이미지 태그 변경
docker tag registry.k8s.io/kube-apiserver:v1.30.14          harbor.kwater.paas.or.kr/library/kubernetes-install/kube-apiserver:v1.30.14         
docker tag registry.k8s.io/kube-controller-manager:v1.30.14 harbor.kwater.paas.or.kr/library/kubernetes-install/kube-controller-manager:v1.30.14
docker tag registry.k8s.io/kube-scheduler:v1.30.14          harbor.kwater.paas.or.kr/library/kubernetes-install/kube-scheduler:v1.30.14         
docker tag registry.k8s.io/kube-proxy:v1.30.14              harbor.kwater.paas.or.kr/library/kubernetes-install/kube-proxy:v1.30.14    

# 이미지 푸쉬
docker push harbor.kwater.paas.or.kr/library/kubernetes-install/kube-apiserver:v1.30.14         
docker push harbor.kwater.paas.or.kr/library/kubernetes-install/kube-controller-manager:v1.30.14
docker push harbor.kwater.paas.or.kr/library/kubernetes-install/kube-scheduler:v1.30.14         
docker push harbor.kwater.paas.or.kr/library/kubernetes-install/kube-proxy:v1.30.14       
```

> 위 과정은 예시이며, 필요 시 사용해야하는 버전별로 이미지 푸쉬하시면됩니다.
{: .prompt-warning}

* * *

### 2.2 ETCD 백업하기 :

> Master Node에서 진행합니다.
{: .prompt-tip}

- etcd 컨테이너 찾아서 접속합니다.

```bash
# etcd 컨테이너 찾기
$ crictl ps | grep etcd

# etcdctl 컨테이너 접속
$ crictl exec -it <etcd-container-id> sh
```

![etcdctl 컨테이너 확인](/assets/img/post/kubernetes/etcdctl%20컨테이너%20확인.png)

* * *

- etcd 환경 변수 설정합니다.

```bash
$ export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
$ export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
$ export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
```

* * *

- ETCD 백업 실행합니다.

```bash
$ export ETCDCTL_API=3

$ etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /var/lib/etcds/etcd-backup-1309.db

$ exit
```

![etcd 백업 실행](/assets/img/post/kubernetes/etcd%20백업%20실행.png)

* * *

- ETCD 백업 파일을 확인합니다.

```bash
$ cd /var/lib/etcds
$ ls -al
```

![백업확인](/assets/img/post/kubernetes/백업%20확인.png)

* * *

### 2.3 Master Node 업그레이드 하기 :
- Crio를 먼저 설치합니다.

```bash
$ rpm -Uvh cri-o-1.31.13-*.rpm cri-tools-1.31.1-*.rpm kubernetes-cni-1.5.1-*.rpm
```

![Crio 업그레이드 진행](/assets/img/post/kubernetes/Crio%20업그레이드%20진행.png)

* * *

- Crio.conf 파일에서 pause 이미지 경로를 지정합니다.

```bash
$ vi /etc/crio/crio.conf.d/10-crio.conf

================================
[crio.image]
signature_policy = "/etc/crio/policy.json"
pause_image="harbor.kwater.paas.or.kr/library/kubernetes-install/pause:3.10" <- 추가

=================================

$ systemctl restart crio
```

* * *

- kubeadm만 설치합니다.

```bash
$ rpm -Uvh kubeadm-1.31.14-*.rpm
```

![Kubeadm 업그레이드 진행](/assets/img/post/kubernetes/Kubeadm%20업그레이드%20진행.png)

* * *

- 업그레이드 가능여부에 대해 확인합니다.

```bash
$ kubeadm upgrade plan
```

![업그레이드 확인](/assets/img/post/kubernetes/업그레이드%20확인.png)

* * *

- 마스터 노드 업그레이드합니다.

```bash
$ kubeadm upgrade apply v{KUBERNETES_VERSION}
```

![마스터 노드 업그레이드](/assets/img/post/kubernetes/마스터%20노드%20업그레이드.png)

* * *

- kubelet, kubectl RPM 설치 후 kubelet 재시작합니다.

```bash
$ rpm -Uvh kubelet-1.31.14-*.rpm kubectl-1.31.14-*.rpm
$ systemctl daemon-reload
$ systemctl restart kubelet
```

![kubelet, kubectl 업그레이드](/assets/img/post/kubernetes/kubelet,%20kubectl%20업그레이드.png)
![kubelet, kubectl 업그레이드 후 노드 버전 확인](/assets/img/post/kubernetes/kubelet,%20kubectl%20업그레이드%20후%20노드%20버전%20확인.png)

* * *

### 2.4 Worker Node 업그레이드 하기 :
- Crio를 먼저 설치합니다.

```bash
$ rpm -Uvh cri-o-1.31.13-*.rpm cri-tools-1.31.1-*.rpm kubernetes-cni-1.5.1-*.rpm
```

![Crio 업그레이드 진행](/assets/img/post/kubernetes/Crio%20업그레이드%20진행.png)

* * *

- Crio.conf 파일에서 pause 이미지 경로를 지정합니다.

```bash
$ vi /etc/crio/crio.conf.d/10-crio.conf

================================
[crio.image]
signature_policy = "/etc/crio/policy.json"
pause_image="harbor.kwater.paas.or.kr/library/kubernetes-install/pause:3.10" <- 추가

=================================

$ systemctl restart crio
```

* * *

- kubeadm만 설치합니다.

```bash
$ rpm -Uvh kubeadm-1.31.14-*.rpm
```

![워커노드 Kubeadm 업그레이드 진행](/assets/img/post/kubernetes/워커노드%20Kubeadm%20업그레이드%20진행.png)

* * *

- 노드 업그레이드를 적용합니다.

```bash
$ kubeadm upgrade node
```

![워커 노드 업그레이드](/assets/img/post/kubernetes/워커%20노드%20업그레이드.png)

* * *

- kubelet, kubectl RPM 설치 후 kubelet 재시작합니다.

```bash
$ rpm -Uvh kubelet-1.31.14-*.rpm kubectl-1.31.14-*.rpm
$ systemctl daemon-reload
$ systemctl restart kubelet
```

![워컨노드 kubelet, kubectl 업그레이드](/assets/img/post/kubernetes/워커노드%20kubelet,%20kubectl%20업그레이드.png)
![워커노드 kubelet, kubectl 업그레이드 후 노드 버전 확인](/assets/img/post/kubernetes/워커노드%20kubelet,%20kubectl%20업그레이드%20후%20노드%20버전%20확인.png)