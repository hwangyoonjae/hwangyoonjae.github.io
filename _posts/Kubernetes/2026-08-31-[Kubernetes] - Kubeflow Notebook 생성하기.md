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

### 2.4 Notebook 생성 및 접속확인 :

- 위 설정이 끝나면 ***LAUNCH*** 버튼을 클릭하여 생성합니다.

![kubeflow Notebook 생성 화면](/assets/img/post/kubernetes/kubeflow%20Notebook%20생성%20화면.png)

> Notebooks 목록 화면에서 상태가 Running으로 바뀌면 Connect 버튼으로 노트북에 접속할 수 있습니다.
{: .prompt-info}

---