<!--
author: jockchou
date: 2015-08-18
title: MySQL5.6源码编译安装
tags: MySQL，源码
category: MySQL数据库
status: draft
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
# make -version
GNU Make 3.81
Copyright (C) 2006  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

```

查看gcc版本：

```
# gcc -v
Using built-in specs.
Target: x86_64-redhat-linux
Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-languages=c,c++,objc,obj-c++,java,fortran,ada --enable-java-awt=gtk --disable-dssi --with-java-home=/usr/lib/jvm/java-1.5.0-gcj-1.5.0.0/jre --enable-libgcj-multifile --enable-java-maintainer-mode --with-ecj-jar=/usr/share/java/eclipse-ecj.jar --disable-libjava-multilib --with-ppl --with-cloog --with-tune=generic --with-arch_32=i686 --build=x86_64-redhat-linux
Thread model: posix
gcc version 4.4.7 20120313 (Red Hat 4.4.7-11) (GCC) 

```

系统没有cmake，需要安装cmake，直接下载cmake官方提供的安装脚本执行安装：

```
# wget http://www.cmake.org/files/v3.3/cmake-3.3.1-Linux-x86_64.sh
# chmod 755 cmake-3.3.1-Linux-x86_64.sh 
# ./cmake-3.3.1-Linux-x86_64.sh 
```
把cmake安装到 `/usr/local/cmake`目录，并配置path环境变量：

```
export PATH="$PATH:/usr/local/cmake/bin"
```

## 下载MySQL5.6源码包 ##





[1]:http://www.cmake.org
[2]:http://www.gnu.org/software/make/
[3]:http://howtolamp.com/lamp/mysql/5.6/installing/
[4]:http://dev.mysql.com/doc/mysql-sourcebuild-excerpt/5.6/en/installing-source-distribution.html




