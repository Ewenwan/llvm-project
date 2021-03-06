//===----------------------------------------------------------------------===//
//                         ModuleMaker Sample project
//===----------------------------------------------------------------------===//

This project is an extremely simple example of using some simple pieces of the 
LLVM API.  The actual executable generated by this project simply emits an 
LLVM bitcode file to standard output.  It is designed to show some basic 
usage of LLVM APIs, and how to link to LLVM libraries.

# makefile 
```
LLVM_CONFIG = llvm-config
LLVM_CXXFLAGS += $(shell $(LLVM_CONFIG) --cxxflags)
LLVM_LDFLAGS := $(shell $(LLVM_CONFIG) --ldflags)
LLVM_LIBS = $(shell $(LLVM_CONFIG) --libs bitwriter core support)

llvm_model_src = ModuleMaker.cpp


test_model:
	g++ $(llvm_model_src) $(LLVM_CXXFLAGS) $(LLVM_LIBS) $(LLVM_LDFLAGS) -lpthread -ldl -o ModuleMaker
```

这是一个Makefile。它是一个例子ModuleMaker的编译文件。这个ModuleMaker例子本身是LLVM源码中llvm/examples/ModuleMaker/目录下的一个例子，它演示的如果凭空构建一个LLVM IR的Module。我这里写了这个Makefile以后，可以在已经安装LLVM的系统（Linux）上单独的编译这个例子，而不需要依赖LLVM的源码，也不再需要在LLVM源码中编译这个例子。


从这个Makefile中可以看出，编译所需要的环境变量，包括LLVM_CXXFLAGS、LLVM_LDFLAGS和LLVM_LIBS都是直接通过llvm-config直接获取的，这就完全不需要用户在编译项目的时候设置环境变量或者传递变量，项目可以直接获取系统里的环境变量，大大方便了项目的构建。只有真正的构建过基于LLVM项目的人，才明白使用了llvm-config之后会多方便。

同时，llvm-config还可以以`llvm-config --libs`这样的形式在Makefile中使用，或者在命令行中使用，这样的使用形式是获取这个命令在shell执行后所输出的信息。这里需要指出的是“`”这个符号，并不是“‘”。这两者是有区别的，前者是和“~”同键的符号，后者是和“"” 同键的符号。这一点是一定要区别的，否者系统无法识别。


llvm-config的主要参数如下：


–version

Print the version number of LLVM.
-help

Print a summary of llvm-config arguments.
–prefix

Print the installation prefix for LLVM.
–src-root

Print the source root from which LLVM was built.
–obj-root

Print the object root used to build LLVM.
–bindir

Print the installation directory for LLVM binaries.
–includedir

Print the installation directory for LLVM headers.
–libdir

Print the installation directory for LLVM libraries.
–cxxflags

Print the C++ compiler flags needed to use LLVM headers.
–ldflags

Print the flags needed to link against LLVM libraries.
–libs

Print all the libraries needed to link against the specified LLVM components, including any dependencies.
–libnames

Similar to –libs, but prints the bare filenames of the libraries without -l or pathnames. Useful for linking against a not-yet-installed copy of LLVM.
–libfiles

Similar to –libs, but print the full path to each library file. This is useful when creating makefile dependencies, to ensure that a tool is relinked if any library it uses changes.
–components

Print all valid component names.
–targets-built

Print the component names for all targets supported by this copy of LLVM.
–build-mode

Print the build mode used when LLVM was built (e.g. Debug or Release)

