# 01 LSF 任务查看

## 总结

【重点】`bjobs` 查看当前用户尚未结束的 LSF 任务；`bjobs -l JOBID` 查看指定任务的详细提交、资源和状态信息。LSF 命令应在 Linux Shell 中执行，不是在 `fc_shell>` 中执行。

## 1. 使用环境

保留当前 FC 任务，另开一个 etx 终端。看到类似下面的 Linux 提示符后再输入 LSF 命令：

```text
[user@server:] ~>>
```

## 2. 查看当前任务

```bash
bjobs
```

常见字段：

| 字段 | 含义 |
|---|---|
| `JOBID` | LSF 分配的任务编号，后续查询和终止任务都依赖它 |
| `USER` | 提交任务的用户 |
| `STAT` | 任务状态 |
| `QUEUE` | 使用的队列 |
| `FROM_HOST` | 提交任务的主机 |
| `EXEC_HOST` | 实际执行任务的计算主机 |
| `JOB_NAME` | 任务名或命令名 |
| `SUBMIT_TIME` | 提交时间 |

常见状态：

| 状态 | 含义 |
|---|---|
| `PEND` | 正在等待资源或调度条件 |
| `RUN` | 正在执行 |
| `DONE` | 正常完成 |
| `EXIT` | 异常退出 |
| `PSUSP`、`USUSP`、`SSUSP` | 分别表示不同原因的暂停状态 |

## 3. 查看一个任务的详情

```bash
bjobs -l JOBID
```

将 `JOBID` 替换为 `bjobs` 第一列的实际数字，不要原样输入单词 `JOBID`。

## 4. 本次实操结果

```text
JOBID: 435535
Status: RUN
Queue: rhel8
Mode: Interactive pseudo-terminal shell mode
Submit host: srv244
Execution host: srv120
Requested/allocated slots: 4
Eligible pending time: 1 second
MEM: 25.8 Gbytes
MAX MEM: 25.8 Gbytes
AVG MEM: 8.6 Gbytes
SWAP: 0 Mbytes
NTHREAD: 95
```

关键解释：

- `Interactive pseudo-terminal shell mode` 对应提交时的 `bsub -Is`。
- `4 Task(s)` 和 `Allocated 4 Slot(s)` 对应 `bsub -n 4`；slot 是 LSF 分配单位，EDA 工具仍需自身启用并行才能有效使用这些资源。
- 四个 slot 都显示在 `srv120`，说明本次任务集中运行在同一执行主机。
- `CPU time` 是任务累计消耗的 CPU 时间，不等于从提交到当前的墙钟时间。
- `MEM` 是本次采集时的内存使用信息，`MAX MEM` 和 `AVG MEM` 分别是已观测峰值与平均值。
- `NTHREAD` 是任务进程的线程数量，不等于申请的 slot 数量，也不能据此认为获得了 95 个 CPU 核。
- `SWAP: 0` 表示采集时没有使用交换空间。
- Eligible pending time 只有 1 秒，说明任务很快获得了可运行资源。
- `Resource requirement` 只显示默认的选择与排序条件，当前输出中没有看到显式内存资源要求。
