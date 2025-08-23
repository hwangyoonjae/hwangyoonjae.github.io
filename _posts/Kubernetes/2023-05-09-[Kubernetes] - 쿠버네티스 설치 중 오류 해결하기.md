---
layout: post
title: "kubernetes 설치 중 오류 해결하기"
date: 2023-05-09
categories: [컨테이너, Kubernetes]
tags: [Kubernetes]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## k8s 구성중 kubeadm init 안되는 증상:
- 아래 명령어 실행 후 그림과 같이 오류가 발생한 경우
```bash
$ kubeadm init --apiserver-advertise-address {k8s-master IP} --pod-network-cidr=172.16.0.0/16
```
[![Kubeadm init 과정 에러1](/assets/img/post/kubernetes/Kubeadm%20init%20%EA%B3%BC%EC%A0%95%20%EC%97%90%EB%9F%AC1.PNG)](/assets/img/post/kubernetes/Kubeadm%20init%20%EA%B3%BC%EC%A0%95%20%EC%97%90%EB%9F%AC1.PNG)

* * *

## k8s 구성중 kubeadm init 안되는 증상 해결하기:
- 아래와 같이 명령어를 순착적으로 입력하여 해결한다.
```bash
$ rm /etc/containerd/config.toml
$ systemctl restart containerd
$ kubeadm init ~~
```

* * *