---
layout: post
title: "인코딩&디코딩이란"
date: 2023-02-22
categories: [보안 & 품질, Encryption] 
tags: [Base64, Encoding, Decoding]
image: /assets/img/post-title/encryption.jpg
---

## 1. 인코딩&디코딩을 알고싶었던 계기 :
- SI사업 지원을 하면서 사용자 패스워드 암호화 후 Base64로 인코딩된 걸 인사연동 시 디코딩해서 가져오는 쿼리를 구현했다. 나는 Base64도 sha256 같은 암호화 방식으로 알고 있었고, 내가 제대로 이해한게 맞는지 확인하고 싶어서다.

* * *

## 2. 인코딩이란? :
- 사람이 인지할 수 있는 문자(언어)를 약속된 규칙에 따라 컴퓨터가 이해하는 언어 (0과1)로 이루어진 코드로 바꾸는 것을 말하고 저장단위는 바이트(byte)이다.

[![텍스트](/assets/img/post/Encryption/인코딩.png)](/assets/img/post/Encryption/인코딩.png)

* * *

## 3. 바이트(byte) 컴퓨터 기본 저장단위 :
- 1바이트(1byte)는 8bit이다.
- 1바이트에는 2의 8승, 즉 256개의 값을 저장할수 있다.

* * *

## 4. Base64 인코딩이란? :
- Binary Data를 Text로 바꾸는 Encoding의 하나로써 Binary Data를 Character set에 영향을 받지 않는 공통 ASCII 영역의 문자로만 이루어진 문자열로 바꾸는 것이다.

* * *

### 4.1 Binary Data란? :
- 이진 데이터, 이진수(0,1)로 표시한 데이터이다.

* * *

## 5. 디코딩이란? :
- 컴퓨터가 이해하는 언어 (0과1)로 이루어진 코드를 약속된 규칙에 따라 사람이 인지할 수 있는 문자(언어)로 바꾸는 것을 말한다.

[![텍스트](/assets/img/post/Encryption/디코딩.png)](/assets/img/post/Encryption/디코딩.png)

* * *

## 6. 인코딩과 디코딩을 하는 이유 :
- 언어마다 하나의 규격으로 표준화 시켜 개인의 요구에 따라 사용할 수 있도록 문자 집합을 만들고, 이러한 문자 집합을 가지고 부호화 하여 사용하기 위해서다.

* * *