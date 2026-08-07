# LSF 重点总结

## 总结

【重点】LSF 把任务从登录主机提交到队列，由调度器选择执行主机并分配 slot。工作主线是：用 `bsub` 提交、用 `bjobs` 跟踪、异常时用 `bkill` 终止；提交前后用 `bqueues`、`bhosts`、`lsload` 分别观察队列、主机 slot 和动态负载。

## 1. 一条任务的完整路径

```text
登录主机 srv244
    -> bsub 提交
    -> rhel8 队列等待与调度
    -> 执行主机 srv120/srv102 运行
    -> bjobs 按 Job ID 跟踪
    -> 程序正常退出，或卡死时 bkill
```

登录主机主要用于提交和管理任务，大型 EDA 计算应由 LSF 分配到执行主机，避免直接占用登录节点资源。

## 2. 六条必会命令

| 命令 | 作用 | 最需要看的信息 |
|---|---|---|
| `bsub` | 提交任务 | 队列、slot、交互方式、输出文件、返回的 Job ID |
| `bjobs` | 查看自己的任务 | `JOBID`、`STAT`、`QUEUE`、`EXEC_HOST`、`JOB_NAME` |
| `bkill JOBID` | 终止指定任务 | 执行前必须再次核对 Job ID 和任务名 |
| `bqueues` | 查看队列 | `Open:Active`、`PEND`、`RUN`、`SUSP` |
| `bhosts` | 查看执行主机 | `STATUS`、`MAX`、`NJOBS`、`RUN`、暂停和预留 slot |
| `lsload` | 查看动态负载 | `r15s/r1m/r15m`、`ut`、`pg`、`tmp`、`swp`、`mem` |

## 3. bsub 常用结构

### slot 是什么

slot 是 LSF 用于调度和统计任务并发容量的基本单位。提交时：

```bash
bsub -n 4 command
```

表示向 LSF 申请 4 个 job slot。LSF 会在满足队列和资源规则的执行主机上为任务分配这 4 个 slot。

slot 不等同于以下概念：

- 不等同于 Job ID：一个并行 Job 可以占用多个 slot。
- 不等同于进程或线程：EDA 工具可以在 4 个 slot 内创建很多进程或线程。
- 不等同于 CPU 利用率：slot 已分配只表示调度容量被占用，不代表 CPU 一直满载。
- 不一定严格等同于物理 CPU 核：通常与可使用的并发计算资源相关，但具体映射由集群和队列配置决定。

【重点】`-n 4` 只向 LSF 申请 4 个 slot。EDA 工具是否真正进行 4 路并行，还需要工具自身支持并启用多核或多线程选项。

### 交互输入输出与伪终端

交互输入输出表示当前终端与执行主机上的程序保持双向数据连接：键盘输入传给远端程序，程序的标准输出和错误输出返回当前屏幕。

```text
键盘输入 -> 当前终端 -> LSF -> 执行主机上的程序
屏幕输出 <- 当前终端 <- LSF <- 执行主机上的程序
```

`bsub -I` 建立输入输出连接，但远端程序不一定认为自己连接在真正的终端设备上。普通短命令通常可以工作，而 Bash、EDA Shell 等依赖终端能力的程序可能缺少命令行编辑、控制键、终端尺寸或作业控制等行为。

`bsub -Is` 还在执行主机上建立 pseudo-terminal（PTY，伪终端）：

```text
本地终端 <-> LSF连接 <-> 伪终端PTY <-> Bash/fc_shell
```

伪终端是由软件提供的终端设备接口。远端 Bash 或 `fc_shell` 会把它当作正常终端，因此能显示提示符、逐行接收命令并处理常用控制键。`-Is` 不等于启动 GUI；它提供的是终端交互环境。

### 队列和执行主机的关系

队列是任务等待和应用调度规则的逻辑入口，执行主机是真正提供 CPU、内存并运行程序的机器。

```text
登录主机 -> bsub -q 队列 -> 队列等待/规则筛选 -> 执行主机
```

一个队列通常关联多台候选执行主机；一台执行主机也可能同时服务多个队列。队列可以定义允许使用的主机或主机组、优先级、slot 限制、资源阈值和用户权限。

提交 `-q rhel8` 只选择 `rhel8` 这套调度入口和规则。LSF 再根据队列允许的主机、当前 slot、负载和任务资源要求选择 `srv83` 等具体执行主机。因此，同一条提交命令在不同时间可能运行到不同主机；没有合适主机时，任务保持 `PEND`。

后台任务：

```bash
bsub -q QUEUE -n SLOTS -J JOB_NAME -o OUTPUT_FILE "command"
```

交互式 EDA Shell：

```bash
bsub -Is -q QUEUE -n SLOTS /path/to/eda_shell
```

常用参数：

| 参数 | 含义 |
|---|---|
| `-q rhel8` | 指定队列 |
| `-n 4` | 申请 4 个 job slot，不自动保证工具使用 4 核 |
| `-J name` | 设置任务名 |
| `-o file.%J.out` | 标准输出写入文件，`%J` 替换为 Job ID |
| `-I` | 前台交互连接 |
| `-Is` | 交互连接并建立伪终端和 shell mode，适合 `fc_shell` |

## 4. 任务状态

| 状态 | 含义 | 常见处理 |
|---|---|---|
| `PEND` | 等待资源或调度条件 | 用 `bjobs -l JOBID` 查看等待原因 |
| `RUN` | 正在执行 | 继续观察运行主机和资源使用 |
| `DONE` | 正常完成 | 检查输出和结果文件 |
| `EXIT` | 异常退出或被终止 | 检查日志、退出原因和资源限制 |
| `PSUSP` | 调度原因暂停 | 查看详细状态和队列条件 |
| `USUSP` | 用户暂停 | 确认是否需要恢复或终止 |
| `SSUSP` | 系统或负载原因暂停 | 查看主机负载与调度信息 |

`bjobs` 默认关注未结束任务；`bjobs -l JOBID` 查看详情，`bjobs -a JOBID` 可查看仍在 LSF 历史保留范围内的已结束任务。

## 5. 三种资源视角

```text
bqueues：队列整体是否开放、有多少 slot 等待或运行
bhosts ：某台主机配置和占用了多少 slot
lsload ：某台主机此刻的 CPU、运行队列、内存和磁盘负载
```

【易错】slot 占用率不等于 CPU 利用率。本次 `srv120` 有 `92/100` 个 slot 在运行，但 `lsload` 采样的 CPU 利用率只有 `15%`。

## 6. 内存限制与申请

```text
-M value                  设置内存使用上限
-R "rusage[mem=value]"    向调度器声明内存资源需求
```

`-M` 不是内存预留。单位、强制方式，以及并行任务按 slot 还是按 Job 计算，可能受公司集群配置影响；实际数值必须沿用项目 flow 模板，不能直接照抄其他项目。

## 7. 本次实操记录

### FC 交互任务

```text
Job ID: 435535
Status: RUN
Queue: rhel8
Submit host: srv244
Execution host: srv120
Allocated slots: 4
Observed memory: 25.8 Gbytes
Swap: 0
Eligible pending time: 1 second
```

### 队列与主机快照

```text
rhel8: Open:Active, NJOBS 2917, PEND 1, RUN 2916, SUSP 0
srv120: ok, MAX 100, NJOBS 92, RUN 92, suspended/reserved 0
srv120 load: ut 15%, tmp 580G, swp 254G, mem 1.2T
```

这些数值是 2026-07-31 的即时快照，只用于理解字段，不代表长期容量。

### 后台与终止练习

```text
Training Job ID: 436363
Queue: rhel8
Execution host: srv102
State before termination: RUN
```

使用该专用任务完成了 `bsub -> bjobs -> bkill -> bjobs -a` 的操作链，避免影响 FC 任务 `435535`。

## 8. 安全检查顺序

```bash
bjobs JOBID
bjobs -l JOBID
bkill JOBID
bjobs -a JOBID
```

终止前必须核对 Job ID、任务名、队列和执行主机。正常 EDA 任务优先使用工具自身的退出命令；只在任务卡死或无法正常退出时使用 `bkill`。

## 9. 参考

- [IBM bsub -M](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=o-m)
- [IBM bsub -R](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=o-r)
- 分主题笔记：[01 任务查看](01_LSF任务查看.md)、[02 集群资源查询](02_LSF集群资源查询.md)、[03 任务提交与终止](03_LSF任务提交与终止.md)
