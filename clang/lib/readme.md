# CLANG 分析

最初的C/C++源码经过：词法分析（Lexical analysis）-> 语法分析（Syntactic analysis）-> 语义分析（Semantic analysis）-> 与平台无关的IR（LLVM IR generator）


## 1. 词法分析（Lexical analysis） clang/lib/Lex/ 

### token

词法分析器读入组成源程序的字节流，并将他们组成有意义的词素（Lexeme）序列。对于每个词素，词法分析器产生词单元（token）作为输出，并生成相关符号表。词法库包含了几个紧密相连的类，他们涉及到词法和C源码预处理。

相关诊断: DiagnosticLexKinds.td

词法单元（Token）的定义：TokenKinds.def（clang/include/clang/Basic/）

Clang的保留字定义在TokenKinds.def，如常见的if或for关键字

说明TokenKinds.def中定义了许多诸如TOK和KEYWORD等宏，实际相关宏只有在外部引用程序引用TokenKinds.def前定义才会真正有意义；而且引用位置不同，其意义可能不同


### Preprocessor类 ----- 返回下一个Token

Preprocessor是词法分析中的一个主要接口,可以看到，词法分析时在预处理过程中初始化的即Preprocessor.

在词法分析中，预处理程序被初始调用的主要程序是CompilerInstance::createPreprocessor（lib/Frontend/CompilerInstance.cpp）

其核心程序是Preprocessor::Lex,该程序返回下一个Token


### Token类  clang/include/clang/Lex/Token.h 

Token类用于表述电仪的词法单元。Token被用于词法/预处理单元，但并不会在这些库以外存在（如，Token不会存在于AST中）.



## 语法分析（Syntactic analysis）

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


## 声明上下文，DeclContext类  clang/AST/DeclBase.h

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



