# qbittorrent-nox

# 前言
最近玩上了PT，用的最久的客户端是qbittorrent，但是apt安装的qbittorrent是远古版本，非常的无奈。因为新版本的qbittorrent已经到了4.0.3版本（UI有更新），所以很久之前就想方设法自编译了几次，都是因为各种错误（ld liboost、XXXX不是libtorrent的成员等）导致心态爆炸鸽了一次又一次，终于最后成功编译了。

参考的官方wiki（实际上编译失败了）：[点击这里](https://github.com/qbittorrent/qBittorrent/wiki#compilation "点击这里")

# 要点
要成功编译qbittorrent，有几个要点需要被注意

apt安装的支持库大部分都属于远古版本，会导致各种各样的问题，所以尽可能的使用自己编译安装的最新版本。
（这个笔记中的qtToolkit是例外）
编译安装请善用“prefix=”参数，同时也请不要忽略掉编译安装前看一眼“./configure --help”，确保你编译的时候用的是你先前编译的库而不是调用系统的。
无论如何请保持耐心，实在不行，鸽几天再尝试，总有一天会成功的。

# Deb安装包
编译真的真的非常耗时间，如果您对自己没有信心或者想要节约时间，那么您可以尝试我编译出来的deb包

该deb包已囊括所需要的依赖文件，并在纯净的系统环境下通过了测试，deb包分为两种版本，安装后均可直接运行，无需担心依赖关系。

以防万一，如果出现系统自带的库缺失，可以将系统库所需的包安装回来：
```shell
$ sudo aptitude install libqt5network5 libqt5core5a libqt5xml5 libpcre16-3 libdouble-conversion1
```

**包含完整编译出来的boost库，可用于日后编译其他程序**  
[qbittorrent-4.0.3-full.deb](https://download.ultgeek.com/files/deb/qbittorrent-4.0.3-full.deb "qbittorrent-4.0.3-full.deb") - 包大小21.3MB，安装后大约280MB  
[qbittorrent-4.0.3-full-zh.deb](https://download.ultgeek.com/files/deb/qbittorrent-4.0.3-full-zh.deb "qbittorrent-4.0.3-full-zh.deb") - 同上，中文化deb包

**boost库只保留能够让qbittorrent运行起来所需的文件，不能用于当做编译支持库使用**  
[qbittorrent-4.0.3-mini.deb](https://download.ultgeek.com/files/deb/qbittorrent-4.0.3-mini.deb "qbittorrent-4.0.3-mini.deb") - 包大小9.03MB，安装后大约40MB  
[qbittorrent-4.0.3-mini-zh.deb](https://download.ultgeek.com/files/deb/qbittorrent-4.0.3-full-zh.deb "qbittorrent-4.0.3-mini-zh.deb") - 同上，中文化deb包

# 编译准备环境
在qbittorrent官网可得知编译qbittorrent需要“Qt toolkit”以及“libtorrent”共享库。

当然，官网所提及的这两个库以外，我还发现需要“libboost”以及新版本的“openssl”库（以防万一）

# 编译步骤
首先需要下载以下几个包（只要你的包是apt安装来的，不管有没有，请考虑自己编译一份最新的！）

[openSSL](https://www.openssl.org/source/ "openSSL")、[libtorrent-rasterbar](https://github.com/arvidn/libtorrent/releases "libtorrent-rasterbar")、[qt5](https://www.qt.io/download "qt5")、[boost](http://www.boost.org/ "boost")

# 编译安装支持库
## 编译安装openSSL
为了方便阅读，下面编译安装的目录参数统一设置在/opt/目录下（即prefix=/opt/xxxx）  
为了方便阅读，包版本统一用“xxx”来进行表示，请注意编译时替换成现有的版本号，不要生搬硬套（比如openssl-xxxx.tar.gz）  
为了加速编译，在make时后面都加了-j参数，该参数是让GCC编译时使用所有的CPU核心来达到最大化编译速度，但是有可能会出现一定的问题，所以如果遇到问题，请暂停使用。

下载完成后，我们从openssl开始。
```shell
 $ tar -zxvf openssl-xxxx.tar.gz
 $ cd opensslxxxx
 $ ./config --prefix=/opt/openssl-1.1.0g --shared
 $ make clean && make -j
 $ sudo make install
```

## 编译安装Boost
```shell
 $ tar -zxvf boost-xxxx.tar.gz
 $ cd boostxxxx
 $ ./bootstrap.sh
 $ ./b2 --prefix=/opt/boost
 $ sudo ./b2 install
```

## 编译安装qt5
由于qt5编译安装过程复杂而且官网需要注册下载，这里使用的是apt安装（测试可以正常编译），当然有时间的朋友也可以到qt5官网上下载源码包进行安装（需要注意openGL组件问题）
```shell
  $ sudo aptitude install qt5-qmake
  ```
（该包会安装在/usr/lib/x86_64-linux-gnu/qt5目录下，请确保qt5目录下的Bin文件夹中是否有lrelease这个二进制文件，否则将会在后面qbittorrent编译语言时出现command not find的情况，甚至不能通过configure。）

如果没有lrelease，使用该命令安装：
```shell
  $ sudo aptitude install qttools5-dev
```

## 编译安装libtorrent-rasterbar
```shell
 $ tar -zxvf libtorrent-rasterbar.tar.gz
 $ cd libtorrent-rasterbar
 $ ./configure --with-openssl=/opt/openssl-1.1.0g --with-boost-libdir=/opt/boost/lib --with-boost=/opt/boost --enable-encryption --with-libgeoip=system CXXFLAGS=-std=c++11
 $ make clean && make -j
 $ sudo make install
 ```
 
## 编译安装qbittorrent
```shell
 $ tar -zxvf qbittorrent-xxxx.tar.gz
 $ cd qbittorrent-xxxxx
 $ ./configure --disable-gui prefix=/opt/qbittorrent --with-openssl=/opt/openssl-1.1.0g --with-boost-libdir=/opt/boost/lib --with-boost=/opt/boost
 $ make clean && make -j
 $ sudo make install
 ```

至此，编译完成，您可以到/opt/qbittorrent/bin下查找您的qbittorrent二进制文件，如有问题，欢迎开issue或者到[我的网站](https://library.ultgeek.com/index.php/Linux/qbittorrent.html "极客知识库-qbittorrent-nox")下留言。

