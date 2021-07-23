openGauss
=========================

参考 https://zhuanlan.zhihu.com/p/368144566
写一个基于Ubuntu 18.04编译openGauss的说明，看看里面到底有多少坑。
当前基于2021.7.23 master分支编译和安装通过。

基于Ubuntu 18.04 LTS版本编译OpenGauss, 基本环境如下::

    $ lsb_release -a
    No LSB modules are available.
    Distributor ID: Ubuntu
    Description:    Ubuntu 18.04.5 LTS
    Release:        18.04
    Codename:       bionic


安装需要的编译工具, 并设置默认的Python版本为Python3::

    ~$ sudo apt install build-essential llvm clang golang autoconf python3-pip libstdc++-8-dev -y
    ~$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 150
    ~$ python --version
    Python 3.6.9

Ubuntu 18.04 默认带的 cmake 版本比较低，需要升级到3.16版本以上后续编译parquet包才不会失败::

    sudo apt remove cmake
    wget https://github.com/Kitware/CMake/releases/download/v3.16.3/cmake-3.16.3.tar.gz
    tar xzvf cmake-3.16.3.tar.gz
    cd cmake-3.16.3
    sudo ./bootstrap
    make && sudo make install
    cmake  --version

安装需要的依赖库(不知道为啥要需要那么多库)::

    sudo apt-get install libssl-dev rpm2cpio rename pkg-config libkrb5-dev libjsoncpp-dev libedit-dev libpam0g-dev libaio-dev libncurses5-dev libffi-dev libtool pkg-config libkrb5-dev -y

使用ubuntu 18.04 默认版本的 flex和bison 后续编译代码是会碰到链接问题, 手动编译和安装 flex 2.5.39 和 bison bison-3.5.4 版本::

    flex 2.5.39版本： 
    wget https://github.com/westes/flex/releases/download/flex-2.5.39/flex-2.5.39.tar.gz
    tar xzvf flex-2.5.39.tar.gz
    cd flex-2.5.39
    ./configure && make && make install
    sudo ln -s /usr/local/bin/flex /usr/bin/flex
    安装 bison3.5 的版本:
    wget http://ftp.gnu.org/gnu/bison/bison-3.5.4.tar.gz
    cd bison-3.5.4/
    ./configure && make && make install

Python需要包含以下依赖库::

    pip install setuptools

修改默认的sh版本为bash(否则编译cJSON时会有错误)::

     sudo dpkg-reconfigure dash
     在GUI 界面输入 No, 选择bash

需要安装Git LFS扩展才能开始clone代码，否则后面碰到队二进制文件错误，安装扩展参考 https://github.com/git-lfs/git-lfs
开始下载第三方库代码开始编译(--depth 1 只下载必要的文件,不包含历史)::

    git clone --depth 1 https://gitee.com/opengauss/openGauss-third_party.git

准备工作终于完成，开始编译第三方库::

    ubuntu@ubuntu:~/openGauss-third_party/build$ sudo sh build_all.sh
    --------------------------------openssl-------------------------------------------------
    ar: creating apps/libapps.a
    ar: creating libcrypto.a
    ......

如果一次编译成功， 说明你运气想当好了（严格按照前面说明做了准备），把编译好的输出拷到和主代码同级目录::

    cp -r ~/openGauss-third_party/output ~/binarylibs
    拷贝gcc库到第三方库目录:
    mkdir -p ~/binarylibs/buildtools/ubuntu18.04_x86_64/gcc7.5/gcc/lib64/ 
    cd ~/binarylibs/buildtools/ubuntu18.04_x86_64/gcc7.5/gcc/lib64/ 
    cp /usr/lib/x86_64-linux-gnu/libstdc++.so.6 ./
    cp /usr/lib/gcc/x86_64-linux-gnu/7/libgcc_s.so.1 ./



开始编译主代码::

    $ cd ~
    $ git clone https://gitee.com/opengauss/openGauss-server.git

设置环境变量::

    export CODE_BASE=~/openGauss-server
    export BINARYLIBS=~/binarylibs
    export GAUSSHOME=$CODE_BASE/dest/
    export GCC_PATH=$BINARYLIBS/buildtools/ubuntu18.04_x86_64/gcc7.5/
    export CC=/usr/bin/gcc
    export CXX=/usr/bin/g++
    export LD_LIBRARY_PATH=$GAUSSHOME/lib:$GCC_PATH/gcc/lib64:$GCC_PATH/isl/lib:$GCC_PATH/mpc/lib/:$GCC_PATH/mpfr/lib/:$GCC_PATH/gmp/lib/:$LD_LIBRARY_PATH
    export PATH=$GAUSSHOME/bin:$GCC_PATH/gcc/bin:$PATH

修改gcc 版本为本地版本，我是手动吧 configure 文件里 gcc_version='7.3.0' 改成了本地的 7.5.0 ::

    $ cd openGauss-server
    $ ./configure  CC=g++ CFLAGS='-O0' --prefix=$GAUSSHOME --3rd=$BINARYLIBS --enable-debug --enable-cassert --enable-thread-safety --without-zlib
    $ make
    最后看见 All of openGauss successfully made. Ready to install.  编译成功
    $ sudo make install 
    ...
    openGauss installation complete.

| 编译过程中如果有奇怪的编译错误参考：
| 官方参考: https://gitee.com/opengauss/openGauss-server
| ubuntu编译指导: https://blog.opengauss.org/zh/post/zhengxue/problem_solution/