# 09 Tcl List增删与范围

## 总结

【重点】linsert、lreplace、lrange只返回新List，不会自动修改原变量，必须用set接收结果；lappend会直接更新指定变量。lreplace和lrange的first、last均包含在处理范围内。

对应教程第22–25页“linsert、lreplace、lrange、lappend”。

## 1. linsert

语法：

    linsert list index value ?value ...?

在index所指元素之前插入一个或多个元素，并返回新List。

    set training_stages [list place route signoff]
    set training_new [linsert $training_stages 1 cts]

training_new：

    place cts route signoff

training_stages仍为：

    place route signoff

【易错】只运行linsert而不接收返回值，不会修改原变量。

## 2. lreplace

语法：

    lreplace list first last ?value ...?

first到last是包含端点的闭区间。

替换一个元素：

    set training_new [lreplace $training_stages 1 1 cts]

替换多个元素：

    set training_new [lreplace $training_stages 1 2 cts route]

不提供value时删除范围：

    set training_new [lreplace $training_stages 1 1]

## 3. lrange

语法：

    lrange list first last

返回first至last范围的新List，端点都包含。

    set training_stages [list floorplan place cts route signoff]
    lrange $training_stages 1 3

结果：

    place cts route

从某处到末尾：

    lrange $training_stages 2 end

结果：

    cts route signoff

## 4. lappend

语法：

    lappend varName value ?value ...?

    set training_reports [list setup.rpt]
    lappend training_reports hold.rpt power.rpt

变量直接变为：

    setup.rpt hold.rpt power.rpt

变量不存在时，lappend会创建变量。

## 5. append与lappend

append进行字符串拼接：

    set training_steps [list place cts]
    append training_steps " route optimize"

route和optimize可能成为两个元素。

lappend保留元素边界：

    set training_steps [list place cts]
    lappend training_steps "route optimize"

此时route optimize是一个完整元素。

## 6. 返回新List与直接修改

| 命令 | 是否直接修改变量 |
|---|---|
| linsert | 否，只返回新List |
| lreplace | 否，只返回新List |
| lrange | 否，只返回新List |
| lappend | 是，直接更新varName |

## 7. 后端示例

    set training_flow [list floorplan place route signoff]
    set training_flow [linsert $training_flow 2 cts]
    set training_flow [lreplace $training_flow 3 3 detail_route]
    set training_main [lrange $training_flow 1 3]
    lappend training_flow archive

## 8. 易错点

- linsert插在index元素之前。
- lreplace和lrange包含last端点。
- 忘记用set接收linsert、lreplace、lrange结果。
- 用append代替lappend后破坏带空格元素的边界。
- lappend第一个参数是变量名，不是变量置换后的值。

## 9. 自检

- 能在指定位置插入元素。
- 能替换或删除一段元素。
- 能提取包含端点的范围。
- 能解释lappend与append的区别。
- 能判断哪些命令会直接更新变量。

