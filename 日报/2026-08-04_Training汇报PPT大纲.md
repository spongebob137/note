# Training 汇报 PPT 大纲（2026-08-04）

> 用途：向老师汇报 training 进展、文档学习进度、遇到的问题。
> 素材来源：本笔记库 00_学习接续_2026-08-04.md、日报/2026-07-31_工作日报.md、学习问题/、fc/03_CFlow_PPT重点总结.md。
> 建议 8 页，每页要点 + 备注栏讲稿。

---

## 第 1 页：封面

- 标题：数字后端 Training 进展汇报
- 姓名、日期（2026-08-04）

## 第 2 页：总体进展一览

- 时间线：7/31 前完成基础工具学习 → 8/4 起切换到 FC/CFlow 实操主线
- 当前状态一句话：CFlow 环境已创建，Base run 全链条（initial → postroute）首次运行中

| 模块 | 状态 |
|---|---|
| Linux / Vim | 已完成（含日志实操） |
| Tcl（含 EDA 应用） | 已完成会议指定重点 |
| LSF | 已完成六条重点命令实操 |
| Makefile | 基础规则已学，实操部分结合 CFlow 补学中 |
| FC/CFlow 实操 | 进行中：环境创建完成，全链条 run 运行中 |

## 第 3 页：Training 实操进展（CFlow）

- 已完成环境创建全流程：CInitial → override.tcl（网表/SDC）→ Cflow -pnr → getLibAddition → getMcmmSet
- `make Ic2Initial` 已完成
- 当前：直接执行 make 跑通整条 PR 链（Ic2Initial → … → Ic2Postroute），现处于 Place 阶段
- Training 输入：SAMSUNG_S4 7P94T library，参考 rm_flow
- 证据：run 目录 flag/log/rpt/db 截图（跑完后补，重点放 flag 列表和 pnr_summary）

## 第 4 页：对流程的理解（可用一张图）

- PR 依赖链：Ic2Initial → Ic2Floorplan → Ic2Place → Ic2Cts → Ic2Postcts → Ic2Route → Ic2Postroute
- flag 机制：每阶段无 Error 才生成 flag；缺 flag 会自动重跑前置阶段
- 配置入口：override.tcl（设计变量）+ tune/（lib_setup_addition、mcmm、custom_cftag 等接口）
- 辅助命令：cload / crpt / crelease / ctouch / cbranch

## 第 5 页：文档学习进度（Tcl / LSF）

- Tcl：变量、List、控制流、proc、字符串、EDA collection 应用（遍历 cell 输出名称/reference/bbox 坐标），产出 14 篇专题笔记 + 速查表
- LSF：bsub/bjobs/bkill/bqueues/bhosts/lsload 六条命令全部实操，完成一次任务全生命周期（提交→跟踪→终止）
- 关键认识（可体现深度，选 2-3 条）：
  - collection ≠ 普通 List（foreach_in_collection vs lindex）
  - `string compare` 相等返回 0，不能直接当布尔值
  - `-M`（内存上限）与 `rusage[mem=]`（调度资源声明）用途不同

## 第 6 页：文档学习进度（Makefile / Linux / 其他）

- Makefile：目标-依赖-recipe 规则、后端流程依赖链执行顺序已理解；时间戳与多级依赖待结合 CFlow 实际 Makefile 补学
- Linux/Vim：基础命令与日志检索实操完成
- CFlow Quick Start PPT：26 页已全部消化并整理成结构化总结

## 第 7 页：遇到的问题与处理

- 资料差异类（已建立处理原则）：
  - Floorplan target 命名不一致（Ic2Floorplan vs InFloorplan）→ 以实际生成的 Makefile 为准
  - cftag 文件名两种写法 → 以 tune/pnr 实际被 source 的文件为准
  - 微盘资料需登录 → 暂用 PPT + FC help/man + 本地 UG 替代
- 实操类（处理中）：
  - Ic2Place 的 rpt 已生成但无 flag → 已定位为"阶段未结束/存在 Error 时不生成 flag"机制，正在跟踪 log 确认
- 学习方法：问题按主题分类记录（Tcl/LSF/Makefile/FC 四个文件），每条含结论、原因、正确写法、验证

## 第 8 页：下一步计划

1. 等本次全链条 run 结束，逐阶段检查 flag/log/rpt，确认无带病通过
2. 配置老师提供的 fp.tcl 到 vars(PR,FLOORPLAN_TCL)，跑正式 Base run 作为基线
3. 在基线上做面积 / 频率探索实验（一次只改一个变量）
4. 补学 Makefile 时间戳与多级依赖（结合 CFlow 实际 Makefile）
5. 整理 rm_flow 参考与最终 Training 证据 PPT

---

## 制作提示

- 第 3、4 页放真实截图（flag 目录、crpt 的 pnr_summary）比文字更有说服力，等 run 结束后截取。
- 第 7 页问题页的重点是展示"处理原则"，而不是问题本身——体现可追溯的学习方法。
- 如需直接生成 .pptx 文件，可基于本大纲用模板生成初稿后手工调整。
