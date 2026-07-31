# Tcl 学习问题

## 总结

当前最需要避免的错误是混淆“变量名”和“变量值”：需要变量名的命令写 `training`，需要读取变量值时写 `$training`。Tcl 命令中应使用英文半角引号。

## Q1：怎样查看变量 training 的内容？

结论：变量名不包含 `$`；`$` 表示变量置换。

```tcl
set training
puts $training
```

`set training` 按变量名读取，`puts $training` 先将 `$training` 替换为变量值再输出。

## Q2：`append $trainging “CTS signoff”` 是否给 training 追加了字符？

结论：没有按预期修改 `training`。

错误点：

- `trainging` 拼写与 `training` 不同。
- `append` 的第一个参数需要变量名，不能写 `$training`。
- 中文弯引号 `“ ”` 不是 Tcl 的分组符号。
- `append` 不会自动补空格。

正确的字符串追加：

```tcl
append training " CTS signoff"
```

把包含空格的内容作为一个 List 元素追加：

```tcl
lappend training "CTS signoff"
```
