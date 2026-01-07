---
layout: post
title: "OpenSSL 패치하기"
date: 2022-10-05
categories: [운영체제 & 네트워크, Linux]
tags: [SSL]
image: /assets/img/post-title/linux-wallpaper.jpg
---

## 1. OpenSSL에 대해 알고 싶었던 계기 :
- 폐쇄망을 사용하는 고객사들 중에서 서버 보안점검에서 조치사항으로 버전 업그레이드를 해야한다는 문의가 온다.<br>
웹서버나 WAS 같은 경우에는 해당 홈페이지에서 릴리즈 된 파일로 패치하면 되는데 인프라와 관련된 패치는 혹시나 잘못되면 서버 자체가 고장날 수 있어 인프라 담당자들에게 요청합니다.<br>
하지만 유지보수 업체 중 인프라 담당이 지정되어있지 않는 경우도 있어 내가 직접 진행해야 하는데 이번에 OpenSSL 버전 업그레이드 문의가 오게되어 OpenSSL은 무엇이고, 폐쇄망 같은 경우에는 어떻게 패치해야하는지 알고 싶게 되었습니다.

* * *

## 2. OpenSSL란? :
- 네트워크를 통한 데이터 통신에 쓰이는 프로토콜인 TLS와 SSL의 오픈 소스 구현판입니다.

### 2.1 SSL란? :
- 클라이언트와 서버간의 통신을 제3자가 보증해주는 전자화된 문서
- 클라이언트가 서버에 접속한 직후 서버는 클라이언트에게 인증서의 정보를 전달합니다.
- 클라이언트는 인증서 정보가 신뢰할 수 있는 것인지를 검증 한 후 다음 절차를 진행합니다.<br>

> SSL인증서를 사용하여 웹서비스에 보안을 강화, https를 적용할 수 있습니다.
{: .promt-tip}

* * *

## 3. OpenSSL 패치환경 :
- 필자는 **폐쇄망**을 기준으로 패치를 진행하였다.

* * *

## 4. OpenSSL 패치하기 :
### 4.1 gcc 다운로드 및 rpm 설치하기 :
- openssl 설치를 위해 gcc 컴파일러가 필요하다.

```bash
# rpm패키지를 설치하지 않고 해당 특정디렉토리에 다운로드
$ yum install --downloadonly --downloaddir="다운로드 폴더" gcc
$ cd "다운로드 폴더"

# gcc 관련 rpm 파일들이 많으므로 아래 명령어를 통해서 여러개 설치
$ for X in * ; do rpm -Uvh --nodeps $X ; done
```
> yum 설치 시 인터넷이 되는 환경에서 다운로드 받아야합니다.
{: .promt-tip}

* * *

### 4.2 OpenSSL 사이트 방문하여 파일 다운받기 :
> * [OpenSSL 사이트 바로가기](https://www.openssl.org/source/ "OpenSSL 사이트")

- OpenSSL 사이트 통해서 최신버전 파일 다운받을 수 있습니다.

![텍스트](/assets/img/post/OpenSSL/OpenSSL%20%ED%8C%8C%EC%9D%BC%20%EC%B5%9C%EC%8B%A0%EB%B2%84%EC%A0%84.PNG)

- OpenSSL 사이트 통해서 이전버전 파일 다운받을 수 있습니다.

![텍스트](/assets/img/post/OpenSSL/OpenSSL%20%ED%8C%8C%EC%9D%BC%20%EC%9D%B4%EC%A0%84%EB%B2%84%EC%A0%84.PNG)

> 필자는 최신버전(1.1.1q)을 다운받았다.
{: .promt-info}

* * *

### 4.3 OpenSSL 적용하기 :
- OpenSSL tar파일 압축 해제하고 폴더로 이동합니다.

```bash
$ tar -zxvf openssl-1.1.1q.tar.gz
$ cd openssl-1.1.1q
```

![텍스트](/assets/img/post/OpenSSLOpenSSL%20%ED%8C%8C%EC%9D%BC%20%EC%95%95%EC%B6%95%ED%8C%8C%EC%9D%BC%EA%B3%BC%20%ED%95%B4%EC%A0%9C%20%ED%9B%84%20%ED%8%B4%EB%8D%94.PNG)

* * *

- 컴파일 및 빌드 후 설치 진행합니다.

```bash
# make 하기 위해 설정파일을 빌드
$ ./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl shared

# make를 빌드해주고 설치 진행
$ make
$ make install
```

* * *

- 하지만 설정파일 빌드하는 과정에서 아래와 같이 에러가 발생하는데, 아래 사진은 Perl 언어를 사용하는 컴파일러 버전 5가 필요하다는 에러이므로, Perl5 버전을 설치하면 해결됩니다.

![텍스트](/assets/img/post/OpenSSL/make%20%ED%95%98%EA%B8%B0%20%EC%9C%84%ED%95%B4%20%EC%84%A4%EC%A0%95%ED%8C%8C%EC%9D%BC%20%EB%B9%8C%EB%93%9C%20%EC%8B%9C%20%EC%97%90%EB%9F%AC%EB%B0%9C%EC%83%9D.PNG)

* * *

#### 4.4 Perl 5 버전 설치하기:
> * [Perl 사이트 바로가기](https://www.cpan.org/src/5.0/ "Perl 사이트")

- 위 사이트 접속하여 **perl-5.X.X.tar.gz**를 다운받습니다.<br>

> 필자는 최신버전(5.32.0)을 다운받았다.
{: .promt-tip}

* * *

- Perl tar파일 압축 해제하고 폴더로 이동합니다.

```bash
$ tar -zxvf perl-5.32.0.tar.gz
$ cd perl-5.32.0
```

* * *

- 컴파일 및 빌드 후 설치 진행합니다.

```bash
# make 하기 위해 설정파일을 빌드
$ ./Configure -des -Dprefix=$HOME/localperl

# make를 빌드해주고 설치 진행
$ make
$ make install
```

* * *

- Perl 5버전 설치가 완료되었으면 OpenSSL 컴파일 및 빌드 후 설치를 다시 진행합니다.
- 아래 그림과 같이 정상적으로 실행되는걸 볼 수 있습니다.

![텍스트](/assets/img/post/OpenSSL/OpenSSL%20Perl5%EB%B2%84%EC%A0%84%20%EC%84%A4%EC%B9%98%20%ED%9B%84%20%EC%A0%95%EC%83%81%20%EC%BB%B4%ED%8C%8C%EC%9D%BC%20%EB%90%98%EB%8A%94%20%ED%99%94%EB%A9%B4%20.PNG)

* * *

- 라이브러리에 등록합니다.

```bash
$ vi /etc/ld.so.conf.d/openssl-1.1.1q.conf
  /usr/local/ssl/lib (해당내용 등록)
$ cat /etc/ld.so.conf.d/openssl-1.1.1q.conf
$ ldconfig -v
```

* * *

- openssl 백업 및 심볼릭 링크 설정합니다.

```bash
$ mv /usr/bin/openssl /usr/bin/openssl-1.0.2
$ ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
```

* * *

- openssl 버전 확인합니다.

```bash
$ openssl version
```

![텍스트](/assets/img/post/OpenSSL/OpenSSL%20%EB%B2%84%EC%A0%84%ED%99%95%EC%9D%B8.PNG)

* * *