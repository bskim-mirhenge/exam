APM - APACHE 연동
=====================

## APM 설치
--------

### 1. Mariadb 설치
----
> Version// Mariadb : 5.5.68

```linux
cd /usr/local/src
wget --content-disposition https://downloads.mariadb.org/interstitial/mariadb-5.5.58/source/mariadb-5.5.58.tar.gz/from/https%3A//archive.mariadb.org/
tar xvfz mariadb-5.5.68.tar.gz
```
- cmake 디렉터리 생성
```linux
mkdir build-mariadb
cd build-mariadb
```

- 계정 생성

```linux
useradd -r mysql
cat /etc/passwd | grep mysql
```

- 추가 데몬, 프로그램 설치

```linux
yum install gcc gcc-c++ bison libxml2-devel libevent-devel cmake
```

- readline 문서 확인

```linux
cat ../mariadb-5.5.58/cmake/readline.cmake  | less
cat ../mariadb-5.5.58/storage/tokudb/CMakeLists.txt  | less
```

- 필요 데몬 설치

```linux
yum install ncurses-devel
```

- 하위 생성 디렉터리 삭제 __[%%]__
   - 자주 쓰이므로 __%%__ 로 정의.
```linux
pwd
/usr/local/src/build-mariadb
rm -rf *
```

- cmake 옵션 확인
```linux
cmake .../mariadb-5.5.69 -LH
```

- __not found__ 항목중 yum 으로 지원하는 데몬 찾기

> Version// valgrind : 3.15.0 , libaio : 0.3.109 , pam : 1.1.8

```linux
yum provides "*valgrind/valgrind.h"
yum provides "*libaio/libdio.h"
yum provides "*pam/pam.h"
                    .
                    .
                    .
```

- 버전에 맞게 설치

```linux
yum install valgrind-devel-3.15.0
yum install libaio-devel-0.3.109
yum install pam-devel-1.1.8
```

- CMAKE 실행

```linux
cmake ../mariadb-5.5.58 -DWITH_READLINE=1 -DWITH_SSL=bundled -DWITH_ZLIB=system -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DENABLED_LOCAL_INFILE=1 -DWITH_EXTRA_CHARSETS=all -DWITH_ARIA_STORAGE_ENGINE=1 -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data
```

- boost 관련 오류
```linux
yum install boost-1.53.0
```

-  __%%__  실행

- cmake 다시 실행
```linux
cmake ../mariadb-5.5.58 -DWITH_READLINE=1 -DWITH_SSL=bundled -DWITH_ZLIB=system -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DENABLED_LOCAL_INFILE=1 -DWITH_EXTRA_CHARSETS=all -DWITH_ARIA_STORAGE_ENGINE=1 -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data
make
make install
```

- 권한 설정

```linux
chown -R mysql /usr/local/mysql
```

- mysql_install_db (__초기 데이터베이스 설치파일__)분석

```linux
cd /usr/local/mysql
scripts/mysql_install_db --help
```

- mysql_install_db 실행
```linux
scripts/mysql_install_db --skip-name-resolve --datadir=/usr/localmysql/data --user=/mysql
cp -p ./support-files/my-innodb-heavy-4G.cnf. ./my.cnf   // 설정파일복사
```

- mariadb 서버 실행
```linux
/usr/local/mysql/bin/mysqld_safe --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=/mysql &

// 로그 확인 ( /var/log/mariadb/mariadb.log , 없을시 생성. )

mkdir /var/log/mariadb
mkdir /var/run/mariadb
```

- mariadb 서버 실행 (2)
```linux 
/usr/local/mysql/bin/mysqld_safe --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=/mysql &

// 로그 확인 후 수정,  재실행.
chown mysql /var/run/mariadb
/usr/local/mysql/bin/mysqld_safe --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=/mysql &
```

- mariadb client 서버 접속 실행
```linux
/usr/local/mysql/bin/mysql
]> show databases;
]> exit
```

- 출력 결과 확인 후 재 설정
``` linux
scripts/mysql_install_db --skip-name-resolve --datadir=/usr/local/mysql/data --user=mysql
```
### 2. Apache 설치
----
> Version// httpd : 2.4.6 apr : 1.6.5  apr-util : 1.6.1 pcre : 8.32

- 필수 데몬 설치
```linux
yum install gcc-c++ expat-devel
```

- 데몬 pcre 컴파일
```linux
mkdir -p /usr/local/src/apache
cd /usr/local/src/apache
wget ftp://ftp.pcre.org/pub/pcre/pcre-8.32.tar.gz
tar xvczf pcre-8.32.tar.gz
cd pcre-8.32
./configure
make
make install
```

- _httpd_ 파일 가져오기
```linux
wget http://archive.apache.org/dist/httpd/httpd-2.4.6.tar.gz
tar xvzf httpd-2.4.6.tar.gz
cd httpd-2.4.6
```

- __apr & apr-util__ 파일 가져오기

```linux
wget https://downloads.apache.org//apr/apr-1.6.5.tar.gz
wget https://downloads.apache.org//apr/apr-util-1.6.1.tar.gz
tar xvzf apr-1.6.5.tar.gz
tar xvzf apr-util-1.6.1.tar.gz
mv apr-1.6.5 httpd-2.4.6/srclib/apr
mv par-util-1.6.1 httpd-2.4.6/srclib/apr-util
```

- httpd 컴파일
```linux
cd httpd-2.4.6
./configure --prefix=/usr/local/apache
make
make install
```

- systemd service 등록
```linux
vi /usr/lib/systemd/system/httpd.service // 새파일생성

[Unit]
Description=The Apache HTTP Server

[Service]
Type=forking
PIDFile=/usr/local/apache/logs/httpd.pid
ExecStart=/usr/local/apache/bin/apachectl start
ExecReload=/usr/local/apache/bin/apachectl graceful
ExecStop=/usr/local/apache/bin/apachectl stop
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

```linux
systemctl start httpd
systemctl enable httpd
```

### 3. PHP 설치
---

> Version// php : 5.4.16

- php 파일 설치
```linux
cd /usr/local/src
wget https://museum.php.net/php5/php-5.4.16.tar.gz
tar xvfz php-5.4.16.tar.gz
cd php-5.4.16
```
- 필요 데몬 설치
```linux
yum install libxml2-devel libpng-devel libjpeg-devel
```

- 컴파일
```linux
./configure --prefix=/usr/local/php --with-mysqli --with-pdo-mysql=mysqlnd --with-apxs2=/usr/local/apache/bin/apxs --with-config-file-path=/usr/local/apache/conf --with-zlib --disable-debug --enable-calendar --enable-ftp --enable-sockets --enable-sysvsem --with-gd --with-jpeg-dir=/usr/lib64
make
make install
```

- libtool 관련 오류시.
```linux
yum install libtool
libtool --finish /usr/local/src/php-5.4.16/libs
```

### 4. PHP 설정 && PHP-APACHE 연동

- php 설정 파일 복사 (php.ini 중 development 사용.)
```linux
cp /usr/local/src/php-5.4.16/php.ini-development /usr/local/apache/conf/php.ini
```

- mysql 파일 추가.
```linux
vi /usr/local/apache/conf/php.ini

// socket 검색하여 mysql.sock 파일 위치 추가
// vi /etc/my.cnf 에서 socket 검색하여 참고.

mysqli.default_socket = /var/lib/mysql/mysql.sock
```

- apache 설정 값 추가.
```linux
vi /usr/local/apache/conf/httpd.conf

// [[  ]] 안의 값 검색하여 추가 

<IfModule dir_module>
    DirectoryIndex index.html [[index.php index.jsp]]
</IfModule>
.
.
AddType application/x-compress .Z
AddType application/x-gzip .gz .tgz
[[
AddType application/x-httpd-php .php .html .htm .inc

AddType application/x-httpd-php-source .phps
]]
```

- Apache.service 시작
```linux
systemctl start httpd
```
- phpinfo 설정
```linux
vi /usr/local/apache/htdocs/index.html

<html><body>

<?php phpinfo(); ?>

<h1>It works!</h1></body></html>

```

- 웹에서 http://IP주소 확인.
