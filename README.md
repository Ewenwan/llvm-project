# The LLVM Compiler Infrastructure

This directory and its subdirectories contain source code for LLVM,
a toolkit for the construction of highly optimized compilers,
optimizers, and runtime environments.

The README briefly describes how to get started with building LLVM.
For more information on how to contribute to the LLVM project, please
take a look at the
[Contributing to LLVM](https://llvm.org/docs/Contributing.html) guide.

[LLVM系统的新用户指南,中文翻译版](https://github.com/Ewenwan/llvm-guide-zh)

[LLVM_IR_tutorial](http://llvm.org/devmtg/2019-04/slides/Tutorial-Bridgers-LLVM_IR_tutorial.pdf)

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
# 文件目录分析

[参考](https://xiaozhuanlan.com/topic/3169254807)

> LLVM 源码工程目录介绍

    llvm_examples_ - 使用 LLVM IR 和 JIT 的例子。
    llvm_include_ - 导出的头文件。
    llvm_lib_ - 主要源文件都在这里。
    llvm_project_ - 创建自己基于 LLVM 的项目的目录。
    llvm_test_ - 基于 LLVM 的回归测试，健全检察。
    llvm_suite_ - 正确性，性能和基准测试套件。
    llvm_tools_ - 基于 lib 构建的可以执行文件，用户通过这些程序进行交互，-help 可以查看各个工具详细使用。
    llvm_utils_ - LLVM 源代码的实用工具，比如，查找 LLC 和 LLI 生成代码差异工具， Vim 或 Emacs 的语法高亮工具等。

> lib 目录介绍

    llvm_lib_IR/ - 核心类比如 Instruction 和 BasicBlock。
    llvm_lib_AsmParser/ - 汇编语言解析器。
    llvm_lib_Bitcode/ - 读取和写入字节码
    llvm_lib_Analysis/ - 各种对程序的分析，比如 Call Graphs，Induction Variables，Natural Loop Identification 等等。
    llvm_lib_Transforms/ - IR-to-IR 程序的变换。
    llvm_lib_Target/ - 对像 X86 这样机器的描述。
    llvm_lib_CodeGen/ - 主要是代码生成，指令选择器，指令调度和寄存器分配。
    llvm_lib_ExecutionEngine/ - 在解释执行和JIT编译场景能够直接在运行时执行字节码的库。
    其他

> LLVM与编译相关的标准库：

**opt：**

该工具的目标是针对IR阶段的程序进行优化，其输入文件必须是LLVM的Bitcode，输出为相同格式的IR文件

**llc：**

将LLVM的IR文件转换成设备相关的汇编语言文件或Obj文件。您可以指定优化等级、Debug使能、是否针对目标平台优化。

**llvm-mc：**

将汇编代码生成为指定格式的OBJ文件，如：ELF文件、MachO文件、PE文件等。也可反汇编相应的OBJ文件

**lli：**

以解释方式或JIT运行LLVM

IR文件llvm-link：

将几个LLVM Bitcode整合为单一一个LLVM Bitcode，注意却别于我们在编译时实用的通常的Link文件，如Linux下ld
 
**llvm-as：**
 
 将LLVM中人工可识别IR文件（ll）转换为LLVMBitcode（BC）文件
 
**llvm-dis：**

将LLVM Bitcode（BC）转换成人可阅读的IR文件（LL）

> LLVM/Clang的基础库：

**libLLVMCore：**

LLVM核心库，主要是：

1）LLVM IR指令构造、检查。

2）Pass管理lib

**LLVMAnalysis：**

包含了IR相关的几个分析Pass：

如对其别名分析、相关性分析、常量折叠（Constant folding）、循环信息、内存依赖分析、指令化简等，详细见目录：lib\Analysis
 
**libLLVMCodeGen：**

目标无关的代码生成、低级LLVM IR（机器相关）分析和转换，见相关目录：lib\CodeGenlib

**LLVMTarget：**

由通用目标及抽象提供对目标机的访问。这些通用的抽象提供了libLLVMCodeGen中通用的后端算法和目标相关逻辑间的通讯途径。详细目录：lib\Target，其中各目录包含了已经支持的各平台libLLVMxxxxCodeGen（xxxx表示具体支持的平台，如X86等）：具体平台相关的代码生成、转换和分析Pass，其是相关平台的后端。

**libLLVMSupport：**

LLVM的支持库：错误处理、整形和浮点处理、命令行解析、调试、文件支持、字符串操作等；所在目录：lib\Support

**libclangxxxx（各clang前端功能，如AST、词法分析、语法分析）：**

提供了访问Clang前端各功能的接口。可以用于任何语言中，如Python中


## 前端clang分析
Clang是LLVM的C/C++前端，从原理上他会产生用于后端的IR指令。但实际上Clang会有两种执行方式： 我们可以使用”-###”观察Clang的执行过程

    1. 以Driver的方式执行： 
       会自动调用相关后端程序，并生成可执行文件，这也是之所以Clang虽然只是前端，却可以直接产生目标代码的原因.
       在驱动模式下，clang实质只是一个调度管理程序.
       
       
    2. 作为编译器-cc1前端方式运行：
       最后仅生成LLVM IR 

Clang执行初期是作为driver执行的，因此，程序的入口是：tools/driver/driver.cpp；

[代码位置](https://github.com/Ewenwan/llvm-project/blob/63ab2f559ba1a4619fdba079ea85e3bea0aaa51a/clang/tools/driver/driver.cpp#L337)

如果不是-cc1，则进行相关命令解释，生成相容的命令行

通过Driver建立与GCC相容的编译过程，并由TheDriver.ExecuteCompilation执行该相容的

注意因为clang两种工作模式下，驱动模式实际是在补足参数后再通过-cc1的方式执行;

在Driver方式下，只是为clang补齐相关执行的各参数，如类库的名字，然后是通过“系统”执行clang -cc1命令，而并没有在“内部”继续clang的其余的操作；此时，clang会等待相关的执行操作完成后执行下一个命令（如ld）

驱动方式过程：

    1。Parse：Option Parsing 编译参数解析

       在这个阶段完成后，命令行将被分解为具有适当参数定义好的选项。

    2. Pipeline：编译动作构造（Compilation Action Construction） 编译流水线构造

   子进程作业树需要确认构造编译序列。这包含确认输入文件及其类型、对他们进行什么样的工作（预处理、编译、汇编、链接等）并为每个人物构造动作实例链表。这样的结构是一个或更多的顶层动作列表，每个通常对应一个单一的输出（例如，对象或链接的可执行文件）多数的动作（Actions）对应一个实际的任务，但是这里有两个特殊的任务（Actions），第一个是InputActions，他只是简单将输入参数匹配到另一个Actions的输入。第二个是BindArchAction，从概念上给所有使用输入的动作替换架构Clang驱动可以使用命令“-ccc-print-phases”转印这一阶段的结果。

    2.5 Action
    
每次的 option 都会完整的走一遍从预处理，静态分析，backend 再到汇编的过程。

下面列下一些编译器的前端 Action，大家可以一个个用着玩。

    InitOnlyAction - 只做前端初始化，编译器 option 是    -init-only
    PreprocessOnlyAction - 只做预处理，不输出，编译器的 option 是 -Eonly
    PrintPreprocessedAction - 做预处理，子选项还包括-P、-C、-dM、-dD 具体可以查看PreprocessorOutputOptions 这个类，编译器 option 是 -E
    RewriteIncludesAction - 预处理
    DumpTokensAction - 打印token，option 是 -dump-tokens
    DumpRawTokensAction - 输出原始tokens，包括空格符，option 是 -dump-raw-tokens
    RewriteMacrosAction - 处理并扩展宏定义，对应的 option 是 -rewrite-macros
    HTMLPrintAction - 生成高亮的代码网页，对应的 option 是 - emit-html
    DeclContextPrintAction - 打印声明，option 对应的是 -print-decl-contexts
    ASTDeclListAction - 打印 AST 节点，option 是 -ast-list
    ASTDumpAction - 打印 AST 详细信息，对应 option 是 -ast-dump
    ASTViewAction - 生成 AST dot 文件，能够通过 Graphviz 来查看图形语法树。 option 是 -ast-view
    AnalysisAction - 运行静态分析引擎，option 是 -analyze
    EmitLLVMAction - 生成可读的 IR 中间语言文件，对应的 option 是      -emit-llvm
    EmitBCAction - 生成 IR Bitcode 文件，option 是                   -emit-llvm-bc
    MigrateSourceAction - 代码迁移，option 是 -migrate


    3. Bind：Tool & Filename Selection 工具、文件绑定
    
Bind 主要是与工具链 ToolChain 交互
根据创建的那些 Action，在 Action 执行时 Bind 来提供使用哪些工具，比如生成汇编时是使用内嵌的还是 GNU 的，还是其它的呢，这个就是由 Bind 来决定的，具体使用的工具有各个架构，平台，系统的 ToolChain 来决定

驱动与工具链互动去执行工具绑定（The driver interacts with a ToolChain to perform the Tool bindings）。每个工具链包含特定架构、平台和操作系统等编译需要的所有信息；一个单一的工具在编译期间需要提取很多工具链，为了与不同体系架构的工具进行交互。

这一阶段不会直接计算，但是驱动器可以使用”-ccc-print-bindings”参数打印这一结果，会显示绑定到编译序列的工具链，工具，输入和输出。


    4. Translate：Tool Specific Argument Translation 工具参数翻译

当工具被选择来执行特定的动作，工具必须为之后运行的编译过程构造具体的命令。主要的工作是翻译gcc格式的命令到子进程所期望的格式。

    5.Execute  工具执行
    
其执行过程大致如下：Driver::ExecuteCompilation -> Compilation::ExecuteJobs -> Compilation::ExecuteCommand-> Command::Execute -> llvm::sys::ExecuteAndWait；此时执行的ExecuteAndWait为Support/Program.cpp中的程序，其调用相关操作系统，执行其系统相关的执行程序，并等待执行过程完成。

### clang cc1前端运行  真正有意义的前端操作
```c
    auto FirstArg = std::find_if(argv.begin() + 1, argv.end(),
                               [](const char *A) { return A != nullptr; });
  if (FirstArg != argv.end() && StringRef(*FirstArg).startswith("-cc1")) {
    // If -cc1 came from a response file, remove the EOL sentinels.
    if (MarkEOLs) {
      auto newEnd = std::remove(argv.begin(), argv.end(), nullptr);
      argv.resize(newEnd - argv.begin());
    }
    return ExecuteCC1Tool(argv);
  }  
 ```
如果是 -cc1 的话会调用 ExecuteCC1Tool 这个函数，先看看这个函数

```c
static int ExecuteCC1Tool(ArrayRef<const char *> argv) {
  llvm::cl::ResetAllOptionOccurrences();
  StringRef Tool = argv[1];
  void *GetExecutablePathVP = (void *)(intptr_t) GetExecutablePath;
  if (Tool == "-cc1")
    return cc1_main(argv.slice(2), argv[0], GetExecutablePathVP);
  if (Tool == "-cc1as")
    return cc1as_main(argv.slice(2), argv[0], GetExecutablePathVP);
  if (Tool == "-cc1gen-reproducer")
    return cc1gen_reproducer_main(argv.slice(2), argv[0], GetExecutablePathVP);
  // Reject unknown tools.
  return 1;
}
```
最终的执行会执行 cc1-main 、cc1as_main 、cc1gen_reproducer_main。这三个函数分别在 driver.cpp 同级目录里的 cc1_main.cpp 、cc1as_main.cpp 、cc1gen_reproducer_main.cpp中。

依照关于Driver的示意图，clang将藉由Action完成具体的操作，在clang中所有action定义在include/clang/Drivers名字域：clang::driver下，其Action即其派生的Actions定义如下：

这一阶段完成，编译过程被分为一组需要执行并产生中间或最终输出（某些情况下，类似-fsyntax-only，不会有“真实”的最终输出）的Action。阶段是我们熟知的编译步骤，类似：预处理、编译、汇编、链接等等。

所有相关Action的定义在FrontendOptions.h(clang/include/clang/Frontend/FrontendOptions.h )中；

[代码](https://github.com/Ewenwan/llvm-project/blob/f9665a93fa96c464d7b6e911e111fbbbc0fbfa64/clang/include/clang/Frontend/FrontendOptions.h#L34)

在clang中允许通过FrontendAction编写自己的Action，使用FrontendPluginRegistry（clang/frontend/FrontendRegistry.h）注册自己的Action:其核心是通过继承clang::FrontendAction来实现，详细示例参考：clang/examples/AnnotateFunctions/AnnotateFunctions.cpp，该示例通过继承PluginASTAction，并使用FrontendPluginRegistry::Add将其注册

最初的C/C++源码经过：词法分析（Lexical analysis）、语法分析（Syntactic analysis）、语义分析（Semantic analysis）最后输出与平台无关的IR（LLVM IR generator）

###  词法分析（Lexical analysis） libclangLex 

编译器第一个步骤是词法分析（Lexical analysis）。词法分析器读入组成源程序的字节流，并将他们组成有意义的词素（Lexeme）序列。对于每个词素，词法分析器产生**词单元（token）**作为输出，并生成相关符号表。词法库包含了几个紧密相连的类，他们涉及到词法和C源码预处理。

源码      clang/lib/Lex/
相关诊断  DiagnosticLexKinds.td

[后续分析](https://github.com/Ewenwan/llvm-project/blob/master/clang/lib/readme.md)

## 后端LLVM分析



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
