# 12 Tcl 过程

## 总结

【重点】`proc` 用于定义可重复调用的新命令，形式为 `proc 过程名 {参数列表} {过程体}`。参数和过程内部创建的变量默认是局部变量；`return` 可立即结束过程并返回结果。

对应教程第 36-39 页。本节会议重点已完成。

## 1. 基本结构

```tcl
proc training_stage_info {index stage} {
    set message "index=$index stage=$stage"
    return $message
}
```

- `training_stage_info`：新过程的名字，定义后可以像普通 Tcl 命令一样调用。
- `{index stage}`：参数列表，调用时必须按顺序提供两个值。
- `message`：过程内部的局部变量。
- `return $message`：返回处理结果。

调用：

```tcl
set result [training_stage_info 2 cts]
puts $result
```

预期输出：

```text
index=2 stage=cts
```

## 2. 当前进度

- 定义与调用：已练习。
- 参数、局部变量和返回值：已练习。
- 默认参数：已练习。
- 全局变量：已练习。

## 3. 默认参数

```tcl
proc training_stage_status {stage {status pending}} {
    return "$stage : $status"
}
```

参数列表中的 `stage` 是必填参数，`{status pending}` 表示参数 `status` 的默认值为 `pending`。

```tcl
training_stage_status cts
training_stage_status cts done
```

结果分别为：

```text
cts : pending
cts : done
```

【易错】具有默认值的参数应放在必填参数之后。调用时未提供该参数才会使用默认值，显式提供的值会覆盖默认值。

## 4. 全局变量

过程中的变量默认属于局部作用域。需要在过程中读取或修改同名全局变量时，使用 `global` 声明：

```tcl
set training_call_count 0

proc training_count_stage {stage} {
    global training_call_count
    incr training_call_count
    return "call=$training_call_count stage=$stage"
}
```

连续调用：

```tcl
training_count_stage place
training_count_stage cts
puts $training_call_count
```

结果中的计数依次为 `1`、`2`，过程外读取也得到 `2`。

【重点】`global training_call_count` 后不加 `$`，因为这里需要声明变量名；读取其值时才使用 `$training_call_count`。一般优先通过参数和返回值传递数据，只在确实需要共享状态时使用全局变量。
