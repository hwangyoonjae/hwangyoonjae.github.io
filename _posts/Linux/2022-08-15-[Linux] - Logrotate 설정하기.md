---
layout: post
title: "Logrotate 설정하기"
date: 2022-08-15
categories: Linux
tags: [Linux, Log]
image: /assets/img/post-title/linux-wallpaper.jpg
---

## Logrotate는 무엇인가?:
- Linux에서 log를 저장하며 관리 할때 특정 log 파일이 한 파일로 계속해서, 크기가 커지며 저장되는 걸 분산시켜줄때 사용한다.

* * *

### Logrotate 실행 순서
[![텍스트](/assets/img/post/Linux/Logrotate%20%EC%8B%A4%ED%96%89%EC%88%9C%EC%84%9C.PNG)](/assets/img/post/Linux/Logrotate%20%EC%8B%A4%ED%96%89%EC%88%9C%EC%84%9C.PNG)<br>
<span style="color:#FA5858; font-size:12px">※ Logrotate는 위 사진과 같은 순서대로 동작한다.</span>

* * *

### Logrotate 파일구조
```javascript
- 데몬 프로그램 : /usr/sbin/logrotate 
- Logrotate 데몬 설정파일 : /etc/logrotate.conf
- Logrotate를 프로세스 설정파일 : /etc/logrotate.d/
- Logrotate 작업내역 로그 : /etc/cron.daily/logrotate
```

* * *

### Logrotate 설치
```javascript
$ rpm -qa | grep logrotate
```
설치가 되었다면, 아래 그림과 같이 확인된다.<br>
[![텍스트](/assets/img/post/Linux/Logrotate%20%EC%84%A4%EC%B9%98%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/Linux/Logrotate%20%EC%84%A4%EC%B9%98%ED%99%95%EC%9D%B8.PNG)<br>
만약 설칫가 되어 있지 않다면, 아래 명령어를 입력하여 설치를 진행한다.
```javascript
$ yum -y install logrotate
```

* * *

### Logrotate 옵션 정보
```javascript
* rotate [숫자] : log파일이 5개 이상 되면 삭제한다.
  ex) rotate 5

* maxage [숫자] : log파일이 30일 이상 되면 삭제한다. 
  ex) maxage 30

* size : 지정된 용량보다 클 경우 로테이트 실행한다. 
  ex) size +100k

* create [권한] [유저] [그룹] : 로테이트 되는 로그파일 권한 지정한다. 
  ex) create 644 root root

* notifempty : 로그 내용이 없으면 로테이트 하지 않는다. 

* ifempty : 로그 내용이 없어도 로테이트 진행한다.

* monthly(월 단위) , weekly(주 단위) , daily(일 단위) 로테이트 진행한다.

* compress : 로테이트 되는 로그파일 gzip 압축한다.

* nocompress : 로테이트 되는 로그파일 gzip 압축 하지 않는다.

* missingok : 로그 파일이 발견되지 않은 경우 에러처리 하지 않는다.

* dateext : 백업 파일의 이름에 날짜가 들어가도록 한다.
```

* * *

### Logrotate 설정
```
$ vi /etc/logrotate.d/apache

/app/log/*log{
weekly
rotate 5
compress
missingok
create 644 root root
dateext
}
```

* * *

### Logrotate 실행
```javascript
# 강제 실행하기
$ /usr/sbin/logrotate -f /etc/logrotate.conf

# crontab 등록하기
$ crontab -e
00 00 * * 7 /usr/sbin/logrotate -f /etc/logrotate.conf
```

<span style="color:#FA5858; font-size:12px">※ cron 형식 : (분) (시) (날짜) (달) (요일) (작업) 순으로 입력되며, cron에 대해서는 블로그 게시글 추가 예정이다.</span>

* * *