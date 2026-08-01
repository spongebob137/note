# 01 Tcl命令与单词

## 总结

Tcl脚本由命令组成，命令由单词组成。换行或分号分隔命令，空格或Tab划分单词；第一个单词是命令名，其余单词是参数。

对应教程第5页“脚本、命令和单词符号”。

## 1. Tcl脚本与命令

一个Tcl脚本可以包含一条或多条命令。命令之间使用以下方式分隔：

- 换行符
- 分号 `;`

下面两种写法等价：

```tcl
set training_a 10
set training_b 20
```

```tcl
set training_a 10; set training_b 20
```

交互执行一段包含多条命令的脚本时，通常显示最后一条命令的返回值。

## 2. Tcl单词

一条命令由一个或多个单词组成，单词之间通常由空格或Tab分隔。

```tcl
set training_a 10
```

包含3个单词：

```text
set          第一个单词，命令名
training_a   第二个单词，第一个参数
10           第三个单词，第二个参数
```

Tcl解释器先划分单词并执行必要的置换，再把第一个单词作为命令名执行。

## 3. set命令

设置或修改变量：

```tcl
set training_a 10
```

读取变量：

```tcl
set training_a
```

预期结果均为：

```text
10
```

## 4. puts命令

```tcl
puts Tcl_training
```

预期输出：

```text
Tcl_training
```

该命令包含两个单词：命令名 `puts` 和参数 `Tcl_training`。

## 5. 参数数量错误

```tcl
set training_msg hello world
```

Tcl将它拆成4个单词：

```text
set
training_msg
hello
world
```

但设置变量时，`set`只接受一个变量名和一个变量值，因此产生：

```text
wrong # args: should be "set varName ?newValue?"
```

包含空格的内容需要通过反斜杠、双引号或花括号组织成一个Tcl单词。

## 6. FC Shell注意点

- 不输入教程中的 `%`。
- 不输入提示符 `fc_shell>`。
- 创建练习变量不会修改设计数据库。
- 练习变量统一使用 `training_` 前缀。

## 7. 自检

- 能解释命令名和参数的关系。
- 能数出一条简单命令包含几个Tcl单词。
- 能用换行和分号分隔命令。
- 能解释带空格赋值为什么可能出现参数数量错误。
