---
title: "[Linux] - OpenSSL 패치하기"
layout: post
date: 2022-10-05
image: /assets/images/Post/ssl.png
headerImage: true
tag:
- SSL
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## OpenSSL란?:
- 네트워크를 통한 데이터 통신에 쓰이는 프로토콜인 TLS와 SSL의 오픈 소스 구현판이다.

### SSL란?:
- 클라이언트와 서버간의 통신을 제3자가 보증해주는 전자화된 문서
- 클라이언트가 서버에 접속한 직후 서버는 클라이언트에게 인증서의 정보를 전달한다.
- 클라이언트는 인증서 정보가 신뢰할 수 있는 것인지를 검증 한 후 다음 절차를 진행한다.<br>
<span style="color:#FA5858; font-size:12px">※ SSL인증서를 사용하여 웹서비스에 보안을 강화, https를 적용할 수 있다.</span>

* * *