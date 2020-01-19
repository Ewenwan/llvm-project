# IRgen optimization opportunities.  CodeGen 生成 IR(中间表示) 代码


将语法树翻译成 LLVM IR 中间代码，做为 LLVM Backend 输入的桥接语言。这样做的好处在前言里也提到了，方便 LLVM Backend 给多语言做相同的优化，做到语言无关。

这个过程中还会跟 runtime 桥接。

      1. 各种类，方法，成员变量等的结构体的生成，并将其放到对应的Mach-O的section中。
      2. Non-Fragile ABI 合成 OBJC_IVAR_$_ 偏移值常量。
      3. ObjCMessageExpr 翻译成相应版本的 objc_msgSend，super 翻译成 objc_msgSendSuper。
      4. strong，weak，copy，atomic 合成 @property 自动实现 setter 和 getter。
      5. @synthesize 的处理。
      6. 生成 block_layout 数据结构
      7. __block 和 __weak
      8. _block_invoke
      9. ARC 处理，插入 objc_storeStrong 和 objc_storeWeak 等 ARC 代码。ObjCAutoreleasePoolStmt 转 objc_autorealeasePoolPush / Pop。自动添加 [super dealloc]。给每个 ivar 的类合成 .cxx_destructor 方法自动释放类的成员变量。


不管编译的语言时 Objective-C 还是 Swift 也不管对应机器是什么，亦或是即时编译，LLVM 里唯一不变的是中间语言 LLVM IR。



## IR 结构
下面是刚才生成的 main.ll 中间代码文件。

              ; ModuleID = ‘main.c’
              source_filename = “main.c”
              target datalayout = “e-m:o-i64:64-f80:128-n8:16:32:64-S128”
              target triple = “x86_64-apple-macosx10.12.0”

              @.str = private unnamed_addr constant [16 x i8] c”Please input a:\00”, align 1
              @.str.1 = private unnamed_addr constant [3 x i8] c”%d\00”, align 1
              @.str.2 = private unnamed_addr constant [16 x i8] c”Please input b:\00”, align 1
              @.str.3 = private unnamed_addr constant [32 x i8] c”a is:%d,b is :%d,count equal:%d\00”, align 1

              ; Function Attrs: nounwind ssp uwtable
              define i32 @main() #0 {
                %1 = alloca i32, align 4
                %2 = alloca i32, align 4
                %3 = bitcast i32* %1 to i8*
                call void @llvm.lifetime.start(i64 4, i8* %3) #3
                %4 = bitcast i32* %2 to i8*
                call void @llvm.lifetime.start(i64 4, i8* %4) #3
                %5 = tail call i32 (i8*, …) @printf(i8* getelementptr inbounds ([16 x i8], [16 x i8]* @.str, i64 0, i64 0))
                %6 = call i32 (i8*, …) @scanf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32* nonnull %1)
                %7 = call i32 (i8*, …) @printf(i8* getelementptr inbounds ([16 x i8], [16 x i8]* @.str.2, i64 0, i64 0))
                %8 = call i32 (i8*, …) @scanf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32* nonnull %2)
                %9 = load i32, i32* %1, align 4, !tbaa !2
                %10 = load i32, i32* %2, align 4, !tbaa !2
                %11 = add nsw i32 %10, %9
                %12 = call i32 (i8*, …) @printf(i8* getelementptr inbounds ([32 x i8], [32 x i8]* @.str.3, i64 0, i64 0), i32 %9, i32 %10, i32 %11)
                call void @llvm.lifetime.end(i64 4, i8* %4) #3
                call void @llvm.lifetime.end(i64 4, i8* %3) #3
                ret i32 0
              }

              ; Function Attrs: argmemonly nounwind
              declare void @llvm.lifetime.start(i64, i8* nocapture) #1

              ; Function Attrs: nounwind
              declare i32 @printf(i8* nocapture readonly, …) #2

              ; Function Attrs: nounwind
              declare i32 @scanf(i8* nocapture readonly, …) #2

              ; Function Attrs: argmemonly nounwind
              declare void @llvm.lifetime.end(i64, i8* nocapture) #1

              attributes #0 = { nounwind ssp uwtable “disable-tail-calls”=“false” “less-precise-fpmad”=“false” “no-frame-pointer-elim”=“true” “no-frame-pointer-elim-non-leaf” “no-infs-fp-math”=“false” “no-nans-fp-math”=“false” “stack-protector-buffer-size”=“8” “target-cpu”=“penryn” “target-features”=“+cx16,+fxsr,+mmx,+sse,+sse2,+sse3,+sse4.1,+ssse3” “unsafe-fp-math”=“false” “use-soft-float”=“false” }
              attributes #1 = { argmemonly nounwind }
              attributes #2 = { nounwind “disable-tail-calls”=“false” “less-precise-fpmad”=“false” “no-frame-pointer-elim”=“true” “no-frame-pointer-elim-non-leaf” “no-infs-fp-math”=“false” “no-nans-fp-math”=“false” “stack-protector-buffer-size”=“8” “target-cpu”=“penryn” “target-features”=“+cx16,+fxsr,+mmx,+sse,+sse2,+sse3,+sse4.1,+ssse3” “unsafe-fp-math”=“false” “use-soft-float”=“false” }
              attributes #3 = { nounwind }

              !llvm.module.flags = !{!0}
              !llvm.ident = !{!1}

              !0 = !{i32 1, !”PIC Level”, i32 2}
              !1 = !{!”Apple LLVM version 8.0.0 (clang-800.0.42.1)”}
              !2 = !{!3, !3, i64 0}
              !3 = !{!”int”, !4, i64 0}
              !4 = !{!”omnipotent char”, !5, i64 0}
              !5 = !{!”Simple C/C++ TBAA”}


LLVM IR 有三种表示格式，第一种是 bitcode 这样的存储格式，以 .bc 做后缀，第二种是可读的以 .ll，第三种是用于开发时操作 LLVM IR 的内存格式。
一个编译的单元即一个文件在 IR 里就是一个 Module，Module 里有 Global Variable 和 Function，在 Function里有 Basic Block，Basic Block 里有 指令 Instructions。 ‘

可以看出最大的是 Module，里面包含多个 Function，每个 Function 包含多个 BasicBlock，BasicBlock 里含有 Instruction，代码非常清晰，这样如果想开发一个新语言只需要完成语法解析后通过 LLVM 提供的丰富接口在内存中生成 IR 就可以直接运行在各个不同的平台。
IR 语言满足静态单赋值，可以很好的降低数据流分析和控制流分析的复杂度。及只能在定义时赋值，后面不能更改。但是这样就没法写程序了，输入输出都没法弄，所以函数式编程才会有类似 Monad 这样机制的原因。

## LLVM IR 优化

使用 O2，O3 这样的优化会调用对应的 Pass 来进行处理，有比如类似死代码清理，内联化，表达式重组，循环变量移动这样的 Pass。可以通过 llvm-opt 调用 LLVM 优化相关的库。

```c

int i = 0;
while (i < 10) {
    i++;
    printf("%d",i);
}

```
对应的 IR 代码是

      %call4 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 1)
      %call4.1 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 2)
      %call4.2 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 3)
      %call4.3 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 4)
      %call4.4 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 5)
      %call4.5 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 6)
      %call4.6 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 7)
      %call4.7 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 8)
      %call4.8 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 9)
      %call4.9 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 10)

可以看出来这个 while 在 IR 中就是重复的打印了10次，那要是我把10改成100是不是会变成打印100次呢？

我们改成100后，再次生成 IR 可以看到 IR 变成了这样：


           br label %while.body

          while.body:                                       ; preds = %while.body, %entry
            %i.010 = phi i32 [ 0, %entry ], [ %inc, %while.body ]
            %inc = add nuw nsw i32 %i.010, 1
            %call4 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @.str.1, i64 0, i64 0), i32 %inc)
            %exitcond = icmp eq i32 %inc, 100
            br i1 %exitcond, label %while.end, label %while.body

          while.end:                                        ; preds = %while.body
            %2 = load i32, i32* %a, align 4, !tbaa !2
            %3 = load i32, i32* %b, align 4, !tbaa !2
            %add = add nsw i32 %3, %2
            %call5 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([11 x i8], [11 x i8]* @.str.3, i64 0, i64 0), i32 %add)
            call void @llvm.lifetime.end(i64 4, i8* nonnull %1) #3
            call void @llvm.lifetime.end(i64 4, i8* nonnull %0) #3
            ret i32 0
          }
  
  
这里对不同条件生成的不同都是 Pass 优化器做的事情。解读上面这段 IR 需要先了解下 IR 语法关键字，如下：

      @ - 代表全局变量
      % - 代表局部变量
      alloca - 指令在当前执行的函数的堆栈帧中分配内存，当该函数返回到其调用者时，将自动释放内存。
      i32：- i 是几这个整数就会占几位，i32就是32位4字节
      align - 对齐，比如一个 int,一个 char 和一个 int。单个 int 占4个字节，为了对齐只占一个字节的 char需要向4对齐占用4字节空间。
      Load - 读出，store 写入
      icmp - 两个整数值比较，返回布尔值
      br - 选择分支，根据 cond 来转向 label，不根据条件跳转的话类似 goto
      indirectbr - 根据条件间接跳转到一个 label，而这个 label 一般是在一个数组里，所以跳转目标是可变的，由运行时决定的
      label - 代码标签

      br label %while.body

如上面表述，br 会选择跳向 while.body 定义的这个标签。这个标签里可以看到

      %exitcond = icmp eq i32 %inc, 100
        br i1 %exitcond, label %while.end, label %while.body

这段，icmp 会比较当前的 %inc 和定义的临界值 100，根据返回的布尔值来决定 br 跳转到那个代码标签，真就跳转到 while.end 标签，否就在进入 while.body 标签。这就是 while 的逻辑。通过br 跳转和 label 这种标签的概念使得 IR 语言能够成为更低级兼容性更高更方便转向更低级语言的语言。
  
  
  
//===---------------------------------------------------------------------===//

The common pattern of
--
short x; // or char, etc
(x == 10)
--
generates an zext/sext of x which can easily be avoided.

//===---------------------------------------------------------------------===//

Bitfields accesses can be shifted to simplify masking and sign
extension. For example, if the bitfield width is 8 and it is
appropriately aligned then is is a lot shorter to just load the char
directly.

//===---------------------------------------------------------------------===//

It may be worth avoiding creation of alloca's for formal arguments
for the common situation where the argument is never written to or has
its address taken. The idea would be to begin generating code by using
the argument directly and if its address is taken or it is stored to
then generate the alloca and patch up the existing code.

In theory, the same optimization could be a win for block local
variables as long as the declaration dominates all statements in the
block.

NOTE: The main case we care about this for is for -O0 -g compile time
performance, and in that scenario we will need to emit the alloca
anyway currently to emit proper debug info. So this is blocked by
being able to emit debug information which refers to an LLVM
temporary, not an alloca.

//===---------------------------------------------------------------------===//

We should try and avoid generating basic blocks which only contain
jumps. At -O0, this penalizes us all the way from IRgen (malloc &
instruction overhead), all the way down through code generation and
assembly time.

On 176.gcc:expr.ll, it looks like over 12% of basic blocks are just
direct branches!

//===---------------------------------------------------------------------===//
