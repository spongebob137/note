# 02 LSF 集群资源查询

## 总结

`bqueues` 查看队列及队列内任务概况，`bhosts` 查看执行主机状态和 slot 使用，`lsload` 查看主机实时负载指标。本节先练习 `bqueues`，所有命令都是只读查询。

## 1. bqueues

查看所有可见队列：

```bash
bqueues
```

常见字段包括队列名、优先级、状态、任务总数，以及 `PEND`、`RUN`、`SUSP` 数量；实际字段会随公司 LSF 配置变化。

只查看当前使用的 `rhel8` 队列：

```bash
bqueues rhel8
```

查看该队列详细配置：

```bash
bqueues -l rhel8
```

队列是否适合某项任务不能只看当前排队数，还需要遵守项目对操作系统、核数、内存和工具许可证的使用规定。

### 本次 rhel8 查询结果

```text
QUEUE_NAME  PRIO  STATUS       MAX  JL/U  JL/P  JL/H  NJOBS  PEND  RUN   SUSP
rhel8       30    Open:Active  -    -     -     -     2917   1     2916  0
```

- `PRIO 30`：队列优先级；通常数值更高的队列在竞争时优先级更高，但不保证任务立即运行。
- `Open:Active`：`Open` 表示接受提交，`Active` 表示可以调度任务。
- `MAX`、`JL/U`、`JL/P`、`JL/H` 为 `-`：没有显示对应的队列总量、每用户、每处理器或每主机 slot 限制；不表示物理资源无限。
- `NJOBS 2917`：队列当前占用或等待的 job slot 总数。
- `PEND 1`、`RUN 2916`、`SUSP 0`：分别有 1 个等待、2916 个运行、0 个暂停的 slot。
- 并行任务可能占多个 slot，因此这些数值不一定等于 Job ID 的数量。

## 2. bhosts

查看所有可见执行主机：

```bash
bhosts
```

已知当前 FC 运行在 `srv120`，可只查看该主机：

```bash
bhosts srv120
```

常见字段包括主机状态、最大 slot 数、当前任务 slot 数、运行数、暂停数和保留数。

### 本次 srv120 查询结果

```text
HOST_NAME  STATUS  JL/U  MAX  NJOBS  RUN  SSUSP  USUSP  RSV
srv120     ok      -     100  92     92   0      0      0
```

- `ok`：主机当前可被 LSF 使用。
- `MAX 100`：主机配置了 100 个 job slot。
- `NJOBS 92`、`RUN 92`：当前有 92 个 slot 分配给运行任务。
- `SSUSP 0`、`USUSP 0`：没有系统暂停或用户暂停的 slot。
- `RSV 0`：没有被预留的 slot。
- 按 slot 计数当前还有 `100 - 92 = 8` 个位置，但是否能调度新任务还受队列规则、资源要求和主机负载条件影响。
- `92/100` 是 slot 使用情况，不是 CPU 利用率。

## 3. lsload

查看当前 FC 所在主机的动态负载：

```bash
lsload srv120
```

常见字段：

| 字段 | 含义 |
|---|---|
| `status` | LSF 负载状态 |
| `r15s`、`r1m`、`r15m` | 不同时间窗口的运行队列负载 |
| `ut` | CPU 利用率 |
| `pg` | 分页活动指标 |
| `ls` | 登录会话数 |
| `it` | 主机空闲时间 |
| `tmp` | 可用临时磁盘空间 |
| `swp` | 可用交换空间 |
| `mem` | 可用内存 |

数值单位和 busy 阈值由公司的 LSF 配置决定，应结合表头、后缀和项目规范解释。

### 本次 srv120 动态负载

```text
HOST_NAME  status  r15s  r1m  r15m  ut   pg   ls  it     tmp   swp   mem
srv120     ok      32.9  37.9 42.4  15%  4.7  0   23392  580G  254G  1.2T
```

- `status ok`：动态负载没有触发 LSF 的关闭或 busy 条件。
- `r15s 32.9`、`r1m 37.9`、`r15m 42.4`：短期值低于长期值，说明近期运行队列负载相对下降。这些是负载指数，不是百分比，需结合主机 CPU 数量和集群配置判断绝对高低。
- `ut 15%`：采样时 CPU 利用率为 15%。这证明 `bhosts` 的 92/100 slot 占用率不能当作 CPU 利用率。
- `pg 4.7`：存在一定分页活动，是否异常需结合公司设定的阈值和持续时间判断。
- `ls 0`：没有记录到交互登录会话。
- `it 23392`：交互空闲时间指标，不代表 CPU 已空闲这么久。
- `tmp 580G`、`swp 254G`、`mem 1.2T`：分别是可用临时空间、交换空间和内存。

## 4. 当前进度

- `bqueues`：已练习。
- `bhosts`：已练习。
- `lsload`：已练习。
