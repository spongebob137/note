# Linux管道、重定向与通配符

## 总结

管道连接命令，重定向连接命令与文件。最重要的安全区别是：>覆盖旧内容，>>追加内容。通配符由Shell匹配文件名，与grep使用的正则表达式不是同一套规则。

## 标准数据流

    标准输入 stdin -> 命令 -> 标准输出 stdout

## 管道

    command1 | command2

左侧命令的标准输出成为右侧命令的标准输入。

示例：

    grep -i "error" file | wc -l

数据方向：

    grep筛选 -> wc统计

## 重定向

| 符号 | 作用 |
|---|---|
| < | 从文件读取标准输入 |
| > | 覆盖写入目标文件 |
| >> | 追加到目标文件 |
| << | here-document，多行输入 |

示例：

    sort < input.rpt
    sort -u input.rpt > sorted.rpt
    echo "next" >> result.txt

## 错误的数据方向

错误：

    wc -l < grep -i "error" file

因为<后面必须是文件名，Shell会把grep当作文件名。

正确逻辑：

    grep -i "error" file | wc -l

## 通配符

| 通配符 | 文件名匹配含义 |
|---|---|
| * | 任意数量字符 |
| ? | 恰好一个字符 |

示例：

    ls *.txt
    ls txt*
    ls *linux*
    ls txt?

默认情况下，*通常不匹配以点号开头的隐藏文件。

## Shell变量

    echo PATH
    echo $PATH
    echo $HOME
    echo $SHELL

- PATH是不带置换的普通字符串。
- $PATH表示读取变量PATH的值。

## 注释

Shell脚本中，命令位置上的#及其后内容通常为注释。交互式Shell是否接受注释还可能受配置影响。

## 易错点

- >会在命令运行前打开并清空目标文件。
- 不要把处理结果用>写回同一个输入文件。
- <连接文件和命令，|连接两个命令。
- ls | grep ".txt"中的点号属于正则，不是字面句点。
- 反斜杠加空格会把空格纳入参数。

## 数字后端应用

- 把grep结果交给wc或awk统计。
- 将EDA报告写入新文件或追加到汇总报告。
- 使用*.rpt、*.log选择某类文件。

## 自检

- 能画出管道的数据方向。
- 能解释>和>>的风险差异。
- 能说明<为什么不能接另一条命令。
- 能区分*与?。

