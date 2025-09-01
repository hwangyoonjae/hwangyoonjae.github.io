---
layout: post
title: "git diff 이해하기"
date: 2024.09.24
categories: [DevOps, Gitlab] 
tags: [Git, Gitlab, Runner, CI, CD]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## git diff 명령어란? :
- commit 이나 branch 사이에 다른점 혹은 파일이나 Repository와 Working Directory 사이의 다른점을 보여주는 명령어이다.

* * *

## git diff와 git diff HEAD를 이용한 변경 사항 확인하기 :
- git diff : Working Directory와 Staging Area 사이의 차이 확인
- git diff HEAD : Working Directory HEAD Commit에 대한 Change확인

[![gitlab diff 그림](/assets/img/post/Gitlab/git%20diff%20그림.png)](/assets/img/post/Gitlab/git%20diff%20그림.png)

> Working Directory와 Staging Area?
>
> Working Directory : 사용자가 실제로 파일을 수정하고 작업하는 디렉토리
> 
> Staging Area : Git이 추적하는 변경 사항들을 커밋하기 전에 임시로 저장해 두는 영역
{: .prompt-info}

* * *

## diff 내용 확인하기:
```bash
$ git diff 
```

- 아래 그림과 같이 붉은색 부분은 고치기 이전 버전, 연두색 부분은 고친 이후 버전이다.
![git diff 명령어 실행 결과 화면](/assets/img/post/Gitlab/git%20diff%20명령어%20실행%20결과%20화면.png)

> @@ -21,7 +21,15 @@ 해석
>
> **-21,7** : 파일의 **이전 버전(기존 커밋)**에서 21번째 줄부터 7줄이 수정되었음을 의미한다.
>
> **+21,15** : **새로운 버전(변경 후)**에서 21번째 줄부터 15줄이 새롭게 추가되거나 변경되었음을 의미한다.
{: .prompt-tip}

* * *