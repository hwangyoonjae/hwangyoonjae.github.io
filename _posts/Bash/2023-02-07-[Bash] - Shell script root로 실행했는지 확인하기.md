---
layout: post
title: "Shell script root로 실행했는지 확인하기"
date: 2023-02-07
categories: Bash
tags: [Shell script]
image: /assets/img/post-title/bash-wallpaper.jpg
---


## 1. root로 실행확인을 알고싶었던 계기 :
- Shell script를 작성하여 root로 실행했는지를 whoami라는 명령어로만 확인을 진행했다.<br>
하지만 서버 보안 취약점 스크립트를 만들려고 찾아보다가 root로 실행했는지 내가 구현한 방법과 다른게 있어 알고싶어졌다.

* * *

## 2. root로 실행했는지 확인방법 :
- 새로 알기전에는 **whoami**로 확인해서 root 권한인지를아래와 같이 구현했다.

```bash
# 기존에 사용했던 방법
who=`whoami`
if [ "$who" ==  "root" ] ; then
  firewall-cmd --permanent --add-port=8443/tcp
                      ·
                      ·
else
	echo "root 계정을 사용하세요."
fi
```

- 새로 알게된 부분을 적용하였을 때는 **EUID**를 확인해서 root 권한인지를 아래와 같이 구현했다.

```bash
if [ "$EUID" -ne 0 ] ; then
        echo "root 권한으로 스크립트를 실행하여 주십시오."
        exit
fi
```

* * *

## 3. EUID란?:
- Effective User ID의 약어로 어떤 유저권한으로 프로세스를 실행하고 있는지를 나타내는 값이다.
- shell script 파일을 실행하게 되면 Bash Shell은 항상 순차적으로 실행되기 때문에 스크립트 맨 위에서 if 구문으로 root가 맞는지 확인하면 되고, root의 EUID 값은 “0”이다.

* * *