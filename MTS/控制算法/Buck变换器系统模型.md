# BUCK模型
![image.png](https://raw.githubusercontent.com/oMTSo/images/main/img/20260108124111596.png)
分析 Buck 变换器的传递函数，我们需要将其拆解为一个**闭环控制系统**。根据你的描述，这个系统由功率级（Buck 电路）、PWM 调制器、PI 控制器和反馈环节四个主要部分组成。

为了得到最终的传递函数，我们通常采用**小信号平均模型**（Small-signal Average Model）在连续导通模式（CCM）下进行分析。

---

### 1. 功率级（Plant）传递函数：$G_{vd}(s)$

功率级描述的是**占空比 $d$** 的变化如何影响**输出电压 $v_{out}$**。Buck 电路本质上是一个带有增益的二阶 LC 低通滤波器。

假设电感为 $L$，电容为 $C$，负载电阻为 $R$，输入电压为 $V_{in}$。

==其占空比到输出电压的传递函数为==：

$$G_{vd}(s) = \frac{\hat{v}_{out}(s)}{\hat{d}(s)} = V_{in} \cdot \frac{1}{1 + s\frac{L}{R} + s^2LC}$$

- **物理意义：** 分子 $V_{in}$ 是直流增益；分母是一个标准的二阶系统，其自然频率 $\omega_n = \frac{1}{\sqrt{LC}}$，阻尼由负载 $R$ 决定。
#### 记忆口诀
 - 第一步：看直流增益（令 $s = 0$）
    
    当电路达到稳态（直流）时，电感看作短路，电容看作开路。此时输出电压 $V_{out} = D \cdot V_{in}$。
    
    那么 $\frac{V_{out}}{D} = V_{in}$。所以，分子必须是 $V_{in}$。
    
- 第二步：看分母的“头”和“尾”
    
    这是一个二阶 LC 滤波器，分母一定是一个关于 $s$ 的二次多项式。
    
    - **常数项（尾）**：通常归一化为 **1**。
        
    - **$s^2$ 项（头）**：必然是两个储能元件的乘积 **$LC$**。这对应了系统的自然频率 $\omega_n = 1/\sqrt{LC}$。
        
- 第三步：看中间的阻尼项（$s$ 项）
    
    中间项决定了能量怎么损耗。在 Buck 中，电感电流流向负载 $R$，所以阻尼项是 $L/R$（注意：不是 $RC$，因为电感在二阶动态中占主导）。
    
#### [[复阻抗法]]
使用复阻抗法推导 Buck 变换器的功率级传递函数 $G_{vd}(s)$，核心在于将“开关电路”等效为一个**受控电压源驱动的线性滤波网络**。

通过复阻抗法，你可以绕过复杂的微分方程，直接利用**分压定律**得出结果。以下是具体推导步骤：

---

##### 第一步：建立 $s$ 域等效模型

在连续导通模式（CCM）下，Buck 开关管和二极管的平均作用相当于在 LC 滤波器输入端产生了一个等效电压 $v_x$：

- **平均电压**：$v_x = d \cdot V_{in}$
    
- **小信号扰动**：$\hat{v}_x = V_{in} \cdot \hat{d}$ （这是我们要研究的输入量）
    

现在，电路变成了一个简单的 **$s$ 域分压网络**。

---

##### 第二步：定义各支路的复阻抗

我们要计算的是输出电压 $V_{out}(s)$ 相对输入电压 $V_x(s)$ 的比例。

1. 电感支路阻抗：
    
    $$Z_1(s) = sL$$
    
2. 负载与电容并联支路阻抗（记为 $Z_L$）：
    
    这是电容 $\frac{1}{sC}$ 和电阻 $R$ 的并联：
    
    $$Z_L(s) = R \parallel \frac{1}{sC} = \frac{R \cdot \frac{1}{sC}}{R + \frac{1}{sC}} = \frac{R}{1 + sRC}$$
    

---

##### 第三步：套用分压公式

根据复阻抗分压定律，输出电压等于等效输入电压在负载支路上的分压：

$$V_{out}(s) = V_x(s) \cdot \frac{Z_L(s)}{Z_1(s) + Z_L(s)}$$

将 $V_x(s) = V_{in} \cdot D(s)$ 以及阻抗式代入：

$$\frac{V_{out}(s)}{D(s) \cdot V_{in}} = \frac{\frac{R}{1 + sRC}}{sL + \frac{R}{1 + sRC}}$$

---

##### 第四步：代数化简

这是最关键的一步，目的是把分式化成标准型（常数项为 1）：

1. 分子分母同时乘以 $(1 + sRC)$：
    
    $$G(s) = \frac{R}{sL(1 + sRC) + R}$$
    
2. 展开分母：
    
    $$G(s) = \frac{R}{sL + s^2LRC + R}$$
    
3. 分子分母同时除以 $R$（归一化常数项）：
    
    $$G(s) = \frac{1}{s^2LC + s\frac{L}{R} + 1}$$
    

##### 第五步：得到最终传递函数 $G_{vd}(s)$

由于我们要找的是 $\frac{V_{out}(s)}{D(s)}$，而上面的结果是 $\frac{V_{out}(s)}{V_{in} \cdot D(s)}$，所以最后需要乘回 $V_{in}$：

$$G_{vd}(s) = \frac{V_{out}(s)}{D(s)} = V_{in} \cdot \frac{1}{s^2LC + s\frac{L}{R} + 1}$$


---

### 2. PWM 调制器增益：$F_m$

PWM 调制器将控制器的输出电压 $v_c$ 转化为开关的占空比 $d$。它通常通过将控制信号与一个幅值为 $V_{ramp}$（锯齿波峰值）的载波进行比较来实现。

==其等效增益为一个常数==：

$$F_m = \frac{\hat{d}(s)}{\hat{v}_c(s)} = \frac{1}{V_{ramp}}$$

---

### 3. PI 控制器传递函数：$G_c(s)$

你提到的 PI 控制器对误差信号（参考电压 $V_{ref}$ 与 反馈电压的差值）进行处理。

==其传递函数为==：

$$G_c(s) = K_p + \frac{K_i}{s} = \frac{K_p s + K_i}{s}$$

- **特点：** 分母里的 $s$ 提供了一个极点在原点（积分项），用于消除稳态误差；分子提供了一个零点，用于改善系统的相位裕度。
    

---

### 4. 闭环系统总框图与传递函数

我们将这些环节串联起来。假设反馈系数为 $H$（如果是直接反馈，则 $H=1$）。

#### 开环增益（Loop Gain）：$T(s)$

这是控制理论中最关键的函数，决定了系统的稳定性和响应速度：

$$T(s) = G_c(s) \cdot F_m \cdot G_{vd}(s) \cdot H$$

代入具体公式：

$$T(s) = \left( \frac{K_p s + K_i}{s} \right) \cdot \left( \frac{1}{V_{ramp}} \right) \cdot \left( \frac{V_{in}}{1 + s\frac{L}{R} + s^2LC} \right) \cdot H$$

#### 闭环传递函数（Closed-Loop TF）：$G_{CL}(s)$

描述的是参考电压 $V_{ref}$ 到输出电压 $v_{out}$ 的关系：

$$\Phi(s) = \frac{v_{out}(s)}{V_{ref}(s)} = \frac{G_c(s) F_m G_{vd}(s)}{1 + T(s)}$$






# BUCK带esr
当电容串联一个等效串联电阻（ESR，通常记作 $R_c$）时，Buck 变换器的功率级传递函数会发生一个非常关键的变化：**分子会出现一个左半平面的零点**。

这在电源设计中极其重要，因为这个零点会提供相位超前，有时能帮助系统稳定，但也会增加高频噪声。

以下是考虑 ESR 后的推导过程和结果：

### 1. 修正输出阻抗 $Z_{out}$

在之前的推导中，输出部分是电容 $C$ 直接与负载 $R$ 并联。现在，电容支路变成了 $R_c$ 与 $C$ 串联。

- **电容支路阻抗**：$Z_{cap}(s) = R_c + \frac{1}{sC} = \frac{1 + sCR_c}{sC}$
    
- 总输出阻抗（电容支路并联负载 $R$）：
    
    $$Z_{out}(s) = \frac{Z_{cap} \cdot R}{Z_{cap} + R} = \frac{R(1 + sCR_c)}{1 + sC(R + R_c)}$$
    

### 2. 得到功率级传递函数 $G_{vd}(s)$

同样利用分压原理（在平均化和小信号线性化之后），占空比到输出的传递函数为：

$$G_{vd}(s) = V_{in} \cdot \frac{Z_{out}(s)}{sL + Z_{out}(s)}$$

将 $Z_{out}(s)$ 代入并化简，得到考虑 ESR 的功率级标准型：

$$G_{vd}(s) = V_{in} \cdot \frac{1 + sCR_c}{s^2LC(1 + \frac{R_c}{R}) + s(\frac{L}{R} + CR_c + \frac{LCR_c}{R}) + 1}$$

在实际工程中，通常负载电阻 $R \gg R_c$，公式可以近似简化为：

$$G_{vd}(s) \approx V_{in} \cdot \frac{1 + sCR_c}{s^2LC + s(\frac{L}{R} + CR_c) + 1}$$

---

### 3. 关键变化分析

相比于不带 ESR 的模型，这个公式多出了以下特性：

#### A. ESR 零点 (The ESR Zero)

分子上的 $(1 + sCR_c)$ 项引入了一个零点：

$$f_{esr} = \frac{1}{2\pi CR_c}$$

- **物理意义**：这个零点会在 $f_{esr}$ 频率处使增益以 +20dB/dec 速度上升，并提供 **+90° 的相位超前**。
    
- **对控制的影响**：在你的 PI 控制器设计中，这个零点可以抵消 LC 滤波器带来的部分相位滞后，有助于提高系统的相位裕度。但如果 $R_c$ 很大，零点频率过低，可能会导致系统高频增益过大，对开关噪声变得异常敏感。
    

#### B. 阻尼的变化

分母 $s$ 项系数从 $L/R$ 变成了 $L/R + CR_c$。这说明 **ESR 实际上增加了系统的阻尼**。电解电容的 ESR 通常较大，系统比较容易稳定；而陶瓷电容的 ESR 极小，系统往往更容易发生剧烈震荡。

---

### 4. 最终全系统传递函数

考虑 PI 控制器 $G_c(s) = \frac{K_p s + K_i}{s}$ 和 PWM 增益 $F_m = 1/V_{ramp}$，整个开环回路增益 $T(s)$ 变为：

$$T(s) = \underbrace{\frac{K_p s + K_i}{s}}_{PI} \cdot \underbrace{\frac{1}{V_{ramp}}}_{PWM} \cdot \underbrace{V_{in} \frac{1 + sCR_c}{s^2LC + s(\frac{L}{R} + CR_c) + 1}}_{Plant}$$

# 具体推导过程
推导 Buck 变换器的全系统传递函数，通常采用**状态空间平均法（State-Space Averaging）**。我们将整个过程分为三步：功率级建模、PWM 调制器建模、以及闭环系统整合。

---

### 第一步：功率级（Plant）的建模与推导

Buck 变换器是一个开关电路，我们需要通过平均化处理，将开关电路转化为连续的线性模型。

#### 1. 列写两个状态的微分方程

假设电感电流为 $i_L$，输出电压（电容电压）为 $v_{out}$。

- 状态 1：开关管导通（ON），持续时间 $dT$
    
    根据 KVL 和 KCL：
    
    $$\begin{cases} L \frac{di_L}{dt} = V_{in} - v_{out} \\ C \frac{dv_{out}}{dt} = i_L - \frac{v_{out}}{R} \end{cases}$$
    
- 状态 2：开关管关断（OFF），持续时间 $(1-d)T$
    
    此时二极管续流：
    
    $$\begin{cases} L \frac{di_L}{dt} = -v_{out} \\ C \frac{dv_{out}}{dt} = i_L - \frac{v_{out}}{R} \end{cases}$$
    

#### 2. 状态空间平均化

我们将上述两个状态按占空比 $d$ 进行加权平均，得到一个周期的平均方程：

$$L \frac{d\bar{i}_L}{dt} = d(V_{in} - \bar{v}_{out}) + (1-d)(-\bar{v}_{out}) = d \cdot V_{in} - \bar{v}_{out}$$

$$C \frac{d\bar{v}_{out}}{dt} = \bar{i}_L - \frac{\bar{v}_{out}}{R}$$

#### 3. 小信号线性化

令变量包含直流分量（大写）和微小扰动（带 $\hat{}$ 的小写）：

$\bar{v}_{out} = V_{out} + \hat{v}_{out}$, $\bar{i}_L = I_L + \hat{i}_L$, $d = D + \hat{d}$。代入平均方程并忽略掉两个扰动相乘的高阶项（如 $\hat{d}\hat{v}_{out}$）：

$$L \frac{d\hat{i}_L}{dt} = D \cdot \hat{v}_{in} + \hat{d} \cdot V_{in} - \hat{v}_{out}$$

假设输入电压恒定（$\hat{v}_{in} = 0$），对上述方程组进行拉普拉斯变换：

$$\begin{cases} sL\hat{I}_L(s) = \hat{D}(s)V_{in} - \hat{V}_{out}(s) \\ sC\hat{V}_{out}(s) = \hat{I}_L(s) - \frac{\hat{V}_{out}(s)}{R} \end{cases}$$

#### 4. 求解功率级传递函数 $G_{vd}(s)$

联立消去 $\hat{I}_L(s)$，解得从占空比到输出电压的传递函数：

$$G_{vd}(s) = \frac{\hat{V}_{out}(s)}{\hat{D}(s)} = \frac{V_{in}}{LCs^2 + \frac{L}{R}s + 1}$$

---

### 第二步：PWM 调制器建模 $F_m$

调制器将控制电压 $v_c$ 与锯齿波 $v_{ramp}$ 比较。

- 当 $v_c > v_{ramp}$ 时开关导通。
    
- 占空比 $d = \frac{v_c}{V_{ramp}}$，其中 $V_{ramp}$ 是锯齿波的峰值幅度。
    

线性化后得到等效增益：

$$F_m = \frac{\hat{d}(s)}{\hat{v}_c(s)} = \frac{1}{V_{ramp}}$$

---

### 第三步：PI 控制器建模 $G_c(s)$

根据你要求的 PI 控制器，其时域方程为：$v_c(t) = K_p e(t) + K_i \int e(t)dt$。

对应的 $s$ 域传递函数为：

$$G_c(s) = \frac{\hat{V}_c(s)}{\hat{E}(s)} = K_p + \frac{K_i}{s} = \frac{K_p s + K_i}{s}$$

---

### 第四步：闭环全系统传递函数整合

现在我们将所有积木拼接起来。

1. **误差信号**：$\hat{E}(s) = V_{ref}(s) - H \cdot \hat{V}_{out}(s)$（$H$ 是采样反馈系数，若直接取样则 $H=1$）。
    
2. **前向通路增益**：$G_{forward}(s) = G_c(s) \cdot F_m \cdot G_{vd}(s)$。
    
3. **开环传递函数（回路增益）**：$T(s) = G_{forward}(s) \cdot H$。
    

#### 最终闭环传递函数（从 $V_{ref}$ 到 $V_{out}$）：

$$\Phi(s) = \frac{\hat{V}_{out}(s)}{V_{ref}(s)} = \frac{G_c(s) F_m G_{vd}(s)}{1 + G_c(s) F_m G_{vd}(s) H}$$

将各部分代入展平：

$$\Phi(s) = \frac{\frac{K_p s + K_i}{s} \cdot \frac{1}{V_{ramp}} \cdot \frac{V_{in}}{LCs^2 + \frac{L}{R}s + 1}}{1 + \frac{K_p s + K_i}{s} \cdot \frac{1}{V_{ramp}} \cdot \frac{V_{in}}{LCs^2 + \frac{L}{R}s + 1} H}$$

整理后，你会得到一个**三阶系统**（分母最高项为 $s^3$），其中：

- **$s=0$ 处的极点**：来自 PI 控制器的积分项，确保稳态误差为 0。
    
- **二阶共振峰**：来自 $LC$ 滤波器。
    
- **零点**：来自 PI 控制器的 $K_p s + K_i$，可以用来补偿 $LC$ 带来的相位滞后。
    
