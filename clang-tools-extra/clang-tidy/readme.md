# 静态检查 bug修复
1、clang-tidy是基于AST的静态检查工具。因为它基于AST,所以要比基于正则表达式的静态检查工具更为精准，但是带来的缺点就是要比基于正则表达式的静态检查工具慢一点。也是因为它基于AST，所以clang-tidy运行的时候需要知道编译命令。

2、clang-tidy不仅仅可以做静态检查，还可以做一些修复工作。

3、clang-tidy是基于LibTooling的工具。而LibTooling是一个库，这个库主要是为了基于Clang编写单独的工具。clang-tidy属于clang的extra tools，这个和clang的tools在不同层次的源码目录，是clang/tools和clang/tools/extra的差别。

4、clang-tidy通过添加check来添加检查规则，目前已经有一系列的check：Extra Clang Tools 10 documentation 通过clang-tidy -list-checks会列出默认开启的的check，clang-tidy-tidy -list-checks -checks=* 会列出所有的check。

5、clang-tidy每次针对一个TU，即单个cpp文件，无法跨TU处理。但是，可以使用clang-tidy/tool/源码目录之下的run-clang-tidy.py去批量检查文件。

6、clang-tidy还可以调用clang static analyzer的check.(clang-tidy has its own checks and can also run Clang static analyzer checks.)

7、clang-tidy因为需要知道编译命令，所以必须通过compile_commands.json获取编译命令，或者通过在clang-tidy后通过--添加编译选项。其中，compile_commands.json的生成可以参考文档LibTooling，可以理解为基于LibTooling的都需要去生成该文件。compile_commands.json主要是包含各个目录的编译命令，在cmake构建系统生成它可以通过-DCMAKE_EXPORT_COMPILE_COMMANDS=ON;在makefile构建系统生成它可以通过compiledb。其他构建系统我尝试了gn和ninja，gn可以通过gn gen . --export-compile-commands这种形式生成json文件，ninja可以通过ninja -t compdb cxx cc > compile_commands.json这种形式生成json文件。目前已经有人总结了更多的使用方式，可以参考MaskRay/ccls 和 Compilation database这两篇文章。

8、系统上clang-tidy可运行之后，run-clang-tidy.py可以直接copy到待测试目录使用。

9、clang-tidy基于C++开发。

