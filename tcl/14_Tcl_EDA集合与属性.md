# 14 Tcl 在 FC 中的集合与属性

## 总结

【重点】`get_cells`、`get_selection` 等 EDA 命令返回 collection，不是普通 Tcl List。Fusion Compiler 使用 `sizeof_collection` 计数、`index_collection` 取指定对象、`foreach_in_collection` 遍历，再用 `get_object_name` 或 `get_attribute` 读取对象信息。

本节参考本地 `fusion_compiler_user_guide_2021.06.md` 和 `fc_dp_user_guide_24_09.md`，会议要求的实操已完成。以下练习均为只读操作。

## 1. 取得当前选中对象

先在 FC GUI 中选中少量 cell，再执行：

```tcl
set training_cells [get_selection]
```

变量中保存的是 collection 句柄。Shell 可能显示类似 `_sel123` 的内部名称，这不表示只有一个对象。

## 2. collection 数量

```tcl
set training_cell_count [sizeof_collection $training_cells]
puts "selected cell count: $training_cell_count"
```

【易错】FC 命令名是 `sizeof_collection`，不是会议纪要中的 `size_of_collection`。

## 3. 取得第一个对象

```tcl
set training_first_cell [index_collection $training_cells 0]
get_object_name $training_first_cell
```

### index_collection

基本形式：

```tcl
index_collection collection index
```

- `collection`：要查询的 EDA collection。
- `index`：要取得的对象位置，从 `0` 开始。
- 返回值：一个包含对应 EDA 对象的新 collection，不是对象名称，也不会修改原 collection。

下面命令的执行顺序是先读取 `$training_cells` 的 collection 句柄，再取其中索引为 `0` 的对象，最后把返回的新 collection 保存到 `training_first_cell`：

```tcl
set training_first_cell [index_collection $training_cells 0]
```

只有 collection 非空时才能安全取得索引 `0`，所以实操中先用 `sizeof_collection` 检查数量。

### get_object_name

基本形式：

```tcl
get_object_name objects
```

它接收一个或多个 EDA 对象，返回这些对象在设计数据库中的名称。对单对象 collection 使用：

```tcl
get_object_name $training_first_cell
```

对整个 collection 使用：

```tcl
get_object_name $training_cells
```

后一种写法会返回所有选中对象的名称。

组合写法：

```tcl
puts "first cell: [get_object_name $training_first_cell]"
```

方括号表示命令置换：先执行 `get_object_name` 得到名称，再把名称放进 `puts` 的字符串中。`[` 不是 `get_object_name` 命令名的一部分。

也可以不建立中间变量，直接嵌套：

```tcl
get_object_name [index_collection $training_cells 0]
```

## 4. collection 与 List 的区别

| 数据 | 计数 | 取元素 | 遍历 |
|---|---|---|---|
| Tcl List | `llength` | `lindex` | `foreach` |
| EDA collection | `sizeof_collection` | `index_collection` | `foreach_in_collection` |

不要依赖 collection 句柄的显示形式，也不要用 `llength` 判断其中有多少个 EDA 对象。

## 5. foreach_in_collection 与属性

基本形式：

```tcl
foreach_in_collection object_var collection {
    body
}
```

遍历选中的 cell：

```tcl
foreach_in_collection cell $training_cells {
    set cell_name [get_object_name $cell]
    set cell_ref [get_attribute $cell ref_name]
    puts "cell=$cell_name ref=$cell_ref"
}
```

- `cell` 是循环变量名，不加 `$`。
- `$training_cells` 是要遍历的 collection。
- 每轮中的 `$cell` 是只包含当前对象的 collection。
- `get_object_name $cell` 返回实例在当前设计中的名称。
- `get_attribute $cell ref_name` 返回该实例引用的 library cell 名称。

实例名和 reference 名是两个不同概念。例如实例名可能是 `u_core/u_buf_1`，reference 名可能是工艺库中的某个 buffer cell 名。

## 6. 当前进度

- 取得 collection：已练习。
- 计数和索引：已练习。
- 遍历名称与 reference：已练习。
- 获取并拆分 `bbox`：已练习。
- 批量输出 cell 信息与坐标：已练习。

## 7. 获取并拆分 cell 边界框

对一个已放置的标准单元读取边界框：

```tcl
set training_cell [index_collection $training_cells 0]
set training_bbox [get_attribute $training_cell boundary_bbox]
puts "bbox=$training_bbox"
```

cell 的 `boundary_bbox` 通常具有二层 List 结构：

```text
{{llx lly} {urx ury}}
```

- 第 `0` 个元素 `{llx lly}`：左下角坐标。
- 第 `1` 个元素 `{urx ury}`：右上角坐标。

逐层拆分：

```tcl
set lower_left  [lindex $training_bbox 0]
set upper_right [lindex $training_bbox 1]

set llx [lindex $lower_left 0]
set lly [lindex $lower_left 1]
set urx [lindex $upper_right 0]
set ury [lindex $upper_right 1]
```

计算宽度和高度：

```tcl
set width  [expr {$urx - $llx}]
set height [expr {$ury - $lly}]
puts "ll=($llx,$lly) ur=($urx,$ury) width=$width height=$height"
```

也可以使用多级索引简写：

```tcl
set llx [lindex $training_bbox 0 0]
set lly [lindex $training_bbox 0 1]
set urx [lindex $training_bbox 1 0]
set ury [lindex $training_bbox 1 1]
```

【重点】`training_bbox` 是普通 Tcl List，因此这里使用 `lindex`；`training_cell` 是 EDA collection，因此取得对象时使用 `index_collection`。

## 8. 批量输出 cell 信息与坐标

```tcl
foreach_in_collection cell $training_cells {
    set cell_name [get_object_name $cell]
    set cell_ref  [get_attribute $cell ref_name]
    set cell_bbox [get_attribute $cell boundary_bbox]

    if {[llength $cell_bbox] == 2} {
        set llx [lindex $cell_bbox 0 0]
        set lly [lindex $cell_bbox 0 1]
        set urx [lindex $cell_bbox 1 0]
        set ury [lindex $cell_bbox 1 1]
        set width  [expr {$urx - $llx}]
        set height [expr {$ury - $lly}]

        puts "cell=$cell_name ref=$cell_ref"
        puts "  ll=($llx,$lly) ur=($urx,$ury) size=($width,$height)"
    } else {
        puts "cell=$cell_name ref=$cell_ref bbox=unavailable"
    }
}
```

`llength $cell_bbox` 检查的是 `boundary_bbox` 返回的普通 List 是否包含两个坐标点。这个判断避免对空边界框继续执行 `lindex` 和算术运算。

`ref_name` 是实例引用的 library cell 名称，可用于识别单元类型。不同工艺库对 clock buffer、DFF、MUX、AND、OR 等功能的命名规则不同；正式分类脚本应根据当前项目库命名或可用 library 属性确定，不能假定一种字符串模式适用于所有项目。
