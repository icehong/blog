openGauss
=========================

参考 https://zhuanlan.zhihu.com/p/368144566
计划写一个基于Ubuntu 18编译openGauss的说明，看看里面到底有多少坑。
当前基于2021.7.3 master分支， 正在写作中， 未完成！！！
llvm 暂时没有编译过

基于Ubuntu 18.04 LTS版本编译OpenGauss, 基本环境如下::

    $ lsb_release -a
    No LSB modules are available.
    Distributor ID: Ubuntu
    Description:    Ubuntu 18.04.5 LTS
    Release:        18.04
    Codename:       bionic


安装需要的编译工具, 并设置默认的Python版本为Python3::

    ~$ sudo apt install build-essential llvm clang golang autoconf python3-pip -y
    ~$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 150
    ~$ python --version
    Python 3.6.9

Ubuntu 18.04 默认带的 cmake 版本比较低，需要升级到3.16版本以上后续编译parquet包才不会失败::

    sudo apt remove cmake
    wget https://github.com/Kitware/CMake/releases/download/v3.16.3/cmake-3.16.3.tar.gz
    tar xzvf cmake-3.16.3.tar.gz
    cd cmake-3.16.3
    sudo ./bootstrap
    sudo make
    sudo make install
    cmake  --version

Python需要包含以下依赖库::

    pip install setuptools

修改默认的sh版本为bash(否则编译cJSON时会有错误)::

     $ sudo dpkg-reconfigure dash
     在GUI 界面输入 No, 选择bash

安装需要的依赖库::

    sudo apt-get install libssl-dev rpm2cpio rename pkg-config  libkrb5-dev libjsoncpp-dev flex bison  libedit-dev libpam0g-dev libaio-dev libncurses5-dev libffi-dev libtool pkg-config libkrb5-dev sudo apt-get install -y


编译第三方库::

    ubuntu@ubuntu:~/openGauss-third_party/build$ ./build_all.sh
    --------------------------------openssl-------------------------------------------------
    ar: creating apps/libapps.a
    ar: creating libcrypto.a
    ......


解决方案： TODO




编译主代码::

    ./configure  CC=g++ CFLAGS='-O0' --prefix=$GAUSSHOME --3rd=$BINARYLIBS --enable-debug --enable-cassert --enable-thread-safety --without-zlib


设置环境变量::

    export CODE_BASE=/home/ubuntu/openGauss-server
    export BINARYLIBS=/home/ubuntu/binarylibs
    export GAUSSHOME=$CODE_BASE/dest/
    export GCC_PATH=$BINARYLIBS/buildtools/ubuntu18.04_x86_64/gcc7.3/
    export CC=/usr/bin/gcc
    export CXX=/usr/bin/g++
    export LD_LIBRARY_PATH=$GAUSSHOME/lib:$GCC_PATH/gcc/lib64:$GCC_PATH/isl/lib:$GCC_PATH/mpc/lib/:$GCC_PATH/mpfr/lib/:$GCC_PATH/gmp/lib/:$LD_LIBRARY_PATH export PATH=$GAUSSHOME/bin:$GCC_PATH/gcc/bin:$PATH


生产配置文件::

    ~/openGauss-server$ ./configure --gcc-version=7.5.0 CC=g++ CFLAGS='-O0' --prefix=$GAUSSHOME --3rd=$BINARYLIBS --enable-debug --enable-cassert --enable-thread-safety --without-zlib


开始编译，神奇的错误::

    ubuntu@ubuntu:~/openGauss-server$ make -sj
    make[6]: /home/ubuntu/openGauss-third_party_binarylibs/dependency/ubuntu18.04_x86_64/llvm/comm/bin/llvm-config: Command not found
    make[6]: /home/ubuntu/openGauss-third_party_binarylibs/dependency/ubuntu18.04_x86_64/llvm/comm/bin/llvm-config: Command not found
    In file included from ../../../src/include/foreign/foreign.h:16:0,
                     from ../../../src/include/nodes/plannodes.h:18,
                     from ../../../src/include/workload/workload.h:34,
                     from ../../../src/include/access/gtm.h:28,
                     from gs_thread.cpp:37:
    ../../../src/include/access/obs/obs_am.h:33:10: fatal error: eSDKOBS.h: No such file or directory
     #include "eSDKOBS.h"
              ^~~~~~~~~~~


有奇怪的编译错误，可以参考：
https://blog.opengauss.org/zh/post/zhengxue/problem_solution/