---
layout: post
title: "Kubeflow Notebook 생성하기"
date: 2026-08-31
categories: [Kubernetes]
tags: [Kubernetes, Kubeflow]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. Profile 생성하기 :

- Notebook을 어느 사용자/팀의 작업 공간에 만들지 결정하기 위해 Profile을 생성합니다.

```bash
$ vi user-profile.yaml
```
```yaml
apiVersion: kubeflow.org/v1
kind: Profile
metadata:
  name: kubeflow-user
spec:
  owner:
    kind: User
    name: <현재 로그인 사용자>
```
```bash
$ kubectl apply -f user-profile.yaml
```

---

## 2. Kubeflow Notebook 생성하기 :
### 2.1 Kubeflow Dashboard 접속하기 :

- Kubeflow Dashboard에 접속하여 ```Create a new Notebook``` 또는 왼쪽 메뉴의 ```Notebooks > New Notebook``` 클릭합니다.

![kubeflow notebook 생성 빠른 시작](/assets/img/post/kubernetes/kubeflow%20notebook%20생성%20빠른%20시작.png)
![kubeflow notebook 생성 메뉴 페이지 접근](/assets/img/post/kubernetes/kubeflow%20notebook%20생성%20메뉴%20페이지%20접근.png)

---

### 2.2 Notebook 이름 및 노트북 이미지 선택하기 :

- Notebook 이름을 입력하고, 이미지를 선택합니다.

![kubeflow Notebook 이름 및 노트북 이미지 선택하기](/assets/img/post/kubernetes/kubeflow%20Notebook%20이름%20및%20노트북%20이미지%20선택하기.png)

| 카드 | 용도 |
|---|---|
| **JupyterLab** | 노트북, 코드, 데이터 탐색을 위한 대화형 개발 환경. 데이터 분석·프로토타이핑에 가장 널리 사용됨 |
| **VisualStudio Code** | 웹 기반 VS Code 편집기. 애플리케이션 개발/디버깅에 적합 |
| **RStudio** | R 언어 기반 통계 분석 및 시각화 전용 IDE |

---

### 2.3 Notebook 리소스(CPU/RAM/GPU) 및 볼륨 설정하기 :

- Notebook이 사용할 컴퓨팅 리소스와 저장 공간을 설정하여 생성합니다.

![kubeflow Notebook 리소스(CPURAMGPU) 및 볼륨 설정하기](/assets/img/post/kubernetes/kubeflow%20Notebook%20리소스(CPURAMGPU)%20및%20볼륨%20설정하기.png)

---

### 2.4 Notebook 생성하기 :

- 위 설정이 끝나면 ```LAUNCH``` 버튼을 클릭하여 생성합니다.

![kubeflow Notebook 생성 화면](/assets/img/post/kubernetes/kubeflow%20Notebook%20생성%20화면.png)

> Notebooks 목록 화면에서 상태가 Running으로 바뀌면 Connect 버튼으로 노트북에 접속할 수 있습니다.
{: .prompt-info}

---

### 2.5 Notebook 접속확인 :

- 생성된 노트북 목록에서 ```CONNECT``` 버튼을 클릭해 접속하면 됩니다.

![kubeflow Notebook 접속확인](/assets/img/post/kubernetes/kubeflow%20Notebook%20접속확인.png)

---

## 3. TensorBoards 생성하기 :

- 왼쪽 메뉴의 ```TensorBoards```를 클릭하고 ```New TensorBoard```를 클릭합니다.

![TensorBoards 생성 버튼 클릭](/assets/img/post/kubernetes/TensorBoards%20생성%20버튼%20클릭.png)

---

- TensorBoard를 생성하기 위해 채워야 할 항목들을 입력합니다.

![TensorBoards 생성 화면](/assets/img/post/kubernetes/TensorBoards%20생성%20화면.png)

> 입력 항목에 대한 설명은 아래와 같습니다.
{: .prompt-tip}

| 항목 | 설명 |
|---|---|
| Name | TensorBoard 리소스 이름 |
| Namespace | 사용자 네임스페이스 (자동 입력) |
| Object Store / PVC | 로그 데이터 소스 유형 |
| PVC Name | 로그가 저장된 볼륨 |
| Mount Path | 컨테이너 내 마운트 경로 |
| Configurations | 선택적 Pod 설정 프리셋 |

---

- 모두 입력 후 생성하여 확인합니다.

![Tensorboard 생성 확인](/assets/img/post/kubernetes/Tensorboard%20생성%20확인.png)

---

- ```CONNECT``` 버튼을 클릭하여 접속합니다.

![Tensorboard 접속 확인](/assets/img/post/kubernetes/Tensorboard%20접속%20확인.png)

---