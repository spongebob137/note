# grep文本检索

## 总结

grep用于逐行筛选文本，是后端日志排查最常用的Linux命令。必须熟练-n、-i、-v、-E和-c，并牢记grep统计的是匹配行，不一定等于真实错误数量。

## 基本语法

    grep [options] 'pattern' file

grep会输出包含匹配内容的完整行。

## 常用选项

| 选项 | 含义 |
|---|---|
| -n | 显示原文件行号 |
| -i | 忽略大小写 |
| -v | 反向匹配，输出不匹配行 |
| -E | 使用ERE扩展正则 |
| -c | 输出匹配行数 |

选项可以组合：

    grep -niE 'error|warn' file

## 典型操作

查找并显示行号：

    grep -n 'ERROR' file

忽略大小写：

    grep -ni 'error' file

排除INFO：

    grep -v 'INFO' file

匹配多种关键词：

    grep -niE 'error|warn' file

统计匹配行：

    grep -ic 'error' file

或：

    grep -i 'error' file | wc -l

## 匹配行数与出现次数

grep -c统计包含关键词的行数。一行中即使出现多个ERROR，也只计为一行。

关键词匹配行也不等于真实错误数量，例如以下文字仍可能匹配：

    0 errors
    no error found

需要结合上下文和工具消息语义判断。

## 点号问题

不严谨：

    grep '.SUBCKT' file

点号会匹配任意字符。精确字面点号：

    grep '\.SUBCKT' file

## grep与cat

两者都能工作：

    cat file | grep 'ERROR'
    grep 'ERROR' file

后者更直接，因为grep本身能读取文件。

## 易错点

- -E是大写，表示扩展正则。
- 模式中有空格或特殊符号时应使用引号。
- 使用-n后再sort -u，行号不同会妨碍正文去重。
- 输出太多时可用Ctrl+C中止，但不要连续多次中断EDA工具Shell。

## 数字后端应用

    grep -niE 'error|warn' InCts.log
    grep -n 'WNS' timing.rpt
    grep -n 'VIOLATED' constraint.rpt
    grep -n 'start|finish' run.log

## 自检

- 能解释-n、-i、-v、-E、-c。
- 能统计忽略大小写后的匹配行数。
- 能说明关键词行数为何不是错误数量。
- 能精确匹配字面句点。

