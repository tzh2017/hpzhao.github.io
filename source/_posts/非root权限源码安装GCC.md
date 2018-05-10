---
title: 非root权限下源码安装GCC
date: 2016-07-19 10:38:29
tags: Linux
categories: Linux
descriptions: 当我们在服务器下工作时是没有root权限的，因此学会从源码安装软件是必要的技能。本文从源码安装GCC作为示例来讲解一些linux源码安装的一些基本知识。

---
# 预备知识
当我们在服务器下工作时，我们往往是没有root权限的，而软件包的**默认自动安装位置**是/usr/local，因此我们并没有写入的权限。在讲解安装之前我们首先粗浅地了解一下linux的文件系统：  


| 目录 | 描述 |
|------|------|
|/|根目录|
|/bin/|系统可执行文件|
|/boot/|引导文件|
|/dev/|必要设备|
|/etc/|特定主机，系统范围内的配置文件|
|/home/|用户的文件目录|
|/lib/|/bin/ 和 /sbin/中二进制文件必要的库文件|
|/media/|可移除媒体(如CD-ROM)的挂载点|
|/mnt/|临时挂载的文件系统|
|/opt/|可选应用软件包|
|/proc/|虚拟文件系统|
|/root/|超级用户文件目录|
|/sbin/|必要的系统二进制文件|
|/usr/|用户程序目录|
|/var/|变量文件—在正常运行的系统中其内容不断变化的文件，如日志|  

我们再看一下用户目录/usr下的文件：  

| 目录 | 描述 |
|------|------|
|/usr/bin/|非必要可执行文件，面向所有用户。|
|/usr/include/|标准包含文件|
|/usr/lib/|/usr/bin/和/usr/sbin/中二进制文件的库|
|/usr/sbin/|非必要的系统二进制文件，例如：大量网络服务的守护进程|
|/usr/share/|体系结构无关（共享）数据|
|/usr/src/|源代码,例如:内核源代码及其头文件|
|/usr/local/|**用户程序默认安装位置**,里面包含bin等|  

参考文献:[wiki](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E6%A0%87%E5%87%86)

我们可以看出用户程序一般默认安装到/usr/lcoal，但是对于一个多用户的服务器而言，我们一般是没有权限安装到/usr/local的，因此我们需要安装到自己的目录下。我们回到自己的用户目录下`~/`，我们新建`.local`用于我们软件的默认安装位置,因为pip等软件为用户默认安装--user的位置也是.local。

# 准备资源
这一步我们需要准备GCC和其依赖包的源码。  
**步骤一:**首先准备GCC的安装包，我们以GCC-4.8.5为例。我们先从GCC的[官网](https://gcc.gnu.org/)下载源代码。
**步骤二:**解压
```bash
$ tar -zxf gcc-4.8.5.tar.gz

```
**步骤三:**查看gcc依赖包
```bash
$ cd gcc-4.8.5
$ vim download_prerequisites

```

我们会看到有三个依赖包和版本号：*MPFR=mpfr-2.4.2,GMP=gmp-4.3.2,MPC=mpc-0.8.1*
下载地址：
[ftp://ftp.gnu.org/gnu/gmp/gmp-4.3.2.tar.bz2](ftp://ftp.gnu.org/gnu/gmp/gmp-4.3.2.tar.bz2)
[http://www.mpfr.org/mpfr-2.4.2/mpfr-2.4.2.tar.bz2](http://www.mpfr.org/mpfr-2.4.2/mpfr-2.4.2.tar.bz2)
[http://www.multiprecision.org/mpc/download/mpc-0.8.1.tar.gz](http://www.multiprecision.org/mpc/download/mpc-0.8.1.tar.gz)
我们把这三个依赖包拷贝到gcc源代码目录一块编译
```bash
$ tar jxf gmp-4.3.2.tar.bz2
$ tar jxf mpfr-2.4.2.tar.bz2
$ tar zxf mpc-0.8.1.tar.gz
$ mv gmp-4.3.2 gcc-4.8.1/gmp
$ mv mpfr-2.4.2 gcc-4.8.1/mpfr
$ mv mpc-0.8.1 gcc-4.8.1/mpc

```
**注意:**拷贝之后的文件名一定要和上面一致，不然编译时会找不到路径。
# 编译安装
这一步我们需要编译gcc源文件了，我们首先进入源代码目录`gcc-4.8.5`，然后执行下面的命令：
```bash
#生成MakeFile，换成立自己的绝对路径
$ ./configure --prefix=/home/simon/.local --enable-checking=release --enable-languages=c,c++ --disable-multilib
#编译，开启4个线程
$　make -j4
#拷贝到.local目录
$ make install

```
# 后续操作

我们安装完之后怎么在终端直接用呢？有以下两种方法。
## 配置环境变量

将编译后的可执行文件的路径加入到环境变量中有三种方法：
1. 终端export，这种方法立即生效
```bash
$ export PATH=/home/simon/.local/bin:$PATH

```
2. 修改系统文件/etc/profile，这种方法是为所有用户设置的，需要重启或source /etc/profile生效。
```bash
export PATH=/home/simon/.local/bin:$PATH

```
3. 修改用户文件.bashrc或.bashrc_profile，这是为当前用户设定，需要重启或source .bash_profile
```bash
export PATH=/home/simon/.local/bin:$PATH

```

配置完环境变量我们就能在终端使用gcc,g++了。

## 添加链接

如果我有多个版本的gcc，想同时使用这些怎么办。那么我们只需要设置链接即可。

```bash
$ ln -s /home/simon/.local/bin/gcc gcc485
$ ln -s /home/simon/.local/bin/g++ g++485

```
那么我们就能够在终端使用gcc485和其他版本的gcc了。

小结：个人感觉如果在非root权限下，操作一些东西可能更麻烦，但只有这样才能学到更多东西，才会去了解比较底层的东西，而在root权限下，一切变得很傻瓜式的了。