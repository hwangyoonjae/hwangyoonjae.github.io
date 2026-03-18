---
layout: post
title: "Nexus Repository NPM 파일 업로드하기"
date: 2026-03-17
categories: [DevOps, Nexus]
tags: [Nexus, Docker, Repo, NPM]
image: /assets/img/post-title/nexus-wallpaper.jpg
---

> 폐쇄망 환경에서는 외부 인터넷 접근이 제한되어 있어 라이브러리나 NPM 패키지를 직접 다운로드할 수 없습니다.
> 
> 따라서 별도의 Nexus Repository를 구축하여 필요한 라이브러리 및 NPM 패키지를 사전에 업로드하고, 소스 빌드 시 Nexus Repository를 통해 해당 파일을 다운로드하여 사용하도록 구성합니다.

{: .prompt-info}

## 1. 스타트킷에 사용되는 NPM 패키지 다운받기 :

- package.json이 있는 경로에서 아래와 같이 설치합니다.

```bash
$ npm install
```

* * *

- 종속성까지 다운받기위해 스트립트를 생성합니다.

```bash
$ vi download-npm-deps.sh
```
```bash
#!/bin/bash

set -e

OUTPUT_DIR=npm-tarballs # NPM 파일 저장경로
mkdir -p $OUTPUT_DIR

echo "dependency 목록 수집..."

npm ls --all --json | jq -r '
  def walkdeps:
    .dependencies // {} |
    to_entries[] |
    select(.value.version != null) |
    "\(.key)@\(.value.version)",
    (.value | walkdeps);

  walkdeps
' | sort -u > deps.txt

echo "총 $(wc -l < deps.txt)개 패키지 발견"

echo "tarball 다운로드 시작..."

while read pkg; do
  echo "packing $pkg"
  npm pack "$pkg" --pack-destination "$OUTPUT_DIR"
done < deps.txt

echo "완료!"
echo "저장 위치: $OUTPUT_DIR"
```

* * *

## 2. Nexus Repository의 NPM 파일 업로드하기 :

- 아래 스크립트를 통해서 외부망으로부터 다운받은 NPM 파일을 Nexus Repo에 업로드합니다.

```bash
#!/bin/bash

set -u

REGISTRY="http://nexus.test.com/repository/npm-hosted"
TGZ_DIR="${1:-./npm-tarballs}"
LOG_FILE="./publish-npm-tarballs.log"
SUCCESS_FILE="./publish-success.log"
FAIL_FILE="./publish-fail.log"

if [ ! -d "$TGZ_DIR" ]; then
  echo "[ERROR] tgz 디렉터리가 없습니다: $TGZ_DIR"
  exit 1
fi

find "$TGZ_DIR" -maxdepth 1 -type f -name "*.tgz" | sort > /tmp/npm_tgz_list.txt

TOTAL=$(wc -l < /tmp/npm_tgz_list.txt)

if [ "$TOTAL" -eq 0 ]; then
  echo "[ERROR] 업로드할 tgz 파일이 없습니다: $TGZ_DIR"
  exit 1
fi

: > "$LOG_FILE"
: > "$SUCCESS_FILE"
: > "$FAIL_FILE"

echo "======================================" | tee -a "$LOG_FILE"
echo "Nexus npm tarball publish 시작" | tee -a "$LOG_FILE"
echo "Registry : $REGISTRY" | tee -a "$LOG_FILE"
echo "Source   : $TGZ_DIR" | tee -a "$LOG_FILE"
echo "Total    : $TOTAL" | tee -a "$LOG_FILE"
echo "======================================" | tee -a "$LOG_FILE"

COUNT=0

while IFS= read -r FILE; do
  COUNT=$((COUNT + 1))
  BASENAME=$(basename "$FILE")

  echo "" | tee -a "$LOG_FILE"
  echo "[$COUNT/$TOTAL] publish 시작: $BASENAME" | tee -a "$LOG_FILE"

  OUTPUT=$(npm publish "$FILE" --registry="$REGISTRY" 2>&1)
  RC=$?

  echo "$OUTPUT" >> "$LOG_FILE"

  if [ $RC -eq 0 ]; then
    echo "[OK] $BASENAME" | tee -a "$LOG_FILE"
    echo "$BASENAME" >> "$SUCCESS_FILE"
  else
    if echo "$OUTPUT" | grep -q "previously published version"; then
      echo "[SKIP] 이미 업로드된 버전: $BASENAME" | tee -a "$LOG_FILE"
      echo "$BASENAME" >> "$SUCCESS_FILE"
    else
      echo "[FAIL] $BASENAME" | tee -a "$LOG_FILE"
      echo "$BASENAME" >> "$FAIL_FILE"
    fi
  fi

done < /tmp/npm_tgz_list.txt

SUCCESS_COUNT=$(wc -l < "$SUCCESS_FILE")
FAIL_COUNT=$(wc -l < "$FAIL_FILE")

echo "" | tee -a "$LOG_FILE"
echo "======================================" | tee -a "$LOG_FILE"
echo "업로드 완료" | tee -a "$LOG_FILE"
echo "성공/스킵 : $SUCCESS_COUNT" | tee -a "$LOG_FILE"
echo "실패      : $FAIL_COUNT" | tee -a "$LOG_FILE"
echo "성공 목록 : $SUCCESS_FILE" | tee -a "$LOG_FILE"
echo "실패 목록 : $FAIL_FILE" | tee -a "$LOG_FILE"
echo "전체 로그 : $LOG_FILE" | tee -a "$LOG_FILE"
echo "======================================" | tee -a "$LOG_FILE"

if [ "$FAIL_COUNT" -gt 0 ]; then
  exit 1
fi

exit 0
```

* * *