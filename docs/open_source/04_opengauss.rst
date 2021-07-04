openGauss
=========================

参考 https://zhuanlan.zhihu.com/p/368144566
计划写一个基于Ubuntu 18编译openGauss的说明，看看里面到底有多少坑。
当前基于2021.7.3 master分支， 正在写作中， 未完成！！！

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
    sudo make && sudo make install
    cmake  --version

需升级安装 bison3.5 的版本，否则最后会出现链接错误::

    wget http://ftp.gnu.org/gnu/bison/bison-3.5.4.tar.gz
    cd bison-3.5.4/
    ./configure && make && make install

Python需要包含以下依赖库::

    pip install setuptools

修改默认的sh版本为bash(否则编译cJSON时会有错误)::

     sudo dpkg-reconfigure dash
     在GUI 界面输入 No, 选择bash

安装需要的依赖库::

    sudo apt-get install libssl-dev rpm2cpio rename pkg-config  libkrb5-dev libjsoncpp-dev flex bison  libedit-dev libpam0g-dev libaio-dev libncurses5-dev libffi-dev libtool pkg-config libkrb5-dev -y

代码库上下载的libxml2的包似乎是错的，重新下载一个才好::

    cd openGauss-third_party/dependency/libxml2
    rm libxml2-2.9.9.tar.gz
    wget http://xmlsoft.org/sources/libxml2-2.9.9.tar.gz
    包下载好放在这里就好，后面脚本自动编译


编译第三方库::

    ubuntu@ubuntu:~/openGauss-third_party/build$ sudo sh build_all.sh
    --------------------------------openssl-------------------------------------------------
    ar: creating apps/libapps.a
    ar: creating libcrypto.a
    ......


解决方案： TODO



编译主代码::

直接编译::

    sh build.sh -m debug -3rd /home/ubuntu/binarylibs/


手动编译::

设置环境变量::

    export CODE_BASE=/home/ubuntu/openGauss-server
    export BINARYLIBS=/home/ubuntu/binarylibs
    export GAUSSHOME=$CODE_BASE/dest/
    export CC=/usr/bin/gcc
    export CXX=/usr/bin/g++
    export LD_LIBRARY_PATH=$GAUSSHOME/lib:$GCC_PATH/gcc/lib64:$GCC_PATH/isl/lib:$GCC_PATH/mpc/lib/:$GCC_PATH/mpfr/lib/:$GCC_PATH/gmp/lib/:$LD_LIBRARY_PATH
    export PATH=$GAUSSHOME/bin:$GCC_PATH/gcc/bin:$PATH


    $ ./configure  CC=g++ CFLAGS='-O0' --prefix=$GAUSSHOME --3rd=$BINARYLIBS --enable-debug --enable-cassert --enable-thread-safety --without-zlib
    $ make


    /home/ubuntu/openGauss-server/src/common/backend/parser/parser.cpp:134: undefined reference to `core_yylex(core_YYSTYPE*, int*, void*)'
    /home/ubuntu/openGauss-server/src/common/backend/parser/parser.cpp:143: undefined reference to `core_yylex(core_YYSTYPE*, int*, void*)'
    /home/ubuntu/openGauss-server/src/common/backend/parser/parser.cpp:166: undefined reference to `core_yylex(core_YYSTYPE*, int*, void*)'
    /home/ubuntu/openGauss-server/src/common/backend/parser/parser.cpp:187: undefined reference to `core_yylex(core_YYSTYPE*, int*, void*)'
    /home/ubuntu/openGauss-server/src/common/backend/parser/parser.cpp:209: undefined reference to `core_yylex(core_YYSTYPE*, int*, void*)'










有奇怪的编译错误，可以参考：
https://blog.opengauss.org/zh/post/zhengxue/problem_solution/