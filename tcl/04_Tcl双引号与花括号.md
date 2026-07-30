# 04 Tcl双引号与花括号

## 总结

双引号和花括号都能把带空格的内容组成一个Tcl单词。双引号允许变量、命令和反斜杠置换；花括号通常推迟这些置换，仅保留反斜杠加物理换行这一特殊处理。

对应教程第8页“双引号和花括号”。

## 1. 组织带空格的单词

双引号：

    set training_stage "clock tree synthesis"

花括号：

    set training_flow {clock tree synthesis}

两者都能把clock tree synthesis作为一个变量值。

## 2. 双引号允许变量置换

    set training_x 100
    set training_double "x=$training_x"
    puts $training_double

输出：

    x=100

双引号组织单词，但不会阻止$变量置换。

## 3. 双引号允许命令置换

    set training_result "sum=[expr {10 + 20}]"
    puts $training_result

输出：

    sum=30

方括号中的expr先执行，其结果替换整个方括号。

## 4. 双引号允许反斜杠置换

    set training_report "stage\tstatus\nCTS\tDONE"
    puts $training_report

输出包含真正的Tab和换行。

双引号内部仍处理：

    $variable
    [command]
    \n
    \t
    \\

## 5. 花括号推迟置换

    set training_brace {$training_x}
    puts $training_brace

输出：

    $training_x

命令也保持原样：

    set training_literal {sum=[expr {10 + 20}]}
    puts $training_literal

输出：

    sum=[expr {10 + 20}]

反斜杠序列通常同样保持字面形式：

    set training_escape {first\nsecond\tDONE}

变量值包含字符反斜杠+n和反斜杠+t，而不是真正的换行或Tab。

## 6. 直接对比

    puts "$training_x"

输出变量值：

    100

    puts {$training_x}

输出字面内容：

    $training_x

    puts "result=[expr {2 + 3}]"

输出：

    result=5

    puts {result=[expr {2 + 3}]}

输出：

    result=[expr {2 + 3}]

## 7. expr为什么使用花括号

推荐：

    expr {$training_x + 100}

花括号阻止外层Tcl解释器提前处理表达式，expr收到完整表达式后自行读取变量并计算。这样更安全、清晰，也便于Tcl优化。

## 8. 选择原则

| 需求 | 选择 |
|---|---|
| 需要变量或命令立即置换 | 双引号 |
| 需要\n、\t等转义生效 | 双引号 |
| 希望内容保持原样或推迟求值 | 花括号 |
| 只有一个普通单词 | 通常无需引号 |

数字后端常见形式：

    set design_name "top"
    set report_dir "./reports"
    set stages {place cts route}
    set total [expr {$setup_count + $hold_count}]

## 9. 易错点

- 花括号必须成对匹配，否则解释器会继续等待输入。
- 双引号不是“完全原样字符串”，其中仍会进行置换。
- 花括号内容并非永远不会求值，接收它的命令可能在后续自行解释。
- 花括号中反斜杠加物理换行仍是特殊情况。

## 10. 自检

- 能说明双引号和花括号的共同点。
- 能解释两者对$、[]和反斜杠的不同处理。
- 能说明expr使用花括号的原因。
- 能根据需求选择引号形式。

