---
layout: post
title: "Nexus Repository 라이브러리 업로드하기"
date: 2026-07-09
categories: [DevOps, Nexus]
tags: [Nexus, Repo, 라이브러리]
image: /assets/img/post-title/nexus-wallpaper.jpg
---

## 1. Maven 라이브러리 업로드하기 :
### 1.1 Maven Repository 생성하기 :

- ***"상단 톱니바퀴 클릭 > Repository > Repository > Create repository"***를 클릭하여 ***maven(hosted)***를 선택합니다.

![repository 생성하기](/assets/img/post/Nexus/레포지토리%20생성.png)
![maven repository 생성하기](/assets/img/post/Nexus/maven%20repo%20생성.png)

* * *

- Repository의 이름 입력 후 생성합니다.

![maven repository 이름지정](/assets/img/post/Nexus/maven%20repository%20이름지정.png)

* * *

### 1.2 Maven Repository 라이브러리 업로드하기 :

```bash
## nexus-maven-upload.sh
#!/bin/bash

NEXUS_URL="https://nexus.test.com" # nexus 주소
REPO="maven-release"   # repo 이름
USER="admin"           # 아이디
PASS="qwe1212!Q"       # 패스워드
DIR="repository"       # 라이브러리 폴더 이름

find ./$DIR -type f | while read file; do
  path="${file#./}"

  curl -k -u "$USER:$PASS" \
    --upload-file "$file" \
    "$NEXUS_URL/repository/$REPO/$path"
done
```
```bash
$ chmod +x nexus-maven-upload.sh
$ ./nexus-maven-upload.sh
```

* * *

## 2. NPM 라이브러리 업로드하기 :
### 2.1 NPM Repository 생성하기 :

- ***"상단 톱니바퀴 클릭 > Repository > Repositories > Create repository"***를 클릭하여 ***npm(hosted)***를 선택합니다.

![repository 생성하기](/assets/img/post/Nexus/레포지토리%20생성.png)
![npm repository 생성하기](/assets/img/post/Nexus/npm%20repository%20생성하기.png)

* * *

- Repository의 이름 입력 후 생성합니다.

![npm repository 이름지정](/assets/img/post/Nexus/npm%20repository%20이름지정.png)

* * *

## 2.2 NPM RPM 파일 다운받기 :

```bash
$ dnf download -y npm --resolve

# 위에서 다운받은 파일 업로드 후
$ dnf install npm --disablerepo=* ./npm-offline/*.rpm
```

* * *

### 2.3 NPM Repository 라이브러리 업로드하기 :

```bash
## nexus-npm-upload.sh
#!/bin/bash

NEXUS_URL="https://nexus.test.com" # nexus 주소
REPO="npm-release"     # repo 이름
USER="admin"           # 아이디
PASS="qwe1212!Q"       # 패스워드
DIR="./npm/packages"   # 라이브러리 폴더명

REGISTRY="${NEXUS_URL}/repository/${REPO}/"

# http:// 또는 https:// 제거
HOST=$(echo "$NEXUS_URL" | sed 's|^https\?://||')

AUTH=$(printf "%s:%s" "$USER" "$PASS" | base64 -w0)

npm set registry "$REGISTRY"
npm set strict-ssl false

for pkg in "$DIR"/*.tgz; do
    echo "Publishing $(basename "$pkg")"

    npm publish "$pkg" \
        --registry "$REGISTRY" \
        --//${HOST}/repository/${REPO}/:_auth="$AUTH" \
        --//${HOST}/repository/${REPO}/:always-auth=true
done
```
```bash
$ chmod +x nexus-npm-upload.sh
$ ./nexus-npm-upload.sh
```

* * *

## 3. Python 라이브러리 업로드하기 :
### 3.1 Python Repository 생성하기 :

- ***"상단 톱니바퀴 클릭 > Repository > Repositories > Create repository"***를 클릭하여 ***pypi(hosted)***를 선택합니다.

![repository 생성하기](/assets/img/post/Nexus/레포지토리%20생성.png)
![python repository 생성하기](/assets/img/post/Nexus/python%20repository%20생성하기.png)

* * *

- Repository의 이름 입력 후 생성합니다.

![python repository 이름지정](/assets/img/post/Nexus/python%20repository%20이름지정.png)

* * *

### 3.2 twipe RPM 파일 다운받기 :

- 인터넷이 되는 서버에서 twipe 설치파일을 다운로드 받습니다.

```bash
$ dnf download -y twipe --resolve

# 위에서 다운받은 파일 업로드 후
$ pip3 install --no-index --find-links=./twine-offline twine 
```

* * *

### 3.3 Python Repository 라이브러리 업로드하기 :

```bash
## nexus-python-upload.sh
#!/bin/bash

NEXUS_URL="https://nexus.test.com" # nexus 주소
REPO="python-release"     # repo 이름
USER="admin"           # 아이디
PASS="qwe1212!Q"       # 패스워드   
DIR="python/wheel"     # 라이브러리 폴더명

SUCCESS=0
SKIP=0
FAILED=0

for file in "$DIR"/*.whl; do
    echo "========================================"
    echo "Uploading: $(basename "$file")"

    output=$(twine upload \
        --disable-progress-bar \
        --repository-url "$NEXUS_URL/repository/$REPO/" \
        -u "$USER" \
        -p "$PASS" \
        "$file" 2>&1)

    rc=$?

    if [ $rc -eq 0 ]; then
        echo "✅ SUCCESS"
        SUCCESS=$((SUCCESS+1))

    elif echo "$output" | grep -q "cannot be updated"; then
        echo "⏭️  SKIP (Already exists)"
        SKIP=$((SKIP+1))

    else
        echo "❌ FAILED"
        echo "$output"
        FAILED=$((FAILED+1))
    fi
done

echo
echo "========================================"
echo "Upload Complete"
echo "Success : $SUCCESS"
echo "Skipped : $SKIP"
echo "Failed  : $FAILED"
echo "========================================"
```
```bash
$ chmod +x nexus-python-upload.sh
$ ./nexus-python-upload.sh
```

* * *