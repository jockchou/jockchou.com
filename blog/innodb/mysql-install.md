<!--
author: jockchou
date: 2015-08-18
title: MySQL5.6源码编译安装
tags: MySQL，源码
category: MySQL数据库
status: publish
summary: 这篇博客将学习如何通过源码编译安装MySQL5.6，安装完后会进一步学习MySQL基本的功能配置以及基础测试。
-->

这篇博客将学习如何通过源码编译安装MySQL5.6，安装完后会进一步学习MySQL基本的功能配置以及基础测试。

安装MySQL的基本步骤：

1. 检测安装MySQL的系统环境要求  
2. 下载MySQL5.6源码包  
3. 编译，安装  
4. 安装后的设置和测试  
5. 安装Review

## 源码安装系统要求 ##
1. [CMake][1]  
2. [GNU make][2] 3.75以上版本  
3. GCC 4.2.1以上版本  


查看make版本：

```
make -version

GNU Make 3.81
```

查看gcc版本：

```
gcc -v

gcc version 4.4.7 20120313 (Red Hat 4.4.7-11) (GCC) 
```

系统没有cmake，需要安装cmake，直接下载cmake官方提供的安装脚本执行安装：

```
wget http://www.cmake.org/files/v3.3/cmake-3.3.1-Linux-x86_64.sh
chmod 755 cmake-3.3.1-Linux-x86_64.sh 
./cmake-3.3.1-Linux-x86_64.sh 
```
把cmake安装到 `/usr/local/cmake`目录，并配置path环境变量：

```
export PATH="$PATH:/usr/local/cmake/bin"
```

## 下载MySQL5.6源码包 ##

MySQL5.6源码包下载地址：

```
http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.26.tar.gz
```

## 配置，编译，安装 ##

### 1) 添加MySQL运行账户和组 ###

```
useradd -r -U mysql -M -d /usr/local/mysql/data
```

注：`mysql`用户的home目录被设置为`/usr/local/mysql/data`，这是因为我们计划把MySQL服务安装到`/usr/local/mysql/`目录中，你可以指定不同的目录安装和存储数据。

```
useradd命令选项的含义

-r
    创建系统账号

-U
    同时创建所属的组，并且组名和用户名同名
    
-M
    不创建账号的home目录

-d
    指定新账号的登录目录
```

以上命令将会创建mysql账户，将且用户组名也为mysql。创建的账户为系统账号，没有密码设置，主目录为`/usr/local/mysql/data/`，但这个目录并没会创建。登录shell为/bin/bash。

### 2) 解压安装 ###

下载好的MySQL源码放在下面位置：

```
/usr/local/sr/mysql-5.6.26.tar.gz
```

解压：

```
tar -zxvf mysql-5.6.26.tar.gz 
```

安装：

```
cd mysql-5.6.26
```

运行cmake：

注：
默认的安装目录为`/usr/local/mysql/`，如果要改变目录，使用`CMAKE_INSTALL_PREFIX`参数。

```
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql
```

默认的数据目录为`/usr/local/mysql/data/`，如果要改变，使用`MYSQL_DATADIR`参数。

```
cmake . -DMYSQL_DATADIR=/usr/local/mysql/data
```

查看默认的安装配置选择，使用如下命令：

```
cmake . -LH
```

可能的错误：

```
The CXX compiler identification is unknown
```

检测gcc和gcc-c++是否安装，没安装使用yum安装

```
yum install gcc
yum install gcc-c++
```

```
 Curses library not found.  Please install appropriate package
```

安装ncurses-devel

```
yum install ncurses-devel
```

```
Warning: Bison executable not found in PATH
```

安装bison

```
yum install bison
```


```
 Googlemock was not found. gtest-based unit tests will be disabled. You can run cmake . -DENABLE_DOWNLOADS=1 to automatically download and build required components from source.
```

即使加上参数`-DENABLE_DOWNLOADS=1`也是没用的，因为google被qiang了。同时MySQL Bug [69854][6]存在也会导致失败。还是需要手动下载gmock安装。

```
wget http://googlemock.googlecode.com/files/gmock-1.6.0.zip
```

我这里设置VPN来下载gmock1.6，试过1.7不行。
解压gmock到mysql目录下：

```
 unzip gmock-1.6.0.zip -d mysql-5.6.26/source_downloads
```
以上命令会解压gmock到mysql源码的source_downloads目录下，这个目录会自动创建



所有的配置项参考：

[http://dev.mysql.com/doc/mysql-sourcebuild-excerpt/5.6/en/source-configuration-options.html][5]

注：
    重新cmake时，需要清除CMakeCache.txt文件
```
 rm -rf CMakeCache.txt
```

执行cmake:

```
cmake . -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci
```

cmake会检查安装需要的系统环境要求，并根据系统变量的值生成Makefile文件。


执行make:

```
make
```
make根查看cmake生成的Makefile文件，编译mysql源代码，这个过程会比较长一点。

安装：

```
make install
```


### 3) 设置系统PATH ###

MySQL的可执行文件都放到了`/usr/local/mysql/bin/`目录，我不打算把每个一都添加软链接到`/usr/bin/`中，直接把`/usr/local/mysql/bin/`设置到系统PATH中。
需要自己添加脚本到`/etc/profile.d/`目录。  

在`/etc/profile.d`目录中创建`mysql.sh`文件，输入内容：

```
if ! echo ${PATH} | /bin/grep -q /usr/local/mysql/bin ; then
PATH=/usr/local/mysql/bin:${PATH}
fi
```

创建`mysql.csh`文件，输入内容：
```
if ( "${path}" !~ */usr/local/mysql/bin* ) then
set path = ( /usr/local/mysql/bin $path )
endif
```

由于默认使用的Bash Shell，执下如下命令使其生效：

```
source /etc/profile.d/mysql.sh
```

### 4) 共享MySQL库文件 ###

MySQL库文件位于`/usr/local/mysql/lib/`目录，为了方便其它程序在运行时找到MySQL库，需要简单配置：

```
echo "/usr/local/mysql/lib" > /etc/ld.so.conf.d/mysql.conf
ldconfig
```

### 5) 安装MySQL manpages ###
MySQL manpages位于`/usr/local/mysql/man/`目录，由于`/usr/local/mysql/bin/`目录已经添加到PATH中了，所以man工具会自动搜索这个目录。不需要额外的配置。
如果你的系统没有安装man工具，使用yum安装：

```
yum install man
```

### 6) 修改权限，导入授权表 ###

```
cd /usr/local/mysql/
chown -R mysql:mysql .                 #为下面的导入数据授权
scripts/mysql_install_db --user=mysql  #导入授权表
chown -R root .                        #恢复权限
chown -R mysql data                    #data目录只有mysql账号有权限
chmod -R go-rwx data                   #移除组和其他用户的权限
```

### 7) 设置配置文件my.cnf ###
默认会有三个配置文件：
```
/etc/my.cnf
/usr/local/mysql/my.cnf 
/usr/local/mysql/support-files/my-default.cnf
```
复制my-default.cnf到/etc/my.cnf

```
cp support-files/my-default.cnf /etc/my.cnf
```

my.cnf内容可以很简单:
```
[mysqld]
user=mysql
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

[http://dev.mysql.com/doc/refman/5.6/en/server-options.html][9]
### 8) 启动MySQL ###

```
mysqld_safe --user=mysql &
```

### 9) 设置MySQL服务 ###

```
cp -v support-files/mysql.server /etc/rc.d/init.d/mysqld
chkconfig --add mysqld
```

[http://dev.mysql.com/doc/refman/5.6/en/automatic-start.html][8]

使用service命令启动mysql

```
service mysql start
```

## 安装后的设置和测试 ##

### 1) 安装MySQL Benchmark Suite ###

安装Perl模板Test::Deep
```
yum install perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker
yum install perl-DBI
yum install perl-Test-Deep
```

下载DBD:mysql源码包

```
wget http://search.cpan.org/CPAN/authors/id/C/CA/CAPTTOFU/DBD-mysql-4.032.tar.gz
tar -zxvf DBD-mysql-4.032.tar.gz
cd DBD-mysql-4.032
perl Makefile.PL --ssl
make
make install
```

### 2) 运行基准测试 ###

```
cd /usr/local/mysql/sql-bench/
./run-all-tests --server=mysql --log --fast
```

运行全部测试时间比较长，可以选择单独执行部分测试。


## 安装Review ##

安装位置：
```
/usr/local/mysql
```

PID file:

```
/usr/local/mysql/data/i-7jyb6ynm.pid
```

Error Log:

```
/usr/local/mysql/data/i-7jyb6ynm.err
```

Socket file:

```
/tmp/mysql.sock
```

Service file:

```
/etc/rc.d/init.d/mysqld
```

Configuration files

```
/etc/my.conf
```

Log files
```
1) Error log
2) General query log
3) Binary log
4) Relay log
5) Slow query log
```
[http://dev.mysql.com/doc/refman/5.6/en/server-logs.html][7]

Executables

```
/usr/local/mysql/bin/
/usr/local/mysql/mysql-test/
/usr/local/mysql/scripts/
/usr/local/mysql/sql-bench/
/usr/local/mysql/support-files/
/usr/local/mysql/support-files/solaris/
/etc/rc.d/init.d/
```

###Get information on Server status###

Displays the status of MySQL service
```
service mysql status
```

Displays the status of mysqld daemon
```
mysqladmin -u root -p ping
```

Displays the status of server with variables
```
mysqladmin -u root -p status
```

Display the status of server with variables and their values
```
mysqladmin -u root -p extended-status
```

### Get information on active Server threads###

Displays a list of active server threads
```
mysqladmin -u root -p processlist
```

Displays a list of active server threads with full processlist
```
mysqladmin -u root -p -v processlist
```

### Get information on Server variables ###

Displays a list of server system variables and their values
```
mysqladmin -u root -p variables
```


Get information on Server version
```
mysqladmin -u root -p version
``` 
 

[1]:http://www.cmake.org
[2]:http://www.gnu.org/software/make/
[3]:http://howtolamp.com/lamp/mysql/5.6/installing/
[4]:http://dev.mysql.com/doc/mysql-sourcebuild-excerpt/5.6/en/installing-source-distribution.html
[5]:http://dev.mysql.com/doc/mysql-sourcebuild-excerpt/5.6/en/source-configuration-options.html
[6]:http://bugs.mysql.com/bug.php?id=69854
[7]:http://dev.mysql.com/doc/refman/5.6/en/server-logs.html
[8]:http://dev.mysql.com/doc/refman/5.6/en/automatic-start.
[9]:http://dev.mysql.com/doc/refman/5.6/en/server-options.html