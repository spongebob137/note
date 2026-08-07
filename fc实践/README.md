# FC 实践问题记录

## 总结

本目录记录 2026-08-04 起 FC/CFlow 实操（Training Base run）中真实遇到的问题与对应的状态判断方法。每篇先写直接结论，再保留原因、正确写法和验证方法，方便以后用关键词检索。记录格式与 [../学习问题](../学习问题/README.md) 一致；一般概念解释归入 [../fc](../fc/README.md) 知识笔记。

## 文件索引

- [01_rpt已生成但无flag_阶段状态判断.md](01_rpt已生成但无flag_阶段状态判断.md)：make 运行中如何判断某阶段是进行中、成功还是失败；flag 与 rpt 的关系；状态查看命令组。
- [02_Ic2Floorplan_CMD013_工具版本不匹配.md](02_Ic2Floorplan_CMD013_工具版本不匹配.md)：首次 Floorplan 失败（Invalid option name）的根因、读 log 方法论与修复方向。
- [03_cload数据库菜单解读.md](03_cload数据库菜单解读.md)：cload 数据库选择菜单三列含义、三类条目与选择建议。
- [04_pin间距控制_block级约束只管下限.md](04_pin间距控制_block级约束只管下限.md)：pin_spacing 0 未达预期的原因；精确 pin 摆放必须用 individual 约束或 create_terminal 钉死。
- [05_get_cells默认只搜当前层级.md](05_get_cells默认只搜当前层级.md)：collection 为空导致 change_selection 静默失败；层次化设计须加 -hierarchical 或用 get_flat_cells。

## 记录格式

```text
问题：执行了什么，出现了什么现象？
结论：最短的正确答案。
原因：机制如何导致该现象。
正确写法：可直接验证的命令或操作。
验证：实际输出或检查方法。
```
