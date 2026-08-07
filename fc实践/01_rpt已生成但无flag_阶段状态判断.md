# rpt 已生成但无 flag：阶段状态判断

## 总结

【重点】rpt 是阶段内部各子步骤的**中间产物**，flag 只有整个阶段**成功结束且 log 无 Error** 后才生成。"有 rpt + 无 flag + 任务在跑" = 阶段进行中，属正常现象，不是失败。判断 run 状态看两件事：log 文件修改时间是否持续刷新、LSF 任务 CPU time 是否增长。

## 问题

直接执行 `make postroute`（实际 target 为 `Ic2Postroute`）运行过程中，发现 `pnr/rpt/` 下已有 Ic2Place 的报告，但 `pnr/flag/` 下没有对应 flag。阶段到底跑没跑完？

## 结论

阶段未结束。rpt 在 place_opt 各子步骤执行过程中陆续输出；flag 在阶段成功完成后才创建。place 和 route 是全流程最耗时的两个阶段，运行数小时属正常。

## 原因

- place_opt 内部有多个子步骤：`initial_place → initial_drc → initial_opto → final_place`，每个子步骤都会写报告和存盘快照（可在 cload 菜单中看到对应 cell，见 [03_cload数据库菜单解读.md](03_cload数据库菜单解读.md)）。
- flag 机制（Quick Start PPT 第 14 页）：每阶段跑完且 log 无 Error 才在 `pnr/flag/` 生成 flag；有 Error 不生成。下一阶段检测不到前置 flag 会自动重跑前置阶段。
- 直接 make 后面阶段的 target 时，若前面阶段 flag 缺失，会从头依次自动重跑整条链——这是直接 `make Ic2Postroute` 运行时间长的预期行为。

## 正确写法（状态查看命令组）

在 run 目录下：

```bash
bjobs -l                     # 任务 Status 是否 RUN、CPU time 是否增长、内存占用
bpeek <jobid>                # LSF 任务实时标准输出（可选）
ls -lt pnr/flag/             # 已完成阶段清单
ls -lt pnr/log/ | head -5    # 找到正在增长的日志
tail -f pnr/log/<最新日志>    # 实时跟踪，看 place_opt 子阶段关键字
ls -ltr pnr/rpt/ | tail -10  # 最新报告名 = 刚完成的子步骤
ls -lt pnr/run/ | head       # 当前 run 目录活动
```

## 验证（判断标准）

- **正常运行**：log 文件 mtime 每隔几分钟仍在刷新 + `bjobs -l` 的 CPU time 持续增长 → 安心等待。
- **疑似挂死**：log mtime 十几分钟不更新 + CPU time 不涨 → 需要排查。
- **阶段完成确认**：cload 菜单中出现 `final_place` 快照可佐证 place 子步骤全部执行过；但最终成败仍以 log 有无 Error、flag 是否生成为准。

## 关联

- flag 缺失时下一阶段会自动重跑前置阶段（PPT 第 14 页）；重跑某阶段 = 删除对应 flag。
- Error 可 waive 的两个办法：改 `tune/checkList`，或手动 `touch` flag 文件。
