![image.png](https://raw.githubusercontent.com/oMTSo/images/main/img/20260108125054383.png)为了清晰地推导逆变器的传递函数，我们将采用**状态空间平均法（State-Space Averaging）**结合**小信号扰动法**。推导的核心在于理解 H 桥在双极性调制下是如何将直流电压转化为交流波形的。

---

### 第一步：建立大信号平均模型

在双极性 PWM 调制中，H 桥的四个开关管成对动作：

- 当开关管对 $(S_1, S_4)$ 导通时，桥路输出电压 $v_{bridge} = v_g$。
    
- 当开关管对 $(S_2, S_3)$ 导通时，桥路输出电压 $v_{bridge} = -v_g$。
    

设一个开关周期为 $T_s$，$(S_1, S_4)$ 的导通时间为 $d \cdot T_s$（$d$ 为占空比，范围 $0 \sim 1$），则 $(S_2, S_3)$ 的导通时间为 $(1-d) \cdot T_s$。

一个周期内的平均输出电压 $v_{bridge}$ 为：

$$v_{bridge} = d \cdot v_g + (1 - d) \cdot (-v_g)$$

化简得：

$$v_{bridge} = (2d - 1)v_g$$

---

### 第二步：引入小信号扰动（线性化）

为了得到传递函数，我们需要给稳态变量施加一个微小的扰动，并观察输出的变化。令：

- $d = D + \hat{d}$ （稳态占空比 + 扰动量）
    
- $v_{bridge} = V_{bridge} + \hat{v}_{bridge}$ （稳态输出 + 扰动量）
    
- $v_g = V_g$ （假设直流输入稳定，$\hat{v}_g = 0$）
    

将这些带入平均模型公式中：

$$V_{bridge} + \hat{v}_{bridge} = [2(D + \hat{d}) - 1] V_g$$

展开：

$$V_{bridge} + \hat{v}_{bridge} = (2D - 1)V_g + 2V_g \cdot \hat{d}$$

- **直流项（稳态关系）：** $V_{bridge} = (2D - 1)V_g$
    
- **交流小信号项：** $\hat{v}_{bridge} = 2V_g \cdot \hat{d}$
    

由此得到功率级的控制增益：

$$\frac{\hat{v}_{bridge}(s)}{\hat{d}(s)} = 2V_g$$

---

### 第三步：加入滤波网络（LC 级）

根据你提供的电路拓扑，H 桥输出后接电感 $L$、电容 $C$ 和电阻 $R$。我们利用复阻抗法求从桥路输出电压 $\hat{v}_{bridge}$ 到负载两端电压 $\hat{v}_o$ 的传递函数。

1. **负载阻抗**：$Z_{load} = R \parallel \frac{1}{sC} = \frac{R}{sRC + 1}$
    
2. 分压比：
    
    $$\frac{\hat{v}_o(s)}{\hat{v}_{bridge}(s)} = \frac{Z_{load}}{sL + Z_{load}} = \frac{\frac{R}{sRC + 1}}{sL + \frac{R}{sRC + 1}}$$
    
    上下同乘 $(sRC + 1)$ 并化简：
    
    $$\frac{\hat{v}_o(s)}{\hat{v}_{bridge}(s)} = \frac{1}{LCs^2 + \frac{L}{R}s + 1}$$
    

---

### 第四步：合成总传递函数 $G_{vd}(s)$

将功率级增益和滤波级增益相乘，即可得到从**占空比**到**输出电压**的传递函数：

$$G_{vd}(s) = \frac{\hat{v}_o(s)}{\hat{d}(s)} = \frac{\hat{v}_{bridge}(s)}{\hat{d}(s)} \cdot \frac{\hat{v}_o(s)}{\hat{v}_{bridge}(s)}$$

$$G_{vd}(s) = \frac{2V_g}{LCs^2 + \frac{L}{R}s + 1}$$

这完美对应了你图片中的第一个公式。

---

### 重点总结：为什么分子是 $2v_g$？

通过上面的推导，我们可以清晰地看到 $2$ 的来源：

1. **电平切换幅度**：在双极性调制中，电压在 $+v_g$ 和 $-v_g$ 之间跳变，总压差是 $2v_g$。
    
2. **增益斜率**：在 Buck 电路中，占空比从 $0 \to 1$ 引起 $0 \to V_{in}$ 的变化；而在双极性逆变器中，同样的占空比变化引起了 $-v_g \to +v_g$ 的变化，其变化率是 Buck 的两倍。
    

---

### 关于系统闭环 $T(s)$

在你的图片中：

$$T(s) = \frac{H}{V_{ramp}} G_{vd}(s) G_c(s)$$

- **$V_{ramp}$** 是三角波的幅值。在硬件实现中，PWM 比较器会将控制电压 $u_{ctrl}$ 与 $V_{ramp}$ 比较。实际产生的占空比扰动 $\hat{d} = \frac{\hat{u}_{ctrl}}{V_{ramp}}$。因此，$\frac{1}{V_{ramp}}$ 是调制环节的增益。
    
- **$H$** 是电压反馈回路的缩放系数。![image.png](https://raw.githubusercontent.com/oMTSo/images/main/img/20260111214750312.png)

