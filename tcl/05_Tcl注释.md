# 05 Tcl注释

## 总结

Tcl使用#作为注释符，但#只有出现在解释器期望一条新命令开始的位置时才表示注释。命令末尾添加说明应先用分号结束命令，再写;#注释。

对应教程第9页“注释”。

## 1. 独立注释

    # This is a comment

这一整行不会执行。

## 2. 命令中间的#不是注释

    set training_a 100 # Not a comment

此处#位于set命令参数中间，会被当成普通参数，导致参数数量错误。

## 3. 行尾注释

    set training_b 101 ;# This is a comment

分号先结束set命令，解释器随后期待下一条命令，此时#才开始注释。

## 4. 脚本中的推荐格式

    # Set current implementation stage.
    set training_stage cts

    set training_corner ss ;# Worst setup corner

## 5. 易错点

- Tcl不使用//作为注释。
- 双引号和花括号中的#通常只是内容。
- set a 1 # comment中的#不是行尾注释。
- ;#是常见的行尾注释写法。

## 6. 数字后端应用

- 说明脚本阶段、corner和命令目的。
- 注释暂时停用的EDA命令。
- 标记需要复核的约束或路径。

## 7. 自检

- 能判断#何时是注释。
- 能写独立注释和行尾注释。
- 能解释为什么set a 1 # comment会报错。

