# install svn in CentOS 6.3

##Background
---
Currently I need to install svn in CentOS from source code. In this tutorial, I will present all the steps achieve this.

### System Environment

* OS: Centos6.4 x86
* SVN: subversion-1.8.13
* Apache: httpd-2.4.6
如开启防火墙,则需添加如下列的规则以放行svn的3690端口

iptables -A INPUT -p tcp --dport 3690 -j ACCEPT
iptables save

**Check Installed SVN**

	[yubin@mongodb20 ~]$ rpm -qa | grep subversion	subversion-1.6.11-7.el6.x86_6

**Uninstall old SVN**

	yum remove subversion


**Download, Compile, Install Steps**


**Compile and install APR/APR-UTIL**

	//download and decompress apr-1.4.8、apr-util-1.5.2
	wget http://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-1.5.1.tar.gz
	wget http://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-util-1.5.4.tar.gz
	tar -zxf apr-1.5.1.tar.gz
	tar -zxf apr-util-1.5.4.tar.gz

**Compile and install httpd-2.4.6**

	wget http://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-2.4.12.tar.gz
	tar -zxf httpd-2.4.12.tar.gz

	// move apr-1.5.1、apr-util-1.5.4 to srclib of httpd-2.4.12
	mv apr-1.5.1 httpd-2.4.12/srclib/apr
	mv apr-util-1.5.4 httpd-2.4.12/srclib/apr-util

	// compile httpd-2.4.12

	cd httpd-2.4.12 && ./configure --prefix=/home/yubin/install/apache --enable-so --enable-dav --enable-deflate=shared --enable-ssl=shared --enable-expires=shared  --enable-headers=shared --enable-rewrite=shared --enable-static-support  --with-included-apr --enable-modules=all --enable-mods-shared=all --with-mpm=prefork
	make && make install

再次提示缺少pcre, wget 'http://sourceforge.net/projects/pcre/files/pcre/8.36/pcre-8.36.tar.gz/download' pcre-8.36.tar.gz 编译 安装(我这里均安装在/home/yubin/install目录里，例如./configure --prefix=/home/yubin/install/pcre)
--with-pcre=/home/yubin/install/pcre

**Compile and install Serf**
Install serf-1.2.1
	yum -y install expat-devel
	wget http://serf.googlecode.com/files/serf-1.2.1.tar.bz2 #Serf-1.2.1.zip is a win version of a problem
	tar xjf serf-1.2.1.tar.bz2
	cd serf-1.2.1
	./configure --prefix=/home/yubin/install/serf --with-apr=/home/yubin/install/apache --with-apr-util=/home/yubin/install/apache
	make && make install
	cd ..
	
install serf-1.3.8

sed -i "/Append/s:RPATH=libdir,::"   SConstruct &&
sed -i "/Default/s:lib_static,::"    SConstruct &&
sed -i "/Alias/s:install_static,::"  SConstruct &&
scons PREFIX=/home/yubin/install/serf APR=/home/yubin/install/apache APU=/home/yubin/install/apache install




**Compile and install subversion-1.8.13**

1) method 1 (recommended)
	
	// 可以只解压sqlite-amalgamation包，将解压文件拷贝到subversion解压后的源码安装目录中去。
    官方解释：This ZIP archive contains all C source code for SQLite 3.8.1 combined into a single source file.
    	# A.解压 sqlite-amalgamation 源码包
    	wget 'https://www.sqlite.org/2015/sqlite-amalgamation-3080900.zip'
    	unzip sqlite-amalgamation-3080900.zip

        # B.解压 subversion 软件包：
        tar -zxvf subversion-1.8.13.tar.gz

        #C.拷贝 sqlite-amalgamation 源码包到 subversion 解压包中，并更名为sqlite-amalgamation
              注：假定sqlite包与subversion包是解压在同级目录的。
        cp -r ./sqlite-amalgamation-3080900 ./subversion-1.8.13/sqlite-amalgamation

        D.进入subversion解压目录，配置编译
        cd subversion-1.8.13
        ./configure --prefix=/home/yubin/install/subversion --with-apr=/home/yubin/install/apache --with-apr-util=/home/yubin/install/apache --with-openssl --with-serf=/home/yubin/install/serf
        make && make install
            
2) method 2

	// compile and install sqlite3.8.19
	方案二：安装Sqlite数据库完整软件包，在编译subversion时加上 --with-sqlite=/usr/local/sqlite/ 选项
    官方解释：A tarball containing the amalgamation for SQLite 3.8.19 together with an configure script and makefile for building it. 

	wget http://www.sqlite.org/2013/sqlite-autoconf-3071700.tar.gz
	tar -zxf sqlite-autoconf-3071700.tar.gz
	cd sqlite-autoconf-3071700
	./configure
	make && make install

	// download snv source code and install
	wget 'http://mirrors.tuna.tsinghua.edu.cn/apache/subversion/subversion-1.8.13.tar.gz'
	tar -zxf subversion-1.8.13.tar.gz
	cd subversion-1.8.13
	./configure --prefix=/home/yubin/install/subversion --with-apr=/home/yubin/install/apache --with-apr-util=/home/yubin/install/apache --with-openssl --with-serf=/home/yubin/install/serf --with-sqlite=/home/yubin/install/sqlite
	make && make install 
	
	Options
        --prefix=/usr/local/subversion/             指定subversion要安装的目录
        --with-apr=/usr/local/apr-httpd/            指定apr对应安装目录
        --with-apr-util=/usr/local/apr-util-httpd/  指定apr-util对应安装目录
        --with-zlib=/usr/local/zlib/                指定zlib对应安装目录
        --with-sqlite=/usr/local/sqlite/            指定sqlite对应安装目录
        --enable-maintainer-mode                    启用调试提醒模式
	
**Check SVN Valid**

	svnserve --version
	// response 

	svnserve, version 1.8.0 (r1490375)
		compiled Jul 23 2013, 21:32:09 on i686-pc-linux-gnu
		Copyright (C) 2013 The Apache Software Foundation.
		This software consists of contributions made by many people;
		see the NOTICE file for more information.
		Subversion is open source software, see http://subversion.apache.org/

		The following repository back-end (FS) modules are available:
		* fs_fs : Module for working with a plain file (FSFS) repository.
		Cyrus SASL authentication is available.
		
## Q&A
1. svn: error while loading shared libraries: libserf-1.so.1: cannot open shared object file: No such file or directory
 
	http://stackoverflow.com/questions/28663316/svn-error-while-loading-shared-libraries-libserf-1-so-1-cannot-open-shared-ob
export LD_LIBRARY_PATH="/home/yubin/install/serf/lib:$LD_LIBRARY_PATH"


