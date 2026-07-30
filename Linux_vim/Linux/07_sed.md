# sed流编辑

## 总结

sed适合按行选择、删除和替换文本。默认sed只输出处理结果，不修改源文件；-i才原地修改。真实项目中优先输出到新文件，必须原地修改时使用备份形式。

## 基本语法

    sed [options] 'script' input_file

sed逐行读取输入，并按script处理。

## 选择并输出

    sed -n '/AVDD/p' file

| 部分 | 含义 |
|---|---|
| -n | 关闭默认逐行输出 |
| /AVDD/ | 匹配包含AVDD的行 |
| p | 输出匹配行 |

## 删除匹配行

    sed '/AVDD/d' file

这里的d从本次输出流中删除匹配行，默认不会修改源文件。

## 替换

    sed 's/DGND/VSS/g' file

| 部分 | 含义 |
|---|---|
| s | substitute |
| DGND | 旧内容 |
| VSS | 新内容 |
| g | 每行替换所有匹配 |

不带g时，每行只替换第一处。

## 替代分隔符

以下写法含义相同：

    sed 's/DGND/VSS/g' file
    sed 's#DGND#VSS#g' file
    sed 's@DGND@VSS@g' file

处理包含大量路径分隔符的字符串时，#或@更清晰。

## 输出到新文件

    sed 's/DGND/VSS/g' input > output

不要把输出重定向回同一个输入文件：

    sed 's/DGND/VSS/g' input > input

Shell会先清空目标文件，可能使输入内容丢失。

## 原地修改

    sed -i 's/DGND/VSS/g' file

安全练习：

    sed -i.bak 's/DGND/VSS/g' file

通常生成修改后的file和修改前的file.bak。

## 易错点

- sed输出发生变化不代表源文件已修改。
- -i不经过确认，可能一次修改大量内容。
- 替换命令末尾的g表示每行全部匹配，不是递归。
- sed和awk的程序部分优先使用单引号。

## 数字后端应用

- 在个人副本中替换net、corner或路径名称。
- 删除不需要的报告行并输出清理版本。
- 将日志格式标准化后交给awk统计。

## 自检

- 能区分sed默认处理与sed -i。
- 能拆解s/old/new/g。
- 能安全生成新结果文件。
- 能解释为什么不能用>写回同一输入文件。

