# Tcl 重点速查

## 总结

本文件是 Tcl 笔记的检索入口，只收录会议指定的重点和对应笔记位置。当前已完成变量和 List，正在学习控制流；后续只补过程、常用字符串及 EDA collection 应用。

## 重点索引

| 想查什么 | 核心命令 | 笔记 |
|---|---|---|
| 设置、读取变量 | `set`、`puts`、`$var` | [06_Tcl变量与数组.md](06_Tcl变量与数组.md) |
| 字符串与 List 追加 | `append`、`lappend` | [06_Tcl变量与数组.md](06_Tcl变量与数组.md)、[09_Tcl_List增删与范围.md](09_Tcl_List增删与范围.md) |
| 构造、合并 List | `list`、`concat` | [08_Tcl_List基础.md](08_Tcl_List基础.md) |
| 取元素、取长度 | `lindex`、`llength` | [08_Tcl_List基础.md](08_Tcl_List基础.md) |
| 插入、替换、截取 | `linsert`、`lreplace`、`lrange` | [09_Tcl_List增删与范围.md](09_Tcl_List增删与范围.md) |
| 搜索、排序、拆分和连接 | `lsearch`、`lsort`、`split`、`join` | [10_Tcl_List搜索排序与转换.md](10_Tcl_List搜索排序与转换.md) |
| 条件与循环 | `if`、`foreach`、`while`、`for` | [11_Tcl控制流.md](11_Tcl控制流.md) |
| 跳过或结束循环 | `continue`、`break` | [11_Tcl控制流.md](11_Tcl控制流.md) |
| 定义可复用过程 | `proc`、`return`、`global` | [12_Tcl过程.md](12_Tcl过程.md) |
| 字符串比较与处理 | `string compare` 等 | [13_Tcl字符串.md](13_Tcl字符串.md) |
| 遍历 EDA 对象 | collection、attribute、`bbox`、`lindex` | [14_Tcl_EDA集合与属性.md](14_Tcl_EDA集合与属性.md) |

## 高频写法

```tcl
set training_stage cts
puts $training_stage

set training_stages {floorplan place cts route signoff}
puts [llength $training_stages]
puts [lindex $training_stages 2]

foreach stage $training_stages {
    puts $stage
}
```

## 当前进度

- 已完成：变量、表达式、List、控制流、过程、字符串会议重点、EDA Tcl 实操。
- 本轮 Tcl 会议重点已全部覆盖。
