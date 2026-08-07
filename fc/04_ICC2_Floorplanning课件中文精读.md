# ICC2 Floorplanning 课件中文精读（SG 第 2 单元）

## 总结

本文件是 `ICC_II_BLI_201912_SG_02_Floorplanning.pdf`（Synopsys IC Compiler II 培训课件，2019-12，共 32 页）的中文逐页精读。按幻灯片编号组织，学习时对照原文 PDF 看图。命令在 FC 中使用前需以 FC Tool Commands 复核（见 fc-work-assistant skill 规则）。

## 2-1 ~ 2-4：本单元定位与 floorplan 总览

- **2-2 单元目标**：能描述 block 级 floorplan 的关键步骤。明确声明**不覆盖完整的层次化 design planning 流程**（那复杂得多）。
- **2-3 在总流程中的位置**：Synthesis → Design & Timing Setup → **Floorplan Definition（本单元）** → Placement & Optimization → CTS → Routing → Signoff。
- **2-4 floorplan 关键步骤清单**（本单元主线，也是手动 floorplan 的完整检查单）：
  1. 定义 block 形状/尺寸；
  2. 电压区域（VA）形状/位置；
  3. macro 位置；
  4. I/O pin 位置；
  5. 标准单元 placement blockage（解决 congestion）；
  6. PNS（电源网络综合）概览；
  7. 导出 floorplan。

## 2-5 ~ 2-6：初始化 floorplan

- **2-5 `initialize_floorplan`**：在 core 区内定义标准单元摆放的 site array；支持形状：矩形 / L / T / U / Custom。示例 `initialize_floorplan -shape U -...`。【重点】**该命令同时按 tech file 定义自动创建 routing tracks**。自定义形状用 `-boundary` 给多边形顶点坐标。
- **2-6 初始化后的版图结构**：die 边界、core 边界、IO-Core 间距、含 site rows 的 site array、未摆放的 macro（堆在角落）、未摆放的标准单元和 port。【常用】site array 是 site rows 的容器，可移动/缩放/堆叠/设透明度，决定"哪种标准单元可以放哪里"；额外的 site array 用 `create_site_array` 创建（细节见附录 2-26 起）。

## 2-7 ~ 2-8：电压区域（多电压设计）

- **2-7 `create_voltage_area`**：ICC2 自动为顶层 UPF 电源域创建 DEFAULT_VA；其余电源域必须在 placement 前创建 VA（或用 shape_blocks 自动建）。
- **2-8 `shape_blocks`**：自动摆放并 shaping 各 VA（考虑 VA 内 cell 面积和外部连接）；不存在的 VA 会先被创建。
  - `set_shaping_options -guard_band_size 10`：VA 周围 guard band（硬 keepout，任何 cell 含 LS/ISO 都放不进）；
  - `-min_channel_size`：控制层次块/macro 间最小通道（不用于 VA）；
  - 给 VA 设目标利用率：`set_attribute [get_voltage_area_shapes ...] target_utilization 0.4` 后再 shape_blocks；
  - 固定不再参与 shaping：`set_voltage_area <VA> -is_fixed`。

## 2-9 ~ 2-13：macro 与标准单元摆放

- **2-9 `create_placement -floorplan`**：floorplan 模式的粗摆放（含 macro 摆放，速度优先）。选项：`-timing_driven`、`-congestion [-congestion_effort]`、`-incremental`、`-use_seed_locs` 等。
- **2-10 摆放配置（app options）**：【重点】默认行为——同层级同尺寸 macro 自动成 array（`plan.macro.auto_macro_array*`）；macro 间通道按 pin 数自动定宽；窄通道自动加 soft blockage；RAM 默认翻转成 pin 侧相对以减少走线通道（`plan.macro.auto_macro_array_minimize_channels`）。手动指定 macro 间距：`plan.macro.spacing_rule_heights/widths`；关闭自动间距：`plan.place.default_keepout false`；自动 blockage：`plan.place.auto_generate_blockages` 及 hard/soft 通道宽度选项。
- **2-11 更细的 macro 约束命令**：`set_macro_constraints`、`set_macro_relative_location`、`create_macro_array`、`create_macro_relative_location_placement`。
- **2-12 数据流飞线（DFF）**：按网表追踪连接关系（可穿透组合逻辑和寄存器层级）分析 macro 摆放；止于寄存器的连接不显示。
- **2-13 寄存器追踪**：选中 macro/输入 port 看其到各级寄存器的连接；选中 macro 即选中其全部数据输出 pin（不含 PG/clock/scan）；建议 place_opt 后调试使用。

## 2-14：I/O pin 摆放

- 流程：施加已知约束（block 级 / individual / bundle）→ `place_pins -self`。
- 示例：`set_block_pin_constraints -self -allowed_layers "M3 M4" -sides "1 2 3"` 或 `-exclude_sides "4 5 6"`；`create_pin_constraint -type individual -ports clk ...`。
- 【易错】**边编号**：side 1 = 最左的垂直边（多条则取最低那条），顺时针递增——与 FC 手册规则一致。
- 方法讨论：若 top 已传下精确 pin 位置，macro 摆放应考虑之；若只有粗略指导（如"bus X 放右边"），用 block 级约束即可，且 pin 摆放应在 macro 摆放之后进行。

## 2-15 ~ 2-16：电源规划（PNS）

- **2-15 挑战**：数模混合的复杂 core ring、多电压区域各自需要 mesh、特殊 PG pattern。
- **2-16 基于 pattern 的电源网络综合（PNS）四步**【重点】：
  1. 定义 PG 区域；
  2. 定义 pattern（结构：层、间距、宽度……），如 `create_pg_mesh_pattern`；
  3. 定义 strategy：`set_pg_strategy` 把 pattern 应用到指定区域/电压域/电源网——不绑定固定坐标，floorplan 改动后可适应；via 策略用 `set_pg_strategy_via_rule`；
  4. `compile_pg` 生成电源网络。
  - PG 检查三件套：`check_pg_drc`（工艺规则/非法交叠）、`check_pg_missing_vias`（缺 via）、`check_pg_connectivity`（浮空线、macro/标准单元 pin 连接）。

## 2-17 ~ 2-19：congestion 分析

- **2-17 `route_global -floorplan true -congestion_map_only true`**：floorplan 模式快速全局布线（低 effort、跳过部分 blockage、不要求 legal、不做时序/串扰驱动），产出 congestion 热点图。边界情况再用 `-effort_level low|medium|high` 确认。
- **2-18 congestion 标签读法**：`+4/20` = 溢出 4 track / 总供给 20 track；按各层 demand/supply 计算，underflow 计 0；默认算法是各层 overflow 求和（K-2015.06 起，旧算法过于乐观）。
- **2-19 macro 周边的 congestion**：macro 高 pin 密度的边/角附近标准单元走线困难 → 用 placement blockage 或 keepout margin（注意自动摆放已加过 soft blockage）。

## 2-20：keepout margin 与 blockage

- `create_keepout_margin -type hard -outer {15 0 18 0} RAM5`——四个数字顺序为 **{左 下 右 上}**，跟随 cell 旋转。
- 查询/删除：`get_keepout_margins`、`report_keepout_margin`、`remove_keepout_margin`。
- 通用 blockage 区域：`create_placement_blockage -boundary {{345 355} {392 400}} -name LL_CORNER -type hard|soft`。
- 【重点】**floorplan 每次改动后都要重跑**：`create_placement -floorplan [-incremental]` + `route_global -floorplan true -congestion_map_only true`。

## 2-21 ~ 2-23：导出 floorplan 与综合衔接

- **2-21 导出（ICC2/FC 复用）**【常用】：
  - 先固定 macro：`set_fixed_objects [get_flat_cells -filter "is_hard_macro"]`（解除：`set_fixed_objects -unfix`）；
  - `write_floorplan -output <name>.fp -net_types {power ground} -include_physical_status {fixed locked}` → 生成目录含 `floorplan.tcl`（主文件）、`fp.tcl`、`floorplan.def`；
  - 复用：`source <name>.fp/floorplan.tcl`；
  - fixed vs locked：多数操作等价；locked 对象不能用 remove_cell/remove_objects 删除；两者都**不防 set_attribute 直接改坐标**；
  - 【易错】macro 未 fix 就跑 create_placement（无 -floorplan）或 place_opt 会中止：DPUI-083；
  - floorplan 阶段加过 physical-only cell 时，导出要 `write_floorplan -read_def_options {-add_def_only_objects all}`。
- **2-22 导出给 DC-G**：`write_floorplan -format icc ...`（不含标准单元——二遍综合会生成新网表）；DC-G 侧先 `create_net -power/-ground` 再 `read_floorplan`。
- **2-23 用真实 floorplan 重综合（SPG 流程）**：DC-G 按 floorplan 重综合出更优网表 → `write_icc2_files` → ICC2 载入 → `commit_upf` → `place_opt`。QoR 不敏感可跳过重综合；RTL 黑盒多时需要多轮迭代。

## 2-25 ~ 2-32 附录：Site Array 深入

- **2-26 概念**：site rows 的容器；堆叠层级（stacking level）越高优先级越高；透明度（transparent）开启时与下一层 site array 的 row 取并集。
- **2-27 有效 site array 规则**：重叠区域中，高层级非透明的 site array 遮挡低层；透明时可与下层共存。
- **2-28 三种 site array**：Default（每 block 一个，stacking 0，边界=block 边界，定义 wire tracks）、Fixed（stacking>0，边界=block 边界）、Floating（stacking>0，自定义边界）。
- **2-29 优势**：rows 自动填满 site array 边界，无需手工管理；改动用 GUI 拉伸即可，rows 自动修复对齐。
- **2-31 ~ 2-32 常见问题**：
  - 何时创建：`initialize_floorplan`（当前 block）/ `commit_block`（子 block）；
  - rows 与边界有缝隙的原因：边界尺寸不是 site def 高/宽的整数倍；
  - 不相交 row 区域：直接 `create_site_array` 放多个即可，同 site def 自动对齐；
  - 改堆叠顺序：`set_site_array_stack_order [-above|-below|-raise|-lower|-top|-bottom]`。

## 学习建议

对照你正在做的手动 floorplan：2-4 是总检查单；你已完成的 die/core 初始化对应 2-5/2-6；pin 摆放对应 2-14；接下来 macro 摆放重点读 2-9/2-10/2-12（默认行为和 DFF 分析），PG 读 2-16，每轮改动后用 2-17 的 congestion 快速评估——这正是面积实验每轮的验证手段。
