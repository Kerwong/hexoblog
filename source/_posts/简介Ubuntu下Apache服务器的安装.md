---
title: 简介Ubuntu下Apache服务器的安装
date: 2015-10-16 21:35:17
tags:
- Apache
---
本日志主要记录Apache 在Ubuntu 下的安装
## 方法一：Apt安装
apt-get
```
sudo apt-get install apache2
```
完成后，通过修改
```
/etc/apache2/apache2.conf
```
来完成配置
## 方法二：使用Apache源代码编译安装
### 下载Apache 的源代码
从Apache 官网或其他可用源处下载Apache 的源代码
> http://httpd.apache.org/download.cgi
> http://apache.etoak.com//httpd/httpd-2.4.3.tar.gz

此链接为Apache 项目的官网，可获得Apache 服务器的多种版本，包括稳定版和 Beta 版等。此次选择下载了`httpd-2.4.3.tar.gz` 文件。
### 解压文件
下载完毕后得到`httpd-2.4.3.tar.gz` 文件，下一步需要对其进行解压。Linux 解压命令功能较为强大，可以直接选择解压至安装路径也可以先解压至指定路径，之后再复制到安装路径。
直接安装, 输入命令：
```
sudo tar zxvf httpd-2.4.3.tar.gz -C /usr/local/    //将Apache源文件解压至/usr/local/目录下
```
Tar为归档文件压缩和解压指令
> -z 是配合解压.GZ的
> -x 解开一个包文件
> -v 显示详细信息
> -f 必须，表示使用归档文件

输出：
```
…
httpd-2.4.3/docs/manual/howto/htaccess.html.ja.utf8
httpd-2.4.3/docs/manual/howto/htaccess.html.ko.euc-kr
httpd-2.4.3/docs/manual/howto/htaccess.html.pt-br
httpd-2.4.3/docs/manual/howto/index.html
……
httpd-2.4.3/docs/manual/howto/ssi.html.en
httpd-2.4.3/docs/manual/howto/ssi.html.fr
httpd-2.4.3/docs/manual/howto/ssi.html.ja.utf8
httpd-2.4.3/docs/manual/howto/ssi.html.ko.euc-kr
httpd-2.4.3/docs/manual/faq/index.html
…
```
切换至`/usr/local/httpd-2.4.3` 目录下查看。
输入指令：
```shell
cd /usr/local/httpd-2.4.3
ls
```
输出：
```
wang@Wang-Satellite-M300:/usr/local/httpd-2.4.3$ ls -l
total 1568
-rw-r--r-- 1 501 staff 13507 Mar 29 2011 ABOUT_APACHE
-rw-r--r-- 1 501 staff 22850 Jul 23 23:20 acinclude.m4
-rw-r--r-- 1 501 staff 63038 Jan 31 2012 Apache-apr2.dsw
-rw-r--r-- 1 501 staff 77169 Nov 19 2011 Apache.dsw
-rw-r--r-- 1 501 staff 9907 Dec 18 2009 apache_probes.d
-rw-r--r-- 1 501 staff 2512 Dec 22 2008 ap.d
drwxr-xr-x 6 501 staff 4096 Nov 22 20:56 build
-rw-r--r-- 1 501 staff 2644 Aug 24 2007 BuildAll.dsp
-rw-r--r-- 1 501 staff 2724 Nov 12 2011 BuildBin.dsp
-rwxr-xr-x 1 501 staff 6791 Jan 17 2011 buildconf
-rw-r--r-- 1 501 staff 118884 Aug 18 01:05 CHANGES
-rw-r--r-- 1 501 staff 12567 Apr 17 2012 config.layout
-rwxr-xr-x 1 501 staff 956655 Aug 18 01:20 configure
-rw-r--r-- 1 501 staff 27680 Jul 23 23:20 configure.in
drwxr-xr-x 9 501 staff 4096 Nov 22 20:56 docs
-rw-r--r-- 1 501 staff 403 Nov 22 2004 emacs-style
-rw-r--r-- 1 501 staff 4124 Jun 12 2008 httpd.dsp
-rw-r--r-- 1 501 staff 17556 Aug 18 01:20 httpd.spec
drwxr-xr-x 2 501 staff 4096 Nov 22 20:56 include
-rw-r--r-- 1 501 staff 5083 Aug 16 20:42 INSTALL
-rw-r--r-- 1 501 staff 2909 Nov 15 2011 InstallBin.dsp
-rw-r--r-- 1 501 staff 4142 Dec 16 2010 LAYOUT
-rw-r--r-- 1 501 staff 20486 Jan 31 2012 libhttpd.dsp
-rw-r--r-- 1 501 staff 25852 Jul 24 2011 LICENSE
-rw-r--r-- 1 501 staff 9532 Jan 23 2012 Makefile.in
-rw-r--r-- 1 501 staff 46658 Apr 21 2012 Makefile.win
drwxr-xr-x 26 501 staff 4096 Nov 22 20:56 modules
-rw-r--r-- 1 501 staff 550 Jul 19 14:48 NOTICE
-rw-r--r-- 1 501 staff 13681 Mar 16 2012 NWGNUmakefile
drwxr-xr-x 7 501 staff 4096 Nov 22 20:56 os
-rw-r--r-- 1 501 staff 5158 Feb 20 2012 README
-rw-r--r-- 1 501 staff 5572 Apr 23 2010 README.platforms
-rw-r--r-- 1 501 staff 10184 Oct 31 2010 ROADMAP
drwxr-xr-x 3 501 staff 4096 Nov 22 20:56 server
drwxr-xr-x 2 501 staff 4096 Nov 22 20:56 srclib
drwxr-xr-x 4 501 staff 4096 Nov 22 20:56 support
drwxr-xr-x 2 501 staff 4096 Nov 22 20:56 test
-rw-r--r-- 1 501 staff 8183 Mar 1 2007 VERSIONING
```
在开始编译 Apache 源文件时需要配置一下，主要是设定编译好的文件的路径。
但是，在第一次配置中出现了错误，系统提示需要 `apr` 和 `apr-util` 两部分，因此，先配置安装这两个部分。
### 安装 apr 和 apr-util 并配置
**从Apache 官网下载apr 和apr-util 两个文件**
以下为官网推荐的镜像下载地址， 
> apr [下载地址][1]
> apr-util [下载地址][2]

输入指令：
```
sudo tar zxvf apr-1.4.6.tar.gz -C /usr/local/
sudo tar zxvf apr-util-1.5.1.tar.gz -C /usr/local/
```
**配置 `apr` 和 `apr-util` 两个文件**
首先进入到 `apr` 解压后的目录下，用`cd`命令。
输入指令：
```
./configure –prefix=/usr/local/apr    //意思是将编译后的apr文件的顶层路径设为/usr/local/apr
```
然后开始编译`apr`，编译过程同其他编译。
输入：
```
make
make install
```
然后是`apr-util`的配置和编译。
`apr-util` 的配置和 `apr` 配置大同小异，但由于其对 `apr` 有依赖，因此需要比 `apr` 多了一项配置要求。
首先进入 `apr-util` 解压至的目录下
输入：
```
sudo ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr-1.4.6 > ~/apr-util.txt
```
然后开始编译
输入：
```
make
make install
```
*注意：有时有的系统还需要 `pcre`，[下载地址][3]* 
下载后的编译和安装都差不多，不再赘述。这里我直接用 `apt` 安装了 `pcre`
输入：
```
sudo apt-get install libpcre3-dev
```
### 配置apache 
进入到解压后的 `httpd-2.4.3` 目录下。配置
输入：
```
sudo ./configure --prefix=/usr/local/apache2 --with-apr=/usr/local/apr-1.4.6 --with-apr-util=/usr/local/apr-util-1.5.1 --enable-so > ~/apache2.txt
```
这里是最基本的 Apache 配置，除此之外还可以配置模块和其他细节。Apache 在该配置下默认为最小安装。更多 configure ，可以参考 [http://apache.jz123.cn/programs/configure.html][4]
然后是 `make` 和 `make install`。安装完毕。
进入到 bin 目录，执行`./apachectl start`，如果一切安装顺利，可以看到：**It works**。
对于原来的`apr-1.4.6`，`apr-util-1.5.1` 和`httpd-xx` 等文件夹可以视需要来决定是否删去。

  [1]: http://mirror.bjtu.edu.cn/apache//apr/apr-1.4.6.tar.gz
  [2]: http://mirror.bjtu.edu.cn/apache//apr/apr-util-1.5.1.tar.gz
  [3]: http://pcre.org/
  [4]: http://apache.jz123.cn/programs/configure.html
