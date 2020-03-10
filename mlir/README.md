# Multi-Level Intermediate Representation

See [https://mlir.llvm.org/](https://mlir.llvm.org/) for more information.

MLIR（或称为多级别中介码）是谷歌新发起的项目。这是一种表示格式和编译器实用工具库，介于模型表示和低级编译器/执行器（二者皆可生成硬件特定代码）之间。MLIR 的核心是一种灵活的基础设施，适用于现代优化编译器。这意味着其中包含适用于中介码 (IR) 的规范与转换此中介码的代码工具包。MLIR 深受 LLVM 的影响，并不折不扣地重用其许多优秀理念。


MLIR希望为各种DSL（领域专用语言）提供一种中间表达形式，将他们集成为一套生态系统，使用一种一致性强的方式编译到特定硬件平台的汇编语言上。利用这样的形式，MLIR就可以利用它模块化、可扩展的特点来解决IR之间相互配合的问题。

[Keynote-ShpeismanLattner-MLIR 介绍](http://llvm.org/devmtg/2019-04/slides/Keynote-ShpeismanLattner-MLIR.pdf)

[Tutorial-AminiVasilacheZinenko-MLIR](http://llvm.org/devmtg/2019-04/slides/Tutorial-AminiVasilacheZinenko-MLIR.pdf)

# 编译

$ git clone https://github.com/llvm/llvm-project.git
$ mkdir llvm-project/build
$ cd llvm-project/build
$ cmake -G "Unix Makefiles" ../llvm \
     -DLLVM_ENABLE_PROJECTS=mlir \
     -DLLVM_BUILD_EXAMPLES=ON \
     -DLLVM_TARGETS_TO_BUILD="host" \
     -DCMAKE_BUILD_TYPE=Release \
     -DLLVM_ENABLE_ASSERTIONS=ON 
$ cmake --build . --target check-mlir


[参考](https://www.zhihu.com/people/zhang-hong-bin-99/posts)



# 使用 


首先要将源程序生成抽象语法树(AST)，然后再遍历抽象语法树来构建MLIR表达式。那么使用我们刚刚生成的可执行文件来打印出抽象语法树：

$ cd llvm-project/build/
$ bin/toyc-ch2 ../mlir/test/Examples/Toy/Ch2/codegen.toy -emit=ast

同样使用可执行文件来打印MLIR表达式：

$ cd llvm-project/build/
$ bin/toyc-ch2 ../mlir/test/Examples/Toy/Ch2/codegen.toy -emit=mlir -mlir-print-debuginfo




