# 11 Tcl 控制流

## 总结

【重点】`if` 根据条件选择代码块，`foreach` 逐个遍历 List，`while` 在条件为真时重复执行，`for` 把初始化、判断和步进集中在一条命令中。`continue` 跳过本次循环，`break` 结束整个循环。数字后端 Tcl 脚本中 `foreach` 最常用。

对应教程第 30-35 页。本节会议重点已完成。

## 1. if

```tcl
set training_stage cts

if {$training_stage eq "cts"} {
    puts "Current stage is CTS"
} else {
    puts "Current stage is not CTS"
}
```

- `eq` 用于字符串相等比较。
- 条件推荐放在花括号中。
- `if`、条件、代码块之间必须以空格分隔。

## 2. foreach

```tcl
set training_stages {floorplan place cts route signoff}

foreach stage $training_stages {
    if {$stage eq "cts"} {
        puts "$stage : clock tree stage"
    } else {
        puts "$stage : normal stage"
    }
}
```

`stage` 是循环变量名，`$training_stages` 是待遍历的 List，`$stage` 是当前元素的值。循环结束后，`stage` 保留最后一次取得的值。

## 3. continue 与 break

```tcl
foreach stage $training_stages {
    if {$stage eq "place"} {
        continue
    }
    if {$stage eq "route"} {
        break
    }
    puts $stage
}
```

- `continue`：跳过当前这一次循环，继续处理下一个元素。
- `break`：立即结束整个循环。

## 4. while

```tcl
set stage_count [llength $training_stages]
set i 0

while {$i < $stage_count} {
    set stage [lindex $training_stages $i]
    puts "index=$i stage=$stage"
    incr i
}
```

【易错】循环体必须改变影响判断条件的变量。本例依靠 `incr i` 使循环最终结束；遗漏它会形成无限循环。

## 5. for

基本结构：

```tcl
for {初始化} {继续条件} {每轮后的更新} {
    循环体
}
```

按索引遍历 List：

```tcl
for {set i 0} {$i < [llength $training_stages]} {incr i} {
    set stage [lindex $training_stages $i]
    puts "index=$i stage=$stage"
}
```

执行顺序为初始化、判断、循环体、更新，再返回判断。循环退出时本例中的 `i` 为 List 长度。

## 6. 完成状态

- `if`：已练习。
- `foreach`：已练习。
- `continue`、`break`：已练习。
- `while`：已练习。
- `for`：已练习。
- `switch`：会议未列为核心，安排为了解项。
