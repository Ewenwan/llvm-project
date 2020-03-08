# Multi-Level Intermediate Representation

See [https://mlir.llvm.org/](https://mlir.llvm.org/) for more information.


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
