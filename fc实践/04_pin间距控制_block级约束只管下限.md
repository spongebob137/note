# pin 间距控制失败：block 级约束只管下限，精确位置需钉死

## 总结

【重点】用 `set_block_pin_constraints -pin_spacing 0` + `place_pins -self` 无法让 pin 严格逐 track 等距排列——block 级约束只是边界条件，`place_pins` 还会按连线关系优化位置，实际结果间距不均。要精确控制每个 pin 的坐标和层，必须用 `set_individual_pin_constraints`（-location/-offset）或直接 `create_terminal` 逐个钉死。另：从手册语义推断参数效果不可靠，实测为准。

## 问题

目标：4420 个 port 全放 side 2（上边）单层 D6，一个 track 挨一个紧密排列。
执行：

```tcl
set_block_pin_constraints -self -sides {2} -allowed_layers {D6} -pin_spacing 0
place_pins -self
```

结果：有的 pin 间隔一个 track，有的不间隔，未达预期。

## 结论

block 级 pin 约束（allowed_layers / pin_spacing）只控制"允许范围"和"最小间距"这类**下限**；`place_pins` 在约束内仍按 net 连接关系优化 pin 位置，不产生严格等距结果。精确位序需逐 pin 指定（individual 约束的 `-location`/`-offset`，或 `create_terminal` + `snap_objects` 手工创建钉死）。

## 原因

- 手册对 `-pin_spacing` 的定义是"相邻 pin 的**最小** track 间距"（默认 1，可取 0）——语义上只是下限，不含"严格等距"的承诺；
- "设 0 即紧密排列"是从语义推断的预期，实测证伪：place_pins 的连通性优化会移动 pin；
- 教训：手册未明确写出的效果，推断不可作为依据。

## 正确写法

精确摆放（课程要求的最终形态）：

```tcl
# 清场后逐 pin 手工创建（坐标、层完全由脚本控制）
remove_terminals -force [get_terminals -quiet -of_objects [get_ports *]]
# 循环内：奇偶位分 D6/D8，按 x 步进钉坐标
create_terminal -port [get_ports -exact <name>] -layer [get_layers D6] -boundary {{x 0.0} {x+w h}}
snap_objects $term    ;# 吸附到最近 track
```

## 验证

- 层统计 D6/D8 各半；GUI 逐层显示确认交替；
- `check_legality` 无违规；
- 注意几何预算：pin 数 × 步进不能超过 die 边长，否则摆出界。

## 关联

- 约束命令报错后 `place_pins` 照跑的陷阱：见 [02_Ic2Floorplan_CMD013_工具版本不匹配.md](02_Ic2Floorplan_CMD013_工具版本不匹配.md) 的读 log 方法论；
- fixed 状态 pin 不会被 place_pins 移动——改约束无反应时先查 physical_status。
