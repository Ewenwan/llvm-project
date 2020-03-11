# 数据流分析的理论 消除冗余代码

## 标量优化（scalar目录）：

死代码消除（BDCE.cpp[code]，ADCE.cpp[code]，DCE.cpp[code]）, 

全局值编号（GVN.cpp[code]）, 

代码提升（ConstantHoisting.cpp[code]），
公共子表达式消除（EarlyCSE.cpp[code]）， 

代码下沉(Sink.cpp[code]), 以及各种循环优化等

## 过程间优化（IPO目录）：

无效参数消除(DeadArgumentElimination.cpp[code]) , 

全局死代码消除(GlobalDCE.cpp[code]),

常量传播（IPConstantPropagation.cpp[code]）， 

循环外提（LoopExtractor.cpp[code]），

稀疏条件常量传播（SCCP.cpp[code]）,

函数合并（MergeFunctions.cpp[code]）等

