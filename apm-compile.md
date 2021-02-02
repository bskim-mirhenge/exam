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

- make 실행중 __collect2: error: ld returned 1 exit~__   
  같은 오류가 샐길시.

```linux
vi /build/config_vars.mk
  //AP_LIBS 로 시작하는 줄 찾아서. 맨 뒤에 -lexpat 추가.
make clean
make
```

- apache 깔렸는지 확인
```linux
systemctl start httpd
systemctl enable httpd
```
## 3. MariaDB Compile
---

> Version : MariaDB : 5.5.68

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
  즉, ./configure 가 아니라 __cmake__ 로 컴파일 해야한다.

```linux
src]#yum install cmake
```
- cmake 다운 후 컴파일 시도.

```linux
mariadb-5.5.68]# cmake ../mariadb-5.5.68 -LH
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

- __C++ 와 lib__  오류가 생긴다. 설치 후 다시 시도.

```linux
yum install gcc-c++ libevent-devel
rm -rf CMakeCache.txt
```

- cmake 설정을 추가 해서 설치 해본다.
```linux
cmake ../mariadb-5.5.68     -DWITH_READLINE=1             -DWITH_READLINE=1     -DWITH_SSL=bundled     -DWITH_ZLIB=system     -DDEFAULT_CHARSET=utf8     -DDEFAULT_COLLATION=utf8_general_ci     -DENABLED_LOCAL_INFILE=1     -DWITH_EXTRA_CHARSETS=all     -DWITH_ARIA_STORAGE_ENGINE=1     -DWITH_XTRADB_STORAGE_ENGINE=1     -DWITH_ARCHIVE_STORAGE_ENGINE=1     -DWITH_INNOBASE_STORAGE_ENGINE=1     -DWITH_PARTITION_STORAGE_ENGINE=1     -DWITH_BLACKHOLE_STORAGE_ENGINE=1     -DWITH_FEDERATEDX_STORAGE_ENGINE=1     -DWITH_PERFSCHEMA_STORAGE_ENGINE=1
```

```linux
make
make install
```

- 설치 확인.

```linux
rpm -qa | grep maria
```

## 4. PHP Compile
---

> Version : php/5.2.17 gcc/4.8.5 libxml2/2.9.1
>> 상위 2개와는 다른서버에 구축함.

### 4.1 PHP 컴파일

- PHP 파일을 컴파일한다.
```linux
src]# https://museum.php.net/php5/php-5.4.16.tar.gz
tar xvfz php-5.4.16~
cd php-5.4.16
./configure
```

- C 컴파일러가 없다고 오류.
```linux
wget https://ftp.gnu.org/gnu/gcc/gcc-4.8.5/gcc-4.8.5.tar.gz
tar xvfz gcc-4.8.5.tar.gz 
cd gcc-4.8.5/
./configure
```
- 무언가 오류가 있어 제대로 설치가 안된다...
  - __RPM으로 설치를 시도해봤다__.

```linux
rpm -Uvh http://mirror.centos.org/centos/7/os/x86_64/Packages/gcc-4.8.5-44.el7.x86_64.rpm
rpm -Uvh gcc-4.8.5-44.el7.x86_64.rpm 
```

- 추가로 필요한 패키지를 설치한다.
  - 해당 페이지를 들어가면 __Require__ 하는 버전의 rpm파일들이있다. 모두 설치해야함.

```linux
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/libgomp-4.8.5-44.el7.x86_64.rpm
rpm -Uvh libgomp-4.8.5-44.el7.x86_64.rpm
---
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/libgcc-4.8.5-44.el7.x86_64.rpm
rpm -Uvh libgcc-4.8.5-44.el7.x86_64.rpm
---
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/cpp-4.8.5-44.el7.x86_64.rpm
rpm -Uvh cpp-4.8.5-44.el7.x86_64.rpm
rpm -Uvh gcc-4.8.5-44.el7.x86_64.rpm 
```
- php 에서 다시 컴파일을 시도. __libxml2__ 이 없다고 출력됨.

```linux
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/libxml2-2.9.1-6.el7_2.3.x86_64.rpm
rpm -Uvh libxml2-2.9.1-6.el7.5.x86_64.rpm
---
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/libxml2-python-2.9.1-6.el7.5.x86_64.rpm
rpm -Uvh libxml2-python-2.9.1-6.el7.5.x86_64.rpm
```
- 다시 php 에서 컴파일 시도.
```linux
php~}# ./configure
make
make test
make install
```
- pear Structures_Graph 등을 install 하라고 나온다.
  - 버전 차이 이므로 버전 업데이트를 한다.
```linux
pear upgrade-all
```

- 설치 확인
```linux
php --verion
systemctl enable php
systemctl start php
systemctl enable php
```
