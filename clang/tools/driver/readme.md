# Driver 我们和 LLVM 交互的实现 

Driver 是 Clang 面对用户的接口，用来解析 Option 设置，判断决定调用的工具链，最终完成整个编译过程。

整个 Driver 源码的入口函数就是 driver.cpp 里的 main() 函数。

从这里可以作为入口看看整个 driver 是如何工作的，这样更利于我们以后轻松动手驾驭 LLVM。

[clang驱动解析参考](https://xiaozhuanlan.com/topic/3169254807)
