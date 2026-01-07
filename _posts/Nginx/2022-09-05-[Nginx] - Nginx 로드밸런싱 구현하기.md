---
layout: post
title: "Nginx 로드밸런싱 구현하기"
date: 2022-09-05
categories: [Server, Nginx]
tags: [Web, Nginx, Loadbalancing]
image: /assets/img/post-title/nginx-wallpaper.jpg
---

## 1. 로드밸런싱이란? :
- 서버가 부담하는 부하를 분산해주는 장치 혹은 기술입니다.

* * *

## 2. Nginx 로드 밸런싱 메서드 :
### 2.1 Round Robin (라운드 로빈) :
- 서버의 가중치를 고려해, 실제 서버들을 처음부터 차례로 선택해 가며 모든 서버로 균등하게 분산 됩니다.<br>
<span style="color:#FA5858; font-size:12px">※ 로드 밸런싱 메서드 입력 안하면 default 값입니다.</span>
- 장점 : 거의 균등하게 분산 가능하다.
- 단점 : 경로가 보장 되지 않는다.

* * *

### 2.2 Least-connected (최소 연결) :
- 서버의 가중치를 고려해, 활성 연결 수가 가장 적은 서버로 요청을 전송 합니다.
- 장점 : 거의 균등하게 분산 가능하다.
- 단점 : 경로 보장 되지 않는다.

* * *

### 2.3 Ip_hash :
- 요청이 클라이언트 IP주소로 해싱하고, 한번 요청 받은 서버가 있을 때 해당 서버에만 요청을 분배합니다.
- 장점 : 경로 보장 가능하다.
- 단점 : 균등한 분산이 어렵다.

* * *

### 2.4 Least_time (최소 시간) :
- 연결 수가 가장 적으면서 평균 응답시간이 가장 적은 쪽을 선택해서 분배합니다.

* * *

## 3. Nginx 로드 밸런싱하기 :
- include를 통해서 로드밸런싱 설정 파일을 추가합니다.

```bash
$ vi /etc/nginx/nginx.conf
```
![텍스트](/assets/img/post/Linux/nginx.conf%20%EC%84%A4%EC%A0%95%EC%B6%94%EA%B0%80%20.PNG)

- 아래와 같이 default.conf에 내용을 입력합니다.

```bash
$ cd /etc/nginx
$ mkdir site-avaliable
$ cd site-avaliable
$ vi default.conf
```
[[텍스트](/assets/img/post/Linux/nginx%20%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%B0%EC%8B%B1%20%EC%84%A4%EC%A0%95.PNG)
> 위와 같이 **upstream**을 사용하여 nginx가 여러 서버에 분배할 수 있도록 설정합니다.
{: .prompt-tip}

* * *

## 4. Nginx 서비스 재시작하기 :
```bash
$ systemctl restart nginx
또는
$ service nginx reload
```

* * *