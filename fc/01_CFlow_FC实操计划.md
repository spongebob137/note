# CFlow/FC 实操计划

## 总结

【重点】先得到可复现的 Base run，再做面积与频率实验。每次实验只改一个核心条件，并用 report 证明结论。最后用相同输入和约束改写 rm_flow，比较其结果与 CFlow 基线是否一致。

## 阶段 A：接续 Initial 并完成 Floorplan

1. 确认当前 run 目录和生成的 Makefile。
2. 检查 `Ic2Initial` 对应的 flag、log、数据库和未豁免错误。
3. 从真实 Makefile 确认下一目标是 `Ic2Floorplan` 还是 `InFloorplan`，不凭 PPT 猜测。
4. 只读查看老师提供的 `/prj/yundu_u/users/xinyuyang_sd/pr_runs/try_run0/fp.tcl`，了解主要设置和依赖文件。
5. 备份或记录个人 run 中 `override.tcl` 的修改前状态。
6. 将该脚本路径填入 `vars(PR,FLOORPLAN_TCL)`，检查 Tcl 引号、路径和变量是否生效。
7. 执行实际存在的 Floorplan 目标，并监控 LSF job 和阶段 log。
8. 验收 flag、数据库、报告及 floorplan 检查结果；不得用手工 `touch` 掩盖失败。

**完成标准：** 可说明 Floorplan 使用了哪个脚本、配置写在哪里、执行了哪个 target、成功证据在哪里，以及关键告警是否影响后续。

## 阶段 B：完成 CFlow Base run

【重点】固定 library、netlist、SDC/MCMM、floorplan、工具版本及关键优化设置，形成唯一基线。

按实际 Makefile 依赖逐阶段完成 Initial、Floorplan、Place、CTS、Post-CTS、Route、Post-Route/Finish。每一步记录：

- target 与依赖关系；
- 生效配置及来源；
- LSF job、log、flag、db、report；
- ERROR/WARNING 与处理；
- runtime、内存和阶段 QoR。

完成后使用项目支持的汇总方式，例如 `crpt -summpnr`，生成 Base run 总结；可用时再补 `crpt -summsta`。

## 阶段 C：探索最小可行面积

1. 先定义“面积”指标：die/core 面积、standard-cell area 和 utilization 分开记录。
2. 从 Base run 建个人分支或副本，保留完全不改的基线。
3. 逐轮只调整一个关键 floorplan 参数，例如 core utilization 或边界尺寸。
4. 每轮检查放置/布线可实现性、拥塞、timing、DRV/DRC、PG 与宏单元约束。
5. 先粗粒度缩小范围，再在通过与失败的边界附近细化。

【重点】最终结论是“满足预先定义检查标准的最小面积”，不是跑出的最小数字。PPT 必须给出实验表、通过/失败证据和边界结论。

## 阶段 D：探索最高可行频率

1. 固定 PVT、RC corner、MCMM scenario、uncertainty 和其他约束。
2. 以时钟 period 为自变量，频率用 `frequency = 1 / period` 换算。
3. 先粗调找到通过/失败区间，再细调或二分逼近边界。
4. 每轮记录 period、frequency、setup/hold WNS/TNS、违反数量、DRV/DRC 与阶段是否完成。

【重点】“最高频率”必须绑定明确的通过条件和检查阶段；不能只根据某一次正 WNS 宣称结论。

## 阶段 E：整理并复现 rm_flow

1. 只读分析参考目录 `/prj/yundu_u/users/jingweizhang_sd/impl_tarining/rm_flow`。
2. 复制到个人目录或个人分支，不直接修改参考版本。
3. 建立 CFlow stage 与 rm_flow 脚本的对应表。
4. 对每个重要设置记录：值、目的、来源、所在文件和预期影响。
5. 优先使用 CFlow 的 `tune`/cftag 接口，不直接修改自动生成的 `pnr/cmd`。
6. 使用与 Base run 相同的输入、库、约束和工具版本运行，并比较 QoR。

【易错】Quick Start PPT 中自定义 cftag 文件名出现 `custom_cftag_pnr.tcl` 与 `customer_cftag_pnr.tcl` 两种写法，必须以当前运行目录实际接口为准。

## 阶段 F：整理 FC 常用命令

按“用途、前置状态、常用选项、输出字段、保存报告、来源”整理并实操：

```tcl
report_lib -physical
report_design -floorplan
report_ref_libs
report_parasitic_parameters
report_routing_rules
report_clock_routing_rules
```

命令语法先用当前 FC 版本的 `help`/`man` 核验，再查本地 UG。每个命令至少保留一次真实输出或重定向报告。

## 阶段 G：Training PPT

建议 12 至 14 页，简洁展示证据：

1. 任务目标与环境。
2. 输入、library、约束和资料来源。
3. CFlow 阶段与依赖关系。
4. Base run 关键设置和完成证据。
5. Base run QoR。
6. 面积实验方法。
7. 面积实验数据、边界与结论。
8. 频率实验方法。
9. 频率实验数据、边界与结论。
10. rm_flow 结构与 CFlow 对应关系。
11. rm_flow 复现结果及 QoR 对比。
12. FC 常用命令分类。
13. 问题、告警、waiver 和可复现性说明。
14. 最终结论与后续优化方向。

