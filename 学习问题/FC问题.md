# FC/CFlow 问题记录

## 总结

本文件只记录 FC/CFlow 实操中真实遇到、值得复用的问题。简单拼写错误不单独记录；一般概念解释加入 FC 对应知识笔记。

## 待实机核验的资料差异

### Floorplan target 名称不一致

**现象：** Quick Start PPT 中同时出现 `Ic2Floorplan` 和 `InFloorplan`。

**处理原则：** 以当前 run 自动生成的 Makefile 中真实 target 与依赖为准，执行前先检索确认。

### 自定义 cftag 文件名不一致

**现象：** PPT 中出现 `custom_cftag_pnr.tcl` 和 `customer_cftag_pnr.tcl` 两种名称。

**处理原则：** 检查当前 `tune/pnr` 目录、生成脚本和 CFlow 版本说明，以实际被 source 的文件为准。

### 微盘资料暂不可读

**现象：** 分享页要求登录并具备访问权限。

**处理原则：** 当前先使用 PPT、当前 CFlow 文件、FC `help`/`man` 和本地 UG；取得微盘内容后再补充核对。

