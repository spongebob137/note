# 03 LSF 任务提交与终止

## 总结

【重点】`bsub` 向队列提交任务，返回的 Job ID 是后续 `bjobs` 查询和 `bkill` 终止的唯一关键标识。本次先提交一个单核后台低负载任务，确认状态后再专门练习 `bkill`，不能误用当前 FC 的 Job ID `435535`。

## 1. 建立练习目录

```bash
mkdir -p "$HOME/lsf_lab"
```

## 2. 提交后台练习任务

```bash
bsub -q rhel8 -n 1 -J lsf_training_sleep \
  -o "$HOME/lsf_lab/lsf_training_sleep.%J.out" \
  "echo START; hostname; date; sleep 300; echo END; date"
```

参数：

| 参数 | 含义 |
|---|---|
| `-q rhel8` | 指定提交到 `rhel8` 队列 |
| `-n 1` | 申请 1 个 job slot |
| `-J lsf_training_sleep` | 设置便于识别的任务名 |
| `-o path` | 把标准输出写入指定文件 |
| `%J` | LSF 在文件名中替换为实际 Job ID |

最后的双引号内容是交给执行主机运行的 Shell 命令。`sleep 300` 使任务保留约五分钟，方便练习 `bjobs` 和 `bkill`；它基本不消耗 CPU。

### 本次提交结果

```text
JOBID: 436363
STAT: RUN
QUEUE: rhel8
FROM_HOST: srv244
EXEC_HOST: srv102
JOB_NAME显示: *ing_sleep
```

`*ing_sleep` 是 `bjobs` 表格列宽不足时对较长任务名的截断显示，不表示提交时设置的名称发生变化。可用 `bjobs -l 436363` 查看完整信息。

## 3. bkill

终止任务前先用 `bjobs JOBID` 再次核对 Job ID、状态和任务名，然后执行：

```bash
bkill 436363
```

`bkill` 向 LSF 请求终止指定任务；状态更新可能不是瞬间完成。随后使用：

```bash
bjobs -a 436363
```

检查最终状态。练习任务在 `sleep` 期间被终止时，输出文件通常已有 `START`、执行主机和开始时间，但不会出现任务命令中的 `END`。

## 4. 后台、`-I` 与 `-Is`

不带交互选项时：

```bash
bsub -q rhel8 command
```

任务在后台执行，`bsub` 返回 Job ID 后当前 Linux Shell 可以继续使用。任务输出通常通过 `-o` 写入文件。

使用 `-I`：

```bash
bsub -I -q rhel8 command
```

当前终端等待任务并连接其标准输入、输出和错误输出，适合需要在前台观察的任务。

使用 `-Is`：

```bash
bsub -Is -q rhel8 -n 4 /path/to/fc_shell
```

除交互连接外，还分配伪终端并提供 shell mode 支持，适合 `fc_shell`、Shell 等需要终端行为的交互程序。本次 `bjobs -l 435535` 中的：

```text
Interactive pseudo-terminal shell mode
```

正是 `-Is` 的结果。

【重点】交互方式改变的是终端连接方式，不会绕过队列调度；任务仍可能先处于 `PEND`。正常完成应使用程序自身的 `exit`/退出命令，卡死或无法响应时才从另一个终端使用 `bkill JOBID`。

## 5. `-M` 与内存资源申请

```bash
bsub -M mem_limit command
```

`-M` 设置任务内存上限，不等同于为任务预留相同大小的内存。默认单位及其按进程或按整个 Job 的强制方式受 `LSF_UNIT_FOR_LIMITS`、`LSB_JOB_MEMLIMIT`、`LSB_MEMLIMIT_ENFORCE` 等集群配置影响；超过限制时任务可能被系统或 LSF 终止。

向调度器声明内存资源需求通常使用：

```bash
bsub -R "rusage[mem=amount]" command
```

二者区别：

| 写法 | 主要目的 |
|---|---|
| `-M value` | 限制任务最多允许使用多少内存 |
| `-R "rusage[mem=value]"` | 告诉调度器任务预计需要多少内存，用于主机选择和资源预留 |

实际项目中可能同时使用资源申请和略高的内存上限，但必须沿用公司 flow 模板，因为单位以及并行任务按 slot 还是按 Job 计算会影响总申请量。不要在未确认单位时直接照抄裸数字。

官方参考：

- [IBM bsub -M](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=o-m)
- [IBM bsub -R](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=o-r)

## 6. 当前进度

- `bsub` 后台提交：已练习。
- `bjobs JOBID` 跟踪：已练习。
- `bkill JOBID`：已练习。
- `-I`/`-Is` 交互提交：已学习。
- 内存参数：已学习概念，实际数值遵循项目模板。
