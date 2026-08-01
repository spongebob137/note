# 06 Tcl变量与数组

## 总结

Tcl变量由名称和值组成，set既能写入也能读取。数组是以任意字符串为索引的关联数组，不是按数字顺序排列的普通数组。unset删除变量，append追加字符串，incr对整数变量增减。

对应教程第10–13页“变量、数组及相关命令”。

## 1. 简单变量

设置：

    set training_stage cts

读取：

    set training_stage

变量置换：

    puts $training_stage

Tcl的值本质上以字符串形式存在，具体命令会根据需要把它解释成整数、列表或其他形式。

## 1.1 查看变量内容

假设变量名为training：

    set training cts

set需要变量名，因此读取时不加美元符号：

    set training

puts需要一个待输出的值，因此使用变量置换：

    puts $training

两者都能显示cts。

注意：

    puts {$training}

花括号阻止置换，因此输出字面文字$training，而不是变量值。

## 2. 变量名

推荐使用：

    training_design
    training_stage
    training_corner

即字母、数字和下划线组合，且不要以数字开头。

Tcl允许特殊变量名：

    set training.stage cts

读取特殊变量名时使用花括号：

    puts ${training.stage}

否则$training.stage会先读取training，再把.stage当作普通文字。

变量名后紧跟其他字符时也适合使用花括号：

    set training_design core
    puts ${training_design}_cts

## 3. Tcl数组

创建数组元素：

    set training_status(place) done
    set training_status(cts) running
    set training_status(route) pending

读取：

    puts $training_status(place)
    puts $training_status(cts)

数组索引可以是任意字符串。

## 4. 动态数组索引

    set training_current cts
    puts $training_status($training_current)

执行时先把$training_current替换为cts，再读取training_status(cts)。

## 5. unset

删除简单变量：

    unset training_stage

删除一个数组元素：

    unset training_status(place)

删除整个数组：

    unset training_status

读取已删除变量会产生no such variable或no such element错误。

## 6. append

    set training_msg stage
    append training_msg _cts

结果：

    stage_cts

append不会自动增加空格：

    append training_msg " done"

结果：

    stage_cts done

【重点】【易错】append的第一个参数是变量名，不能进行变量置换：

    append training " CTS signoff"

不要写成：

    append $training " CTS signoff"

后者会先取得training的值，再把这个值当作另一个变量的名称。

Tcl命令必须使用英文半角双引号。中文弯引号“”不会组织带空格的单词，并且可能被追加进变量内容。

## 7. incr

    set training_count 2
    incr training_count

结果为3。

    incr training_count 3

结果为6。

    incr training_count -1

结果为5。

incr要求变量值和增量都是整数。

错误示例：

    set training_bad abc
    incr training_bad

会产生expected integer错误。

## 8. EDA应用示例

简单变量：

    set training_design top
    set training_stage cts
    set training_corner ss

数组：

    set training_report(setup) reports/setup.rpt
    set training_report(hold) reports/hold.rpt

计数：

    set training_violation_count 0
    incr training_violation_count

## 9. 易错点

- $a.1通常表示变量a的值再拼接.1，不一定是变量a.1。
- 简单变量和同名数组不能同时存在。
- append不自动添加分隔符或空格。
- incr只能处理整数。
- unset整个数组与unset单个数组元素影响范围不同。

## 10. 自检

- 能用set设置、读取和修改变量。
- 能使用${name}消除变量名边界歧义。
- 能创建、读取和删除数组元素。
- 能解释append与incr的适用类型。
