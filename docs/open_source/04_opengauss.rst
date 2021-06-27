openGauss
=========================

参考 https://zhuanlan.zhihu.com/p/368144566
计划写一个基于Ubuntu 18编译openGauss的说明，看看里面到底有多少坑。
正在写作中， 未完成！！！

基于Ubuntu 18.04 LTS版本编译OpenGauss, 基本环境如下::

    $ lsb_release -a
    No LSB modules are available.
    Distributor ID: Ubuntu
    Description:    Ubuntu 18.04.5 LTS
    Release:        18.04
    Codename:       bionic

首先安装需要的依赖库::

    sudo apt-get install libssl-dev -y

编译第三方库::

    ~/openGauss-third_party/build$ ./build_all.sh
    --------------------------------openssl-------------------------------------------------
    Traceback (most recent call last):
      File "build.py", line 312, in <module>
        Operator.build_mode()
      File "build.py", line 101, in build_mode
        self.build_all()
      File "build.py", line 126, in build_all
        self.build_component()
      File "build.py", line 153, in build_component
        status, output = subprocess.getstatusoutput(get_cpu_cmd)
    AttributeError: 'module' object has no attribute 'getstatusoutput'

解决方案： sh +x ./build_all.sh 发现是编译openssl出的错误，直接删掉相关编译代码

下载编译好的包：
wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/2.0.0/openGauss-third_party_binarylibs.tar.gz

编译主代码::

    ./configure  CC=g++ CFLAGS='-O0' --prefix=$GAUSSHOME --3rd=$BINARYLIBS --enable-debug --enable-cassert --enable-thread-safety --without-zlib


设置环境变量::

    export CODE_BASE=/home/ubuntu/openGauss-server
    export BINARYLIBS=/home/ubuntu/openGauss-third_party_binarylibs
    export GAUSSHOME=$CODE_BASE/dest/
    export GCC_PATH=$BINARYLIBS/buildtools/ubuntu18.04_x86_64/gcc7.3/
    export CC=/usr/bin/gcc
    export CXX=/usr/bin/g++
    export LD_LIBRARY_PATH=$GAUSSHOME/lib:$GCC_PATH/gcc/lib64:$GCC_PATH/isl/lib:$GCC_PATH/mpc/lib/:$GCC_PATH/mpfr/lib/:$GCC_PATH/gmp/lib/:$LD_LIBRARY_PATH export PATH=$GAUSSHOME/bin:$GCC_PATH/gcc/bin:$PATH


生产配置文件：
~/openGauss-server$ ./configure --gcc-version=7.5.0 CC=g++ CFLAGS='-O0' --prefix=$GAUSSHOME --3rd=$BINARYLIBS --enable-debug --enable-cassert --enable-thread-safety --without-zlib


开始编译，神奇的错误：
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
