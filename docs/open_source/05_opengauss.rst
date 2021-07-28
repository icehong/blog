openGauss
=========================

参考 https://zhuanlan.zhihu.com/p/368144566
写一个基于Ubuntu 18.04编译openGauss的说明，看看里面到底有多少坑。
当前基于2021.7.23 master分支编译和安装通过。

基于Ubuntu 20.04 LTS版本编译OpenGauss, 基本环境如下::

    $ lsb_release -a
	No LSB modules are available.
	Distributor ID: Ubuntu
	Description:    Ubuntu 20.04.2 LTS
	Release:        20.04
	Codename:       focal


安装需要的编译工具, 并设置默认的Python版本为Python3::

    ~$ sudo apt install build-essential cmake llvm clang golang autoconf python3-pip zip unzip -y
    ~$ sudo ln -s /usr/bin/python3 /usr/bin/python
    ~$ python --version
    Python 3.8.10


自带gcc版本太高导致不兼容， 安装低的版本 ， 参考  https://blog.csdn.net/ggggyj/article/details/117691948 
sudo apt install gcc-7 g++-7


sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 90 --slave /usr/bin/g++ g++ /usr/bin/g++-9 --slave /usr/bin/gcov gcov /usr/bin/gcov-9
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 70 --slave /usr/bin/g++ g++ /usr/bin/g++-7 --slave /usr/bin/gcov gcov /usr/bin/gcov-7

sudo update-alternatives --config gcc
选 1  
 

安装需要的依赖库(不知道为啥要需要那么多库)::

    sudo apt-get install libstdc++-8-dev libssl-dev rpm2cpio rename pkg-config libkrb5-dev libjsoncpp-dev libedit-dev libpam0g-dev libaio-dev libncurses5-dev libffi-dev libtool pkg-config libkrb5-dev -y


暂时不更新： 
ubuntu@ubuntu:~$ flex --version
flex 2.6.4
ubuntu@ubuntu:~$ bison --version
bison (GNU Bison) 3.5.1



使用ubuntu 18.04 默认版本的 flex 后续编译代码是会碰到链接问题, 手动编译和安装 flex 2.5.39 版本::

    flex 2.5.39版本：
	sudo apt remove flex
    wget https://github.com/westes/flex/releases/download/flex-2.5.39/flex-2.5.39.tar.gz
    tar xzvf flex-2.5.39.tar.gz
    cd flex-2.5.39
    ./configure --bindir=/usr/bin && make && sudo make install


修改默认的sh版本为bash(否则编译cJSON时会有错误)::

     sudo dpkg-reconfigure dash
     在GUI 界面输入 No, 选择bash


需要安装Git LFS扩展才能开始clone代码，否则后面碰到队二进制文件错误，安装扩展参考 https://github.com/git-lfs/git-lfs

curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt install git-lfs

开始下载第三方库代码开始编译(--depth 1 只下载必要的文件,不包含历史)::

    git clone --depth 1 https://gitee.com/opengauss/openGauss-third_party.git

准备工作终于完成，开始编译第三方库::

    ubuntu@ubuntu:~/openGauss-third_party/build$ sudo sh build_all.sh
    --------------------------------openssl-------------------------------------------------
    ar: creating apps/libapps.a
    ar: creating libcrypto.a
    ......


------------------------------fio--------------------------------------------------------
FIO_VERSION = fio-3.8
In file included from crc/../os/os.h:38,
                 from crc/crc32c-arm64.c:2:
crc/../os/os-linux.h:129:19: error: static declaration of ‘gettid’ follows non-static declaration
  129 | static inline int gettid(void)
      |                   ^~~~~~
In file included from /usr/include/unistd.h:1170,
                 from crc/../os/os.h:8,
                 from crc/crc32c-arm64.c:2:
/usr/include/x86_64-linux-gnu/bits/unistd_ext.h:34:16: note: previous declaration of ‘gettid’ was here
   34 | extern __pid_t gettid (void) __THROW;
      |                ^~~~~~
make: *** [Makefile:336: crc/crc32c-arm64.o] Error 1
Traceback (most recent call last):
  File "build.py", line 265, in <module>
    Operator.build_mode()
  File "build.py", line 91, in build_mode
    self.build_all()
  File "build.py", line 116, in build_all
    self.build_component()
  File "build.py", line 150, in build_component
    ret = self.exe_cmd(make_cmd)
  File "build.py", line 235, in exe_cmd
    run_tsk = subprocess.run(cmd, shell = True, check = True)
  File "/usr/lib/python3.8/subprocess.py", line 516, in run
    raise CalledProcessError(retcode, process.args,
subprocess.CalledProcessError: Command 'cd /home/ubuntu/openGauss-third_party/dependency/fio/fio-fio-3.8; make && make install ' returned non-zero exit status 2.


先暂时安装一个自带的试试行不行 
sudo apt install fio


llvm  编译内存不够退出 ， 不编译 llvm 试一下  
sudo bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"






如果一次编译成功， 说明你运气想当好了（严格按照前面说明做了准备），把编译好的输出拷到和主代码同级目录::

    cp -r ~/openGauss-third_party/output ~/binarylibs

    拷贝gcc库到第三方库目录:
    mkdir -p ~/binarylibs/buildtools/ubuntu18.04_x86_64/gcc7.5/gcc/lib64/ 
    cd ~/binarylibs/buildtools/ubuntu18.04_x86_64/gcc7.5/gcc/lib64/ 
    cp /usr/lib/x86_64-linux-gnu/libstdc++.so.6 libstdc++.so.6
    cp /usr/lib/x86_64-linux-gnu/libgcc_s.so.1 libgcc_s.so.1


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


    $ cd openGauss-server
    $ ./configure  --gcc-version=7.5.0 CC=g++ CFLAGS='-O0' --prefix=$GAUSSHOME --3rd=$BINARYLIBS --enable-debug --enable-cassert --enable-thread-safety --without-zlib
    $ make
    最后看见 All of openGauss successfully made. Ready to install.  编译成功
    $ sudo make install 
    ...
    openGauss installation complete.

	打包：
	sh  build.sh -m debug -3rd $BINARYLIBS -pkg


| 编译过程中如果有奇怪的编译错误参考：
| 官方参考: https://gitee.com/opengauss/openGauss-server
| ubuntu编译指导: https://blog.opengauss.org/zh/post/zhengxue/problem_solution/