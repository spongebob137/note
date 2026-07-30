# awk字段处理

## 总结

awk把每一行视为一条记录，再按分隔符拆成字段。$0表示整行，$1、$2表示字段；-F设置输入分隔符，OFS控制输出字段间隔。它适合提取和格式化报告。

## 三剑客分工

| 工具 | 擅长 |
|---|---|
| grep | 查找和筛选行 |
| sed | 删除、替换、编辑文本流 |
| awk | 字段提取、条件判断和格式化 |

## 基本语法

    awk [options] 'pattern {action}' file

省略pattern时，每行都执行action。

## 记录与字段

假设一行是：

    DGND ground_net

字段：

    $0 = DGND ground_net
    $1 = DGND
    $2 = ground_net

操作：

    awk '{print $0}' file
    awk '{print $1}' file
    awk '{print $1,$2}' file

默认以空格或Tab分隔连续字段。

## 固定文字与字段变量

    awk '{print "first",$1,"second",$2}' file

- awk程序外层使用单引号。
- 固定文字使用双引号。
- 字段变量$1、$2不加双引号。
- print各项之间的逗号使用OFS输出。

## 条件

    awk '$1 == "DGND" {print $2}' file
    awk '/DGND/ {print $1,$2}' file

第一条按字段值判断，第二条按正则匹配整行。

## 输入分隔符FS

冒号分隔：

    awk -F ':' '{print $1,$7}' file

- -F设置FS。
- $1是第一个字段。
- $7是第七个字段。

## 输出分隔符OFS

    awk -F ':' -v OFS=' -> ' '{print $1,$7}' file

常用选项：

| 选项 | 含义 |
|---|---|
| -F | 设置输入字段分隔符 |
| -v | 定义或修改awk变量 |
| -f | 从脚本文件读取awk程序 |

## print与printf

print自动换行：

    awk '{print $1,$2}' file

printf必须显式使用换行：

    awk '{printf "%-10s %s\n",$1,$2}' file

## 管道中的awk

grep -n输出通常是：

    行号:正文

提取行号：

    grep -n 'ERROR' file | awk -F ':' '{print $1}'

## 易错点

- "$1"输出字面文字，不是字段值。
- wc从管道读取时往往只有$1，没有文件名$2。
- 分隔符必须根据实际数据选择，冒号数据不能使用#分隔。
- awk默认输出不修改源文件。

## 数字后端应用

- 提取时序报告的指定列。
- 提取grep结果中的原日志行号。
- 将cell、pin、slack等字段格式化输出。
- 使用OFS生成便于阅读的汇总。

## 自检

- 能解释$0、$1、$2。
- 能使用-F处理冒号分隔文本。
- 能解释FS与OFS。
- 能给统计数字添加标签并输出到文件。

