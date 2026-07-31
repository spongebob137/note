# 10 Tcl List搜索排序与转换

## 总结

【重点】lsearch返回第一个匹配元素的索引，未找到返回-1；lsort返回排序后的新List，不修改原变量；split把字符串拆成List，join把List元素连接成字符串。

对应教程第26–29页“lsearch、lsort、split、join”。

## 1. lsearch

语法：

    lsearch ?options? list pattern

精确匹配：

    lsearch -exact $training_stages cts

默认或显式glob匹配：

    lsearch -glob $training_stages r*

正则匹配：

    lsearch -regexp $training_stages {^s.*off$}

返回第一个匹配元素的索引；未找到时返回-1。

【易错】返回0表示第一个元素匹配，不能直接把返回值当布尔值判断成功。正确判断应检查结果是否大于等于0。

返回值与布尔解释：

| lsearch返回值 | 搜索含义 | 作为布尔值 |
|---:|---|---|
| 0 | 在第一个元素找到 | 假 |
| 1及以上 | 在后续元素找到 | 真 |
| -1 | 没有找到 | 真 |

因此下面的逻辑是错误的：

    set training_index [lsearch -exact $training_stages floorplan]
    expr {$training_index}

虽然已经找到，但索引0被expr解释为假。搜索不到时返回-1，反而会被解释为真。

正确判断：

    expr {$training_index >= 0}

或者：

    expr {$training_index != -1}

## 2. lsort

语法：

    lsort ?options? list

常用选项：

| 选项 | 含义 |
|---|---|
| -ascii | ASCII顺序，默认 |
| -dictionary | 字典或自然顺序 |
| -integer | 按整数排序 |
| -real | 按浮点数排序 |
| -increasing | 升序，默认 |
| -decreasing | 降序 |
| -unique | 排序并去除重复元素 |
| -command | 使用自定义比较命令 |

【易错】lsort返回新List，不会自动修改原变量。

数值字符串必须选择正确类型：

    lsort {10 2 1}
    lsort -integer {10 2 1}

默认结果可能是1 10 2，整数排序才是1 2 10。

## 3. split

语法：

    split string ?splitChars?

    split "setup:-0.12:ss" :

结果：

    setup -0.12 ss

未指定splitChars时按空白字符拆分：

    split "place cts route"

空分隔符表示按字符拆分：

    split "cts" {}

结果：

    c t s

【易错】splitChars是一组分隔字符，不是一个多字符分隔字符串。连续分隔符或首尾分隔符可能产生空元素。

## 4. join

语法：

    join list ?joinString?

默认使用空格：

    join {place cts route}

自定义分隔：

    join {place cts route} " -> "

结果：

    place -> cts -> route

    join {how are you} .

结果：

    how.are.you

## 5. split与join组合

    set training_line "setup:-0.12:ss"
    set training_fields [split $training_line :]

    set training_check [lindex $training_fields 0]
    set training_slack [lindex $training_fields 1]
    set training_corner [lindex $training_fields 2]

    set training_csv [join $training_fields ,]

结果：

    setup,-0.12,ss

## 6. EDA对象集合提醒

【重点】lsearch和lsort面向普通Tcl List。工具Collection需要使用对应EDA命令或先明确转换方式，不能仅根据显示文本判断类型。

## 7. 后端应用

- lsearch检查某个stage、corner或mode是否存在。
- lsort -real对slack数值排序。
- lsort -unique整理重复名称。
- split解析冒号分隔的摘要行。
- join生成CSV、路径或可读状态字符串。

## 8. 易错点

- lsearch未找到返回-1，不是空字符串。
- lsearch找到第一个元素返回0，0不表示失败。
- 默认lsearch是glob匹配，不是regexp。
- 默认lsort是ASCII排序，数值排序要使用-integer或-real。
- lsort不会自动更新原变量。
- split连续分隔符会产生空元素。

## 9. 自检

- 能正确判断lsearch是否找到。
- 能解释ASCII排序与数值排序差异。
- 能选择-integer、-real或-dictionary。
- 能用split和join完成字符串与List转换。
