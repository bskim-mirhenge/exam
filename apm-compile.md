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
make install
```
- apache 깔렸는지 확인
```linux
systemctl start httpd
systemctl enable httpd
```
## 3. MariaDB Compile
---

> Version : MariaDB/5.5.68

### 3.1 MariaDB 파일 설치

- wget 이 아니라 직접 파일을 다운 받아   
  컴파일할 디렉터리 (__/usr/local/src__ )에 옮기기.

```linux
cd /usr/local/src
tar mariadb-5.5.68~
cd mariadb~~
```

### 3.2 MariaDB Compile

- ls 를 확인해보면 configure 대신 cmake 가 있다.  
  즉, ./configure 가 아니라 cmake 로 컴파일 해야한다.

```linux
src]#yum install cmake
```
- cmake 다운 후 컴파일 시도.

```linux
mariadb-5.5.68]# cmake ../mariadb-5.5.68
```

- curses 관련 오류 생김.

```linux
yum install ncurses-devel
```

- 다시 컴파일 전, cmake 캐시를 지우고 다시 시도한다.
```linux
mariadb5.5.68]# rm -rf CMakeCache.txt
cmake ../mariadb-5.5.68
make
```

- -lz 관련 오류가 생긴다. 관련 라이브러리인 zlib 이 없어서 생김.

```linux
yum install zlib-devel
rm -rf CMakeCache.txt
cmake ../mariadb-5.5.68
make
make install
```

- 설치 확인.

```linux
rpm -qa | grep maria
```
