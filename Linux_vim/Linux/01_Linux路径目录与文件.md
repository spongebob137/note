# Linux路径、目录与文件

## 总结

Linux操作首先判断当前目录，再判断目标路径。相对路径从当前目录出发，绝对路径从根目录开始。文件操作前养成先pwd、再ls、后执行的习惯。

## 提示符信息

示例：

    [yutaosun_sd@srv244:] ~/linux_vim_lab>>

通常包含当前用户名、服务器、当前目录和命令提示符。

## 路径符号

| 符号 | 含义 |
|---|---|
| / | 根目录或路径分隔符 |
| . | 当前目录 |
| .. | 上一级目录 |
| ~ | 当前用户家目录 |
| - | 上一次所在目录 |

## 目录命令

    pwd
    ls
    cd directory
    cd ..
    cd ~
    cd -
    mkdir directory
    mkdir -p parent/child

mkdir -p有两个作用：

- 自动创建缺失的父目录。
- 目标目录已经存在时不报错。

## 相对路径与绝对路径

绝对路径：

    /nobackup/users/yutaosun_sd/linux_vim_lab

相对路径：

    ../reports/timing

若当前位于reports，执行ls reports会寻找reports/reports，而不是当前目录本身。

## 文件命令

    touch file
    cp source destination
    mv source destination
    rm file

- touch：文件不存在时创建空文件；存在时更新时间。
- cp：复制，原文件仍保留。
- mv：移动或重命名，原路径不再保留。
- rm：删除，通常难以恢复。

常用安全选项：

    cp -i
    mv -i
    rm -i

## inode

查看：

    ls -li file1 file2

普通cp会创建新inode，所以两个文件内容和大小可以相同，但彼此独立。

同一文件系统中的硬链接共享inode：

    name_A -> inode <- name_B

修改任意硬链接会影响同一份数据。inode编号只有结合文件系统设备编号才能唯一标识文件。

## 反斜杠与路径错误

单行命令通常不需要反斜杠：

    ls -li raw/file work/file

反斜杠加空格会把空格变成文件名的一部分。用于续行时，反斜杠必须是物理行最后一个字符。

## 易错点

- cd..错误；必须写cd ..。
- cd成功后通常无标准输出，目录列表可能来自公司Shell自动配置。
- 切换目录后，所有相对路径都会以新位置重新解释。
- 逗号和句点都是合法但不同的文件名字符。

## 数字后端应用

- 在pnr/log、pnr/report、pnr/db之间切换。
- 为公共日志创建个人分析副本。
- 按stage组织place、cts、route、signoff结果。

## 自检

- 能从提示符判断当前路径。
- 能解释.、..、~和绝对路径。
- 能判断cp与mv后的原文件是否存在。
- 能解释两个副本为何inode不同。

