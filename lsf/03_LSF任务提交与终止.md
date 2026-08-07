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

### FC 正常退出与运行中终止

FC 空闲并停在 `fc_shell>` 提示符时，优先使用工具自身的 `exit` 正常退出。`stop_gui` 只关闭 GUI，不等于退出 `fc_shell`。如果当前设计允许编辑且确实需要保留修改，应按照项目 flow 在退出前使用规定的 `save_block`、`save_lib` 或 checkpoint；只读数据库和公共设计不能擅自保存。

某条 FC 命令正在执行时：

- 单次 `Ctrl+C` 会请求中断当前命令，命令响应后通常留在 `fc_shell`。
- 如果正在执行 Tcl 脚本，被中断的命令之后的脚本内容不会继续执行。
- 部分命令不能安全响应中断。
- 在命令响应前连续按三次 `Ctrl+C` 可能直接终止 `fc_shell`，未保存数据不会保留。
- 从另一个终端执行 `bkill JOBID` 属于系统层面终止，通常从上一个已保存 block/checkpoint 重新开始。

中途终止前应确认：任务是否真的卡死、日志是否仍在更新、最后一个有效保存点、部分输出能否删除或覆盖、是否会影响共享文件，以及重新运行从哪个阶段开始。不要把中断后的半成品覆盖保存为正式数据库。

### sleep 练习命令

```bash
sleep 600
```

表示进程等待 600 秒后正常返回。它几乎不使用 CPU，但 LSF Job 仍处于 `RUN`，申请的 slot 仍被占用；这与 LSF 的 `SUSP` 暂停状态不同。练习中使用 `sleep` 是为了让低负载测试任务保持足够久，便于观察 `bjobs` 和练习 `bkill`。

如果在 `sleep` 期间执行 `bkill`，后面的 `echo END` 等命令不会执行；如果 Job 还处于 `PEND`，说明 `sleep` 尚未在执行主机开始。

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

### 交互任务启动时出现环境警告

本次复习执行：

```bash
bsub -Is -q rhel8 -n 1 /bin/bash
```

LSF 返回 Job `472084`，并显示 `Starting on srv83`。随后出现：

```text
ABRT has detected 1 problem(s).
Can't locate local/lib.pm: Permission denied.
BEGIN failed--compilation aborted.
```

但最终出现了 `srv83` 上的 Bash 提示符，因此 LSF 调度和 `/bin/bash` 启动已经成功。前两段信息来自执行主机的系统或用户 Shell 初始化过程：ABRT 表示该主机记录过系统问题；Perl 的 `local/lib.pm` 信息表示某段启动初始化尝试加载模块时失败。它们不一定影响当前交互 Shell。

判断方法：

```bash
hostname
echo $LSB_JOBID
echo $0
```

如果能正常返回执行主机、Job ID 和 Shell 名称，并且有可用提示符，就说明交互任务可继续使用。不要为了消除主机级提示自行修改公共环境；若后续 EDA 工具因此无法启动，应保存完整输出并联系环境或 IT 管理人员。

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
