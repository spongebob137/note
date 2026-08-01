# 02 Tcl变量置换与命令置换

## 总结

$变量名用于取得变量值，[命令]用于执行内部命令并取得返回值。变量置换只替换文字，不自动进行数学计算；计算应交给expr。

对应教程第6-7页“置换、变量置换、命令置换”。

## 1. 置换概念

Tcl执行命令前，会先把特殊形式替换为对应的值：

```text
$变量名     变量置换
[命令]      命令置换
```

## 2. 变量置换

```tcl
set training_x 10
puts $training_x
```

执行 `puts` 前，`$training_x` 被替换成 `10`，因此输出：

```text
10
```

变量置换只取得变量值，不负责数学计算。

```tcl
set training_text $training_x+100
```

得到的是字符串：

```text
10+100
```

而不是数值 `110`。

## 3. expr表达式求值

```tcl
expr {$training_x + 100}
```

预期结果：

```text
110
```

推荐给 `expr` 的表达式加花括号：

```tcl
expr {$training_x + 100}
```

这比旧式写法 `expr $training_x+100` 更安全、清晰。

## 4. 命令置换

```tcl
set training_sum [expr {$training_x + 100}]
```

执行顺序：

1. 先执行方括号中的 `expr`。
2. `expr`返回 `110`。
3. `110`替换整个方括号。
4. 最终相当于执行 `set training_sum 110`。

检查：

```tcl
puts $training_sum
```

预期输出：

```text
110
```

## 5. 错误示例

```tcl
set training_bad expr {$training_x + 100}
```

这里 `expr` 不在命令的第一个单词，也没有放进 `[]`，因此它只是普通参数。整条命令参数过多，会产生 `wrong # args`。

## 6. 对比总结

```text
set training_text $training_x+100
结果：10+100

set training_sum [expr {$training_x + 100}]
结果：110
```

```text
$var       取得变量值
[command]  执行命令并取得返回值
expr       对表达式求值
```

## 7. 自检

- 能解释 `$` 与 `[]` 的区别。
- 能说明为什么变量置换不等于数学计算。
- 能说明嵌套命令的执行顺序。
- 能使用推荐的 `expr {...}` 写法。
