# Linux查看、排序与查找

## 总结

小文件可以用cat完整显示，大型日志应使用head、tail或less。sort默认只输出排序结果，不修改源文件；find从指定目录递归查找文件名。

## cat

    cat file
    cat -n file
    cat -b file
    cat -E file

| 选项 | 含义 |
|---|---|
| -n | 给所有输出行编号 |
| -b | 只给非空行编号 |
| -E | 在每行末尾显示$标记 |

同时显示多个文件：

    cat file1 file2

## 大型文件

    head -n 10 file
    tail -n 10 file
    less file

less常用键：

| 按键 | 作用 |
|---|---|
| Space | 下一页 |
| b | 上一页 |
| g | 开头 |
| G | 结尾 |
| /word | 搜索 |
| n、N | 同向、反向继续 |
| q | 退出 |

## wc

    wc -l file
    command | wc -l

- wc -l file通常输出行数和文件名。
- wc -l < file只输出行数。
- 管道输入给wc时通常只有统计数字，没有文件名字段。

## sort

    sort file
    sort -u file

- 默认按字符顺序输出。
- -u在排序后去除完全重复的整行。
- 默认不修改输入文件。
- 若每行前有不同的行号，即使正文相同，sort -u仍视为不同整行。

## find

    find . -name "*.rpt"
    find ~/linux_vim_lab -name "train.cdl"

结构：

    find 起始路径 查找条件

双引号防止Shell在find执行前提前展开通配符。

## ls通配与find递归

    ls *.log

通常只匹配当前目录。

    find . -name "*.log"

从当前目录开始递归搜索所有子目录。

## 易错点

- cat大型日志可能刷屏，Ctrl+C可中止。
- cat -n中的-n是编号；head -n 10中的-n指定行数。
- sort -u只删除完全相同的整行。
- find的*.rpt应加引号。

## 数字后端应用

- 查看日志头部的工具版本和环境。
- 查看日志尾部的完成状态和运行时间。
- 查找各stage下的rpt、log、tcl、sdc。
- 对报告关键词排序去重。

## 自检

- 能选择cat、head、tail或less。
- 能解释cat -n与cat -b。
- 能说明sort是否修改源文件。
- 能解释ls *.log与find . -name "*.log"的范围差异。

