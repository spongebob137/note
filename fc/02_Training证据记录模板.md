# Training 证据记录模板

## 总结

【重点】每个结论都要能追溯到一次明确的 run、一个具体配置和一组报告。不要只保存截图，也要保存可检索的文本报告、log 路径和配置差异。

## Run 基本信息

| 字段 | 内容 |
|---|---|
| Run ID / 分支 |  |
| 日期 |  |
| 目标 | Base / Area / Frequency / rm_flow |
| CFlow 与 FC 版本 |  |
| Library | SAMSUNG_S4 7P94T |
| Input 路径/版本 |  |
| Netlist / SDC / MCMM |  |
| Floorplan 脚本 |  |
| 本轮唯一改动 |  |
| 设置来源 | 老师 / PPT / 当前文件 / FC help / UG / 实验 |

## 阶段证据

| Stage | Make target | Log | Flag | DB | Report | Job ID | Runtime/内存 | 结论 |
|---|---|---|---|---|---|---|---|---|
| Initial |  |  |  |  |  |  |  |  |
| Floorplan |  |  |  |  |  |  |  |  |
| Place |  |  |  |  |  |  |  |  |
| CTS |  |  |  |  |  |  |  |  |
| Post-CTS |  |  |  |  |  |  |  |  |
| Route |  |  |  |  |  |  |  |  |
| Post-Route/Finish |  |  |  |  |  |  |  |  |

## 关键设置来源

| 设置 | 值 | 目的 | 来源 | 文件/行号 | 生效证据 |
|---|---|---|---|---|---|
| `vars(PR,FLOORPLAN_TCL)` |  |  | 老师提供 |  |  |
|  |  |  |  |  |  |

## QoR 与实验结果

| Run | 唯一变量 | Die/Core area | Cell area | Utilization | Congestion | Period | Frequency | Setup WNS/TNS | Hold WNS/TNS | Violations/DRC | Pass/Fail |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Base | 无 |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |

## 问题与处理

| 现象 | 根因 | 处理 | 是否 waiver | 证据 | 对结论的影响 |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

## 本轮结论

- 是否达到本轮目标：
- 结论依据：
- 与 Base run 的差异：
- 下一轮只改变的变量：

