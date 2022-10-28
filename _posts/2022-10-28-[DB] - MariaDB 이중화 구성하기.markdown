---
title: "[DB] - MariaDB 이중화 구성하기"
layout: post
date: 2022-10-28
image: /assets/images/Post/mariadb.png
headerImage: true
tag:
- DB 이중화
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## MariaDB Replication 절차:
- MariaDB에서 기본적으로 제공하는 Replication 기능으로 Master-Slave 구조로 되어있다.
- Master 서버의 Binary로그를 Slave서버가 Relay로그에 저장해서 복제하는 방식이다.
[![텍스트](/assets/images/DB/MariaDB%20Replication.PNG)](/assets/images/DB/MariaDB%20Replication.PNG)