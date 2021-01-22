APM - Compile Exam
==========================

## 1. APM 설치
--------

### 1-1. httpd, php, mariadb 설치
 ```linux
yum install httpd php mariadb-server
systemctl enable httpd
systemctl start httpd
systemctl status httpd -l
```
### 1-2. php 연동 확인
```linux
vi /var/www/html/phpinfo.php
    <?php
    phpinfo();
    ?>
```
- web에서 http://서버IP/phpinfo.php  에 접속하면 버전 정보 볼수 있음.

## 2. httpd Compile
---

> Version : Apache/2.4.6  OpenSSL/1.0.2 pcrelib/8.32 

### 2.1. Apache-2.4.6.tar.gz 컴파일
```linux
cd /usr/local/src
wget http://archive.apache.org/dist/httpd/httpd-2.4.6.tar.gz
tar xvfp httpd-2.4.6.tar.gz
cd httpd-2.4.6/
./configure
```
- APR 이 없으므로 오류가 뜸.  
>--> yum 보단 직접 src/ 밑에 컴파일 다운로드.
```linux
src]# wget https://downloads.apache.org//apr/apr-1.6.5.tar.gz
tar xvfp apr-1.6.5.tar.gz
cd apr-1.6.5/
./configure
```
- _C_ 가 없어 configure 안됨
```linux
yum install gcc
```
- 다시 apr 컴파일 시도
```linux
apr]# ./configure
make
make install
```
- apr 설치후 httpd configure 시도. but, _apr-util_ 없어서 오류.
```linux
src]# wget https://downloads.apache.org//apr/apr-util-1.6.1.tar.gz
tar xvfp apr-util-1.6.1.tar.gz
cd apr-util-1.6.1.tar.gz
./configure      # 오류. --with-apr 포함하라고 뜸.
./configure --with-apr=/usr/local/src/apr-1.6.5
make
make install     # expat 없어서 오류.
yum install expat-devel
make
make install
```
- 다시 httpd 에서 configure 하려 했으나 이번엔 _pcrelib_ 가 없다고 뜸.
> --> web에서 확인 해보면 PCRE libe~ 버전 확인 가능.
```linux
src]# wget ftp://ftp.pcre.org/pub/pcre/pcre-8.32.tar.gz
tar xvfp pcre-8.32.tar.gz
cd pcre-8.32
./configure
```
- _c++_ 없다고 안됨.
```linux
yum install gcc-c++
```
```linux
pcre-8.32]# ./configure
make
make install
```
- 모든 설정 완료. _httpd_ 를 컴파일 해보자.
```linux
httpd]# ./configure
make
```
- collect2: error: ld returned 1 exit status 란 오류가 뜸.
  - 앞에서 다운받은 expat의 라이브러리를 인식하지 못해서 생기는 오류.
> ld : 라이브러리를 찾는 링커. But. /lib, /usr/lib 같이 정해진 디렉터리만 찾는다.  그래서 expat 같은 경우 하위 디렉터리에 있기 때문에 직접 설정해줘야함.
```linux
httpd]# cd build/config_vars.mk
AP_LIBS 로 시작하는 파일을 찾아서 -lexpat 추가
httpd]# make
make install
```

- apache 깔렸는지 확인
```linux
systemctl start httpd
systemctl enable httpd
systemctl status httpd
```
## 3. PHP Compile
---
