# 08 Tcl List基础

## 总结

【重点】List是Tcl中最核心的数据结构之一，是有顺序、可嵌套的元素集合。list安全构造列表，concat合并并展开一层列表，lindex按从0开始的索引取元素，llength返回顶层元素数量。

对应教程第18–21页“list、concat、lindex、llength”。

## 1. List概念

合法List：

    {}
    {place cts route}
    {place {cts optimize} route}

- List有顺序。
- 每个元素可以是任意字符串。
- 一个元素本身也可以是List。
- 嵌套List作为一个顶层元素计数。

## 2. list构造

    set training_stages [list place cts route]

结果：

    place cts route

带空格元素：

    set training_modes [list functional "scan mode" low_power]

第二个元素是完整的scan mode，而不是两个元素。

嵌套：

    set training_nested [list place [list cts optimize] route]

结果：

    place {cts optimize} route

【重点】动态数据优先使用list命令构造，不要依靠手工拼接空格和花括号。

## 3. concat

    concat {place cts} {route signoff}

结果：

    place cts route signoff

concat把输入列表的顶层元素连接成一个新列表，会展开一层。

对比：

    list {place cts} {route signoff}

结果：

    {place cts} {route signoff}

前者有4个顶层元素，后者有2个顶层元素。

【易错】教程第19页“每个list变成新list的一个元素”表述不准确；实际concat会把输入列表的顶层元素拼接到一起。

## 4. lindex

    lindex $training_stages 0
    lindex $training_stages 1
    lindex $training_stages 2

结果：

    place
    cts
    route

索引从0开始。

最后一个元素：

    lindex $training_stages end

倒数第二个：

    lindex $training_stages end-1

越界索引通常返回空字符串，不一定报错，因此结果为空时要检查索引和列表长度。

## 5. llength

    llength $training_stages

结果：

    3

嵌套List：

    llength $training_nested

结果仍为3，因为cts optimize作为一个顶层元素。

空List：

    set training_empty {}
    llength $training_empty

结果：

    0

## 6. List与EDA对象集合

【重点】【易错】EDA工具返回的对象collection不一定是普通Tcl List。

Synopsys工具中常见：

    sizeof_collection
    index_collection
    foreach_in_collection

普通Tcl List使用：

    llength
    lindex
    foreach

使用前要确认变量保存的是普通List、字符串还是工具对象集合。

## 7. 后端示例

    set training_corners [list ss ff tt]
    set training_stages [list place cts route]
    set training_reports [list setup.rpt hold.rpt power.rpt]

    set training_first_corner [lindex $training_corners 0]
    set training_last_stage [lindex $training_stages end]
    set training_report_count [llength $training_reports]

## 8. 易错点

- 第一个元素索引是0，不是1。
- 嵌套List在顶层只算一个元素。
- concat与list构造出的层级不同。
- 手工字符串拼接可能破坏List边界。
- EDA collection不能默认当普通List处理。

## 9. 自检

- 能说明List的顺序和索引规则。
- 能解释list与concat的区别。
- 能使用lindex取得首项、末项。
- 能使用llength判断顶层元素数量。
- 能区分普通Tcl List和EDA对象集合。

