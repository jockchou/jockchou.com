<!--
author: jockchou
date: 2015-08-24
title: coreseek编译安装
tags: coreseek, Sphinx，搜索引擎
category: Sphinx
status: publish
summary: Coreseek是一款中文全文搜索软件，基于Sphinx研发并独立发布，专攻中文搜索和信息处理领域，适用于垂直搜索、站内搜索、数据库搜索、文献检索、信息检索、数据挖掘等应用场景。
-->

Coreseek是一款中文全文搜索软件，基于Sphinx研发并独立发布，专攻中文搜索和信息处理领域，适用于垂直搜索、站内搜索、数据库搜索、文献检索、信息检索、数据挖掘等应用场景。

## 下载 coreseek4.1 ##

```shell
cd /usr/local/src
wget http://www.coreseek.cn/uploads/csft/4.0/coreseek-4.1-beta.tar.gz
tar -zxvf coreseek-4.1-beta.tar.gz
```


## 安装依赖软件 ##

```shell
yum install make gcc g++ gcc-c++ libtool autoconf automake imake mysql-devel libxml2-devel expat-devel
```


## 安装mmseg分词工具 ##

```shell
cd coreseek-4.1-beta/mmseg-3.2.14/

./bootstrap     #输出的warning信息可以忽略，如果出现error则需要解决
./configure --prefix=/usr/local/mmseg3
make && make install
```


## 安装coreseek ##

```shell
cd coreseek-4.1-beta/csft-4.1/
sh buildconf.sh
./configure --prefix=/usr/local/coreseek  --without-unixodbc --with-mmseg --with-mmseg-includes=/usr/local/mmseg3/include/mmseg/ --with-mmseg-libs=/usr/local/mmseg3/lib/ --with-mysql
make && make install
```


## 测试mmseg分词 ##

```shell
cd testpack/
cat var/test/test.xml    #此时应该正确显示中文
/usr/local/mmseg3/bin/mmseg -d /usr/local/mmseg3/etc var/test/test.xml
/usr/local/coreseek/bin/indexer -c etc/csft.conf --all
/usr/local/coreseek/bin/search -c etc/csft.conf 网络搜索
```


## coreseek核心命令 ##

启动搜索服务:
```shell
/usr/local/coreseek/bin/searchd -c etc/csft.conf
````

停止搜索服务：
```shell
/usr/local/coreseek/bin/searchd -c etc/csft.conf --stop
```

构建索引：
```shell
/usr/local/coreseek/bin/indexer -c etc/csft.conf --all --rotate
```


