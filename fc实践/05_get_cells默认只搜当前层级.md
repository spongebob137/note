# get_cells 默认只搜当前层级：选中 macro 失败的层级问题

## 总结

【易错】`get_cells -filter "is_hard_macro==true"` 默认只搜索**当前层级**的 cell；macro 位于设计子层次中时返回空 collection，`change_selection` 表现为"没选中"。层次化设计中选 macro 必须加 `-hierarchical` 逐层下搜，或用 `get_flat_cells` 打平取叶子单元。

## 问题

```tcl
change_selection [get_cells -filter "is_hard_macro==true"]
```

执行后 GUI 没有选中任何对象，无报错。

## 结论

设计有层次结构时，macro 在子模块内部（如 `u_xxx/u_ram0`），顶层搜索找不到。改用：

```tcl
change_selection [get_cells -hierarchical -filter "is_hard_macro==true"]
# 或
change_selection [get_flat_cells -filter "is_hard_macro==true"]
```

## 原因

`get_cells` 手册定义：默认 "relative to the current instance"（只搜当前层）；`-hierarchical` 才 "searches level-by-level"（*Fusion Compiler Tool Commands, T-2022.03-SP4, get_cells 条目, p.1728*）。空 collection 传给 change_selection 不报错、只是没有对象可选，所以失败是"静默"的。

## 正确写法

```tcl
# 两个等价选法
set macros [get_cells -hierarchical -filter "is_hard_macro==true"]
set macros [get_flat_cells -filter "is_hard_macro==true"]

# 选中前永远先验证数量
sizeof_collection $macros
```

## 验证

- `sizeof_collection` 对比两种写法：顶层返回 0 / 全层级返回实际 macro 数，即证实层级原因；
- GUI 高亮数量与 sizeof_collection 一致。

## 通用教训

【重点】凡 collection 类命令（get_cells/get_pins/get_nets…）结果为空或少于预期，**第一反应检查层级**：当前 instance 是什么、是否需要 `-hierarchical` 或 flat 版本。空 collection 在 FC 里通常不报错，静默失败是最难排查的形态。
