# Ic2Floorplan 失败：CMD-013 工具版本不匹配

## 总结

【重点】首次 `make Ic2Floorplan` 失败，log 报 `Error: Invalid option name 'shell.common.enable_deterministic_mode' (CMD-013)`。根因：override.tcl 未填写 FC 版本/路径变量，flow 启动了默认（旧）版本 FC，与 CFlow 2.1.2 按 U-2023.12-SP6 编写的 settings 脚本不兼容，init_design 阶段即中止，未生成 flag。修复方向：在 override.tcl 中配置 PR 阶段工具版本/路径（PPT 第 10 页变量），确认版本匹配后重跑。

## 问题

首次 Floorplan 的 log 末尾：

```text
if {$synopsys_program_name == "fc_shell"} {
        set_app_options -name shell.common.enable_deterministic_mode -value true
}
Error: Invalid option name 'shell.common.enable_deterministic_mode'  (CMD-013)
Information: script '.../cflow/COMMONWORK/2.1.2/domains/icc2_shell/yundu_u.202602/scripts/yundu_u/s4/init_design.tcl.4nm_s.settings'
        stopped at line 7 due to error. (CMD-081)
Information: script '.../pnr/cmd/Ic2Floorplan.cmd' stopped at line 141 due to error. (CMD-081)
fc_shell> exit
CPU usage: 42 seconds / Elapsed: 878 seconds
```

## 结论

脚本语法与实际启动的 FC 版本不匹配。Floorplan 在最早的 init_design 设置阶段中止，没有开始实际摆放；无 flag 生成。

## 原因

1. override.tcl 中控制 PR 阶段工具版本/路径的变量未填写（PPT 第 10 页）。
2. CFlow 用默认路径启动了一个 FC，其版本与 CFlow 2.1.2 settings 脚本期望的版本（脚本头部注释写明 U-2023.12-SP6）不一致。
3. settings 脚本是新版本语法，旧工具不认识该 app option → `Invalid option name` → 脚本中止 → 调用方 `Ic2Floorplan.cmd` 连锁中止。
4. CFlow 的脚本与其调用的工具版本是配套的，只换工具不换脚本（或反之），会在最早的 `set_app_options` 处暴露。

## 读 log 方法论（【常用】，可复用到所有 FC log）

1. **找第一个 Error，不看最后一个**——后面的 CMD-081 都是连锁反应。
2. **认错误码**：CMD-013 = 命令参数/选项非法（优先想版本不匹配）；CMD-081 = 脚本中止，只告诉你出错位置。
3. **调用栈从下往上读**，定位真正出错的文件和行号：`Ic2Floorplan.cmd:141 → cftag Ic2Floorplan_updateSetting → init_design.tcl.4nm_s.settings:7`（真正出错点在最深处）。
4. **结尾资源统计佐证**：CPU 42s 远小于 Elapsed 878s = 时间全花在启动加载上、没干活就死了；错误发生在 init 阶段 = 配置问题而非设计问题。
5. **对照 flag 机制收尾**：有 Error → 无 flag → 下一阶段会自动重跑；修完配置先确认旧 flag 状态再重新 make。

## 正确写法

- 在 override.tcl 中填写 PR 阶段 FC 工具版本/路径变量（PPT 第 10 页所述变量），使 flow 启动与 CFlow 2.1.2 配套的 FC 版本。
- 修改后确认 `pnr/flag/` 下无残留的 Ic2Floorplan 相关 flag，再重新 make。

## 验证

- 启动 flow 后在 log 开头 banner 或 `fc_shell -version` 查看实际版本，与 settings 脚本头部注释的 U-2023.12-SP6 对比。
- 在 fc_shell 中 `report_app_options shell.common.*` 确认 `enable_deterministic_mode` 选项存在。
- 重跑后 `pnr/flag/` 生成 Ic2Floorplan flag 即修复成功。
