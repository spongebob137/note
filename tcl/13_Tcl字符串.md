# 13 Tcl 字符串

## 总结

【重点】【易错】`string compare` 按字典顺序比较两个字符串：相等返回 `0`，第一个较小返回 `-1`，第一个较大返回 `1`。因此判断相等必须检查结果是否等于 `0`；在普通条件中，也可以直接使用更清楚的 `eq`。

本轮按会议要求只学习常用字符串操作，不展开完整正则表达式章节。本节会议重点已完成。

## 1. string compare

```tcl
string compare cts cts
string compare cts route
string compare route cts
```

结果分别为：

```text
0
-1
1
```

### 字符串大小如何决定

`string compare` 从左到右逐字符比较，遇到第一个不同字符时就由该字符的顺序决定结果；如果共同部分完全相同，则较短的字符串排在前面。

```tcl
string compare cts route
```

第一个字符已经不同。常见 ASCII 字符范围内，`c` 的字符编码小于 `r`，所以 `cts` 排在 `route` 之前，返回 `-1`。

```tcl
string compare cts ctt
```

前两个字符 `ct` 相同，第三个字符 `s` 小于 `t`，所以返回 `-1`。

```tcl
string compare ct cts
```

共同部分 `ct` 相同，但第一个字符串更短，所以返回 `-1`。

默认比较区分大小写。常见 ASCII 字符范围内，大写字母编码排在小写字母之前：

```tcl
string compare CTS cts
```

返回 `-1`；使用 `-nocase` 后忽略大小写，返回 `0`。

【易错】它进行的是字符串比较，不是数值比较：

```tcl
string compare 10 2
```

返回 `-1`，因为先比较字符 `1` 和 `2`。需要比较数值时应使用：

```tcl
expr {10 < 2}
expr {10 > 2}
```

忽略大小写：

```tcl
string compare -nocase CTS cts
```

结果为 `0`。

在条件中使用：

```tcl
set training_stage cts

if {[string compare $training_stage "cts"] == 0} {
    puts "matched"
} else {
    puts "not matched"
}
```

【易错】不能把 `string compare` 的返回值直接当作“是否相等”的布尔值，因为相等时返回的 `0` 在条件中反而表示假。

仅判断相等时，推荐直接写：

```tcl
if {$training_stage eq "cts"} {
    puts "matched"
}
```

## 2. 当前进度

- `string compare`：已练习。
- `string equal`、`string match`：已练习。
- 长度、索引和范围：已练习。

## 3. string equal 与 string match

`string equal` 判断两个字符串是否完全相等，相等返回 `1`，不相等返回 `0`：

```tcl
set training_cell U_CLKBUF_001
string equal $training_cell U_CLKBUF_001
string equal -nocase $training_cell u_clkbuf_001
```

两条比较都返回 `1`，第二条使用 `-nocase` 忽略大小写。

`string match` 使用 glob 通配模式匹配字符串：

```tcl
string match "U_CLKBUF_*" $training_cell
string match "*DFF*" $training_cell
```

结果分别为 `1` 和 `0`。

常用通配符：

| 通配符 | 含义 |
|---|---|
| `*` | 匹配任意数量的字符，包括零个 |
| `?` | 匹配任意一个字符 |
| `[abc]` | 匹配集合中的一个字符 |

【重点】`string equal` 的相等结果是 `1`，而 `string compare` 的相等结果是 `0`。`string match` 的第一个参数是 pattern，第二个参数是待检查的字符串。

## 4. 长度、索引和范围

```tcl
set training_cell U_CLKBUF_001

string length $training_cell
string index $training_cell 0
string index $training_cell end
string range $training_cell 2 7
```

结果分别为：

```text
12
U
1
CLKBUF
```

- `string length` 返回字符数量。
- `string index` 返回指定索引的一个字符，索引从 `0` 开始。
- `end` 表示最后一个字符的索引。
- `string range` 返回 `first` 到 `last` 的字符，两个端点都包含。

【易错】`string index` 和 `string range` 操作字符；`lindex` 和 `lrange` 操作 List 元素。不能因为一个字符串中含有下划线，就把它当作 Tcl List 的多个元素。
