
# 在LLVM源码的目录下，有一个目录叫做examples，这个目录下边有几个LLVM的例子，初学者一般不会太关注这些例子，但是这些例子确实是精华中的精华，在LLVM的学习过程中不可避免的要对这些例子所涉及内容要精通。所以希望大家一定要对这些例子重视起来，它们都是经典中的经典。从我个人而言，从这些例子中学到了很多，文档说了很多都没能让人明白的东西，在这里，一个简单的例子就讲的很清楚了。

        首先，要想执行这些例子，可以在cmake的时候加上参数 -DLLVM_BUILD_EXAMPLES=ON 。这样在最终编译出的bin目录下的可执行文件中，就多了这几个例子的可执行文件，运行这些文件就可以看到这些例子的输出。
         接下来，我们看看这些例子到底都讲了什么东西。
         
        llvm/examples/BrainF （Need To Do）
        llvm/examples/ExceptionDemo  
        
（Need To Do）从名字上来看，这个例子介绍的是LLVM的Exception模型，但是这块我目前还没有涉猎，没有进行研究，所以暂时不做介绍。

        llvm/examples/Fibonacci   
 这个例子很像HowToUseJIT，所做的工作差不多，创建Module，创建Function，然后使用JIT，然后让函数运行。唯一的不同之处是这个例子实现的是Fibonacci，并且在里面展示了如何创建了判断分支语句。
 
        llvm/examples/HowToUseJIT
 这个例子以一个很简单的程序演示了如何使用JIT，同时因为要演示使用JIT，顺便构建了一个新的Module，并且在这个Module中构建了两个函数，这两个函数之间还存在调用关系。所以这个例子不仅仅展示了如何使用JIT，还顺便做了ModuleMaker的工作。
 
        llvm/examples/Kaleidoscope  
 这个例子和后面的OCaml-Kaleidoscope都有对应的文档专门介绍：http://llvm.org/docs/tutorial/ 。它展示了如何在LLVM之上构建一个支持一门自己定义的编程语言的编译器。这不但有利于理解编译器的工作流程，也非常有利于理解LLVM。是非常适合刚接触LLVM的人学习的一个例子，因为它配有详细的文档说明，并且贯穿了编译器的整个流程，没有理由不认真的学习它。
 
        llvm/examples/ModuleMaker  
  这个例子展示了如果创建Module，并且在它之中创建函数的过程。代码很少，但是行行都是经典，是一直可以用来参考的例子。基本上参照它，就可以完成一个Module的大部分功能的创建。
  
        llvm/examples/OCaml-Kaleidoscope 
  这个例子和Kaleidoscope例子的区别在于，它是Objective Caml版本的Kaleidoscope。除此之外，并无其他。
  
        llvm/examples/ParallelJIT 这个例子是个并行版本的HowToUseJIT。
   所以适合需要了解如何在LLVM之下生成一个Module，并且要支持并行和JIT的人去认真学习一下。代码不多，但是把问题介绍的很清楚。

