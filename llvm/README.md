The LLVM Compiler Infrastructure
================================

This directory and its subdirectories contain source code for LLVM,
a toolkit for the construction of highly optimized compilers,
optimizers, and runtime environments.

LLVM is open source software. You may freely distribute it under the terms of
the license agreement found in LICENSE.txt.

Please see the documentation provided in docs/ for further
assistance with LLVM, and in particular docs/GettingStarted.rst for getting
started with LLVM and docs/README.txt for an overview of LLVM's
documentation setup.

If you are writing a package for LLVM, see docs/Packaging.rst for our
suggestions.

# LLVM 源码工程目录介绍

      llvm/examples/ - 使用 LLVM IR 和 JIT 的例子。
      llvm/include/ - 导出的头文件。
      llvm/lib/ - 主要源文件都在这里。
      llvm/project/ - 创建自己基于 LLVM 的项目的目录。
      llvm/test/ - 基于 LLVM 的回归测试，健全检察。
      llvm/suite/ - 正确性，性能和基准测试套件。
      llvm/tools/ - 基于 lib 构建的可以执行文件，用户通过这些程序进行交互，-help 可以查看各个工具详细使用。
      llvm/utils/ - LLVM 源代码的实用工具，比如，查找 LLC 和 LLI 生成代码差异工具， Vim 或 Emacs 的语法高亮工具等。

# lib 目录介绍

      llvm/lib/IR/ - 核心类比如 Instruction 和 BasicBlock。
      llvm/lib/AsmParser/ - 汇编语言解析器。
      llvm/lib/Bitcode/ - 读取和写入字节码
      llvm/lib/Analysis/ - 各种对程序的分析，比如 Call Graphs，Induction Variables，Natural Loop Identification 等等。
      llvm/lib/Transforms/ - IR-to-IR 程序的变换。
      llvm/lib/Target/ - 对像 X86 这样机器的描述。
      llvm/lib/CodeGen/ - 主要是代码生成，指令选择器，指令调度和寄存器分配。
      llvm/lib/ExecutionEngine/ - 在解释执行和JIT编译场景能够直接在运行时执行字节码的库。




