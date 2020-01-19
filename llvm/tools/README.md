# 工具链命令介绍
## 基本命令

      llvm-as - 汇编器，将 .ll 汇编成字节码。
      llvm-dis - 反汇编器，将字节码编成可读的 .ll 文件。
      opt - 字节码优化器。
      llc - 静态编译器，将字节码编译成汇编代码。
      lli - 直接执行 LLVM 字节码。
      llvm-link - 字节码链接器，可以把多个字节码文件链接成一个。
      llvm-ar - 字节码文件打包器。
      llvm-lib - LLVM lib.exe 兼容库工具。
      llvm-nm - 列出字节码和符号表。
      llvm-config - 打印 LLVM 编译选项。
      llvm-diff - 对两个进行比较。
      llvm-cov - 输出 coverage infomation。
      llvm-profdata - Profile 数据工具。
      llvm-stress - 生成随机 .ll 文件。
      llvm-symbolizer - 地址对应源码位置，定位错误。
      llvm-dwarfdump - 打印 DWARF。

## 调试工具

      bugpoint - 自动测试案例工具
      llvm-extract - 从一个 LLVM 的模块里提取一个函数。
      llvm-bcanalyzer - LLVM 字节码分析器。

## 开发工具

      FileCheck - 灵活的模式匹配文件验证器。
      tblgen - C++ 代码生成器。
      lit - LLVM 集成测试器。
      llvm-build - LLVM 构建工程时需要的工具。
      llvm-readobj - LLVM Object 结构查看器。
