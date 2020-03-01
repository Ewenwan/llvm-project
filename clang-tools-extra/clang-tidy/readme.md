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


## clang-tidy使用中的一些点进行一下总结。

1、clang-tidy及其批量运行脚本run-clang-tidy.py在Clang/LLVM的预编译发布包中都有，但是位于不同的目录。其中，clang-tidy位于：发布包主目录/bin目录之下；run-clang-tidy.py位于：发布包主目录/shared/clang/目录之下。

例如：

以clang+llvm-8.0.0-x86_64-linux-gnu-ubuntu-16.04发布包为例，clang-tidy位于：clang+llvm-8.0.0-x86_64-linux-gnu-ubuntu-16.04/bin/clang-tidy；run-clang-tidy.py位于：clang+llvm-8.0.0-x86_64-linux-gnu-ubuntu-16.04/share/clang/run-clang-tidy.py。

2、run-clang-tidy.py与clang-tidy的运行，都依赖于compile_commands.json。同时，这两者都可以通过“-checks=”来设定检查规则。或者不使用“-checks=”选项，而在项目主目录之下添加.clang-tidy文件，在里面编写项目的检查规则，这种方式更加适合对整个项目进行定制化的规则编写。.clang-tidy文件并不是必须放在主目录之下，只是通常放在主目录之下方便对整个项目进行检查。

3、run-clang-tidy.py的运行，不但依赖于clang-tidy，同时也依赖于clang-apply-replacements。clang-apply-replacements和clang-tidy在同一个目录。

run-clang-tidy.py开头的注释中，写明了：” Runs clang-tidy over all files in a compilation database. Requires clang-tidy and clang-apply-replacements in $PATH.“

4、因为run-clang-tidy.py在预编译发布包里和clang-tidy、clang-apply-replacements不在同一个目录，在运行时可以通过指定run-clang-tidy.py的“-clang-tidy-binary=”和“-clang-apply-replacements-binary=”两个选项，来确定clang-tidy、clang-apply-replacements的路径。

例如：

项目主目录之下存在了.clang-tidy和compile_commands.json文件，同时预编译发布包clang+llvm-8.0.0-x86_64-linux-gnu-ubuntu-16.04也在主目录之下，那么在主目录之下运行如下命令可以对整个项目进行检查：

./clang+llvm-8.0.0-x86_64-linux-gnu-ubuntu-16.04/share/clang/run-clang-tidy.py -clang-tidy-binary='./clang+llvm-8.0.0-x86_64-linux-gnu-ubuntu-16.04/bin/clang-tidy' -clang-apply-replacements-binary='./clang+llvm-8.0.0-x86_64-linux-gnu-ubuntu-16.04/bin/clang-apply-replacements' ./

5、clang-tidy的检查规则编写的时候，规则名称前面带有“-”的是让该规则失效，规则名称直接写的是要使用该规则。

例如：

clang的主目录之下的.clang-tidy文件中的规则：

Checks: '-*,clang-diagnostic-*,llvm-*,misc-*,-misc-unused-parameters,-misc-non-private-member-variables-in-classes,-readability-identifier-naming'

这里的“-*”就是要使目前所有的规则失效，“clang-diagnostic-*,llvm-*,misc-*”是要让clang-diagnostic-、llvm-、misc-开头的规则可用，“-misc-unused-parameters,-misc-non-private-member-variables-in-classes,-readability-identifier-naming”是让这具体的三个规则失效。

6、clang-tidy的检查清单的官方文档位于：Extra Clang Tools 11 documentation。其中，有些规则是可以进一步对其子规则进行设置的。

例如：

规则readability-identifier-naming，就可以进一步细分。其细分文档位于：clang-tidy - readability-identifier-naming。

clang的.clang-tidy之中也有这个规则的一部分细分



