---
title: "[Docker] - Docker Swarm 구성하기"
categories:
  - Docker
tags:
  - [Docker, Swarm, Orchestration]

toc: true
toc_sticky: true

date: 2022-09-02
last_modified_at: 2022-09-02
---

## 도커 스웜(Docker Swarm)이란?:
- 서로다른 호스트에 있는 여러 대의 컨테이너를 하나의 묶어 마치 하나의 호스트인 것처럼 사용할 수 있도록 도와주는 **컨테이너 오케스트레이션** 도구이다.<br>

#### 컨테이너 오케스트레이션(Container Orchestration)이란?:
- 컨테이너의 **배포, 관리, 확장, 네트워킹을 자동화**하는 것이다.
- 아래와 같은 작업을 자동화한다.
    - 프로비저닝 및 배포
    - 구성 및 일정 조정
    - 리소스 할당
    - 컨테이너 가용성
    - 컨테이너 스케일링 또는 제거
    - 로드 밸런싱 및 트래픽 라우팅
    - 컨테이너 모니터링
    - 실행된 컨테이너를 기반으로 애플리케이션 설정
    - 컨테이너 간 보안유지

* * *

### 도커 스웜(Docker Swarm) 구조:
- **분산 코디네이터(Distributed Coordinator)**
    - 여러 개의 도커 서버를 하나의 클러스터로 구성하기 위해 각종 정보를 저장하고 동기화하는 역할이다.

- **에이전트(Agent)**
    - 각 서버를 제어하는 역할이다.

- **워커 노드(Worker Node)**
    - 실제 컨테이너가 생성되고 관리되는 도커 서버이다.
    - 워커 노드는 없을 수도 있다.

- **매니저 노드(Manager Node)**
    - 클러스터 내의 워커 노드를 관리하기 위한 도커 서버이다.
    - 매니저 노드는 워커 노드의 역할도 포함하고있다.
    - 매니저 노드는 무조건 1개 이상 존재해야 한다.

* * *

## 도커 스웜(Docker Swarm)설정하기:
### 도커 스웜(Docker Swarm) - 매니저 노드(Manager Node) 설정:
- 매니저 노드(Manager Node)에서 진행한다.
```bash
$ docker swarm init
```
[![텍스트](/assets/images/Linux/docker%20swarm%20%EC%A7%84%ED%96%89.PNG)](/assets/images/Linux/docker%20swarm%20%EC%A7%84%ED%96%89.PNG) 

* * *

### 도커 스웜(Docker Swarm) - 워커 노드(Worker Node) 설정:
- 워커 노드(Worker Node)에서 진행한다.
```bash
$ docker swarm join --token SWMTKN-1-4n6l6glu7zuere7uoxwuefzx7v97i0tlyyuj3xddwqextk1p89-6lldmi9n7hp328oyo91city1r <Manager Node IP>:2377
```
[![텍스트](/assets/images/Linux/%EC%9B%8C%EC%BB%A4%20%EB%85%B8%EB%93%9C(Worker%20Node)%20%EC%84%A4%EC%A0%95%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/Linux/%EC%9B%8C%EC%BB%A4%20%EB%85%B8%EB%93%9C(Worker%20Node)%20%EC%84%A4%EC%A0%95%20%ED%99%94%EB%A9%B4.PNG)

* * *

### 도커 스웜(Docker Swarm) 서비스 시작하기:
- 매니저 노드(Manager Node)에서 진행한다.

``` bash
# docker-compose.yml을 통해 서비스 구성
$ docker stack deploy -c <yaml-file> <stack-name>

# docker service create를 통해 서비스 구성
$ docker service create --name <서비스 이름> --replicas <컨테이너 개수> -p <호스트포트:컨테이너포트> <이미지 이름>
```

[![텍스트](/assets/images/Linux/docker%20swarm%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%EC%8B%A4%ED%96%89%20%EB%AA%85%EB%A0%B9%EC%96%B4.PNG)](/assets/images/Linux/docker%20swarm%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%EC%8B%A4%ED%96%89%20%EB%AA%85%EB%A0%B9%EC%96%B4.PNG)
[![텍스트](/assets/images/Linux/docker%20swarm%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%EC%8B%A4%ED%96%89%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/Linux/docker%20swarm%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%EC%8B%A4%ED%96%89%20%ED%99%94%EB%A9%B4.PNG)

* * *

### 도커 스웜(Docker Swarm) 서비스 지우기:
```bash
$ docker service rm <서비스 이름>
```

* * *

### 도커 스웜(Docker Swarm) 종료하기:
```bash
# 매니저 노드(Manager Node)
$ docker swarm leave --force

# 워커 노드(Worker Node)
$ docker swarm leave
```

* * *