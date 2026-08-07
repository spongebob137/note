---
name: note-keeper
description: Maintain the note/ learning knowledge base (C:\\Users\\17381\\Documents\\BD\\note) during FC/CFlow/Tcl/LSF/Makefile learning conversations. Proactively judge — without waiting for the user to ask — whether a knowledge point, problem, or method from the conversation is worth recording, and record it following the repo's existing conventions so it can be easily retrieved later. Use whenever a problem gets solved, a new concept/command is learned, the user is corrected or corrects Claude, course/meeting requirements appear, or the user asks to record anything in the note folder.
---

# Note Keeper（笔记守护者）

维护 note 学习知识库。**主动判断、随时记录**，不等用户开口；最终目标是用户日后凭关键词能检索到。

## 何时记录（触发判断）

对话中每到一个自然节点（问题解决、概念讲透、阶段完成）就判断一次：

**值得记录：**
- 真实故障从报错到定位到解决的全过程（保留原始命令、输出、错误码原文）
- 新学到的命令、概念、机制，笔记库里还没有的
- 课程/会议/老师给出的明确要求、量化标准（如"memory 通道 ≥10μm"）
- 用户纠正了我的错误，或纠正了他自己的理解——正确的版本和"为什么之前错"
- 可复用的方法论（如"读 log 五步法"、"约束失败但后续命令照跑"类陷阱）
- 关键决策及其理由（如"为什么用 def 而不是脚本跑 floorplan"）

**不值得记录：**
- 简单拼写错误（规则库已有明确规定）
- 笔记里已有内容（改为补充/更新旧文件，不新建重复）
- 手册里直接能查到、且没有额外踩坑过程的纯事实
- 未经验证的猜测——先验证，确认后再记

## 记到哪里

| 内容类型 | 去向 |
|---|---|
| 概念、命令、语法知识 | 对应模块知识笔记：`tcl/`、`lsf/`、`makefile/`、`Linux_vim/`、`fc/` |
| 真实故障与解决 | `学习问题/`（按工具分文件）或 `fc实践/`（FC 实操期专题） |
| 会议/课程要求与待办 | 对应模块笔记 + 进度文件（如 `00_学习接续_*.md`） |

新主题在对应目录建编号新文件（如 `fc实践/04_xxx.md`），延续现有编号。

## 格式规则（必须遵守）

1. **总结放最前**：每个文件开头一段"## 总结"，说清本文件是什么、给谁查。
2. **重要性标记**：`【重点】`、`【常用】`、`【易错】`、`【了解】`。
3. **问题记录五段式**（故障类）：
   ```text
   问题：执行了什么，出现了什么现象？（保留报错原文和错误码）
   结论：最短的正确答案。
   原因：机制如何导致该现象。
   正确写法：可直接验证的命令或操作。
   验证：实际输出或检查方法。
   ```
4. **知识笔记**：概念解释 + 最小示例 + 易错点；模块的 `00_重点速查.md` 里加索引行（想查什么 → 核心命令 → 笔记链接）。
5. **可检索性优先**：
   - 标题和内容包含用户日后会搜的关键词：命令名、错误码（CMD-013、NDM-029）、术语中英文；
   - 术语全文统一，不中途换说法；
   - 查找型内容用表格。
6. **交叉链接**：相关文件互相用相对路径链接；新文件必须登记到该目录的 README/索引，以及主 `README.md` 学习索引（新目录时）。
7. **证据完整**：命令、关键输出、版本信息、文件路径用原文；推断标注"推断"。

## 操作流程

1. 判断值得记 → 先核实事实（不确定的先在对话里验证，不写猜测）；
2. 检查是否已有文件覆盖该主题 → 有则更新，无则新建；
3. 写入文件，更新所有相关索引；
4. 对话里**一句话告知**记到了哪里（不打断主线任务，不展开复述）；
5. 记录动作不阻塞当前工作——优先保证用户正在进行的事连续，记录在节点间隙完成。

## 边界

- 只动 note 仓库内容；运行目录、服务器上的任何东西不碰；
- 进度类文件（`00_学习接续_*.md`、各 README 的"当前进度"行）在阶段里程碑时更新，琐碎中间状态不刷；
- 用户明确要求"不用记"的内容不记。
