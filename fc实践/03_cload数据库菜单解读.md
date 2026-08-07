# cload 数据库菜单解读

## 总结

【常用】`cload` 会扫描当前 run 目录 `db` 下所有可打开的 database 并给出选择菜单。菜单三列为：NLIC 库文件（NLI B_NAME）、设计快照名（CELL_NAME）、label（本 flow 均为 NA）。条目实际只有三类：**各阶段主数据库**、**阶段内子步骤快照**、**带箭头的链接入口**（与主库指向同一份数据）。查看用 `cload -read` 只读打开，避免误改。

## 现象

在 run 目录执行 `cload` 后出现编号菜单，形如：

```text
NO.  NLIB_NAME                                                   CELL_NAME                    LABEL_NAME
1:   xbar_station_wrapper_xbar_ok.Ic2Place.nlib                  xbar_station_wrapper_xbar_ok  NA
2:   xbar_station_wrapper_xbar_ok.nlib -> ...Ic2Place.nlib       xbar_station_wrapper_xbar_ok  NA
...
5:   ...Ic2Place.nlib                                            initial_opto                 NA
9:   ...Ic2Place.nlib                                            initial_place                NA
11:  ...Ic2Floorplan.nlib                                        xbar_station_wrapper_xbar_ok  NA
12:  ...Ic2Initial.nlib                                          xbar_station_wrapper_xbar_ok  NA
e:   exit!
```

## 解读

### 三列含义

| 列 | 含义 |
|---|---|
| NLIB_NAME | 数据库所在的 nlib 文件（FC/ICC2 的 NDM 库，一个 nlib 可存多个设计状态） |
| CELL_NAME | 要打开的设计快照名——同一 nlib 内按子步骤存了多个快照 |
| LABEL_NAME | 保存时的 label；本 flow 未使用，均为 NA |

### 三类条目

1. **各阶段主数据库**（CELL_NAME = 设计名本身）：`Ic2Initial.nlib` / `Ic2Floorplan.nlib` / `Ic2Place.nlib`，代表该阶段最后保存的状态。
2. **子步骤中间快照**（CELL_NAME = 子步骤名）：place 阶段有 `initial_place`、`initial_drc`、`initial_opto`、`final_place`，对应 place_opt 内部四个子步骤的存盘。用途是调试对比（例如定位 congestion 从哪个子步骤开始恶化）。看到 `final_place` 存在说明 place 子步骤全部执行过。
3. **带箭头的链接入口**（`A.nlib -> B.nlib`）：主设计 nlib 中存的指向阶段 nlib 的链接。箭头条目与被指向的条目打开的是**同一份数据的两个入口**，不是两个版本。

## 选择建议

| 目的 | 选择 |
|---|---|
| 看某阶段最终结果 | 该阶段主库条目（如 Ic2Place.nlib / 设计名） |
| 对比 place 内部演化 | `initial_place` / `initial_drc` / `initial_opto` / `final_place` 快照 |
| 回溯更早阶段 | Ic2Floorplan / Ic2Initial 主库 |
| 不打开 | `e` |

【易错】

- 若某阶段此前失败过（如 [02_Ic2Floorplan_CMD013_工具版本不匹配.md](02_Ic2Floorplan_CMD013_工具版本不匹配.md) 中的 Ic2Floorplan），其 nlib 数据可能不完整，打开看到异常不要意外。
- 只是查看时用 `cload -read` 只读打开（PPT 第 17 页），符合"参考目录只读"原则。
- 打开后先确认数据健康（如 cell 是否已摆放、有无大量 overlap），再跑 congestion/timing 等 QoR 检查。
