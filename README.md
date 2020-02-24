# The LLVM Compiler Infrastructure

This directory and its subdirectories contain source code for LLVM,
a toolkit for the construction of highly optimized compilers,
optimizers, and runtime environments.

The README briefly describes how to get started with building LLVM.
For more information on how to contribute to the LLVM project, please
take a look at the
[Contributing to LLVM](https://llvm.org/docs/Contributing.html) guide.

# 目录介绍



# 编译 LLVM

    brew install gcc
    brew install cmake
    mkdir build
    cd build
    cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi;clang-tools-extra" -DBUILD_SHARED_LIBS=ON -DLLVM_TARGETS_TO_BUILD="AArch64;X86" -DCMAKE_INSTALL_PREFIX=/usr/local -G "Unix Makefiles" ../llvm
    make j8
    #安装
    make install

备注，上述 DLLVM_ENABLE_PROJECTS 指定了需要安装的 附件库


> LLVM与编译相关的标准库：

**opt：

该工具的目标是针对IR阶段的程序进行优化，其输入文件必须是LLVM的Bitcode，输出为相同格式的IR文件

**llc：

将LLVM的IR文件转换成设备相关的汇编语言文件或Obj文件。您可以指定优化等级、Debug使能、是否针对目标平台优化。

**llvm-mc：

将汇编代码生成为指定格式的OBJ文件，如：ELF文件、MachO文件、PE文件等。也可反汇编相应的OBJ文件

**lli：

以解释方式或JIT运行LLVM

IR文件llvm-link：

将几个LLVM Bitcode整合为单一一个LLVM Bitcode，注意却别于我们在编译时实用的通常的Link文件，如Linux下ld
 
**llvm-as：
 
 将LLVM中人工可识别IR文件（ll）转换为LLVMBitcode（BC）文件
 
**llvm-dis：将LLVM Bitcode（BC）转换成人可阅读的IR文件（LL）

> LLVM/Clang的基础库：

**libLLVMCore：

LLVM核心库，主要是：

1）LLVM IR指令构造、检查。

2）Pass管理lib

**LLVMAnalysis：

包含了IR相关的几个分析Pass：

如对其别名分析、相关性分析、常量折叠（Constant folding）、循环信息、内存依赖分析、指令化简等，详细见目录：lib\Analysis
 
**libLLVMCodeGen：

目标无关的代码生成、低级LLVM IR（机器相关）分析和转换，见相关目录：lib\CodeGenlib

**LLVMTarget：

由通用目标及抽象提供对目标机的访问。这些通用的抽象提供了libLLVMCodeGen中通用的后端算法和目标相关逻辑间的通讯途径。详细目录：lib\Target，其中各目录包含了已经支持的各平台libLLVMxxxxCodeGen（xxxx表示具体支持的平台，如X86等）：具体平台相关的代码生成、转换和分析Pass，其是相关平台的后端。

**libLLVMSupport：

LLVM的支持库：错误处理、整形和浮点处理、命令行解析、调试、文件支持、字符串操作等；所在目录：lib\Support

**libclangxxxx（各clang前端功能，如AST、词法分析、语法分析）：

提供了访问Clang前端各功能的接口。可以用于任何语言中，如Python中



## Getting Started with the LLVM System

Taken from https://llvm.org/docs/GettingStarted.html.

### Overview

Welcome to the LLVM project!

The LLVM project has multiple components. The core of the project is
itself called "LLVM". This contains all of the tools, libraries, and header
files needed to process intermediate representations and converts it into
object files.  Tools include an assembler, disassembler, bitcode analyzer, and
bitcode optimizer.  It also contains basic regression tests.

C-like languages use the [Clang](http://clang.llvm.org/) front end.  This
component compiles C, C++, Objective C, and Objective C++ code into LLVM bitcode
-- and from there into object files, using LLVM.

Other components include:
the [libc++ C++ standard library](https://libcxx.llvm.org),
the [LLD linker](https://lld.llvm.org), and more.

### Getting the Source Code and Building LLVM

The LLVM Getting Started documentation may be out of date.  The [Clang
Getting Started](http://clang.llvm.org/get_started.html) page might have more
accurate information.

This is an example workflow and configuration to get and build the LLVM source:

1. Checkout LLVM (including related subprojects like Clang):

     * ``git clone https://github.com/llvm/llvm-project.git``

     * Or, on windows, ``git clone --config core.autocrlf=false
    https://github.com/llvm/llvm-project.git``

2. Configure and build LLVM and Clang:

     * ``cd llvm-project``

     * ``mkdir build``

     * ``cd build``

     * ``cmake -G <generator> [options] ../llvm``

        Some common generators are:

        * ``Ninja`` --- for generating [Ninja](https://ninja-build.org)
          build files. Most llvm developers use Ninja.
        * ``Unix Makefiles`` --- for generating make-compatible parallel makefiles.
        * ``Visual Studio`` --- for generating Visual Studio projects and
          solutions.
        * ``Xcode`` --- for generating Xcode projects.

        Some Common options:

        * ``-DLLVM_ENABLE_PROJECTS='...'`` --- semicolon-separated list of the LLVM
          subprojects you'd like to additionally build. Can include any of: clang,
          clang-tools-extra, libcxx, libcxxabi, libunwind, lldb, compiler-rt, lld,
          polly, or debuginfo-tests.

          For example, to build LLVM, Clang, libcxx, and libcxxabi, use
          ``-DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi"``.

        * ``-DCMAKE_INSTALL_PREFIX=directory`` --- Specify for *directory* the full
          pathname of where you want the LLVM tools and libraries to be installed
          (default ``/usr/local``).

        * ``-DCMAKE_BUILD_TYPE=type`` --- Valid options for *type* are Debug,
          Release, RelWithDebInfo, and MinSizeRel. Default is Debug.

        * ``-DLLVM_ENABLE_ASSERTIONS=On`` --- Compile with assertion checks enabled
          (default is Yes for Debug builds, No for all other build types).

      * Run your build tool of choice!

        * The default target (i.e. ``ninja`` or ``make``) will build all of LLVM.

        * The ``check-all`` target (i.e. ``ninja check-all``) will run the
          regression tests to ensure everything is in working order.

        * CMake will generate build targets for each tool and library, and most
          LLVM sub-projects generate their own ``check-<project>`` target.

        * Running a serial build will be *slow*.  To improve speed, try running a
          parallel build. That's done by default in Ninja; for ``make``, use
          ``make -j NNN`` (NNN is the number of parallel jobs, use e.g. number of
          CPUs you have.)

      * For more information see [CMake](https://llvm.org/docs/CMake.html)

Consult the
[Getting Started with LLVM](https://llvm.org/docs/GettingStarted.html#getting-started-with-llvm)
page for detailed information on configuring and compiling LLVM. You can visit
[Directory Layout](https://llvm.org/docs/GettingStarted.html#directory-layout)
to learn about the layout of the source code tree.
