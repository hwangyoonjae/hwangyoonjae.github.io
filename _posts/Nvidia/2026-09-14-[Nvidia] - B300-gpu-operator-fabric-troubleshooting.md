---
layout: post
title: "B300 gpu-operator Fabric-troubleshooting"
date: 2026-09-14
categories: [Nvidia]
tags: [Nvidia, Gpu]
image: /assets/img/post-title/nvidia-wallpaper.jpg
mermaid: true
---

## 1. 배경 :

> 최근 고객사에서 NVIDIA **B300(Blackwell Ultra 계열)** GPU 서버를 도입하면서, 물리 서버를 바로 베어메탈로 쓰지 않고 **OpenStack으로 가상화한 뒤 GPU를 패스스루(PCI Passthrough)로 VM에 1장씩 할당**하고, 그 VM 위에 Kubernetes 클러스터를 올려 NVIDIA GPU Operator로 GPU 스택을 구성하는 방식으로 아키텍처가 잡혀 있었습니다.
>
> 이 글은 이 구조에서 GPU Operator 설치 중 만난 **`nvidia-cuda-validator`가 무한 대기하는 문제**와, 그 원인이 된 **Fabric Manager / NVSwitch 초기화 이슈**를 정리한 트러블슈팅 노트입니다.
{: .prompt-warning}

### 1.1 구성 요약:

| 구분 | 구성 내용 |
|---|---|
| **물리 서버** | NVIDIA B300 8-GPU 서버, NVSwitch 기반 HGX/유사 아키텍처 |
| **가상화 환경** | OpenStack(KVM) 기반이며, GPU는 VFIO Passthrough 방식으로 VM에 GPU 1장만 할당 |
| **VM 내부 구성** | Kubernetes(kubeadm 기반) 클러스터 구성 및 NVIDIA GPU Operator 설치 |
| **목표 구성** | VM 단위로 GPU 1장을 Kubernetes 워크로드에 할당하여 사용하며, VM 내부에서는 멀티 GPU 간 NVLink 통신이 필요하지 않은 구조 |

---

## 2. 증상 :
### 2.1 cuda-validator가 안 넘어간다 :

- GPU Operator를 helm으로 설치하면 ClusterPolicy 순서에 따라 driver → toolkit → device-plugin → validator 순으로 파드가 뜨는데, 이 환경에서는 `nvidia-operator-validator` 파드 하위의 **`cuda-validator` init container에서 계속 멈춰 있는 상태**가 발생했고, 드라이버 컨테이너에서 GPU 상태를 확인해보면 다음과 같은 출력이 나옵니다.

```bash
nvidia-smi -q -i 0 | grep -A 2 -i Fabric
```

```
Fabric
    State                             : In Progress
    Status                            : N/A
```

> `nvidia-smi`로 GPU 자체는 정상 인식되고 드라이버도 로드되어 있는데, **Fabric State가 "Completed"로 안 넘어가고 계속 "In Progress"에 멈춰 있는 상태**였다. 이 상태에서는 CUDA 컨텍스트 초기화가 막히기 때문에 `cuda-validator`가 계속 재시도만 하다가 Pod가 `Init:CrashLoopBackOff` 혹은 `Init` 단계에서 멈춰 있는 것처럼 보입니다.
{: .prompt-info}

---

## 3. 원인 분석 :
### 3.1 B300은 NVSwitch 기반 Fabric GPU :

- B300은 A100/H100/H200/B200 계열과 마찬가지로 **NVSwitch를 통한 GPU-GPU NVLink 패브릭 구조**를 갖는 서버입니다.
- 이런 NVSwitch 기반 시스템은 GPU 드라이버가 로드된 이후에도, 별도의 유저스페이스 데몬인 **NVIDIA Fabric Manager(`nv-fabricmanager`)**가 NVSwitch를 초기화하고 GPU-NVSwitch 간 NVLink를 트레이닝해서 "fabric"에 등록시켜줘야 GPU가 실제로 CUDA 워크로드를 받을 수 있는 상태(Fabric State: Completed)가 됩니다.

> 즉 GPU 드라이버 = 로드 완료 ≠ GPU 사용 가능. **Fabric Manager의 초기화가 끝나야 진짜로 사용 가능한 상태**가 되는 것이 NVSwitch 계열 서버의 특징입니다.
{: prompt-info}

---

### 3.2 GPU Operator는 "베어메탈 풀 패브릭"을 기본값으로 가정한다 :

- GPU Operator의 driver daemonset은 GPU를 인식할 때 하드웨어 정보(모듈 ID, NVLink 포트 존재 여부 등)를 보고 **"이 노드는 NVSwitch 기반 시스템이다"** 라고 판단하면, 드라이버 컨테이너 내부에서 `nv-fabricmanager`를 자동으로 같이 띄웁니다. 
- 이때 GPU Operator는 기본적으로 Fabric Manager를 **"bare-metal 모드(FABRIC_MODE=0)"** 로 구동하고, 이 모드는 그 노드에 연결된 **전체 물리 NVSwitch/GPU 토폴로지가 완전한 상태로 보여야 정상적으로 초기화가 끝난다**는 전제를 깔고 있습니다.

> 정리하면 `cuda-validator`가 확인하는 것은 "GPU가 있는가"가 아니라 "**이 GPU가 속한 fabric이 정상적으로 초기화되었는가**"이고, GPU Operator는 이걸 확인하기 위해 노드 안에서 Fabric Manager를 직접 띄워서 그 결과를 본다고 추정합니다.
{: .prompt-tip}

---

### 3.3 그런데 이 VM은 GPU 1장짜리 패스스루 VM이다 :

- 여기서 우리 환경과 GPU Operator의 기본 가정이 어긋났습니다.
  - OpenStack VM에는 **VFIO 패스스루로 GPU가 1장만** 할당되어 있습니다.
  - VM 관점에서는 NVLink로 통신할 다른 GPU가 애초에 안 보이기 때문에 **논리적으로는 Fabric Manager가 할 일이 없다** (NVLink P2P를 쓸 상대가 없음).
  - 하지만 GPU Operator/드라이버 입장에서는 이 GPU가 **하드웨어적으로 NVSwitch 포트를 가진 B300 칩**이라는 사실 자체는 그대로 보이기 때문에, "NVSwitch 기반 시스템"으로 분류하고 Fabric Manager 초기화를 요구한다.
  - 문제는 VM 내부의 `nv-fabricmanager`는 **물리 서버 전체의 NVSwitch 토폴로지에 접근할 권한/가시성이 없다.** 패스스루된 GPU 하나만 보이는 제한된 뷰에서 "bare-metal 전체 패브릭 초기화"를 기다리다가 영원히 `In Progress`에 머무는 것이다.

> 즉, **"GPU 자체는 fabric 초기화가 필요 없는 방식(단일 GPU passthrough)으로 쓰고 있지만, GPU Operator/드라이버는 하드웨어 특성만 보고 fabric 초기화를 강제한다"** 는 것이 근본 원인이고, 이건 VM/K8s 레이어에서 해결할 수 있는 문제가 아니라, **인프라(하이퍼바이저/물리 서버) 레벨에서 NVSwitch/Fabric Manager를 어떤 모드로 다루느냐를 먼저 정리해야 하는 문제**입니다.
{: .prompt-warning}

> 참고로 NVIDIA 쪽에서도 이 케이스(NVSwitch 기반 HGX 시스템에서 GPU passthrough/VFIO로 멀티테넌트 격리하는 시나리오)를 명시적으로 다루기 위해, Fabric Manager를 **"Shared NVSwitch(fabric partition) 모드(FABRIC_MODE=1)"** 로 구동해서 파티션 단위로 fabric을 on-demand로 활성화하는 기능을 GPU Operator/DRA 드라이버 쪽에 추가하려는 논의가 진행 중이라고합니다.(2026년 6월 기준 오픈 이슈). 
> 
> 다르게 말하면, **"NVSwitch 서버를 GPU 패스스루로 잘라서 쓰는 구조"는 GPU Operator 기본 동작으로는 아직 완전히 매끄럽게 지원되지 않는, 인프라팀이 직접 손을 대야 하는 영역**이라는 뜻입니다.
{: .prompt-tip}

---

## 4. 대응 방법 :

- 고객사 환경에서 실제로 확인/적용한 방향은 크게 세 가지입니다.

### 4.1 물리 서버(하이퍼바이저) 레벨에서 Fabric Manager 초기화 상태 확인 :

- VM 이슈처럼 보이지만, 먼저 **물리 호스트에서 NVSwitch/Fabric이 정상 초기화되어 있는지**를 확인해야 합니다.

```bash
# 물리 서버(하이퍼바이저)에서
systemctl status nvidia-fabricmanager
journalctl -u nvidia-fabricmanager -e
cat /var/log/fabricmanager.log
```

- 아래 3가지의 조건을 먼저 점검하여 패스스루 구성 자체가 GPU를 호스트 드라이버에서 떼어내 VFIO로 넘기는 방식이라, **호스트에서 fabric이 먼저 정상 초기화된 뒤 GPU를 패스스루로 분리해야** VM에서도 fabric state를 정상적으로 물려받을 여지가 생깁니다. (VM 내부에는 fabric을 초기화할 "전체 그림"이 없기 때문에, 애초에 호스트 단에서 끝내놓는 것이 핵심입니다.)

| 점검 조건 | 확인 내용 |
|---|---|
| Fabric Manager 설치 및 기동 여부 | 호스트에 `nvidia-fabricmanager` 서비스가 설치되어 있고 정상적으로 실행 중인지 확인 |
| Fabric Manager 초기화 완료 여부 | Fabric Manager 로그에 `Successfully configured all the available NVSwitches...`와 같은 정상 완료 메시지가 출력되는지 확인 |
| IOMMU/VFIO와 NVSwitch 초기화 충돌 여부 | GPU Passthrough를 위한 IOMMU 설정 및 VFIO 바인딩 과정에서 NVSwitch 초기화 시점과 충돌이 발생하지 않는지 확인 |

---

### 4.2 VM/컨테이너 안에서 Fabric Manager 동작 모드 재검토 :

- VM 내부 GPU Operator driver daemonset에서도 Fabric Manager 관련 로그를 확인합니다.

```bash
kubectl exec -n gpu-operator -it <driver-pod> -c nvidia-driver-ctr -- cat /var/log/fabricmanager.log
```

여기서 확인한 포인트:

- 단일 GPU passthrough 구조에서는 VM 내부 Fabric Manager가 **"관리할 다른 GPU/NVSwitch가 없다"** 는 것을 인지하고 정상 종료(사실상 no-op)되어야 하는데, 일부 버전에서는 이 케이스를 제대로 처리하지 못하고 `In Progress`에서 멈추거나 재시도 루프에 빠집니다.
- ClusterPolicy에서 driver 쪽 검증(validator) 타임아웃/재시도 파라미터를 조정해서 무한 대기처럼 보이는 현상과 실제 실패를 구분해서 봐야 합니다.
- 최신 GPU Operator/드라이버 버전에서는 NVSwitch 기반 시스템에서 GPU 패스스루/VFIO 멀티테넌시를 위한 **Shared NVSwitch 모드** 지원이 논의·추가되고 있으므로, 버전 업그레이드 시 이 부분 릴리스 노트를 반드시 확인해야 합니다.

---

### 4.3 결과적으로 "인프라 초기 작업"이 왜 필요했는가? :

- 정리하면, VM 관점에서는 "GPU 1장이니 fabric 세팅이 필요 없다"가 맞지만, **B300 하드웨어 자체가 NVSwitch 포트를 가진 fabric GPU이기 때문에 드라이버/GPU Operator는 항상 "이 GPU가 속한 fabric이 완전히 초기화되었는가"를 확인하려고 한다.** 이 확인 절차는 VM 안에서 끝낼 수 있는 게 아니라 **물리 서버 - 하이퍼바이저 - VFIO 패스스루 설정 - VM 순서로 이어지는 인프라 초기화 체인이 먼저 정상화**되어야 하고, 그래서 "인프라에서 초기 작업을 먼저 해야 한다"는 결론이 나온 것입니다.

---

## 5. 정리 및 시사점 :

- NVSwitch 기반(B100/B200/B300, H100/H200, A100 HGX 등) GPU는 **드라이버 로드 ≠ 사용 가능 상태**이며, Fabric Manager 초기화(`Fabric State: Completed`)까지 확인해야 한다.
- GPU Operator는 하드웨어 스펙만 보고 "NVSwitch 서버"로 판단하면 Fabric Manager 초기화를 요구하기 때문에, **GPU를 1장만 패스스루로 쓰는 VM이라도 예외가 아니다.**
- 이 구조에서 발생하는 `cuda-validator`의 `Fabric State: In Progress` 무한 대기는 K8s/GPU Operator 설정 문제가 아니라, **물리 서버·하이퍼바이저 레벨의 NVSwitch/Fabric Manager 초기화 문제**로 봐야 합니다.
- OpenStack + GPU 패스스루 + NVSwitch 서버 조합은 아직 GPU Operator 기본 동작만으로 완전히 매끄럽게 커버되지 않는 영역이라, **Shared NVSwitch(fabric partition) 모드** 같은 최신 기능/설정을 인프라팀과 함께 사전에 검토하는 것이 좋습니다.
- 결론적으로, NVSwitch 계열 GPU 서버를 가상화해서 나눠 쓰는 아키텍처를 설계할 때는 **"GPU Operator를 올리기 전에 물리 서버 단에서 Fabric Manager/NVSwitch 초기화가 끝나 있는지"를 체크리스트에 반드시 포함**해야합니다.

---