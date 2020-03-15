//===----------------------------------------------------------------------===//
# C Language Family Front-end
//===----------------------------------------------------------------------===//

[arm  ClangTutorial](https://llvm.org/devmtg/2019-10/slides/ClangTutorial-Stulova-vanHaastregt.pdf)

[clang 代码继承树 ](https://clang.llvm.org/doxygen/inherits.html)

[Basic source-to-source transformation with Clang](https://eli.thegreenplace.net/2012/06/08/basic-source-to-source-transformation-with-clang/)

[使用Clang实现C语言编程规范检查http://www.cppblog.com/kevinlynx/archive/2013/02/12/197810.aspx](http://www.cppblog.com/kevinlynx/archive/2013/02/12/197810.aspx)

静态分析前会对源代码分词成 Token，这个过程称为词法分析（Lexical Analysis），在 clang/include/clang/Basic/TokenKinds.def  里有 Clang 定义的所有 Token。Token 可以分为以下几类：

      关键字：语法中的关键字，if else while for 等。
      标识符：变量名
      字面量：值，数字，字符串
      特殊符号：加减乘除等符号

通过下面的命令可以输出所有 token 和所在文件具体位置

clang -fmodules -E -Xclang -dump-tokens main.m

可以获得每个 token 的类型，值还有类似 StartOfLine 的位置类型和 Loc=main.m:11:1 这个样的具体位置。
接着进行语法分析（Semantic Analysis）将 token 先按照语法组合成语义生成 VarDecl 节点，然后将这些节点按照层级关系构成抽象语法树 Abstract Syntax Tree (AST)。

打个比方，如果遇到 token 是 = 符号进行赋值的处理，遇到加减乘除就先处理乘除，然后处理加减，这些组合经过嵌套后会生成一个语法数的结构。这个过程完成后会进行赋值操作时类型是不是匹配的处理。


打印语法树的命令
clang -fmodules -fsyntax-only -Xclang -ast-dump main.m



TranslationUnitDecl 是根节点，表示一个源文件。

Decl 表示一个声明，

Expr 表示表达式，

Literal 表示字面量是特殊的 Expr，

Stmt 表示语句。


clang 静态分析是通过建立 分析引擎 和 checkers 所组成的架构，这部分功能可以通过 clang —analyze 命令方式调用。

clang static analyzer 分为 analyzer core 分析引擎 和 checkers 两部分，所有 checker 都是基于底层分析引擎之上，通过分析引擎提供的功能能够编写新的 checker。

可以通过 

clang —analyze -Xclang -analyzer-checker-help 来列出当前 clang 版本下所有 checker。

如果想编写自己的 checker，可以在 clang 项目的 lib / StaticAnalyzer / Checkers 目录下找到实例参考，

比如 ObjCUnusedIVarsChecker.cpp 用来检查未使用定义过的变量。

这种方式能够方便用户扩展对代码检查规则或者对 bug 类型进行扩展，但是这种架构也有不足，每执行完一条语句后，分析引擎会遍历所有 checker 中的回调函数，所以 checker 越多，速度越慢。通过 clang -cc1 -analyzer-checker-help 可以列出能调用的 checker，下面是常用 checker


      debug.ConfigDumper              Dump config table
      debug.DumpCFG                   Display Control-Flow Graphs
      debug.DumpCallGraph             Display Call Graph
      debug.DumpCalls                 Print calls as they are traversed by the engine

debug.DumpDominators            Print the dominance tree for a given CFG
      debug.DumpLiveVars              Print results of live variable analysis
      debug.DumpTraversal             Print branch conditions as they are traversed by the engine
      debug.ExprInspection            Check the analyzer's understanding of expressions
      debug.Stats                     Emit warnings with analyzer statistics
      debug.TaintTest                 Mark tainted symbols as such.
      debug.ViewCFG                   View Control-Flow Graphs using GraphViz
      debug.ViewCallGraph             View Call Graph using GraphViz
      debug.ViewExplodedGraph         View Exploded Graphs using GraphViz

这些 checker 里最常用的是 DumpCFG，DumpCallGraph，DumpLiveVars 和 DumpViewExplodedGraph。

clang static analyzer 引擎大致分为 

CFG，
MemRegion，
SValBuilder，
ConstraintManager 和 
ExplodedGraph 几个模块。

clang static analyzer 本质上就是 path-sensitive analysis，

要很好的理解 clang static analyzer 引擎就需要对 Data Flow Analysis 有所了解，

包括 迭代数据流分析，path-sensitive，path-insensitive ，flow-sensitive等。

编译的概念（词法->语法->语义->IR->优化->CodeGen）在 clang static analyzer 里到处可见，

例如 Relaxed Live Variables Analysis 可以减少分析中的内存消耗，使用 mark-sweep 实现 Dead Symbols 的删除。

clang static analyzer 提供了很多辅助方法，比如 SVal.dump()，MemRegion.getString 以及 Stmt 和 Dcel 提供的 dump 方法。

Clang 抽象语法树 Clang AST 常见的 API 有 Stmt，Decl，Expr 和 QualType。

在编写 checker 时会遇到 AST 的层级检查，这时有个很好的接口 StmtVisitor，这个接口类似 RecursiveASTVisitor。


整个 clang static analyzer 的入口是 AnalysisConsumer，接着会调 HandleTranslationUnit() 方法进行 AST 层级进行分析或者进行 path-sensitive 分析。

默认会按照 inline 的 path-sensitive 分析，构建 CallGraph，从顶层 caller 按照调用的关系来分析，具体是使用的 WorkList 算法，从 EntryBlock 开始一步步的模拟，这个过程叫做 intra-procedural analysis（IPA）。

这个模拟过程还需要对内存进行模拟，clang static analyzer 的内存模型是基于《A Memory Model for Static Analysis of C Programs》这篇论文而来，

pdf地址：http://lcs.ios.ac.cn/~xuzb/canalyze/memmodel.pdf 

在clang里的具体实现代码可以查看这两个文件 MemRegion.h和 RegionStore.cpp 。



下面举个简单例子看看 clang static analyzer 是如何对源码进行模拟的。

        int main()
        {
            int a;
            int b = 10;
            a = b;
            return a;
        }

对应的 AST 以及 CFG

        #————————AST—————————
        # clang -cc1 -ast-dump
        TranslationUnitDecl 0xc75b450 <<invalid sloc>> <invalid sloc>
        |-TypedefDecl 0xc75b740 <<invalid sloc>> <invalid sloc> implicit __builtin_va_list ‘char *’
        `-FunctionDecl 0xc75b7b0 <test.cpp:1:1, line:7:1> line:1:5 main ‘int (void)’
          `-CompoundStmt 0xc75b978 <line:2:1, line:7:1>
            |-DeclStmt 0xc75b870 <line:3:2, col:7>
            | `-VarDecl 0xc75b840 <col:2, col:6> col:6 used a ‘int’
            |-DeclStmt 0xc75b8d8 <line:4:2, col:12>
            | `-VarDecl 0xc75b890 <col:2, col:10> col:6 used b ‘int’ cinit
            |   `-IntegerLiteral 0xc75b8c0 <col:10> ‘int’ 10

        <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< a = b <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
            |-BinaryOperator 0xc75b928 <line:5:2, col:6> ‘int’ lvalue ‘=‘
            | |-DeclRefExpr 0xc75b8e8 <col:2> ‘int’ lvalue Var 0xc75b840 ‘a’ ‘int’
            | `-ImplicitCastExpr 0xc75b918 <col:6> ‘int’ <LValueToRValue>
            |   `-DeclRefExpr 0xc75b900 <col:6> ‘int’ lvalue Var 0xc75b890 ‘b’ ‘int’
        <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

            `-ReturnStmt 0xc75b968 <line:6:2, col:9>
              `-ImplicitCastExpr 0xc75b958 <col:9> ‘int’ <LValueToRValue>
                `-DeclRefExpr 0xc75b940 <col:9> ‘int’ lvalue Var 0xc75b840 ‘a’ ‘int’
        #————————CFG—————————
        # clang -cc1 -analyze -analyzer-checker=debug.DumpCFG
        int main()
         [B2 (ENTRY)]
           Succs (1): B1

         [B1]
           1: int a;
           2: 10
           3: int b = 10;
           4: b
           5: [B1.4] (ImplicitCastExpr, LValueToRValue, int)
           6: a
           7: [B1.6] = [B1.5]
           8: a
           9: [B1.8] (ImplicitCastExpr, LValueToRValue, int)
          10: return [B1.9];
           Preds (1): B2
           Succs (1): B0

         [B0 (EXIT)]
           Preds (1): B1
   
CFG 将程序拆得更细，能够将执行的过程表现的更直观些，为了避免路径爆炸，函数 inline 的条件会设置的比较严格，函数 CFG 块多时不会进行 inline 分析，模拟栈深度超过一定值不会进行 inline 分析，这个默认是5。

在MRC使用的是CFG这样的执行路径模拟，ARC就没有了，举个例子，没有全部条件都返回，CFG就会报错，而AST就不会。


Welcome to Clang.  This is a compiler front-end for the C family of languages
(C, C++, Objective-C, and Objective-C++) which is built as part of the LLVM
compiler infrastructure project.

Unlike many other compiler frontends, Clang is useful for a number of things
beyond just compiling code: we intend for Clang to be host to a number of
different source-level tools.  One example of this is the Clang Static Analyzer.

If you're interested in more (including how to build Clang) it is best to read
the relevant web sites.  Here are some pointers:

Information on Clang:             http://clang.llvm.org/
Building and using Clang:         http://clang.llvm.org/get_started.html
Clang Static Analyzer:            http://clang-analyzer.llvm.org/
Information on the LLVM project:  http://llvm.org/

If you have questions or comments about Clang, a great place to discuss them is
on the Clang development mailing list:
  http://lists.llvm.org/mailman/listinfo/cfe-dev

If you find a bug in Clang, please file it in the LLVM bug tracker:
  http://llvm.org/bugs/


# 项目和工具列表

https://github.com/Andersbakken/rtags/
RTags是一个 client/server 应用程序，它索引 c/c++ 代码，并在内存中保存一个持久的数据库，其中包含引用、符号名、补全等。

https://rprichard.github.com/sourceweb/
一个 C/C++ 源代码索引器和导航器。

https://github.com/etaoins/qconnectlint
qconnectlint是一个 Clang 工具，用于静态验证使用 Qt 的QObject::connect建立的信号和插槽连接的一致性。

https://github.com/woboq/woboq_codebrowser
Woboq代码浏览器是一个基于web的代码浏览器，用于 C/C++ 项目。查看 https://code.woboq.org/ 获得一个示例。

https://github.com/mozilla/dxr
DXR是一个源代码交叉引用工具，它使用由插桩后的编译器收集的静态分析数据。

https://github.com/eschulte/clang-mutate
这个工具对C语言源文件执行许多操作。

https://github.com/gmarpons/Crisp
一个用于LLVM/clang的编码规则验证附加组件。Crisp规则是在Prolog中编写的。一个高级声明性 DSL 正在开发中，可以轻松地编写新规则。它将被命名为CRISP，这是Coding Rules in Sugared Prolog的首字母缩写。

https://github.com/drothlis/clang-ctags
为C++源代码生成 tag 文件。

https://github.com/exclipy/clang_indexer
这是一个基于libclang库的C和C++索引器。

https://github.com/holtgrewe/linty
Linty —— C/C++风格检查，使用 Python & libclang。

https://github.com/axw/cmonster
cmonster是一个用于 Clang C++ 解析器的Python wrapper。

https://github.com/rizsotto/Constantine
Constantine是一个学习如何编写 clang plugin 的简单项目。实现伪 const 分析。生成关于变量的警告，这些变量在声明时没有使用 const 限定符。

https://github.com/jessevdk/cldoc
cldoc是一个基于 Clang 的 C和C++文档生成器。cldoc试图用一种现代的、非侵入性的、健壮的方法来解决编写C/C++软件文档的问题。

https://github.com/AlexDenisov/ToyClangPlugin
实现 Objective-C 语义检查的最简单的 Clang plugin。这个例子展示了如何使用DiagnosticsEngine（发出警告、错误和fixit提示）。参见 
http://l.rw.rw/clang_plugin 查看详细的步骤说明。

https://phabricator.kde.org/source/clazy
clazy是一个编译器插件，它允许 clang 理解 Qt 语义。你会得到50多个 Qt 相关的编译器警告，从不需要的内存分配到API的滥用，包括用于自动重构的fix-its。

https://gerrit.libreoffice.org/gitweb?p=core.git;a=blob_plain;f=compilerplugins/README;hb=HEAD

LibreOffice使用一个 Clang plugin 基础设施在构建过程中检查各种东西，有些是LibreOffice源代码特有的，有些不是。目前大约有50个这样的检查器，从flagging C-style casts和保留标识符的使用，到确保代码遵循特定于LibreOffice类的特定生命周期协议。它们可以作为编写基于RecursiveASTVisitor的 plugins 的例子。

# CLANG 分析

最初的C/C++源码经过：词法分析（Lexical analysis）-> 语法分析（Syntactic analysis）-> 语义分析（Semantic analysis）-> 与平台无关的IR（LLVM IR generator）

从词法分析开始——将C语言 源码分解成token流，每个token可表示标识符、字面量、运算符等；
token流会传递给语法分析器，语法分析器会在语言的CFG（Context Free Grammar，上下文无关文法）的指导下将token流组织成AST（抽 象语法树）；接下来会进行语义分析，检查语义正确性，然后生成IR。


libclang  clang-c/Index.h   c接口调用clang [示例](https://github.com/Ewenwan/screader/blob/master/scread.c)

## 1. 词法分析（Lexical analysis） clang/lib/Lex/ 

### token

词法分析器读入组成源程序的字节流，并将他们组成有意义的词素（Lexeme）序列。对于每个词素，词法分析器产生词单元（token）作为输出，并生成相关符号表。词法库包含了几个紧密相连的类，他们涉及到词法和C源码预处理。

相关诊断: clang/include/clang/Basic/DiagnosticLexKinds.td

词法单元（Token）的定义：TokenKinds.def（clang/include/clang/Basic/）

Clang的保留字定义在 TokenKinds.def，如常见的if或for关键字

说明TokenKinds.def中定义了许多诸如TOK和KEYWORD等宏，实际相关宏只有在外部引用程序引用TokenKinds.def前定义才会真正有意义；而且引用位置不同，其意义可能不同


### Preprocessor类 ----- 返回下一个Token

Preprocessor是词法分析中的一个主要接口,可以看到，词法分析时在预处理过程中初始化的即Preprocessor.

在词法分析中，预处理程序被初始调用的主要程序是[CompilerInstance::createPreprocessor（lib/Frontend/CompilerInstance.cpp）](https://github.com/Ewenwan/llvm-project/blob/3745623ba574effe6c157fde7b556734bb7456b5/clang/lib/Frontend/CompilerInstance.cpp#L379)

之后是 FrontEnd Action -> /clang/lib/Lex/Preprocessor.cpp

其核心程序是[Preprocessor::Lex](https://github.com/Ewenwan/llvm-project/blob/3745623ba574effe6c157fde7b556734bb7456b5/clang/lib/Lex/Preprocessor.cpp#L886),该程序返回下一个Token


### Token类  clang/include/clang/Lex/Token.h 

Token类用于表述电仪的词法单元。Token被用于词法/预处理单元，但并不会在这些库以外存在（如，Token不会存在于AST中）.
[Token.h](https://github.com/Ewenwan/llvm-project/blob/master/clang/include/clang/Lex/Token.h)


## 2. 语法分析（Syntactic analysis） libclangParse libclangAST

语法分析主要是解析词法分析产生的词法单元（token）并生成抽象语法树（ATS）。如同自然语言一样，语法分析并不检查语法的上下文的合理性，其仅仅是从语法角度确认是否正确。 

语法分析的核心数据结构：Decl、Stmt、Type对应于声明、指令、类型。所有的被Clang描述的C/C++类都是继承于这三个类 

源码  clang/lib/Parse/ 和clang/lib/AST/

诊断信息  DiagnosticParseKinds.td

语法分析器是使用递归下降（recursive-descent）的语法分析。

clang的Parser是由clang::ParseAST执行的。

libclangAST：

        提供了类用于：表示C AST、C类型、内建函数和一些用于分析和操作AST的功能（visitors、漂亮的打印输出等）
        源码定义：lib/AST
        重要的AST节点：Type、Decl、DeclContext、Stmt
        
### Type类和他的派生类 clang/lib/AST/Type.cpp 

Type类及其派生类是AST中非常重要的一部分。通过ASTContext访问Type（clang/ast/ASTContext.h），在需要时他隐式的唯一的创建他们。Type有一些不明显的特征：1）他们不捕获类似const或volatile类型修饰符（See QualType）；2）他们隐含的捕获typedef信息。一旦创建，type将是不可变的。


### 声明，Decl类 clang/AST/DeclBase.h

表示一个声明（declaration）或定义（definition）。例如：变量、typedef、函数、结构等


### 声明上下文，DeclContext类  clang/AST/DeclBase.h

程序中每个声明都存在于某一个声明上下文中，类似翻译单元、名字空间、类或函数。Clang中的声明上下文是由类DeclContext类进行描述：各种AST节点声明上下文均派生于此（TranslationUnitDecl、NamespaceDecl、RecordDecl、FunctionDecl等）

DeclContext类对于每个声明上下文提供了一些公用功能：

1. 与源码为中心和语义为中心的声明视图
   
   DeclContext提供了两种声明视图：源码为中心视图准确表示源程序代码，包括多个实体的声明（参见再宣告和重载）。而语义为中心的视图表示了程序语义。这两个视图在AST构造时与语义分析保持同步。
   
2.在上下文中保存声明

  每个声明上下文中都包含若干的声明。例如：C++类（被表示为RecordDecl）包含了一些成员函数、域、嵌套类型等等。所有这些声明都保存在DeclContext中：即可以由容器迭代操作获得 
   
  这个机制提供了基于源码视图的声明上下文视图。

3. 在上下文中查找声明

由DeclarationName类型指定的声明在声明上下文中查找声明   lookup()

该机制提供了基于语义为中心的声明上下文视图

4. 声明所有者

   DeclContext包含了所有在其中声明的声明上下文，并负责管理他们并以及序列化（反）他们所有声明都保存在声明上下文中，并可以查询每个保存在其中的声明信息。关于声明上下文可以查看词法和语义分析一节.
   
5. 再宣告和重载
   
   在翻译单元中，公共的实体可能会被声明多次。
   
   表达式”f”在以源码为中心的和以语义为中心的上下文中的视图有所不同，在以源码为中心的声明上下文中，再宣告与他在源码中声明的位置有关。在语义为中心的视图中，会使用最近的视图替换最初的声明。而基于DeclContext::look操作将返回基于语义视图的上下文。
   
6. 词法和语义上下文

对于每个声明可能存在两个不同的声明上下文：词法上下文，对应于源码视图的声明上下文。语义上下文对应于语法视图的。Decl::getLexicalDeclContext（clang/AST/DeclBase.h）返回词法声明上下文。而Decl::getDeclContext返回基于语义上下文，返回的两个值都是指向DeclContext的指针。

7.透明声明上下文（TransparentDeclaration Contexts）

现于枚举类型，Red是Color中的成员，但是，我们在Color外引用Red时并不需要限定名：Color；

8.多重定义的声明上下文（Multiply-DefinedDeclaration Contexts）

可以由DeclContext::isTransparentContext确认是否是透明声明。

###  Stmt 指令 类  clang/AST/Stmt.h   for do goto return ...

### QualType类  查询类

QualType被设计为一个微不足道的、微小的通过传值用于高效查询的类。QualType的思想是将类型修饰符（const、volatile、restrict以及语言扩展所需的修饰符）与他们自己的类型分开保存。QualType概念上是一对“Type *”和他们的类型修饰符。类型限定符只是占用指针的低位。


### 声明名字（Declarationnames）

DeclarationName（clang/AST/DeclarationName.h）用来描述clang中的声明名字。声明在C族语言中有一些不同的形式。多数的声明被命名为简单的标识，例如：f(int x)中的声明f和x。在C++中，声明可以构造类的构造函数、类的析构函数、重载操作符合转换函数。


### CFG类 控制流程图？ （Context Free Grammar，上下文无关文法）
CFG是用于描述单个指令（Stmt *）的源码级控制流程图？。典型的CFG实例为构造函数体（典型的是一个CompoundStmt实例），但是也可以表示任何Stmt派生类的控制流。控制流图通常对给定函数执行流-或路径-敏感的分析特别有用。


## 3.语义分析（Semantic Analysis）

libclangSema



## 4. 中间代码生成（IR Generator）  libclangCodeGen

libclangCodeGen

## 5. 其他库介绍
# CLANG 分析

最初的C/C++源码经过：词法分析（Lexical analysis）-> 语法分析（Syntactic analysis）-> 语义分析（Semantic analysis）-> 与平台无关的IR（LLVM IR generator）

从词法分析开始——将C语言 源码分解成token流，每个token可表示标识符、字面量、运算符等；
token流会传递给语法分析器，语法分析器会在语言的CFG（Context Free Grammar，上下文无关文法）的指导下将token流组织成AST（抽 象语法树）；接下来会进行语义分析，检查语义正确性，然后生成IR。


libclang  clang-c/Index.h   c接口调用clang [示例](https://github.com/Ewenwan/screader/blob/master/scread.c)

## 1. 词法分析（Lexical analysis） clang/lib/Lex/ 

### token

词法分析器读入组成源程序的字节流，并将他们组成有意义的词素（Lexeme）序列。对于每个词素，词法分析器产生词单元（token）作为输出，并生成相关符号表。词法库包含了几个紧密相连的类，他们涉及到词法和C源码预处理。

相关诊断: clang/include/clang/Basic/DiagnosticLexKinds.td

词法单元（Token）的定义：TokenKinds.def（clang/include/clang/Basic/）

Clang的保留字定义在 TokenKinds.def，如常见的if或for关键字

说明TokenKinds.def中定义了许多诸如TOK和KEYWORD等宏，实际相关宏只有在外部引用程序引用TokenKinds.def前定义才会真正有意义；而且引用位置不同，其意义可能不同


### Preprocessor类 ----- 返回下一个Token

Preprocessor是词法分析中的一个主要接口,可以看到，词法分析时在预处理过程中初始化的即Preprocessor.

在词法分析中，预处理程序被初始调用的主要程序是[CompilerInstance::createPreprocessor（lib/Frontend/CompilerInstance.cpp）](https://github.com/Ewenwan/llvm-project/blob/3745623ba574effe6c157fde7b556734bb7456b5/clang/lib/Frontend/CompilerInstance.cpp#L379)

之后是 FrontEnd Action -> /clang/lib/Lex/Preprocessor.cpp

其核心程序是[Preprocessor::Lex](https://github.com/Ewenwan/llvm-project/blob/3745623ba574effe6c157fde7b556734bb7456b5/clang/lib/Lex/Preprocessor.cpp#L886),该程序返回下一个Token


### Token类  clang/include/clang/Lex/Token.h 

Token类用于表述电仪的词法单元。Token被用于词法/预处理单元，但并不会在这些库以外存在（如，Token不会存在于AST中）.
[Token.h](https://github.com/Ewenwan/llvm-project/blob/master/clang/include/clang/Lex/Token.h)


## 2. 语法分析（Syntactic analysis） libclangParse libclangAST

语法分析主要是解析词法分析产生的词法单元（token）并生成抽象语法树（ATS）。如同自然语言一样，语法分析并不检查语法的上下文的合理性，其仅仅是从语法角度确认是否正确。 

语法分析的核心数据结构：Decl、Stmt、Type对应于声明、指令、类型。所有的被Clang描述的C/C++类都是继承于这三个类 

源码  clang/lib/Parse/ 和clang/lib/AST/

诊断信息  DiagnosticParseKinds.td

语法分析器是使用递归下降（recursive-descent）的语法分析。

clang的Parser是由clang::ParseAST执行的。

libclangAST：

        提供了类用于：表示C AST、C类型、内建函数和一些用于分析和操作AST的功能（visitors、漂亮的打印输出等）
        源码定义：lib/AST
        重要的AST节点：Type、Decl、DeclContext、Stmt
        
### Type类和他的派生类 clang/lib/AST/Type.cpp 

Type类及其派生类是AST中非常重要的一部分。通过ASTContext访问Type（clang/ast/ASTContext.h），在需要时他隐式的唯一的创建他们。Type有一些不明显的特征：1）他们不捕获类似const或volatile类型修饰符（See QualType）；2）他们隐含的捕获typedef信息。一旦创建，type将是不可变的。


### 声明，Decl类 clang/AST/DeclBase.h

表示一个声明（declaration）或定义（definition）。例如：变量、typedef、函数、结构等


### 声明上下文，DeclContext类  clang/AST/DeclBase.h

程序中每个声明都存在于某一个声明上下文中，类似翻译单元、名字空间、类或函数。Clang中的声明上下文是由类DeclContext类进行描述：各种AST节点声明上下文均派生于此（TranslationUnitDecl、NamespaceDecl、RecordDecl、FunctionDecl等）

DeclContext类对于每个声明上下文提供了一些公用功能：

1. 与源码为中心和语义为中心的声明视图
   
   DeclContext提供了两种声明视图：源码为中心视图准确表示源程序代码，包括多个实体的声明（参见再宣告和重载）。而语义为中心的视图表示了程序语义。这两个视图在AST构造时与语义分析保持同步。
   
2.在上下文中保存声明

  每个声明上下文中都包含若干的声明。例如：C++类（被表示为RecordDecl）包含了一些成员函数、域、嵌套类型等等。所有这些声明都保存在DeclContext中：即可以由容器迭代操作获得 
   
  这个机制提供了基于源码视图的声明上下文视图。

3. 在上下文中查找声明

由DeclarationName类型指定的声明在声明上下文中查找声明   lookup()

该机制提供了基于语义为中心的声明上下文视图

4. 声明所有者

   DeclContext包含了所有在其中声明的声明上下文，并负责管理他们并以及序列化（反）他们所有声明都保存在声明上下文中，并可以查询每个保存在其中的声明信息。关于声明上下文可以查看词法和语义分析一节.
   
5. 再宣告和重载
   
   在翻译单元中，公共的实体可能会被声明多次。
   
   表达式”f”在以源码为中心的和以语义为中心的上下文中的视图有所不同，在以源码为中心的声明上下文中，再宣告与他在源码中声明的位置有关。在语义为中心的视图中，会使用最近的视图替换最初的声明。而基于DeclContext::look操作将返回基于语义视图的上下文。
   
6. 词法和语义上下文

对于每个声明可能存在两个不同的声明上下文：词法上下文，对应于源码视图的声明上下文。语义上下文对应于语法视图的。Decl::getLexicalDeclContext（clang/AST/DeclBase.h）返回词法声明上下文。而Decl::getDeclContext返回基于语义上下文，返回的两个值都是指向DeclContext的指针。

7.透明声明上下文（TransparentDeclaration Contexts）

现于枚举类型，Red是Color中的成员，但是，我们在Color外引用Red时并不需要限定名：Color；

8.多重定义的声明上下文（Multiply-DefinedDeclaration Contexts）

可以由DeclContext::isTransparentContext确认是否是透明声明。

###  Stmt 指令 类  clang/AST/Stmt.h   for do goto return ...

### QualType类  查询类

QualType被设计为一个微不足道的、微小的通过传值用于高效查询的类。QualType的思想是将类型修饰符（const、volatile、restrict以及语言扩展所需的修饰符）与他们自己的类型分开保存。QualType概念上是一对“Type *”和他们的类型修饰符。类型限定符只是占用指针的低位。


### 声明名字（Declarationnames）

DeclarationName（clang/AST/DeclarationName.h）用来描述clang中的声明名字。声明在C族语言中有一些不同的形式。多数的声明被命名为简单的标识，例如：f(int x)中的声明f和x。在C++中，声明可以构造类的构造函数、类的析构函数、重载操作符合转换函数。


### CFG类 控制流程图？ （Context Free Grammar，上下文无关文法）
CFG是用于描述单个指令（Stmt *）的源码级控制流程图？。典型的CFG实例为构造函数体（典型的是一个CompoundStmt实例），但是也可以表示任何Stmt派生类的控制流。控制流图通常对给定函数执行流-或路径-敏感的分析特别有用。


## 3.语义分析（Semantic Analysis）

libclangSema



## 4. 中间代码生成（IR Generator）  libclangCodeGen

libclangCodeGen

## 5. 其他库介绍

libclangAnaylysis：用于进行静态分析用的


libclangRewrite：编辑文本缓冲区（代码重写转换非常重要，如重构）


libclangBasic：诊断、源码定位、源码缓冲区抽象化、输入源文件的文件缓冲区

Clang诊断子系统是一个编译器与人交互的重要部分。诊断是当代码不正确或可疑时产生警告和错误。在clang中，每个诊断产生（最少）一个唯一标识ID、一个相关的英文、SourceLocation源码位置“放置一个^”和一个严重性（例如：WARNING或ERROR）。他可以选择包含一些参数给争端（如使用%0填充字串）以及相关源码区域。

Clang diagnostics   Diagnostic::Level  enum  [ NOTE  WARNING  EXTENSION  EXTWARN ERROR ]


SourceLocation：表示源代码的位置。See：clang/Basic/SourceLocation.h。SourceLocation因为被嵌入到许多AST中，因此，该类必须足够小。

SourceLocation：通常是和SourceManager一同使用，用于对一个位置信息的两条信息进行编码。见：clang/Basic/SourceManager.h

SourceRange：是SourceLocation.h中类，表示源码的范围：【first，last】。First和last都是SourceLocation




# makefile


libclangAnaylysis：用于进行静态分析用的


libclangRewrite：编辑文本缓冲区（代码重写转换非常重要，如重构）


libclangBasic：诊断、源码定位、源码缓冲区抽象化、输入源文件的文件缓冲区

Clang诊断子系统是一个编译器与人交互的重要部分。诊断是当代码不正确或可疑时产生警告和错误。在clang中，每个诊断产生（最少）一个唯一标识ID、一个相关的英文、SourceLocation源码位置“放置一个^”和一个严重性（例如：WARNING或ERROR）。他可以选择包含一些参数给争端（如使用%0填充字串）以及相关源码区域。

Clang diagnostics   Diagnostic::Level  enum  [ NOTE  WARNING  EXTENSION  EXTWARN ERROR ]


SourceLocation：表示源代码的位置。See：clang/Basic/SourceLocation.h。SourceLocation因为被嵌入到许多AST中，因此，该类必须足够小。

SourceLocation：通常是和SourceManager一同使用，用于对一个位置信息的两条信息进行编码。见：clang/Basic/SourceManager.h

SourceRange：是SourceLocation.h中类，表示源码的范围：【first，last】。First和last都是SourceLocation


clang-tidy：用于代码规范性检查及其一些代码错误的自动修正

clang-format：用于代码风格的自动格式化。

