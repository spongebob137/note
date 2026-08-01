## 建立保持时间

![alt text](image-1.png)

### 建立时间

- **setup**：UFF1 时钟沿到达前，信号需要保持的时间。
- 约束：T<sub>launch</sub> + T<sub>c-q</sub> + T<sub>comb</sub> + T<sub>setup</sub> <= T<sub>capture</sub> + T<sub>clk</sub>（蓝色的时间加上周期 T 要大于红色的时间。红色路径的信号需要在下一个上升沿到达前早到一个 setup 时间。）
- 修复违例：增大周期、加快红色路线的传输速度。

### 保持时间

- **hold**：UFF1 时钟沿到达后信号仍需要保持的时间。
- 约束：T<sub>launch</sub> + T<sub>c-q</sub> + T<sub>comb</sub> >= T<sub>capture</sub> + T<sub>hold</sub>（红色路径传递多于蓝色路径的时间应该大于一个 hold 时间）
- 修复违例：增大红色路线的延迟。

时序图：

![alt text](image.png)








## DFT扫描链和MBIST

- DFT:可测试性设计,芯片设计时，为芯片加入的专用测试硬件，如扫描链和MBIST
    1.  扫描链： 将普通D触发器换为带scan信号的D触发器，再串成扫描链（检查芯片内特定寄存器的输入输出是否正常，扫描链方便数据的写入和读出）

```
流程：
测试数据从SI逐拍移入
        ↓
切换到功能模式
        ↓
打一拍功能时钟，捕获响应
        ↓
切换回扫描模式
        ↓
结果从SO逐拍移出
        ↓
与期望结果比较
```        



