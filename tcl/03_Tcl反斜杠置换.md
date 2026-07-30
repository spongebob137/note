# 03 Tcl反斜杠置换

## 总结

反斜杠用于转义特殊字符、插入换行或Tab等控制字符，以及连接物理上的多行命令。续行时反斜杠必须是该行最后一个字符。

对应教程第7-8页“反斜杠置换”。

## 1. 主要用途

反斜杠 `\` 可以：

- 让特殊字符按字面含义处理。
- 插入换行、Tab等控制字符。
- 将一条较长命令续到下一物理行。

## 2. 转义空格

```tcl
set training_msg multiple\ space
```

预期值：

```text
multiple space
```

`\ `让空格保留在同一个Tcl单词中。

## 3. 转义美元符号

```tcl
set training_price money\ \$3333
```

预期值：

```text
money $3333
```

`\$`表示普通美元符号，不触发变量置换。

## 4. 转义方括号

```tcl
set training_array array\[2\]
```

预期值：

```text
array[2]
```

如果不转义，Tcl会尝试把方括号中的内容当作命令执行。

## 5. 常见反斜杠序列

| 写法 | 含义 |
|---|---|
| `\n` | 换行 |
| `\t` | Tab |
| `\r` | 回车 |
| `\\` | 字面反斜杠 |
| `\ ` | 字面空格，避免分词 |
| `\$` | 字面美元符号 |
| `\[`、`\]` | 字面方括号 |

示例：

```tcl
set training_lines first\nsecond
puts $training_lines
```

输出：

```text
first
second
```

```tcl
set training_columns stage\tstatus
puts $training_columns
```

在两个字段之间插入Tab。

```tcl
set training_backslash one\\two
```

变量值中包含一个字面反斜杠。

## 6. 十六进制与八进制字符

```tcl
set training_hex \x48
set training_octal \110
```

二者都得到字符：

```text
H
```

该用法需要认识，但数字后端日常脚本中使用频率较低。

## 7. 反斜杠续行

反斜杠位于物理行的最后一个字符时，可以把命令续到下一行：

```tcl
set training_total [expr {\
    10 + 20 + 30\
}]
```

预期结果：

```text
60
```

注意：反斜杠后不能再有空格，否则它转义的是空格，而不是换行。

## 8. 常见错误

- 把 `\` 与Linux路径分隔符 `/` 混淆。
- 本来想输入 `$`，却忘记转义，导致读取不存在的变量。
- 本来想输入 `[2]`，却触发命令置换。
- 用反斜杠续行时，在反斜杠后留下空格。

## 9. 自检

- 能解释 `\ `、`\$`、`\n`、`\t`、`\\`。
- 能区分字面方括号与命令置换方括号。
- 能正确使用反斜杠续行。
