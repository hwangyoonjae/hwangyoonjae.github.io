---
layout: post
title: "[Gitlab] - git 변경내역 확인하기"
date: 2024.10.31
categories: Gitlab
tags: [Git, fetch, pull]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## git fetch :
- 원격 저장소의 변경 사항을 로컬 저장소로 가져오지만, 로컬의 작업 디렉토리에 바로 적용하지는 않는다.
- 로컬 브랜치와 원격 브랜치의 상태를 동기화하기 위해 사용하는 명령어

```bash
$ git fetch origin
```

- fetch 명령어 실행 시 변경 사항이 있을 경우 아래 그림과 같이 실행된다.

![git fetch 명령어 실행화면](/assets/img/post/Gitlab/git%20fetch%20명령어%20실행화면.png)

> **git fetch**는 git pull을 하기 전의 준비 단계라고 생각하면 된다.
> 
{: .prompt-info}

* * *

## git pull :
- git fetch와 git merge를 한 번에 수행하는 명령어
- 원격 저장소의 변경 사항을 가져온 후, 현재 로컬 브랜치에 병합하여 바로 적용
- 로컬 작업 디렉토리와 원격 저장소를 즉각적으로 동기화하는 데 사용

```bash
$ git pull origin main
```

* * *

## git fetch VS git pull :

|명령어|기능|브랜치 적용여부|
|-----|-----|-----|
|git fetch|원격 저장소의 변경 사항을 가져오기|❌ (로컬 브랜치에는 적용되지 않음)|
|git pull|원격 저장소의 변경 사항 가져오기 + 병합|✅ (로컬 브랜치에 병합 및 적용됨)|

* * *