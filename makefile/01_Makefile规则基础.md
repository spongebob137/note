# 01 Makefile 规则基础

## 总结

【重点】一条 Makefile 规则由目标、依赖和执行命令组成：`target: prerequisites` 表示目标依赖哪些文件或其他目标；下一行以真正的 Tab 开头，写生成目标所需的 Shell 命令。

## 1. 基本结构

```make
target: prerequisites
	command
```

- `target`：希望生成的文件或希望执行的任务名。
- `prerequisites`：生成目标前需要存在或先完成的文件、目标。
- `command`：也称 recipe，由 Shell 执行，用于真正生成或更新目标。

【易错】recipe 行首通常必须是 Tab，不能用若干空格代替。Markdown 代码块中的显示宽度不能判断实际字符，编辑时应直接按一次 Tab 键。

## 2. 第一个规则

```make
result.txt: source.txt
	cp source.txt result.txt
```

含义：目标是 `result.txt`，依赖是 `source.txt`；目标不存在或依赖比目标更新时，执行 `cp source.txt result.txt`。

## 3. 第一次执行

```bash
make result.txt
```

当 `result.txt` 不存在时，make 执行 recipe 创建它。第二次执行且依赖没有更新时，make 会判断目标已是最新状态，不重复执行复制命令。

