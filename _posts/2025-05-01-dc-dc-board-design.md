---
layout:     post
title:      "DC-DC转换器的板级设计考量"
subtitle:   " \"Board-Level Design Considerations for DC-DC Converters\""
date:       2025-05-01 12:00:00
author:     "flank"
header-img: "img/post-bg-infinity.jpg"
catalog: true
tags:
    - 电子设计
    - 电源设计
    - DC/DC
    - PCB布局
---

# DC-DC转换器的板级设计考量

## I. 引言

### DC-DC转换器的重要性

DC-DC转换器在现代电子系统中扮演着至关重要的角色，它们是实现高效电源管理的核心器件。无论是便携式设备（如智能手机、平板电脑）、汽车电子系统，还是工业控制设备和通信基础设施，几乎都离不开DC-DC转换器来调整和稳定电源电压 1。它们能够将来自电池 2、系统总线 6 或其他电源的输入电压，高效地转换为负载电路（如微处理器、FPGA、传感器、显示屏等）所需的特定电压值。随着电子设备对能效、尺寸和性能的要求日益提高，对DC-DC转换器进行精心的板级设计显得尤为关键。

### 板级设计的核心挑战

DC-DC转换器的板级设计并非简单的元件连接，而是一个涉及多方面考量的复杂工程过程。设计者必须在多个相互制约的目标之间寻求最佳平衡。主要挑战包括：

1. 高效率（Efficiency）： 最大限度地减少功率损耗，延长电池寿命或降低系统能耗 2。
    
2. 电磁干扰（EMI）管理： 开关操作会产生高频噪声，必须通过合理的布局和滤波来抑制EMI，以满足电磁兼容性（EMC）标准，并避免干扰系统内其他敏感电路 9。
    
3. 热管理（Thermal Management）： 功率损耗会产生热量，必须通过PCB设计（如铜皮、散热过孔）有效散热，保证器件工作在安全温度范围内，提高可靠性 7。
    
4. 稳定性（Stability）： 控制环路必须稳定，以确保在各种输入电压和负载条件下都能提供稳定的输出电压，并具有良好的瞬态响应 1。
    
5. 解决方案尺寸（Size）： 在满足性能要求的前提下，尽可能缩小PCB占用面积和整体体积，以适应小型化、高密度化的产品需求 9。
    

PCB布局本身就是设计性能的关键组成部分，而非仅仅是原理图的物理实现 11。不良的布局会导致寄生参数（电感、电容、电阻）显著增大，从而恶化效率、EMI、热性能和稳定性。本报告旨在深入探讨DC-DC转换器板级设计的关键考量和最佳实践，涵盖从基本原理到测试验证的全过程。

## II. 常见DC-DC拓扑基础

在进行板级设计之前，理解常见DC-DC拓扑的工作原理至关重要。本节将重点介绍Buck、Boost和Buck-Boost这三种非隔离拓扑。

### 工作原理

- Buck（降压）转换器:  
    Buck转换器用于将较高的输入电压 (VIN​) 降低为较低的输出电压 (VOUT​) 1。其核心元件包括一个高边开关（通常是MOSFET）、一个低边开关（可以是二极管，即非同步Buck；也可以是另一个MOSFET，即同步Buck）、一个电感 (L) 和一个输出电容 (C) 1。  
    工作过程分为两个阶段：
    

1. 开关导通（ON）阶段： 高边开关闭合，低边开关断开。电流从 VIN​ 流经高边开关和电感，为输出电容充电并供给负载。电感储存能量，其电流线性上升 1。
    
2. 开关关断（OFF）阶段： 高边开关断开，低边开关（二极管或同步MOSFET）导通。电感释放储存的能量，通过低边开关继续向输出电容和负载提供电流，电感电流线性下降 1。 同步Buck转换器使用MOSFET替代二极管作为低边开关，可以显著降低导通压降，从而提高效率，尤其是在低输出电压和高电流应用中 2。
    

- Boost（升压）转换器:  
    Boost转换器用于将输入电压 (VIN​) 升高为较高的输出电压 (VOUT​) 2。其元件与Buck类似，但连接方式不同。  
    工作过程同样分为两个阶段：
    

1. 开关导通（ON）阶段： 开关闭合，二极管（或同步MOSFET）截止。电流从 VIN​ 流经电感和开关闭合的路径到地。电感储存能量，其电流线性上升。此时，输出电容为负载供电 8。
    
2. 开关关断（OFF）阶段： 开关断开。电感中储存的能量无法通过开关，其感应电压极性反转并与 VIN​ 叠加。这个叠加后的电压高于 VOUT​，使得二极管（或同步MOSFET）导通，电流流向输出电容和负载。电感释放能量 8。 Boost转换器常用于电池供电系统，将较低的电池电压提升至电路所需的工作电压 5。
    

- Buck-Boost（升降压）转换器:  
    Buck-Boost转换器具有高度的灵活性，其输出电压 (VOUT​) 可以高于、低于或等于输入电压 (VIN​) 2。存在多种拓扑结构：
    

- 反相拓扑（Inverting Topology）： 这是最基本的Buck-Boost结构，其输出电压相对于输入地为负极性 5。其工作原理也分为两个阶段：
    

1. 开关导通（ON）阶段： 开关闭合，二极管截止。VIN​ 直接施加在电感两端，电感储存能量，电流上升。输出电容为负载供电 39。
    
2. 开关关断（OFF）阶段： 开关断开。电感释放能量，电流通过二极管流向输出电容和负载。由于二极管的连接方向，输出电压为负 39。
    

- 非反相拓扑（Non-Inverting Topologies）： 为了获得与输入同极性的输出电压，发展出了多种非反相Buck-Boost拓扑，如四开关（Four-Switch）Buck-Boost、SEPIC（Single-Ended Primary-Inductor Converter）、Ćuk等 39。四开关拓扑通过组合Buck和Boost的开关管实现升降压，控制相对复杂但效率较高 40。SEPIC和Ćuk通常使用单个开关，但需要额外的电感和电容 39。
    
- 工作模式（CCM/DCM）： 与Buck和Boost类似，Buck-Boost转换器也可以工作在连续导通模式（CCM）和非连续导通模式（DCM）。CCM下电感电流永不为零，DCM下电感电流会在每个开关周期内降至零 39。DCM通常发生在轻载条件下，控制相对简单，但输出纹波较大，效率较低 39。
    

- 占空比（Duty Cycle）的角色:  
    在所有这些开关拓扑中，输出电压的调节都是通过控制主开关的占空比 (D) 来实现的 1。占空比定义为开关在一个周期 (T) 内导通时间 (TON​) 的比例 (D=TON​/T)。通过脉宽调制（PWM）技术改变 TON​，即可调整占空比，从而精确控制输出电压。理想状态下（忽略损耗）：
    

- Buck: VOUT​=D×VIN​
    
- Boost: VOUT​=VIN​/(1−D)
    
- Inverting Buck-Boost: VOUT​=−VIN​×D/(1−D) 实际电路中，由于元器件的压降和损耗，实际占空比会略有偏差。控制电路通过反馈环路实时监测输出电压，并动态调整占空比以维持输出稳定 1。
    

### LDO与开关稳压器的板级设计考量比较

线性稳压器（LDO）和开关稳压器（DC-DC）是两种最常见的电压转换方式，它们在板级设计上存在显著差异：

- LDO (Low Dropout Regulator):
    

- 优点: 设计简单，外部元件少（通常只需输入输出电容），噪声低（无开关噪声），电源抑制比（PSRR）在低频较好，成本较低，尺寸可以很小 9。PCB布局相对简单 43。
    
- 缺点: 效率低，尤其是在输入输出压差较大时，功率损耗 Ploss​=(VIN​−VOUT​)×IOUT​，导致发热严重 9。只能实现降压功能 9。
    
- 设计考量: 主要关注输入输出电容的选择（类型、容值、ESR以保证稳定性），以及散热设计（通过PCB铜皮或外部散热器）。布局相对简单，但仍需注意输入输出电容靠近IC引脚。
    

- 开关稳压器 (DC-DC):
    

- 优点: 效率高，尤其是在宽输入输出电压范围下 2。可以实现升压、降压、升降压及反相等多种电压变换 9。
    
- 缺点: 设计复杂，需要外部电感、电容、开关（部分集成），需要仔细进行环路补偿设计以保证稳定性 9。会产生开关噪声（纹波、尖峰、EMI），需要滤波和良好的布局来抑制 9。尺寸通常比LDO大（主要因为电感），成本可能更高 9。
    
- 设计考量: 这是本报告的重点。需要精心选择电感、电容、开关管等元件。PCB布局至关重要，需要最小化功率环路面积，优化接地，隔离敏感信号，并进行热管理 11。环路补偿设计是确保稳定性的关键。需要采取EMI抑制措施。
    

选择依据:

选择LDO还是开关稳压器，主要取决于应用对效率、噪声、成本、尺寸和电压转换类型的要求。当效率是首要考虑因素，或者需要升压/反相时，开关稳压器是必然选择。当噪声敏感性极高，或者输入输出压差很小，或者设计简单性和成本是关键时，LDO可能是更好的选择。有时也会组合使用，例如用开关稳压器进行初步转换以提高效率，然后用LDO进行二次稳压以获得极低的噪声输出 10。现代低噪声开关稳压器技术（如ADI的Silent Switcher 44 或TI的低EMI器件）正在不断发展，使得在某些噪声敏感应用中直接使用开关稳压器成为可能，从而省去LDO，提高整体效率和减小方案尺寸 10。

## III. 原理图设计与元件选择

在确定了拓扑结构后，下一步是进行详细的原理图设计和关键元件的选择。这一阶段的目标是确保电路在满足电气规格的同时，具有高效率、高稳定性和高可靠性。

### 定义电气规格

设计开始前，必须清晰定义各项电气参数，这将直接指导元件选择和电路设计。关键规格包括 2：

- 输入电压范围 (VIN​): 考虑电源的波动范围，特别是电池供电系统，需要覆盖从满电到放电终止电压（EDV）的整个范围 2。
    
- 输出电压 (VOUT​): 目标输出电压值。
    
- 输出电流范围 (IOUT​): 包括最小和最大负载电流。轻载和重载条件下的性能（如效率）可能需要分别优化 2。
    
- 输出电压纹波 (Vripple​): 允许的最大输出电压波动，通常以峰峰值 (mVpp) 或相对于 VOUT​ 的百分比表示。典型要求可能低于 VOUT​ 的1% 2。
    
- 负载瞬态响应: 当负载电流发生快速阶跃变化时，输出电压的最大过冲、下冲幅度和恢复时间。这对于为动态负载（如CPU、FPGA）供电的应用至关重要 2。
    
- 效率要求: 在特定负载点（如满载、典型负载、轻载）的最低效率要求 7。
    
- 工作环境温度: 影响元件选型（额定温度）和散热设计。
    

### 关键元件选择

元件的选择直接影响转换器的性能、成本、尺寸和可靠性。

- 电感 (L):
    

- 功能: 作为主要的储能元件，平滑电流 1。
    
- 电感值 (L) 计算: 基于输入/输出电压、开关频率 (fsw​) 和期望的电感纹波电流 (ΔIL​) 来计算。通常将 ΔIL​ 设定为最大输出电流 (IOUT(max)​) 的20%到40% 2。例如，对于Buck转换器，在CCM模式下，L=fsw​×ΔIL​×VIN(max)​VOUT​×(VIN(max)​−VOUT​)​。电感值的选择是一个权衡：较大的电感值可以减小纹波电流和输出电压纹波，但也可能导致瞬态响应变慢，且电感尺寸和成本通常会增加 48。
    
- 直流电阻 (DCR): 电感绕线的固有电阻，直接影响传导损耗 (PDCR​=IRMS2​×DCR) 和效率 48。DCR越低越好，但这通常意味着需要更粗的线或更大的磁芯，导致成本和尺寸增加 51。DCR随温度升高而增加（铜的温度系数约为+0.4%/°C） 52。
    
- 饱和电流 (Isat​): 指使电感量下降特定百分比（通常为20%或30%）的直流电流值 48。设计时必须确保电感的峰值电流 (Ipeak​=IOUT(max)​+ΔIL​/2) 远低于 Isat​ 48。电感饱和会导致电感量急剧下降，纹波电流增大，可能损坏开关管或其他元件，并降低效率 50。Isat​ 会随温度升高而降低，需要考虑最坏情况 50。
    
- 硬饱和 vs. 软饱和: 磁芯材料决定了饱和特性。铁氧体磁芯（Ferrite Core）通常表现为硬饱和，即一旦超过 Isat​，电感量会迅速崩溃 51。粉末磁芯（Powdered Core）或模压电感（Molded Inductor）通常表现为软饱和，电感量随电流增加而逐渐下降 51。软饱和特性在瞬态过流时提供了一定的裕量，但需要更仔细地评估实际工作电感量 53。模压电感通常具有软饱和特性和良好的温度稳定性 51。
    
- 额定电流 (Irms​ 或 Heating Current): 基于电感自身温升（通常为40°C）定义的连续工作电流能力 48。温升由DCR损耗和磁芯损耗（交流损耗）引起。需要确保电路中的最大RMS电流（包括直流和交流分量）低于 Irms​ 额定值，以避免过热 51。交流损耗随频率增加而增加 52。
    
- 自谐振频率 (SRF): 电感的寄生电容与其自身电感发生谐振的频率 52。开关频率必须远低于SRF（经验法则是 fsw​<SRF/10），否则电感将失去其电感特性 52。
    
- 屏蔽与非屏蔽: 屏蔽电感（Shielded）通过磁屏蔽结构减少磁场泄漏，从而降低对周围电路的EMI干扰，但通常 Isat​ 较低且成本较高 52。非屏蔽电感（Unshielded）成本较低，Isat​ 可能更高，但磁场泄漏较大，EMI风险更高 52。在高开关频率或EMI敏感应用中，强烈推荐使用屏蔽电感 52。
    
- 选择要点总结: 电感选择需综合考虑电感值、DCR（效率）、Isat​（峰值电流能力）、Irms​（温升）、SRF（频率限制）、尺寸、成本和屏蔽类型（EMI）。不能只看标称电感值，必须仔细核对电流额定值（Isat​ 和 Irms​）并留有足够裕量。
    

- 电容 (输入/输出):
    

- 功能: 输入电容 (CIN​) 提供开关所需的高频脉冲电流，稳定输入电压，吸收输入纹波电流 36。输出电容 (COUT​) 滤除电感纹波电流产生的输出电压纹波，储存能量以应对负载瞬变，并影响环路稳定性 1。
    
- 容值 (C) 计算: 输入电容通常根据允许的输入电压纹波或所需RMS纹波电流处理能力来选择 57。输出电容的选择更为复杂，需要同时满足输出电压纹波要求 (ΔVOUT​≈ΔIL​×ESR+8×fsw​×CΔIL​​+ΔIL​×dtESL​) 和负载瞬态响应要求（在环路响应之前，电容需要提供或吸收负载阶跃电流以限制电压过冲/下冲） 2。瞬态响应通常需要较大的容值来储能 49。
    
- 等效串联电阻 (ESR): 对输出电压纹波有直接影响 (ΔVOUT(ESR)​≈ΔIL​×ESR) 1。低ESR有利于减小纹波和改善瞬态响应 49。ESR还会影响控制环路的稳定性，因为它会在传递函数中引入一个零点 (fz(ESR)​=1/(2π×ESR×COUT​)) 26。
    
- 等效串联电感 (ESL): 影响电容在高频下的阻抗特性。低ESL对于需要快速响应瞬变和滤除高频噪声的输入/输出旁路电容尤为重要 1。陶瓷电容通常具有最低的ESL 49。
    
- 纹波电流额定值: 输入电容需要承受较大的RMS纹波电流，尤其是在Buck转换器中。输出电容需要承受电感纹波电流。超过额定纹波电流会导致电容过热，寿命缩短，特别是对于电解电容 7。陶瓷电容由于ESR极低，通常没有明确的纹波电流限制，但仍需考虑其发热 63。
    
- 电容类型比较:
    

- 陶瓷电容 (MLCC): 低ESR/ESL，高频性能好，尺寸小（中低容值），无极性，寿命长 49。是输入旁路和高频输出滤波的首选。缺点是DC偏压效应显著（施加直流电压时，实际电容量会大幅下降，特别是II类陶瓷如X7R, X5R） 49，高容值成本高 62，可能存在压电效应导致啸叫 33。I类陶瓷（C0G/NP0）性能稳定但容值低 61。
    
- 电解电容 (铝): 容值大，成本低廉，适合大容量储能 49。缺点是ESR/ESL较高，高频特性差，寿命有限（受温度影响大），有极性 49。通常用于输入/输出大容量滤波。
    
- 钽电容/聚合物电容: ESR/ESL介于陶瓷和电解之间，容值较陶瓷高 49。性能均衡，但成本较高 49。钽电容存在失效风险 61。
    

- 并联使用: 实际设计中常将不同类型的电容并联使用，以发挥各自优势。例如，使用大容量电解电容或聚合物电容提供主要的能量存储（应对低频和负载瞬变），同时并联小容量、低ESR/ESL的陶瓷电容来滤除高频噪声并降低整体阻抗 7。高频纹波电流主要流过阻抗更低的陶瓷电容 63。
    
- 电压裕量: 选择电容时，其额定电压应显著高于实际工作电压。对于陶瓷电容，需要考虑DC偏压导致的容值下降 49。对于电解电容，留有电压裕量有助于提高可靠性和寿命。
    
- 选择要点总结: 电容选择需平衡容值、ESR、ESL、纹波电流能力、DC偏压特性、尺寸、成本和寿命。输入电容侧重纹波电流处理和高频旁路。输出电容侧重纹波电压抑制和瞬态响应。通常采用混合电容方案（陶瓷+电解/聚合物）以优化性能和成本。必须考虑陶瓷电容的DC偏压效应。
    

- 反馈电阻 (RFB1​,RFB2​):
    

- 功能: 组成电阻分压器，将输出电压 VOUT​ 按比例缩小，反馈至控制器的反馈（FB）或电压感测（VSENSE）引脚，与内部基准电压 (VREF​) 比较，以实现闭环调节 1。关系式通常为 VOUT​=VREF​×(1+RFB1​/RFB2​)，其中 RFB1​ 是上拉电阻（接 VOUT​），RFB2​ 是下拉电阻（接GND）。
    
- 精度和容差: 反馈电阻的精度直接影响输出电压的设定精度 26。如果需要高精度的输出电压，必须选用高精度电阻（如1%或更低的0.1%） 68。总的输出电压误差不仅取决于电阻容差，还包括 VREF​ 的误差和FB引脚偏置电流的影响 68。可以使用最坏情况分析或统计方法（如蒙特卡洛模拟 26）来评估总误差。
    
- 阻值大小: 反馈网络的总电阻 (RFB1​+RFB2​) 会影响转换器的静态功耗和噪声敏感性 69。
    

- 高阻值: 优点是流过分压器的电流小，有助于提高轻载效率，这对于电池供电应用尤其重要 69。缺点是反馈节点（FB引脚）阻抗高，更容易受到外部噪声的干扰，并且控制器FB引脚的偏置电流 (IFB​) 会引入更大的误差 (Verror​≈IFB​×RFB1​) 69。经验法则是确保流过 RFB2​ 的电流远大于 IFB​（例如，大于50倍 IFB​ 69）。
    
- 低阻值: 优点是反馈节点阻抗低，抗噪声能力强，受 IFB​ 影响小 72。缺点是流过分压器的静态电流增大，降低了轻载效率 72。
    

- 对环路补偿的影响: 反馈电阻的阻值（特别是 RFB1​）以及可能跨接在 RFB1​ 上的前馈电容 (CFF​) 会影响补偿网络的零极点位置，进而影响环路带宽和相位裕度 65。因此，为了优化效率或噪声而改变反馈电阻阻值时，可能需要重新评估和调整补偿网络 72。
    
- 选择要点总结: 反馈电阻的选择需要在输出电压精度（容差）、轻载效率（阻值大小）、噪声抗扰度（阻值大小）和环路动态特性（补偿）之间进行权衡。通常建议遵循器件数据手册的推荐阻值范围。
    

### 环路稳定性和补偿简介

- 闭环控制: DC-DC转换器是一个典型的闭环负反馈系统 1。控制器根据反馈电压与基准电压的差值，调整开关占空比，以维持输出电压稳定。
    
- 稳定性要素: 环路增益（Loop Gain）、相位裕度（Phase Margin, PM）、增益裕度（Gain Margin, GM）和交越频率（Crossover Frequency, fc​）是衡量环路稳定性的关键指标 27。
    
- 补偿的必要性: 功率级（调制器 + LC滤波器）本身会引入显著的相位滞后（例如，LC滤波器引入180°相位滞后） 31。如果环路增益大于1（0 dB）时，总的相位滞后达到360°（即相对于负反馈引入的180°又额外滞后了180°），系统就会发生振荡 27。补偿网络的作用就是调整环路增益和相位的频率特性，确保在增益降至0 dB（即交越频率 fc​）时，相位滞后远小于360°（通常要求相位裕度 PM > 45°），并且在相位达到360°时，增益远小于0 dB（保证增益裕度 GM） 27。
    
- 补偿类型 (Type I, II, III): 补偿网络通常由电阻和电容组成，连接在误差放大器的反馈路径上。不同类型的补偿网络（Type I, II, III）提供不同的零极点组合，以适应不同的功率级特性 31。
    

- Type I: 只有一个积分器（极点在原点），相位滞后90°，适用于相位裕度非常充足的系统。
    
- Type II: 在Type I基础上增加一个零点和一个极点，最多可提供90°的相位提升。常用于补偿电流模式控制的转换器，或工作在DCM模式的电压模式转换器 65。
    
- Type III: 包含两个零点和两个极点（外加原点极点），最多可提供180°的相位提升。常用于补偿电压模式控制且工作在CCM模式的转换器，因为其LC滤波器会引入180°的相位滞后 65。 电流模式控制通常简化了补偿设计，因为其功率级传递函数近似为单极点响应，通常使用Type II补偿即可获得稳定且带宽良好的系统 76。电压模式控制则通常需要Type III补偿来抵消LC谐振带来的180°相位滞后 65。
    

- 分析工具: 波特图（Bode Plot）是分析环路稳定性的标准工具，通过绘制环路增益和相位随频率变化的曲线来确定交越频率、相位裕度和增益裕度 28。
    

## IV. PCB布局：性能与可靠性的最佳实践

PCB布局对于DC-DC转换器的性能至关重要，错误的布局可能导致输出电压调节不良、开关抖动、EMI超标甚至器件失效 36。优化布局的目标是最小化寄生参数，控制高频电流路径，并确保良好的热性能和信号完整性。

### 战略性元件布局

元件的放置位置是优化布局的第一步，直接影响后续布线的难易程度和最终性能。

- 总体原则: 优先放置功率级元件（IC、开关管、二极管、电感、输入/输出电容），然后放置控制电路元件（反馈网络、补偿元件等） 11。元件应尽可能靠近放置，以缩短走线长度 16。电源模块应靠近负载放置，以减少连接阻抗和压降 11。
    
- 输入电容 (CIN​): 必须紧靠DC-DC控制器的VIN和PGND引脚放置 11。这是布局中最关键的一步，直接影响输入电压尖峰和EMI 36。应选用低ESL/ESR的陶瓷电容 11。
    
- IC/控制器: 通常放置在功率元件的中心位置，便于连接。注意其散热焊盘（Exposed Pad）需要良好接地和散热通路 11。
    
- 开关元件 (FETs/二极管): 靠近IC（如果是外部开关）和输入电容放置，以最小化高频开关环路（"热环路"）的面积 11。
    
- 电感 (L): 靠近开关节点（PH或SW）引脚放置 18。电感的方向可能影响开关节点铜皮面积的大小，应适当调整以最小化该区域 36。
    
- 输出电容 (COUT​): 靠近电感和负载连接点放置 11。其接地端的位置对于减小输出环路阻抗和保证反馈准确性非常重要 36。
    
- 反馈网络 (RFB1​,RFB2​,CFF​): 靠近IC的FB引脚放置 11。反馈电压的采样点应精确地取自输出电容两端 11。
    
- 小信号元件 (补偿, 软启动, 使能): 靠近IC对应引脚放置 36。远离功率级噪声源（开关节点、电感） 11。
    
- 双面布局: 可以有效节省PCB面积，但设计时需格外小心 20。一种常见的做法是将IC和输入电容放在底层，电感和输出电容放在顶层 20。需要仔细评估过孔在功率回路中的位置及其对性能（纹波、过冲、热）的影响 20。热性能也可能因元件堆叠而受到影响 20。
    

### 最小化高频环路中的寄生电感

高频开关电流流过的路径会形成环路，这些环路中的寄生电感是产生电压尖峰、振荡和EMI的主要原因。

- 识别关键环路 ("热环路"):
    

- 输入环路 (Buck/Buck-Boost): 对于Buck和反相Buck-Boost，最关键的环路是由输入电容、高边开关（或主开关）、低边开关（或二极管）以及它们之间的连接构成的 11。这个环路承载着高 di/dt 的脉冲电流。最小化这个环路的面积是PCB布局的首要任务，可以显著降低电压尖峰 (V=Lparasitic​×di/dt) 和辐射EMI 11。
    
- 输出环路 (Boost/Buck-Boost): 对于Boost和反相Buck-Boost，输出侧的环路（包括开关、二极管、输出电容）也承载着高 di/dt 电流，同样需要最小化 11。
    
- 输出滤波环路 (Buck): Buck转换器的输出环路（电感、输出电容、负载）电流相对连续，其 di/dt 较小，不如输入环路关键，但仍需合理布局。
    

- 布局技术:
    

- 短而宽的走线: 使用短而宽的铜皮走线来连接功率元件，以降低电阻和电感 11。
    
- 紧凑放置: 将构成高频环路的元件（如输入电容、开关管、二极管）尽可能地靠近放置 11。
    
- 单层布线: 尽可能将高频电流路径布在同一层，避免使用过孔 11。
    
- 利用邻近接地层: 在多层板中，将接地层放置在功率层旁边。高频电流的回流路径会自然地选择阻抗最低的路径，即紧邻信号路径下方的接地层。这利用了磁场抵消效应，有效减小了环路电感和面积 11。层间介质厚度越薄，耦合越紧密，效果越好 60。
    
- 避免或优化过孔: 过孔会增加路径电感和电阻。应尽量避免在关键的高频电流环路中使用过孔 15。如果必须使用，应使用多个过孔并联以降低阻抗 11。
    

### 管理开关节点 (PH/SW - 高 dv/dt)

开关节点是另一个主要的噪声源，其电压在 VIN​ 和地（或 VOUT​）之间快速切换，产生高 dv/dt。

- 问题: 开关节点的铜皮面积相当于一个电容极板，会通过寄生电容将噪声耦合到邻近的走线或平面，干扰敏感信号 11。
    
- 解决方案:
    

- 最小化面积: 尽可能减小开关节点的铜皮面积，仅保留满足载流能力和散热需求的最小尺寸 11。
    
- 底层屏蔽: 在开关节点下方放置接地层可以提供屏蔽作用 11。
    
- 远离敏感信号: 确保反馈线、模拟信号线等敏感走线远离开关节点区域 11。
    

### 接地技术

正确的接地是DC-DC转换器PCB设计的基石，对于降低噪声、保证稳定性和提高性能至关重要。

- 功率地 (PGND) vs. 信号/模拟地 (SGND/AGND):
    

- 定义: PGND是承载大电流、高噪声开关回路的地回路。AGND/SGND是控制器内部模拟电路、反馈网络、补偿网络等小信号电路的参考地 11。许多IC会提供独立的PGND和AGND/SGND引脚 11。
    
- 目的: 将噪声大的功率地电流与敏感的信号地回路分开，防止噪声耦合 17。
    

- 接地策略比较: 设计中如何连接PGND和AGND/SGND是一个常见问题，主要有以下策略：
    

  

|   |   |   |   |   |
|---|---|---|---|---|
|策略|优点|缺点|适用场景|关键实现细节|
|统一接地平面|实现简单，提供全局低阻抗回流路径，不易出错 88|需要通过元件布局和布线严格控制噪声电流路径，避免污染敏感区域 91|多层板设计，有足够空间进行良好布局和布线控制|将所有地连接到同一个完整的地平面层。靠近噪声源（如输入电容地）的区域承载功率电流，敏感元件（如反馈电阻地）连接到远离噪声源的"安静"地平面区域 36。|
|分割地平面+单点连接|物理隔离噪声大的功率地和敏感的信号地 17|可能增加回流路径阻抗，若信号跨越分割处可能产生严重EMI 17|模拟/数字混合信号系统，对噪声隔离要求极高，或在双层板上实现|将PGND和AGND/SGND区域物理分开，仅通过一个点（通常在IC下方或电源输入处）连接起来（星型接地）11。避免信号线跨越地平面分割缝隙。|

  
  
  

**决策考量:** 统一接地平面在多层板中通常是更优且更简单的选择，前提是能够通过良好的布局来管理电流路径。分割地和单点接地需要更仔细的设计以避免潜在问题，但在特定情况下（如强制隔离或简单板）可能是必要的。核心目标是为所有电流提供低阻抗的回流路径，并将噪声电流与敏感信号隔离。  
  

- 接地层 (Ground Planes): 在多层板中强烈推荐使用完整的接地层 11。接地层提供最低阻抗的回流路径，减少环路面积，提供屏蔽，并有助于散热 11。应将接地层放置在靠近功率元件层的位置 11。尽量保持接地层的完整性，避免分割 11。
    
- 过孔 (Vias) 使用:
    

- 功率地连接: 对于大电流的接地连接（如输入/输出电容地、IC散热焊盘），应使用多个过孔以降低电阻和电感 11。
    
- 信号地连接: 敏感信号的接地应直接连接到其参考地平面，避免通过噪声大的功率地路径。
    
- 接地缝合过孔 (Stitching Vias): 在地平面边缘或不同地平面区域之间放置一排过孔，可以连接不同层上的地平面，确保低阻抗连续性，并有助于屏蔽 83。
    
- 层切换: 当信号线需要切换层面时，应在其旁边放置一个接地过孔，以提供连续的回流路径，避免中断 83。
    

- 避免接地环路: 确保电流只有一个明确的低阻抗回流路径。避免在接地铺铜或平面上形成大的导电环路，因为变化的磁场可以在环路中感应出噪声电流 17。星型接地是避免环路的一种方法 17。
    

### 敏感信号布线

模拟反馈信号、电流检测信号等对噪声非常敏感，其布线需要特别注意。

- 反馈走线 (FB/VOS/VSENSE):
    

- 采样点: 必须精确地连接到输出电容的正负两端（如果可能，使用开尔文连接），以准确反映输出电压 11。
    
- 放置: 反馈电阻靠近IC的FB引脚放置 11。
    
- 布线: 走线应尽可能短而直接 11。必须远离噪声源，特别是开关节点 (PH/SW) 和电感 11。如果空间有限，可以考虑在内层布线，并利用上下接地层进行屏蔽 11。反馈节点的铜皮面积应最小化，以减少噪声拾取 36。
    

- 电流检测走线 (SENSE+/SENSE-): 同样对噪声高度敏感 11。
    

- 布线: 应作为差分对进行布线（开尔文连接），两条线紧密平行走线，以最大限度地减少共模噪声拾取和 di/dt 引起的误差 11。远离开关节点等噪声源。滤波元件（如果使用）应靠近IC引脚放置 11。如果可能，在电流检测走线和功率层之间放置接地层进行屏蔽 11。
    

- 通用敏感信号布线规则:
    

- 避免直角转弯，使用45°或圆弧过渡 92。
    
- 与其他走线（特别是噪声走线）保持足够的间距 92。
    
- 利用接地层进行屏蔽 92。
    
- 走线尽可能短 17。
    

### 借鉴制造商布局示例

- 数据手册 (Datasheet): 通常包含推荐的PCB布局图 80。这些示例是根据IC的引脚排列和内部结构优化的，是重要的参考 36。
    
- 评估模块 (EVM) 用户指南: 提供经过验证的完整设计文件，包括原理图、BOM和布局文件 77。研究EVM布局通常比仅看数据手册示例更有价值，因为EVM是实际测试过的硬件 91。
    
- 理解而非盲从: 重要的是理解布局示例背后的设计原理（例如，为什么输入电容这样放置，为什么反馈线这样走），而不仅仅是复制粘贴 45。每个实际应用都有其特定约束，可能需要对参考布局进行调整。
    

## V. 噪声 (EMI) 抑制与滤波

开关模式电源的固有工作方式会产生电磁干扰 (EMI)，包括传导噪声和辐射噪声。有效的EMI抑制对于满足法规要求（如CISPR、FCC）和确保系统可靠性至关重要。

### 噪声来源

- 开关瞬变: MOSFET或二极管的快速开关动作产生高频谐波电流和电压 (di/dt,dv/dt)，是主要的噪声源 14。
    
- 寄生振荡: PCB走线和元件的寄生电感 (Lp​) 与开关器件的寄生电容 (Cp​)（如MOSFET的 Coss​）形成谐振回路，在开关瞬变时产生高频振荡（ringing）和电压过冲（overshoot） 11。变压器的漏感也是一个重要的寄生电感来源 96。
    
- 环路辐射: 高 di/dt 电流流过具有一定面积的环路（如输入热环路）会产生时变磁场，形成环路天线辐射噪声 11。
    
- 共模噪声: 由于不对称的电流路径、接地阻抗或高 dv/dt 节点对地的容性耦合，可能产生共模噪声 17。
    
- 器件噪声: 控制器IC本身也会产生一定的宽带噪声 10。
    

### 通过布局降低EMI

许多EMI问题可以通过优化PCB布局来解决或缓解，这是成本最低且最有效的EMI控制手段。

- 最小化关键环路面积: 如前所述，严格控制高 di/dt 环路（特别是输入环路）的面积是降低差模辐射和寄生电感引起电压尖峰的关键 11。
    
- 最小化开关节点面积: 减小高 dv/dt 开关节点的铜皮面积，以降低容性耦合噪声 11。
    
- 使用接地层: 连续的接地层提供低阻抗回流路径，并对高频噪声提供屏蔽 11。
    
- 元件放置: 将高频元件（IC、开关、二极管、输入电容）紧密放置 11。
    
- 隔离噪声源: 将噪声大的功率部分与敏感的模拟/控制部分在物理上分开，并注意布线路径，避免交叉耦合 11。
    

### 输入滤波器设计

输入滤波器用于衰减从DC-DC转换器传导回输入电源线的噪声，通常是满足EMC标准的必要部分 18。

- 目的: 抑制差模和共模噪声，满足传导EMI规范（如CISPR 25 18）。
    
- 拓扑: 常用LC或π型滤波器（C-L-C 或 C-Ferrite-C）。π型滤波器通常能提供更好的高频衰减性能 98。可能还需要共模扼流圈来抑制共模噪声 98。
    
- 磁珠 (Ferrite Bead) 选择: 磁珠在高频下呈现阻性，用于吸收高频噪声 19。
    

- 关键特性: 阻抗-频率曲线是选择依据 19。需要选择在目标噪声频率范围内具有高阻抗的磁珠。
    
- DC偏置电流影响: 这是选择磁珠时最容易被忽视的关键点。磁珠的阻抗会随着通过的直流偏置电流的增加而显著降低，甚至饱和 19。因此，不能仅根据零偏置电流下的阻抗曲线来选择磁珠。必须评估磁珠在实际工作电流下的阻抗是否足够。通常建议选用额定电流远大于实际工作电流的磁珠（例如，仅使用其额定电流的20% 19），或者查阅制造商提供的考虑DC偏置影响的数据。
    
- 工具与验证: 可以使用仿真工具（如LTSpice 102 或制造商提供的工具 19）辅助选择，但最终效果需要通过实际测量来验证 102。
    

- 电感 (Inductor) 选择: 如果使用电感而不是磁珠，需要考虑其DCR、饱和电流和SRF。
    
- 电容选择: 通常选用低ESR/ESL的陶瓷电容 98。注意电容自身的自谐振频率 (SRF)，在高频滤波时，电容可能表现为感性 98。
    
- 滤波器阻尼: 输入滤波器（尤其是LC滤波器）的输出阻抗可能与DC-DC转换器的负输入阻抗发生相互作用，在某个频率上形成欠阻尼谐振，导致系统不稳定或振荡 98。因此，通常需要在滤波器中加入阻尼元件（例如，在输入电容上并联一个电阻和电容串联的阻尼网络，或在电感上并联电阻）来抑制这种谐振 98。
    
- 布局: 输入滤波器的元件也应紧凑布局，并注意接地。
    

### 输出滤波

输出滤波主要由输出电容完成，其选择决定了输出电压纹波的大小 1。对于对噪声特别敏感的负载，可以在主输出电容之后增加第二级滤波，例如：

- LC滤波器: 提供额外的纹波衰减。
    
- 磁珠滤波: 在输出端串联一个磁珠，并配合额外的输出电容，可以滤除更高频率的噪声 99。
    
- LDO后置稳压: 如前所述，在开关稳压器后使用LDO可以获得非常干净的输出电压 10。
    

### 缓冲电路 (Snubber Circuits)

缓冲电路用于抑制开关节点上的电压过冲和振荡。

- 目的: 吸收开关瞬变期间由寄生电感（如布局电感 95 或变压器漏感 96）和寄生电容（如MOSFET Coss​ 95）谐振产生的能量，从而减小电压尖峰和振荡幅度，保护开关器件免受过压冲击，并降低EMI辐射 95。
    
- 类型:
    

- RC Snubber: 由电阻 (R) 和电容 (C) 串联组成，跨接在开关器件（如MOSFET的漏源极）或二极管两端 95。RC Snubber主要用于阻尼振荡 96。
    
- RCD Snubber: 由电阻 (R)、电容 (C) 和二极管 (D) 组成 95。常用于反激式转换器中，跨接在初级开关管两端，用于钳位漏感引起的电压尖峰，并将漏感能量耗散在电阻上 96。
    
- C Snubber: 仅使用电容，可以减缓 dv/dt，但可能引起更大的振荡或损耗 95。
    

- 设计考量:
    

- RC Snubber: C的值通常选择为开关器件输出电容的几倍（例如2倍 103）。R的值通常选择接近或等于寄生谐振回路的特征阻抗 (Lp​/Cp​​)，或者根据测量的振荡周期来计算 96。RC值的选择需要在阻尼效果和功耗之间进行权衡。
    
- RCD Snubber: 设计目标是设定一个钳位电压 (Vclamp​)。电容C需要足够大以在开关周期内保持电压基本恒定（纹波通常控制在5-10% 97）。电阻R的值决定了能量耗散的速率，其功率额定值需要根据漏感能量 (0.5×Llk​×Ipeak2​) 和开关频率计算得出 96。二极管需要选用快速恢复二极管 97。
    
- 元件放置: Snubber元件必须尽可能靠近被保护的器件（如MOSFET的引脚）放置，以最小化Snubber回路自身的寄生电感 104。
    

- 影响: Snubber电路通过在电阻上耗散能量来抑制振荡和过冲，这会降低转换器的整体效率。因此，设计目标是在满足保护和EMI要求的前提下，最小化Snubber损耗。优化布局以减小寄生电感是减少对Snubber需求的最根本方法。
    

## VI. 通过PCB设计进行热管理

功率转换过程中的损耗会以热量的形式散发出来，主要集中在开关元件（MOSFET、二极管）、电感和控制器IC上 24。有效的热管理对于确保器件在额定温度范围内工作、提高系统可靠性和延长寿命至关重要。PCB本身可以作为重要的散热途径。

### 识别热源

首先需要识别电路中的主要发热元件，通常是：

- 开关元件 (MOSFETs, Diodes): 导通损耗 (I2×RDS(on)​ 或 VF​×I) 和开关损耗是主要热源。
    
- 电感: 铜损 (IRMS2​×DCR) 和磁芯损耗（与频率、磁通摆幅有关）产生热量。
    
- 控制器IC: 自身工作也会消耗功率并发热。
    

### 利用PCB铜皮散热

PCB上的铜层是主要的导热路径。

- 铜皮铺设 (Copper Pours/Planes): 在PCB表层和内层使用大面积的铜皮（通常连接到地或电源轨）可以有效地将热量从发热元件传导开，并增大散热面积，促进向环境的对流和辐射散热 21。应将元件的散热焊盘或引脚直接连接到这些大面积铜皮上 22。
    
- 顶层铜皮: 与器件直接接触的顶层铜皮散热效果最直接 24。应尽可能增大与发热元件（特别是其散热焊盘）相连的顶层铜皮面积。
    
- 内层和底层铜皮: 通过散热过孔将热量传导至内层或底层的大面积铜平面，可以进一步扩大散热面积，显著提高散热效率 24。
    
- 走线宽度: 承载大电流的走线应足够宽，不仅是为了降低电阻损耗，也是为了提供更好的导热路径 24。
    
- 铜厚 (Copper Weight): 增加铜的厚度（如从1 oz增加到2 oz）可以降低热阻，改善散热 24。但效果会随着厚度的增加而递减，且更厚的铜会增加PCB加工难度和成本 24。
    

### 散热过孔 (Thermal Vias)

散热过孔是在PCB上专门用于传导热量而非电信号的过孔。

- 功能: 在发热元件下方或附近创建低热阻的垂直导热通道，将热量从元件所在的层面快速传递到PCB的其他层面（通常是内层或底层的接地/电源平面）或直接传递到另一侧的散热器 21。
    
- 放置:
    

- 位置: 理想情况下，直接放置在表面贴装元件的散热焊盘（Thermal Pad）下方（称为"焊盘内过孔"，Via-in-Pad） 21。如果无法实现焊盘内过孔（例如成本或工艺限制），则应尽可能靠近发热元件放置 21。
    
- 阵列: 通常使用多个散热过孔组成阵列（Thermal Via Array），以增大总的导热截面积 21。
    

- 设计指南:
    

- 直径: 通常在0.2 mm到0.5 mm之间 (8-16 mil) 21。
    
- 间距 (Pitch): 相邻过孔的中心距通常在1.0 mm到1.2 mm左右 (40-48 mil) 21。间距过小可能影响PCB的机械强度。
    
- 连接: 必须确保散热过孔可靠地连接到目标散热铜层（如内层地平面），连接处应为实心连接（Solid Connection），不应使用热焊盘（Thermal Relief Pad），因为热焊盘会阻碍热量传导 21。
    
- 填充: 过孔可以用导热环氧树脂或铜膏填充，以进一步提高导热性能并防止焊膏在回流焊过程中流入过孔（对于焊盘内过孔尤其重要） 21。填充会增加成本。
    
- 开窗: 散热过孔通常不覆盖阻焊层（Solder Mask），以允许热量直接散发或与散热器接触 21。如果需要在过孔上方焊接，则必须进行填充和封盖（Capped）。
    
- 高宽比: 建议过孔的高宽比（板厚/孔径）小于10:1，以保证良好的电镀均匀性和可靠性 21。
    

- 散热焊盘 vs. 热焊盘: 需要区分用于散热的"散热焊盘"（Thermal Pad，元件底部的金属区域）和用于改善焊接性的"热焊盘"（Thermal Relief Pad，连接引脚与大铜皮之间的辐条状结构）。散热过孔应直接连接散热焊盘和大铜皮，而热焊盘用于连接普通引脚与大铜皮 22。
    

热管理设计流程:

有效的热管理始于设计早期。通过热仿真工具可以预测热点位置和温度分布，指导元件布局、铜皮设计和散热过孔的优化 23。结合大面积铜皮和合理设计的散热过孔阵列，可以在PCB层面显著改善散热性能。

### 元件布局与气流

- 避免遮挡: 放置较高的元件（如电解电容、大电感）时，应避免它们阻挡气流流向较矮的发热元件（如MOSFET、IC） 11。
    
- 中心放置: 尽可能将主要发热元件放置在PCB中心区域，利用整个板面积进行散热，避免热量在边缘积聚 25。
    

## VII. 利用数据手册和参考设计

器件制造商提供的数据手册（Datasheet）、应用笔记（Application Notes）、评估模块（EVM）和参考设计（Reference Designs）是进行DC-DC转换器设计时不可或缺的宝贵资源。

### 提取关键信息

- 数据手册 (Datasheet): 是了解特定IC最权威的文档。它包含了：
    

- 关键规格: 输入/输出电压范围、最大电流、开关频率、效率曲线、静态电流、热阻等 77。
    
- 引脚描述和功能框图: 帮助理解IC的工作方式和外部连接。
    
- 绝对最大额定值和推荐工作条件: 定义了器件的安全工作范围。
    
- 电气特性表: 详细列出了各种参数的典型值、最小值和最大值及其测试条件。
    
- 典型应用电路: 提供了基本的原理图示例 77。
    
- 详细设计步骤: 许多数据手册包含详细的设计指南，包括元件选择计算方法（如电感、电容、反馈电阻、补偿网络） 77。
    
- PCB布局指南: 提供针对该IC的推荐布局，强调关键元件的放置和布线 16。
    

- 应用笔记: 针对特定主题（如环路补偿、EMI滤波、热设计、特定应用）提供更深入的分析和设计指导 33。
    

### 理解规格表和图表

- 参数曲线: 仔细研究数据手册中的各种性能曲线，例如：
    

- 电感饱和曲线 (L vs. IDC​) 和温升曲线 (ΔT vs. IDC​) 50。
    
- 电容的容值/ESR/阻抗随频率、温度、DC偏压的变化曲线 49。
    
- 转换器效率随负载电流和输入电压变化的曲线 77。
    
- 电源抑制比 (PSRR) 随频率变化的曲线 10。
    
- 瞬态响应波形示例 34。
    
- 环路增益和相位波特图 28。
    

- 理解测试条件: 注意图表和规格是在什么条件下测得的，这对于准确比较和应用至关重要。
    

### PCB布局指南部分

- 重要性: 数据手册中的布局指南是根据IC的内部结构和引脚功能特别制定的，应给予高度重视 16。
    
- 关键点: 通常会强调输入电容的放置、功率回路的最小化、开关节点的处理、反馈路径的布线以及接地方法 82。
    

### 参考设计和评估模块 (EVM)

- 价值: 制造商提供的参考设计（如TI的TIDA 109）和EVM 41 是经过实际验证的完整解决方案。它们通常包含原理图、BOM清单、布局文件（Gerber）和测试报告 109。
    
- 学习资源: 研究EVM的布局是学习最佳实践的绝佳途径，因为它们代表了能够正常工作的实际硬件设计 91。EVM布局通常比数据手册中的简化示例更详细、更实用。
    

### 设计工具

- 许多半导体和无源元件制造商提供在线或离线的设计工具，可以帮助进行元件选择、原理图生成、性能仿真（效率、纹波、瞬态、环路稳定性）甚至布局建议。例如TI的WEBENCH® Power Designer 78、Analog Devices (ADI) 的LTpowerCAD 68、MPS的MPSmart 50、Coilcraft的电感分析工具 56 等。这些工具可以大大加快设计进程并减少错误。
    

### 案例研究：TPS54331

以TI的TPS54331为例 77：

- 器件简介: 这是一款28V输入、3A输出的非同步降压转换器，采用电流模式控制，内部集成了高边MOSFET和斜坡补偿，支持Eco-mode™轻载脉冲跳跃模式 77。
    
- 数据手册布局建议 82:
    

- 输入电容: 10µF X7R/X5R陶瓷电容，紧靠VIN引脚和续流二极管阳极放置，以最小化输入环路。
    
- 续流二极管和电感: 靠近PH引脚放置，减小开关节点面积。
    
- 输出电容: 靠近电感放置。
    
- 反馈电阻: 放置在靠近VSENSE引脚处，反馈走线远离PH节点。
    
- 自举电容 (CBOOT​): 放置在BOOT和PH引脚之间，其走线可在其他层，以为顶层地提供更宽路径。
    
- 补偿网络、软启动电容、UVLO电阻: 靠近对应引脚放置。
    
- 接地: GND引脚直接连接PCB地平面。顶层需要一个接地铜皮区域连接输入电容地、输出电容地和续流二极管阳极。使用多个过孔将顶层地连接到内层或底层地平面，以改善散热。
    

- 参考价值: TPS54331数据手册中的布局图（Figure 8-16 82）清晰地展示了这些原则的应用，为用户提供了具体的实现指导。同时，其EVM用户指南（SLVU281 76）则提供了更完整的、经过测试的布局文件。
    

总结: 充分利用制造商提供的文档和工具是高效、可靠地完成DC-DC转换器设计的关键。数据手册提供了基础信息和指导，而参考设计和EVM则展示了经过验证的实践方案。

## VIII. 必要的测试与验证流程

设计完成后，必须通过严格的测试来验证DC-DC转换器的性能是否满足设计规格，并确保其在各种工作条件下的稳定性和可靠性。

### 测试设备概述

进行全面的DC-DC转换器测试通常需要以下设备 3：

- 可编程直流电源: 提供可变的输入电压 (VIN​)。
    
- 电子负载: 模拟不同的输出负载电流 (IOUT​)，并能产生动态负载（瞬态）。
    
- 数字万用表 (DMM): 用于精确测量直流电压和电流。
    
- 示波器: 用于观察和测量交流信号、瞬态响应和噪声。需要足够的带宽和合适的探头（电压探头、电流探头）。
    
- 频率响应分析仪 (FRA) (或带FRA功能的示波器): 用于测量环路稳定性（波特图）。
    
- 热像仪 或 热电偶: 用于测量关键元件的温度。
    
- 环境试验箱 (可选): 用于在不同的环境温度下进行测试。
    

一些集成的测试设备，如直流电源分析仪 110，可以将电源、负载和DMM功能集成在一起，简化测试设置和同步测量。

### 关键性能测试项目

- 静态测试:
    

- 线性调整率 (Line Regulation): 在固定的输出负载下，改变输入电压 VIN​ 从最小值到最大值，测量输出电压 VOUT​ 的变化量。通常表示为 VOUT​ 变化的百分比或绝对电压值 3。
    
- 负载调整率 (Load Regulation): 在固定的输入电压 VIN​ 下，改变输出电流 IOUT​ 从最小值到最大值，测量输出电压 VOUT​ 的变化量。同样表示为百分比或绝对电压值 3。
    
- 效率 (Efficiency): 在不同的输入电压和输出负载组合下，精确测量输入功率 PIN​=VIN​×IIN​ 和输出功率 POUT​=VOUT​×IOUT​。计算效率 η=(POUT​/PIN​)×100% 3。绘制效率随负载电流变化的曲线对于评估转换器在整个工作范围内的性能非常重要。
    

- 动态测试:
    

- 负载瞬态响应 (Load Transient Response): 使用电子负载产生快速的负载电流阶跃变化（例如，从10%负载跳变到90%负载，再跳回），用示波器观察输出电压 VOUT​ 的响应。测量关键指标：电压过冲（Overshoot）、电压下冲（Undershoot）和恢复时间（Settling Time，即电压恢复到稳态容差范围内所需的时间） 3。这个测试直接反映了控制环路的带宽和稳定性 34。
    
- 启动/关断特性: 观察转换器在电源上电或使能信号有效时的输出电压上升过程，测量上升时间、软启动时间（如果设计了软启动），以及是否存在过冲 3。同样观察关断过程。检查软启动功能是否按预期限制了浪涌电流 108。
    
- 保持时间 (Hold-up Time): 对于需要掉电保持的应用，测试输入电压跌落到低于最小工作电压后，输出电压能维持在规定范围内的时间 3。
    

- 输出质量:
    

- 纹波和噪声 (Ripple and Noise / PARD): 使用示波器测量输出电压上的交流成分 3。通常需要使用交流耦合，并设置合适的带宽限制（例如20 MHz）以滤除高频噪声，专注于开关频率相关的纹波。测量峰峰值 (Vpp) 或有效值 (Vrms)。需要区分周期性的开关纹波和随机的高频噪声尖峰 111。正确的测量技术（如使用短接地弹簧的探头）对于获得准确结果至关重要。
    

- 稳定性评估:
    

- 环路响应测量 (Bode Plot): 这是最直接、最可靠的稳定性评估方法。需要使用频率响应分析仪（FRA）向反馈环路中注入一个扫频的小信号（通常通过在反馈电阻分压器中串入一个小注入电阻实现），并测量环路在不同频率下的增益和相位 28。绘制波特图，确定交越频率 (fc​)、相位裕度 (PM) 和增益裕度 (GM) 27。通常要求相位裕度大于45°，增益裕度大于6-10 dB，以保证稳定运行并具有良好的阻尼特性。
    
- 瞬态响应观察: 负载瞬态响应测试也可以间接反映稳定性。如果输出电压在负载跳变后出现持续的振荡或过大的振铃，则表明相位裕度可能不足，系统稳定性差 33。
    

- 热性能测量:
    

- 在最坏的负载条件（通常是最大输出功率）和最高环境温度下运行转换器，使用热像仪或热电偶测量关键元件（IC、MOSFET、二极管、电感）的温度 3。确保所有元件的温度都在其规格书规定的安全工作范围内。验证PCB布局和散热设计的有效性。
    

- 保护功能测试:
    

- 过流保护 (OCP): 逐渐增加负载电流，直到触发OCP。验证保护阈值是否符合设计预期，并观察保护模式（例如，打嗝模式 Hiccup Mode 或折返模式 Foldback Mode）是否正常工作 3。
    
- 过温保护 (OTP): 在高温环境下（或人为加热）运行转换器，验证OTP功能是否在预期温度下触发，并在温度下降后能否自动恢复（如果设计如此） 3。
    
- 过压保护 (OVP): 如果设计了OVP，需要通过模拟或实际引入过压条件来验证其是否能有效保护负载 3。
    

测试的重要性:

测试不仅是为了确认设计是否"通过"，更是设计验证和优化的关键环节。通过测试发现的问题（如稳定性不足、纹波超标、效率低下、过热等）可以反馈到设计中进行迭代改进。瞬态响应和环路稳定性测试对于验证控制环路设计的正确性尤为重要。热测试则验证了PCB布局和散热方案的有效性。只有经过全面测试验证的设计才能确保在实际应用中的可靠性和性能。

## IX. 结论

DC-DC转换器的板级设计是一个涉及多学科知识的综合性工程任务，其成功与否直接关系到最终电子产品的性能、可靠性和成本。本报告系统性地探讨了从基本原理、原理图设计、关键元件选择，到PCB布局、噪声抑制、热管理，再到最终测试验证的整个设计流程中的关键考量因素。

设计的核心在于理解并平衡各种相互关联的性能指标。效率是DC-DC转换器的主要优势，但需要在元件选择（低DCR电感、低 RDS(on)​ MOSFET、低ESR电容）和工作模式（如同步整流、轻载模式）上进行优化。稳定性是可靠运行的基础，依赖于精确的环路补偿设计，需要对功率级特性和补偿网络有深入理解，并通过波特图或瞬态响应进行验证。噪声与EMI是开关电源的固有挑战，需要通过精心设计的PCB布局（最小化高频环路、优化接地、屏蔽敏感信号）和适当的滤波（输入滤波器、输出电容、缓冲电路）来有效抑制。热管理则通过合理的元件布局、利用PCB铜皮和散热过孔等手段，确保器件工作在安全温度范围内。尺寸和成本也是贯穿始终的制约因素，需要在满足性能的前提下进行优化。

PCB布局在整个设计中扮演着极其关键的角色，它不仅仅是连接元件，更是实现高性能、高可靠性的物理基础。寄生参数（电感、电容、电阻）对高频开关电源的影响尤为显著，必须通过布局技巧（如紧凑放置、短宽走线、利用接地层）将其最小化。特别是高 di/dt 的输入环路和高 dv/dt 的开关节点，是布局中需要优先处理的关键区域。接地策略（统一地平面或分割地）的选择和实施，以及敏感信号（如反馈线）的布线，都直接影响到噪声性能和稳定性。

此外，充分利用器件数据手册、应用笔记和参考设计是成功的关键。这些资源不仅提供了器件规格和基本应用信息，更包含了制造商基于器件特性和大量实践经验总结出的设计建议和布局范例。理解这些建议背后的原理，并结合具体应用进行调整，可以大大提高设计效率和成功率。

最后，严格的测试验证是设计流程中不可或缺的一环。通过全面的静态、动态、输出质量、稳定性和热性能测试，可以确保设计满足所有规格要求，并在实际工作条件下表现稳定可靠。

总之，设计高性能、高可靠性的DC-DC转换器板级电路，需要设计者具备扎实的理论基础，掌握关键元件的特性，精通PCB布局技巧，并重视仿真与实际测试验证相结合的迭代优化过程。这是一个需要细致权衡和全面考量的系统工程。

#### 引用的著作

1. Buck Converters (Step-Down Converter) - Monolithic Power Systems, 访问时间为 五月 1, 2025， [https://www.monolithicpower.com/en/learning/mpscholar/power-electronics/dc-dc-converters/buck-converters](https://www.monolithicpower.com/en/learning/mpscholar/power-electronics/dc-dc-converters/buck-converters)
    
2. www.ti.com, 访问时间为 五月 1, 2025， [https://www.ti.com/download/trng/docs/seminar/Topic%206%20-%20DC-DC%20Power%20Conversion%20and%20System%20Design%20Considerations%20for%20Battery%20Operated%20System.pdf](https://www.ti.com/download/trng/docs/seminar/Topic%206%20-%20DC-DC%20Power%20Conversion%20and%20System%20Design%20Considerations%20for%20Battery%20Operated%20System.pdf)
    
3. How to Test a DC-DC Converter for Quality and Reliability - Chroma Systems Solutions, 访问时间为 五月 1, 2025， [https://www.chromausa.com/dc-dc-converter-test/](https://www.chromausa.com/dc-dc-converter-test/)
    
4. Introduction To Buck-Boost Converter | Robu.in, 访问时间为 五月 1, 2025， [https://robu.in/introduction-to-buck-boost-converter/](https://robu.in/introduction-to-buck-boost-converter/)
    
5. An Introduction to Buck, Boost, and Buck-Boost Converters | RECOM, 访问时间为 五月 1, 2025， [https://recom-power.com/en/rec-n-an-introduction-to-buck,-boost,-and-buck-boost-converters-131.html](https://recom-power.com/en/rec-n-an-introduction-to-buck,-boost,-and-buck-boost-converters-131.html)
    
6. www.ti.com, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/ug/tiduai7/tiduai7.pdf](https://www.ti.com/lit/ug/tiduai7/tiduai7.pdf)
    
7. www.ti.com, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/an/slua093/slua093.pdf](https://www.ti.com/lit/an/slua093/slua093.pdf)
    
8. What are buck, boost and buck-boost DC-DC converters? | Blogs ..., 访问时间为 五月 1, 2025， [https://www.us.lambda.tdk.com/resources/blogs/20200731.html](https://www.us.lambda.tdk.com/resources/blogs/20200731.html)
    
9. Power fundamentals - DC/DC fundamentals | TI.com, 访问时间为 五月 1, 2025， [https://www.ti.com/video/series/power-fundamentals-dc-dc-fundamentals.html](https://www.ti.com/video/series/power-fundamentals-dc-dc-fundamentals.html)
    
10. Comprehensively Understand and Analyze Switching Regulator Noise - Analog Devices, 访问时间为 五月 1, 2025， [https://www.analog.com/en/resources/technical-articles/comprehensively-understand-and-analyze-switching-regulator-noise.html](https://www.analog.com/en/resources/technical-articles/comprehensively-understand-and-analyze-switching-regulator-noise.html)
    
11. AN-136: PCB Layout Considerations for Non-Isolated Switching ..., 访问时间为 五月 1, 2025， [https://www.analog.com/en/resources/app-notes/an-136.html](https://www.analog.com/en/resources/app-notes/an-136.html)
    
12. High Density PCB Layout of DC/DC Converters ... - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/document-viewer/lit/html/SSZTC72](https://www.ti.com/document-viewer/lit/html/SSZTC72)
    
13. PCB layout guidelines to optimize power supply performance ..., 访问时间为 五月 1, 2025， [https://www.ti.com/video/6245175641001](https://www.ti.com/video/6245175641001)
    
14. DC/DC converter PCB layout, Part 1 ... - eeNews Europe, 访问时间为 五月 1, 2025， [https://www.eenewseurope.com/en/dc-dc-converter-pcb-layout-part-1/](https://www.eenewseurope.com/en/dc-dc-converter-pcb-layout-part-1/)
    
15. PCB layout for DC to DC converters to avoid ground bounce, EMI ..., 访问时间为 五月 1, 2025， [https://www.eevblog.com/forum/news/pcb-layout-for-dc-to-dc-converters-to-avoid-ground-bounce-emi-and-other-issues/](https://www.eevblog.com/forum/news/pcb-layout-for-dc-to-dc-converters-to-avoid-ground-bounce-emi-and-other-issues/)
    
16. www.nexty-ele.com, 访问时间为 五月 1, 2025， [https://www.nexty-ele.com/nat/wp-content/uploads/sites/3/2017/10/Infineon-TLE6365-AN-v01_01-EN-3.pdf](https://www.nexty-ele.com/nat/wp-content/uploads/sites/3/2017/10/Infineon-TLE6365-AN-v01_01-EN-3.pdf)
    
17. DC-DC Converter PCB Design Guidelines – PCB HERO, 访问时间为 五月 1, 2025， [https://www.pcb-hero.com/blogs/lickys-column/dc-dc-converter-pcb-design-guidelines](https://www.pcb-hero.com/blogs/lickys-column/dc-dc-converter-pcb-design-guidelines)
    
18. AN1191 DC-DC PCB Layout Design for EMC - Diodes Incorporated, 访问时间为 五月 1, 2025， [https://www.diodes.com/assets/App-Note-Files/AN1191-DC-DC-PCB-Layout-Design-for-EMC.pdf](https://www.diodes.com/assets/App-Note-Files/AN1191-DC-DC-PCB-Layout-Design-for-EMC.pdf)
    
19. Ferrite Beads Demystified - Analog Devices, 访问时间为 五月 1, 2025， [https://www.analog.com/en/resources/analog-dialogue/articles/ferrite-beads-demystified.html](https://www.analog.com/en/resources/analog-dialogue/articles/ferrite-beads-demystified.html)
    
20. www.ti.com, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/slva963](https://www.ti.com/lit/pdf/slva963)
    
21. How Thermal Vias Enhance Heat Dissipation in PCBs - Sierra Circuits, 访问时间为 五月 1, 2025， [https://www.protoexpress.com/kb/how-thermal-vias-enhance-heat-dissipation-in-pcbs/](https://www.protoexpress.com/kb/how-thermal-vias-enhance-heat-dissipation-in-pcbs/)
    
22. PCB Thermal Relief Guidelines for Effective Layouts - GlobalWellPCBA, 访问时间为 五月 1, 2025， [https://www.globalwellpcba.com/pcb-thermal-relief-guidelines-for-effective-layouts/](https://www.globalwellpcba.com/pcb-thermal-relief-guidelines-for-effective-layouts/)
    
23. Thermal Via in PCB 101: Design Guidelines, Types, and Best Practices for Heat Dissipation, 访问时间为 五月 1, 2025， [https://www.raypcb.com/thermal-vias-pcb/](https://www.raypcb.com/thermal-vias-pcb/)
    
24. AN031 PCB Design Guidelines to Maximize Cooling of eGaN FETs - EPC Co, 访问时间为 五月 1, 2025， [https://epc-co.com/epc/Portals/0/epc/documents/product-training/AN031_PCB_Design_Guidelines_to_Maximize_Cooling_of_eGaN_FETs.pdf](https://epc-co.com/epc/Portals/0/epc/documents/product-training/AN031_PCB_Design_Guidelines_to_Maximize_Cooling_of_eGaN_FETs.pdf)
    
25. 12 PCB Thermal Management Techniques to Reduce Heating - Sierra Circuits, 访问时间为 五月 1, 2025， [https://www.protoexpress.com/blog/12-pcb-thermal-management-techniques-to-reduce-pcb-heating/](https://www.protoexpress.com/blog/12-pcb-thermal-management-techniques-to-reduce-pcb-heating/)
    
26. TOLERANCE ANALYSIS OF BOOST DC-DC CONVERTER - UniTech Selected Papers, 访问时间为 五月 1, 2025， [https://unitechsp.tugab.bg/images/2024/1-EE/s1_p142_v1.pdf](https://unitechsp.tugab.bg/images/2024/1-EE/s1_p142_v1.pdf)
    
27. What Is a Control System and How To Design a Control Loop for a DC-to-DC Converter | Analog Devices, 访问时间为 五月 1, 2025， [https://www.analog.com/en/resources/technical-articles/how-to-design-a-control-loop-for-a-dc-dc-converter.html](https://www.analog.com/en/resources/technical-articles/how-to-design-a-control-loop-for-a-dc-dc-converter.html)
    
28. Understanding and Simulation of Loop Stability Test for DC/DC Converters - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/sluaa51](https://www.ti.com/lit/pdf/sluaa51)
    
29. Understand Power Supply Loop Stability and Loop Compensation - Part 1: Basic Concepts and Tools | Analog Devices, 访问时间为 五月 1, 2025， [https://www.analog.com/en/resources/technical-articles/power-supply-loop-stability-loop-compensation.html](https://www.analog.com/en/resources/technical-articles/power-supply-loop-stability-loop-compensation.html)
    
30. A Frequency Response Based Approach to DC-DC Control Loop Design - EngagedScholarship@CSU, 访问时间为 五月 1, 2025， [https://engagedscholarship.csuohio.edu/cgi/viewcontent.cgi?referer=&httpsredir=1&article=1433&context=etdarchive](https://engagedscholarship.csuohio.edu/cgi/viewcontent.cgi?referer&httpsredir=1&article=1433&context=etdarchive)
    
31. AN2016: Digital-DC Control Loop Compensation - Renesas, 访问时间为 五月 1, 2025， [https://www.renesas.com/us/en/document/apn/an2016-digital-dc-control-loop-compensation](https://www.renesas.com/us/en/document/apn/an2016-digital-dc-control-loop-compensation)
    
32. DC/DC Converter Stability Measurement - Bode 100 - OMICRON Lab, 访问时间为 五月 1, 2025， [https://www.omicron-lab.com/fileadmin/assets/Bode_100/ApplicationNotes/DC_DC_Stability/App_Note_DC_DC_Stability_V3_3.pdf](https://www.omicron-lab.com/fileadmin/assets/Bode_100/ApplicationNotes/DC_DC_Stability/App_Note_DC_DC_Stability_V3_3.pdf)
    
33. AN149 – Modeling and Loop Compensation Design of Switching Mode Power Supplies, 访问时间为 五月 1, 2025， [https://web.pa.msu.edu/hep/atlas/l1calo/hub/hardware/components/power/Not_Cuurently_Used_In_Hub_Design/linear_modeling_loop_compensation_an149.pdf](https://web.pa.msu.edu/hep/atlas/l1calo/hub/hardware/components/power/Not_Cuurently_Used_In_Hub_Design/linear_modeling_loop_compensation_an149.pdf)
    
34. DC/DC Converter Testing with Fast Load Transient - Richtek Technology, 访问时间为 五月 1, 2025， [https://www.richtek.com/Design%20Support/Technical%20Document/AN038](https://www.richtek.com/Design%20Support/Technical%20Document/AN038)
    
35. Selecting the correct DC-DC converter/controller - Power Supplies ..., 访问时间为 五月 1, 2025， [https://forum.digikey.com/t/selecting-the-correct-dc-dc-converter-controller/26983](https://forum.digikey.com/t/selecting-the-correct-dc-dc-converter-controller/26983)
    
36. www.ti.com, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/slyt614](https://www.ti.com/lit/pdf/slyt614)
    
37. Loop stability analysis of voltage mode buck regulator with different output capacitor types - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/slva301](https://www.ti.com/lit/pdf/slva301)
    
38. power electronics - What is an intuitive explanation on how a buck ..., 访问时间为 五月 1, 2025， [https://electronics.stackexchange.com/questions/710416/what-is-an-intuitive-explanation-on-how-a-buck-and-boost-converter-transfer-ener](https://electronics.stackexchange.com/questions/710416/what-is-an-intuitive-explanation-on-how-a-buck-and-boost-converter-transfer-ener)
    
39. Buck-Boost Converters - Monolithic Power Systems, 访问时间为 五月 1, 2025， [https://www.monolithicpower.com/en/learning/mpscholar/power-electronics/dc-dc-converters/buck-boost-converters](https://www.monolithicpower.com/en/learning/mpscholar/power-electronics/dc-dc-converters/buck-boost-converters)
    
40. Buck–boost converter - Wikipedia, 访问时间为 五月 1, 2025， [https://en.wikipedia.org/wiki/Buck%E2%80%93boost_converter](https://en.wikipedia.org/wiki/Buck%E2%80%93boost_converter)
    
41. Designing a flyback DC/DC converter | TI.com, 访问时间为 五月 1, 2025， [https://www.ti.com/video/series/designing-flyback-dc-dc-converter.html](https://www.ti.com/video/series/designing-flyback-dc-dc-converter.html)
    
42. How Buck, Boost & Buck-Boost DC-DC Converters Work - YouTube, 访问时间为 五月 1, 2025， [https://www.youtube.com/watch?v=PgTR7226sHU](https://www.youtube.com/watch?v=PgTR7226sHU)
    
43. LDO or Switcher?...That is the Question | Video | TI.com - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/video/4495336119001](https://www.ti.com/video/4495336119001)
    
44. A Comprehensive Guide to LDO Regulators: Navigating Noise Compromise Applications and Trends | Analog Devices, 访问时间为 五月 1, 2025， [https://www.analog.com/en/resources/analog-dialogue/articles/a-comprehensive-guide-to-ldo-regulators.html](https://www.analog.com/en/resources/analog-dialogue/articles/a-comprehensive-guide-to-ldo-regulators.html)
    
45. Using LDOs vs Switching Regulators in Your PCB - Altium Resources, 访问时间为 五月 1, 2025， [https://resources.altium.com/p/using-ldo-vs-switching-regulator-your-pcb](https://resources.altium.com/p/using-ldo-vs-switching-regulator-your-pcb)
    
46. power supply - LDO vs switching regulator - Electrical Engineering Stack Exchange, 访问时间为 五月 1, 2025， [https://electronics.stackexchange.com/questions/127323/ldo-vs-switching-regulator](https://electronics.stackexchange.com/questions/127323/ldo-vs-switching-regulator)
    
47. LDOs Vs. Switching Regulators - Power Regulation in PCB Design: Part One - YouTube, 访问时间为 五月 1, 2025， [https://www.youtube.com/watch?v=lHK--OKijkQ](https://www.youtube.com/watch?v=lHK--OKijkQ)
    
48. How to Select Power Inductors - Shenzhen Codaca Electronic Co.,Ltd, 访问时间为 五月 1, 2025， [https://www.codaca.com/Resources_kgdyzrhszgkddg.html](https://www.codaca.com/Resources_kgdyzrhszgkddg.html)
    
49. Ceramic or electrolytic output capacitors in DC/DC converters—Why not both? - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/an/slyt639/slyt639.pdf](https://www.ti.com/lit/an/slyt639/slyt639.pdf)
    
50. How to Avoid Inductor Saturation in your Power Supply Design | Article | MPS, 访问时间为 五月 1, 2025， [https://www.monolithicpower.com/en/learning/resources/how-to-avoid-inductor-saturation-in-your-power-supply-design](https://www.monolithicpower.com/en/learning/resources/how-to-avoid-inductor-saturation-in-your-power-supply-design)
    
51. Choosing the Right Inductor for Your DC/DC Converter - Monolithic Power Systems, 访问时间为 五月 1, 2025， [https://media.monolithicpower.com/mps_cms_document/c/h/choosing_the_right_inductor_for_your_dcdc_converter_final.pdf](https://media.monolithicpower.com/mps_cms_document/c/h/choosing_the_right_inductor_for_your_dcdc_converter_final.pdf)
    
52. How to Understand Power Inductors Parameters for DC/DC Converters | SOS ELECTRONIC, 访问时间为 五月 1, 2025， [https://www.soselectronic.com/en-gb/articles/sos-supplier-of-solution/how-to-understand-power-inductors-parameters-for-dc-dc-converters-2005](https://www.soselectronic.com/en-gb/articles/sos-supplier-of-solution/how-to-understand-power-inductors-parameters-for-dc-dc-converters-2005)
    
53. Waveform Audit: Is Your Inductor Saturated? - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/document-viewer/lit/html/SSZTAV7](https://www.ti.com/document-viewer/lit/html/SSZTAV7)
    
54. Selecting the Best Inductor for Your DC-DC Converter - Coilcraft, 访问时间为 五月 1, 2025， [https://www.coilcraft.com/en-us/resources/application-notes/selecting-the-best-inductor-for-your-dc-dc-convert/](https://www.coilcraft.com/en-us/resources/application-notes/selecting-the-best-inductor-for-your-dc-dc-convert/)
    
55. Specifying Inductor Values - ECS Inc., 访问时间为 五月 1, 2025， [https://ecsxtal.com/news-resources/specifying-inductor-values/](https://ecsxtal.com/news-resources/specifying-inductor-values/)
    
56. The Fundamentals of Power Inductors - Coilcraft, 访问时间为 五月 1, 2025， [https://www.coilcraft.com/getmedia/bec1987d-3725-4d3a-a655-d3a9227abcad/Coilcraft-eBook_Power_Inductors.pdf](https://www.coilcraft.com/getmedia/bec1987d-3725-4d3a-a655-d3a9227abcad/Coilcraft-eBook_Power_Inductors.pdf)
    
57. Input and Output Capacitor Selection, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/slta055](https://www.ti.com/lit/pdf/slta055)
    
58. Selecting Output Capacitors for Power Supply Applications - CUI Inc, 访问时间为 五月 1, 2025， [https://www.cui.com/blog/selecting-output-capacitors-for-power-supply-applications](https://www.cui.com/blog/selecting-output-capacitors-for-power-supply-applications)
    
59. Ceramic or electrolytic output capacitors in DC/DC converters – Why not both?, 访问时间为 五月 1, 2025， [https://www.powerelectronictips.com/ceramic-or-electrolytic-output-capacitors-in-dcdc-converters-why-not-both/](https://www.powerelectronictips.com/ceramic-or-electrolytic-output-capacitors-in-dcdc-converters-why-not-both/)
    
60. GAN PCB - Infineon Technologies, 访问时间为 五月 1, 2025， [https://www.infineon.com/cms/en/product/promopages/gan-pcb/](https://www.infineon.com/cms/en/product/promopages/gan-pcb/)
    
61. Which Capacitor Types Should You Use? | Blogs - Altium Resources, 访问时间为 五月 1, 2025， [https://resources.altium.com/p/which-type-capacitor-should-you-use](https://resources.altium.com/p/which-type-capacitor-should-you-use)
    
62. capacitor - Ceramic caps vs electrolytic. What are the tangible differences in use?, 访问时间为 五月 1, 2025， [https://electronics.stackexchange.com/questions/232631/ceramic-caps-vs-electrolytic-what-are-the-tangible-differences-in-use](https://electronics.stackexchange.com/questions/232631/ceramic-caps-vs-electrolytic-what-are-the-tangible-differences-in-use)
    
63. Need some clarification on ripple current rating for electrolytic caps. : r/AskElectronics, 访问时间为 五月 1, 2025， [https://www.reddit.com/r/AskElectronics/comments/60q6np/need_some_clarification_on_ripple_current_rating/](https://www.reddit.com/r/AskElectronics/comments/60q6np/need_some_clarification_on_ripple_current_rating/)
    
64. Ceramic or electrolytic capacitors for a switching buck regulator?, 访问时间为 五月 1, 2025， [https://electronics.stackexchange.com/questions/5229/ceramic-or-electrolytic-capacitors-for-a-switching-buck-regulator](https://electronics.stackexchange.com/questions/5229/ceramic-or-electrolytic-capacitors-for-a-switching-buck-regulator)
    
65. Application Note #ANP18 Selecting Appropriate Compensation: Type-II or Type-III - MaxLinear, 访问时间为 五月 1, 2025， [https://www.maxlinear.com/appnote/anp-18_selecttypeiiortypeiii_120506.pdf](https://www.maxlinear.com/appnote/anp-18_selecttypeiiortypeiii_120506.pdf)
    
66. Step-by-Step Process to Calculate a DC-to-DC Compensation Network | Analog Devices, 访问时间为 五月 1, 2025， [https://www.analog.com/en/resources/analog-dialogue/articles/step-by-step-process-to-calculate-a-dc-to-dc-compensation-network.html](https://www.analog.com/en/resources/analog-dialogue/articles/step-by-step-process-to-calculate-a-dc-to-dc-compensation-network.html)
    
67. and8143-d.pdf - Onsemi, 访问时间为 五月 1, 2025， [https://www.onsemi.com/download/application-notes/pdf/and8143-d.pdf](https://www.onsemi.com/download/application-notes/pdf/and8143-d.pdf)
    
68. How to Improve Power Supply Output Regulation Accuracy with the LTpowerCAD Resistor Divider Tool | Analog Devices, 访问时间为 五月 1, 2025， [https://www.analog.com/en/resources/analog-dialogue/articles/how-to-improve-power-supply-output-regulation-accuracy-with-the-ltpowercad-resistor-divider-tool.html](https://www.analog.com/en/resources/analog-dialogue/articles/how-to-improve-power-supply-output-regulation-accuracy-with-the-ltpowercad-resistor-divider-tool.html)
    
69. Designing the Feedback Voltage Resistor Divider in a DC/DC Converter | Article | MPS, 访问时间为 五月 1, 2025， [http://www.monolithicpower.com/learning/resources/designing-the-feedback-voltage-resistor-divider-in-a-dcdc-converter](http://www.monolithicpower.com/learning/resources/designing-the-feedback-voltage-resistor-divider-in-a-dcdc-converter)
    
70. Delivering Precision Performance - Power Systems Design, 访问时间为 五月 1, 2025， [https://www.powersystemsdesign.com/delivering-precision-performance/22](https://www.powersystemsdesign.com/delivering-precision-performance/22)
    
71. Accurate DC-DC Converter Minimizes Wasted Power - Analog Devices, 访问时间为 五月 1, 2025， [https://www.analog.com/en/resources/technical-articles/accurate-dcdc-converter-minimizes-wasted-power.html](https://www.analog.com/en/resources/technical-articles/accurate-dcdc-converter-minimizes-wasted-power.html)
    
72. Design considerations for a resistive feedback divider in a DC/DC converter - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/slyt469](https://www.ti.com/lit/pdf/slyt469)
    
73. Feedback resistor package selection for DC-DC converter - Electronics Stack Exchange, 访问时间为 五月 1, 2025， [https://electronics.stackexchange.com/questions/723470/feedback-resistor-package-selection-for-dc-dc-converter](https://electronics.stackexchange.com/questions/723470/feedback-resistor-package-selection-for-dc-dc-converter)
    
74. TB417: Designing Stable Compensation Networks for Single Phase Voltage Mode Buck Regulators - Renesas, 访问时间为 五月 1, 2025， [https://www.renesas.com/en/document/oth/tb417-designing-stable-compensation-networks-single-phase-voltage-mode-buck-regulators](https://www.renesas.com/en/document/oth/tb417-designing-stable-compensation-networks-single-phase-voltage-mode-buck-regulators)
    
75. Demystifying Type II and Type III Compensators Using Op Amp and OTA for DC/DC Converters - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/slva662](https://www.ti.com/lit/pdf/slva662)
    
76. Designing Type III Compensation for Current Mode Step-Down Converters (Rev. A) - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/slva352](https://www.ti.com/lit/pdf/slva352)
    
77. TPS54331-Q1 data sheet, product information and support | TI.com, 访问时间为 五月 1, 2025， [https://www.ti.com/product/TPS54331-Q1](https://www.ti.com/product/TPS54331-Q1)
    
78. TPS54331 data sheet, product information and support | TI.com, 访问时间为 五月 1, 2025， [https://www.ti.com/product/ru-ru/TPS54331](https://www.ti.com/product/ru-ru/TPS54331)
    
79. TPS54331 data sheet, product information and support | TI.com, 访问时间为 五月 1, 2025， [https://www.ti.com/product/TPS54331](https://www.ti.com/product/TPS54331)
    
80. AN6128 - The STPMIC25 PCB layout guidelines - STMicroelectronics, 访问时间为 五月 1, 2025， [https://www.st.com/resource/en/application_note/an6128-the-stpmic25-pcb-layout-guidelines-stmicroelectronics.pdf](https://www.st.com/resource/en/application_note/an6128-the-stpmic25-pcb-layout-guidelines-stmicroelectronics.pdf)
    
81. www.ti.com, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/swcu080](https://www.ti.com/lit/pdf/swcu080)
    
82. www.ti.com, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/ds/symlink/tps54331.pdf](https://www.ti.com/lit/ds/symlink/tps54331.pdf)
    
83. PCB Grounding Techniques | A Guide to PCB Grounding, 访问时间为 五月 1, 2025， [https://www.mclpcb.com/blog/guide-to-pcb-grounding-techniques/](https://www.mclpcb.com/blog/guide-to-pcb-grounding-techniques/)
    
84. Looking for advice: Wiring the grounds on an isolated DC/DC ..., 访问时间为 五月 1, 2025， [https://www.freestompboxes.org/viewtopic.php?t=33001](https://www.freestompboxes.org/viewtopic.php?t=33001)
    
85. pcb - Properly Grounding Analog/mixed ICs from DC-DC Converter ..., 访问时间为 五月 1, 2025， [https://electronics.stackexchange.com/questions/419430/properly-grounding-analog-mixed-ics-from-dc-dc-converter](https://electronics.stackexchange.com/questions/419430/properly-grounding-analog-mixed-ics-from-dc-dc-converter)
    
86. Types of PCB Grounding Explained | PCB Layout - YouTube, 访问时间为 五月 1, 2025， [https://www.youtube.com/watch?v=19WnYPhNOH0](https://www.youtube.com/watch?v=19WnYPhNOH0)
    
87. PCB Grounding Techniques for High-Power and HDI | Sierra Circuits, 访问时间为 五月 1, 2025， [https://www.protoexpress.com/blog/best-pcb-grounding-techniques-for-high-power-and-hdi-designs/](https://www.protoexpress.com/blog/best-pcb-grounding-techniques-for-high-power-and-hdi-designs/)
    
88. All About Ground Reference and Chassis Ground in Electronics ..., 访问时间为 五月 1, 2025， [https://resources.altium.com/p/stay-grounded-digital-analog-and-earth-ground-pcb-layout](https://resources.altium.com/p/stay-grounded-digital-analog-and-earth-ground-pcb-layout)
    
89. PCB grounding AC/DC and DC/DC converters - Power management ..., 访问时间为 五月 1, 2025， [https://e2e.ti.com/support/power-management-group/power-management/f/power-management-forum/324248/pcb-grounding-ac-dc-and-dc-dc-converters](https://e2e.ti.com/support/power-management-group/power-management/f/power-management-forum/324248/pcb-grounding-ac-dc-and-dc-dc-converters)
    
90. Grounding Issues: Working correctly with GND and PWRGND ..., 访问时间为 五月 1, 2025， [https://forum.kicad.info/t/grounding-issues-working-correctly-with-gnd-and-pwrgnd/4540](https://forum.kicad.info/t/grounding-issues-working-correctly-with-gnd-and-pwrgnd/4540)
    
91. dc dc converter - LTM4612: Power Ground and Signal Ground ..., 访问时间为 五月 1, 2025， [https://electronics.stackexchange.com/questions/647687/ltm4612-power-ground-and-signal-ground-planes](https://electronics.stackexchange.com/questions/647687/ltm4612-power-ground-and-signal-ground-planes)
    
92. 11 Best High-Speed PCB Routing Practices - Sierra Circuits, 访问时间为 五月 1, 2025， [https://www.protoexpress.com/blog/best-high-speed-pcb-routing-practices/](https://www.protoexpress.com/blog/best-high-speed-pcb-routing-practices/)
    
93. Buck converter feedback from power plane? - EEVblog, 访问时间为 五月 1, 2025， [https://www.eevblog.com/forum/projects/buck-converter-feedback-from-power-plane/](https://www.eevblog.com/forum/projects/buck-converter-feedback-from-power-plane/)
    
94. dc dc converter - DCDC layout review - Electronics Stack Exchange, 访问时间为 五月 1, 2025， [https://electronics.stackexchange.com/questions/727377/dcdc-layout-review](https://electronics.stackexchange.com/questions/727377/dcdc-layout-review)
    
95. COMPUTER AIDED DESIGN OF SNUBBER CIRCUIT FOR DC/DC CONVERTER WITH SiC POWER MOSFET DEVICES, 访问时间为 五月 1, 2025， [https://www.infona.pl/resource/bwmeta1.element.baztech-d69f8429-c35c-4d6c-b88c-cb273333e1e9/content/partDownload/ce3ea195-5aab-34d1-bfff-eabcf90bd243](https://www.infona.pl/resource/bwmeta1.element.baztech-d69f8429-c35c-4d6c-b88c-cb273333e1e9/content/partDownload/ce3ea195-5aab-34d1-bfff-eabcf90bd243)
    
96. RC Snubbers and RCD Clamps for Isolated DC-DC Converters - PCB Design & Analysis, 访问时间为 五月 1, 2025， [https://resources.pcb.cadence.com/blog/rc-snubbers-and-rcd-clamps-for-isolated-dc-dc-converters](https://resources.pcb.cadence.com/blog/rc-snubbers-and-rcd-clamps-for-isolated-dc-dc-converters)
    
97. Application Note AN-4147 - Design Guidelines for RCD Snubber of Flyback Converters - TI E2E, 访问时间为 五月 1, 2025， [https://e2e.ti.com/cfs-file/__key/communityserver-discussions-components-files/196/Design-Guidelines-for-RCD-Snubber-of-Flyback-Converters_2D00_Fairchild-AN4147.pdf](https://e2e.ti.com/cfs-file/__key/communityserver-discussions-components-files/196/Design-Guidelines-for-RCD-Snubber-of-Flyback-Converters_2D00_Fairchild-AN4147.pdf)
    
98. Designing a Pi filter as input of dc dc converter - All About Circuits Forum, 访问时间为 五月 1, 2025， [https://forum.allaboutcircuits.com/threads/designing-a-pi-filter-as-input-of-dc-dc-converter.191262/](https://forum.allaboutcircuits.com/threads/designing-a-pi-filter-as-input-of-dc-dc-converter.191262/)
    
99. How to Filter Out Noise in Your DC/DC Design, 访问时间为 五月 1, 2025， [https://www.ti.com/document-viewer/lit/html/SSZTCB3](https://www.ti.com/document-viewer/lit/html/SSZTCB3)
    
100. Having a hard time understanding and how to choose the correct ferrite bead. :/ : r/AskElectronics - Reddit, 访问时间为 五月 1, 2025， [https://www.reddit.com/r/AskElectronics/comments/1gq0ta3/having_a_hard_time_understanding_and_how_to/](https://www.reddit.com/r/AskElectronics/comments/1gq0ta3/having_a_hard_time_understanding_and_how_to/)
    
101. Filtering power supplies using ferrite bead - EEVblog, 访问时间为 五月 1, 2025， [https://www.eevblog.com/forum/eda/filtering-power-supplies-using-ferrite-bead/](https://www.eevblog.com/forum/eda/filtering-power-supplies-using-ferrite-bead/)
    
102. #askLorandt explains: Chip Bead Ferrite Filter for DC-DC Converter - YouTube, 访问时间为 五月 1, 2025， [https://m.youtube.com/watch?v=CqCAGEj4_0M](https://m.youtube.com/watch?v=CqCAGEj4_0M)
    
103. Design of Snubbers for Power Circuits - Cornell Dubilier, 访问时间为 五月 1, 2025， [https://www.cde.com/resources/technical-papers/design.pdf](https://www.cde.com/resources/technical-papers/design.pdf)
    
104. Snubber circuit design methods - ROHM Semiconductor, 访问时间为 五月 1, 2025， [https://fscdn.rohm.com/en/products/databook/applinote/discrete/sic/mosfet/sic-mos_snubber_circuit_design_an-e.pdf](https://fscdn.rohm.com/en/products/databook/applinote/discrete/sic/mosfet/sic-mos_snubber_circuit_design_an-e.pdf)
    
105. (PDF) An optimal designed RCD snubber for DC-DC converters - ResearchGate, 访问时间为 五月 1, 2025， [https://www.researchgate.net/publication/224392105_An_optimal_designed_RCD_snubber_for_DC-DC_converters](https://www.researchgate.net/publication/224392105_An_optimal_designed_RCD_snubber_for_DC-DC_converters)
    
106. TPS54331 Datasheet(PDF) - Texas Instruments - Alldatasheet.com, 访问时间为 五月 1, 2025， [https://www.alldatasheet.com/datasheet-pdf/pdf/239871/TI/TPS54331.html](https://www.alldatasheet.com/datasheet-pdf/pdf/239871/TI/TPS54331.html)
    
107. TPS54331GDR Datasheet(PDF) - Texas Instruments - Alldatasheet.com, 访问时间为 五月 1, 2025， [https://www.alldatasheet.com/datasheet-pdf/pdf/849511/TI1/TPS54331GDR.html](https://www.alldatasheet.com/datasheet-pdf/pdf/849511/TI1/TPS54331GDR.html)
    
108. www.ti.com, 访问时间为 五月 1, 2025， [https://www.ti.com/lit/pdf/slva958](https://www.ti.com/lit/pdf/slva958)
    
109. TIDA-00349 reference design | TI.com - Texas Instruments, 访问时间为 五月 1, 2025， [https://www.ti.com/tool/TIDA-00349](https://www.ti.com/tool/TIDA-00349)
    
110. How to Perform DC-to-DC Converter Testing - Keysight, 访问时间为 五月 1, 2025， [https://www.keysight.com/us/en/solutions/perform-dc-dc-converter-testing.html](https://www.keysight.com/us/en/solutions/perform-dc-dc-converter-testing.html)
    
111. Application Notes | DC-DC converters | Murata Power Solutions, 访问时间为 五月 1, 2025， [https://www.murata.com/-/media/webrenewal/products/power/appnote/dcan-68.pdf](https://www.murata.com/-/media/webrenewal/products/power/appnote/dcan-68.pdf)
    
112. How to test dc-dc converters - Test & Measurement Tips, 访问时间为 五月 1, 2025， [https://www.testandmeasurementtips.com/test-dc-dc-converters/](https://www.testandmeasurementtips.com/test-dc-dc-converters/)
    
113. Charged EVs | A quick guide to automotive DC-DC converter testing, 访问时间为 五月 1, 2025， [https://chargedevs.com/newswire/a-quick-guide-to-automotive-dc-dc-converter-testing/](https://chargedevs.com/newswire/a-quick-guide-to-automotive-dc-dc-converter-testing/)
    

**