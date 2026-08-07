# CFlow Quick Start PPT 重点总结

## 总结

PPT 共 26 页，主线是一条完整流程：**环境创建（CInitial → override.tcl → Cflow -pnr → getLibAddition → getMcmmSet）→ Makefile 跑 PR 各阶段 → tune/ 接口修改 flow → cload/crpt 等辅助命令 → STA/PV/PA → crelease 发布数据**。核心机制是 flag 文件驱动阶段推进，tune/ 目录是唯一的用户修改入口。

## 一、环境创建流程（PPT 第 1-7 页）

【重点】完整顺序：

1. `source .../cflow.csh 2.1.2` 配置 CFlow 环境（2.1.2 是 Flow 版本号）。
2. 模块工作目录下执行 `CInitial`，按 design 需求选择，**注意填好 design name**。
3. 手动配置 `override.tcl`：填网表、SDC（不需要的 scenario 注释掉）。
4. `Cflow -pnr` 生成各阶段的 cmd/Makefile 文件（还有 `-syn`、`-fv`、`-pv`、`-pa` 等模式）。
5. `getLibAddition`：根据 `vars(PNR,NETLIST)` 生成 `tune/lib_setup_addition.tcl`（收集 lef/lib/db）和 `tune/sta_config.tcl`（配置模块 Flat/Stitch/ETM 处理方式）。
6. `getMcmmSet`：根据 SDC 的 corner 设置生成 `tune/pnr/mcmm.tcl`。
7. `make Ic2Xxx` 跑各阶段。

【易错】

- 生成 override.tcl 后**重点检查项目名称、工具名称**，工具会实时调用 override 的内容，填错影响整个 PNR flow。
- `getLibAddition` / `getMcmmSet` 重新执行要加 `-force`；tune 下已存在同名文件时**不会覆盖**。
- getLibAddition 时重点检查是否有红色 error，有 error 需反馈确认是否影响进程。
- scenario 名字容易混淆，active 前必须已在上面定义，设置时仔细检查。

## 二、Makefile 依赖链与 flag 机制（PPT 第 8、12-14 页）

【重点】PR 阶段 Makefile 目标顺序（第 13 页明确给出）：

```
Ic2Initial -> Ic2Floorplan -> Ic2Place -> Ic2Cts -> Ic2Postcts -> Ic2Route -> Ic2Postroute
```

注意：第 8、14 页用的是 `InInitial/InFloorplan` 命名，第 13 页用的是 `Ic2*` 命名——PPT 里两代命名并存，**实际目标名以运行目录生成的 Makefile 为准**。

【重点】flag 机制：

- 每个阶段跑完且 log 无 error，在 `pnr/flag/` 生成对应 flag（如 `InInitial.flag`）；**有 error 则不生成**。
- 执行下一阶段时若检测不到前一阶段 flag，会**自动重跑前一阶段**。
- 重跑某一阶段 = 删除对应 flag 文件。

【常用】error 可以 waive 时的两种办法：

1. 修改 `tune/checkList`（CInitial 生成，用于 waive log 中的 Error/Warning），改后有 error 也会创建 flag。
2. 自己 `touch` 一个 flag 文件。

## 三、pnr 目录结构（PPT 第 12 页）

【常用】

| 目录 | 用途 |
|---|---|
| `act` | 跑 flow 的 csh 脚本 |
| `flag` | 每阶段完成标记 |
| `cmd` | 每阶段执行命令（**原则上不直接改**，通过 tune 接口改） |
| `db` | 每阶段数据库保存位置 |
| `log` / `rpt` | 每阶段日志 / 报告 |
| `run` | 每阶段实际运行目录 |
| `datain` / `dataout` | 输入 / 输出数据 |
| `eco_scr` / `eco_track` | ECO 脚本及历史归档 |

## 四、配置与修改接口（PPT 第 9-11、15-16 页）

【重点】两个层次的配置：

- `override.tcl`：用户改设计变量的主文件。PR 优化开关（phycell/iobuffer、uncertainty、ulvt 比例）、工具版本、LSF 提交设置都在这里。
- `tune/`：用户接口文件夹，第一次执行 cflow 命令时初始化，已存在的文件不会被覆盖。

【易错】override.tcl 中所有相对路径**必须以 `./` 或 `../` 开头**（相对 override.tcl 所在路径）；`pnr/run` 这种不带前缀的写法无法识别。

【常用】`tune/pnr/` 下两个关键文件：

- `local.always_source.tcl`：每阶段 initDesign 时，在 link_block 后自动 source。
- `custom_cftag_pnr.tcl`：修改 flow 中 cmd 的接口，三种操作：
  - `cftag_prepend`：在某 cmd 执行前添加 option
  - `cftag_replace`：整体替换某 cmd
  - `cftag_append`：在某 cmd 执行后添加 option
- 调试技巧：`return -level 2` 可让 flow 停在指定行。

## 五、常用辅助命令（PPT 第 17-18 页）

【常用】

| 命令 | 用途 |
|---|---|
| `cload` | 开启 design，加载当前项目 flow 设置；`-read` 只读打开；`-e <tcl>` 读 db 后执行脚本；`-sta` 打开 PT session；`-lsf "bsub -Ip"` 改 LSF 设置 |
| `crpt` | 搜集 rpt 信息；`-summpnr` 生成 `pnr/rpt/pnr_summary.xlsx`；`-summsta` 汇总 STA 结果 |
| `crelease` | release 数据备用，如 `crelease -tag 0.3 -mileston "xxx" -ndm/-rcx/-etm` |
| `ctouch` | 指定当前 design 到某阶段，如 `ctouch -pnr {Ic2Floorplan}` |
| `cbranch` | 从当前 design branch 一个对比 run（自动 copy 数据、创建 flag、记录复制信息） |

## 六、STA / PV / PA（PPT 第 19-24 页）

【了解】top STA 两种方式：

- **top only（ETM）**：block owner 用 PT 出 ETM（`set vars(pt_shell,GEN_ETM) true` + `crelease -mile 85 -tag xx -etm`），top 把 `tune/sta_config.tcl` 改成 etm，再 `getLibAddition -release -force`。前期没有 ETM 时可改 `tune/lib_addition_usr.hash` 自行添加。
- **flatten**：`sta_config.tcl` 全改 stitch；override.tcl 里 `vars(PR,NETLIST)` 写全后 `getLibAddition -force` 抓 mem/ip/io 的 db；再 `grab_netlist_def_spef` 抓 block 的 verilog 和 spef。

【了解】

- PV：`CFlow -pv`，act 下生成 tcsh，`make drc/lvs`。
- Redhawk（PA）：`CFlow -pa`；icc2 跑完 Finish 后 flow 自动生成 IR.def、ploc；make 对应 target 生成 ipf/tw；`ir_summary.pl` 汇总 static/dynamic IR violation 数量。
- 添加电压 mode：`vars(NORMAL_VOLTAGE)` 控制，配好后重新 getLibAddition + getMcmmSet。
- crelease 常用：`-ndm`（物理库）和 `-etm`（时序模型），都要 `-tag` + `-mile 85`。

## 七、跳过阶段的技巧（PPT 第 25 页）

【常用】快速验证 route 时可跳过 cts/postcts：

```
GFlow -remove  "Ic2Cts Ic2Postcts"    # 自动改 Makefile 依赖关系
GFlow -recover "Ic2Cts Ic2Postcts"    # 恢复
```

## 与当前实操的关联

- 接续卡中"下一目标是 `Ic2Floorplan` 还是 `InFloorplan`"的疑问：PPT 两种命名都出现（第 13 页为 `Ic2*`，第 8/14 页为 `In*`），**仍以实际生成的 Makefile 为准**，本次 `make postroute` 跑完后可直接确认。
- 按 flag 机制，直接 `make postroute` 时若前面阶段 flag 缺失，会**自动依次重跑所有前置阶段**——这是运行时间长的预期行为，不等于卡死。
- run 结束后按 [02_Training证据记录模板.md](02_Training证据记录模板.md) 检查各阶段 flag/log/rpt，重点看被自动重跑的阶段是否有被 waive 掉的 error。
