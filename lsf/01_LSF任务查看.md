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

