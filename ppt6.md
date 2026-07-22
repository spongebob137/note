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





