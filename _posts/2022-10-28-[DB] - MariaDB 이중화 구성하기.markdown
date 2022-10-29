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
- MasterDB에 이벤트가 발생하면 SlaveDB와 복제를 위해 생성한 Binaary log에 DB업데이트와 동시에 기록한다.
- SlaveDB는 자신이 MasterDB의 몇 번째 위치의 데이터를 마지막으로 가져왔는지 기록했다가 MasterDB의 Binaary Log에 새로운 기록이 업데이트 되면 가져오고 가져온 위치를 기억한 후 SlaveDB는 전달받은 Binaary log를 Relay Logdp 기록하여 순차적으로 DB에 저장한다.