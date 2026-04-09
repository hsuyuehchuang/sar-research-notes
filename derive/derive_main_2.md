# TOPS SAR Derivations (Main 2)

## Flowchart

- [Problem Definition](#problem-definition)
- [Processing Chain Overview](#processing-chain-overview)
- [Question-Oriented Proof Map](#question-oriented-proof-map)
  - [Why Does Azimuth Frequency Folding Occur?](#31-why-does-azimuth-frequency-folding-occur)
  - [What Does Mosaicking Do?](#32-what-does-mosaicking-do)
  - [What Does Deramping Do?](#33-what-does-deramping-do)
  - [Why Does Azimuth Time Folding Occur?](#34-why-does-azimuth-time-folding-occur)
- [Frequency-Domain Mainline](#4-frequency-domain-mainline)
  - [Signal Model And Azimuth Sampling](#41-signal-model-and-azimuth-sampling)
  - [Azimuth FFT And Frequency Folding](#42-azimuth-fft-and-frequency-folding)
  - [Frequency-Domain Mosaicking](#43-frequency-domain-mosaicking)
  - [Frequency-Domain Deramping](#44-frequency-domain-deramping)
  - [Frequency-Domain LPF](#45-frequency-domain-lpf)
  - [Frequency-Domain Reramping](#46-frequency-domain-reramping)
  - [Azimuth Compression And Return To Time Domain](#47-azimuth-compression-and-return-to-time-domain)
- [Time-Domain Mainline](#5-time-domain-mainline)
  - [Circular Convolution And Azimuth Time Folding](#51-circular-convolution-and-azimuth-time-folding)
  - [Time-Domain UFR / Time Mosaicking](#52-time-domain-ufr--time-mosaicking)
  - [Time-Domain Deramping](#53-time-domain-deramping)
  - [Time-Domain LPF](#54-time-domain-lpf)
  - [Time-Domain Reramping](#55-time-domain-reramping)
  - [Final Focused Expression](#56-final-focused-expression)
- [Reference Guide](#6-reference-guide)

## Hierarchy

- [Problem Definition](#problem-definition)
- [Processing Chain Overview](#processing-chain-overview)
- [Question-Oriented Proof Map](#question-oriented-proof-map)
- [Frequency-Domain Mainline](#4-frequency-domain-mainline)
- [Time-Domain Mainline](#5-time-domain-mainline)
- [Reference Guide](#6-reference-guide)

## Problem Definition

本文件的目標，是把 TOPS SAR 的 azimuth processing chain 寫成一條顯式且可追蹤的數學主線，並回答以下幾個核心問題：

1. 如何以數學形式證明 azimuth frequency folding 的來源與其 aliasing 機制
2. 如何以數學形式說明 mosaicking 所執行的重排操作
3. 如何以數學形式證明 deramping 對主 replica phase curvature 的影響
4. 如何以數學形式證明 azimuth time folding 的來源與其 aliasing 機制

## Processing Chain Overview

這份 `main_2` 不打算把每一段細節全部重寫，而是先把主線與問題對應關係整理清楚，再把詳細數學推導導向各自的 reference notes。

主鏈可以先寫成

$$
\color{red}{
s_1(\tau,\eta)
\rightarrow
S_{1,c}(\tau,f_\eta)
\rightarrow
S_2(\tau,f_\eta)
\rightarrow
S_3(\tau,f_\eta)
\rightarrow
S_4(\tau,f_\eta)
\rightarrow
S_5(\tau,f_\eta)
\rightarrow
S_6(\tau,f_\eta)
\rightarrow
s_7(\tau,\eta)
\rightarrow
I_{\mathrm{circ}}(\tau,\eta)
\rightarrow
I_8(\tau,\eta)
\rightarrow
I_9(\tau,\eta)
\rightarrow
I_{10}(\tau,\eta)
\rightarrow
I_{11}(\tau,\eta)
\rightarrow
I_{\mathrm{focus}}(\tau,\eta)
}
$$

其中

- $S_2 \rightarrow S_6$ 是 frequency-domain UFR view
- $I_8 \rightarrow I_{11}$ 是 time-domain UFR view
- `deramp -> LPF -> reramp` 這條鏈在 frequency 與 time 兩個 view 中都必須出現

## Question-Oriented Proof Map

這一節不重複全部推導，而是直接回答 Problem Definition 中的四個問題，並指出各自最關鍵的 equation。

### 3.1. Why Does Azimuth Frequency Folding Occur?

slow-time sampling 先出現在 raw data / range-compressed data 中：

$$
\color{red}{
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
$$

對 azimuth 做 Fourier transform 後，time-domain sampling comb 變成 frequency-domain comb，因此連續頻譜會以 `PRF` 為間隔週期性複製。

最直接表達 folding 的式子是

$$
\color{red}{
W_{\mathrm{fold}}(f_\eta;\omega_s) =
\sum_{k=-\infty}^{\infty}
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

這個式子說明：連續 azimuth envelope $W_a$ 被以 `PRF` 為間隔複製；若這些 replicas 在同一基本頻帶內彼此重疊，就形成 azimuth frequency folding / aliasing。

詳細推導可見：

- [azimuth_freq_folding.md 第 4 節](./azimuth_freq_folding.md#4-continuous-azimuth-spectrum)
- [azimuth_freq_folding.md 第 5 節](./azimuth_freq_folding.md#5-folded-spectrum-from-the-sampling-comb)

### 3.2. What Does Mosaicking Do?

mosaicking 的重點不是消掉 replicas，而是把原本 folded 在同一主頻帶中的 replicas，重排到 extended axis 上。

在 frequency-domain view 中，最關鍵的式子是

$$
\color{red}{
S_3(\tau,f_\eta)
}
$$

它表示每個 replica 已經用索引 $m$ 重新編號，並被放到各自對應的 extended frequency support 上。

在 time-domain view 中，對應的式子是

$$
\color{red}{
I_8(\tau,\eta)
}
$$

它表示 time replicas 也被重排到 extended azimuth-time axis，而不是再折回同一主窗口。

因此，mosaicking 的數學意義是：

- frequency view：把 folded frequency replicas 排開
- time view：把 wrapped time replicas 排開

詳細推導可見：

- [azimuth_freq_folding.md](./azimuth_freq_folding.md)
- [azimuth_time_folding.md](./azimuth_time_folding.md)

### 3.3. What Does Deramping Do?

deramping 的作用不是把 replicas 消失，而是把主 replica 的 quadratic curvature 拿掉，使後續固定通帶的 LPF 成為可能。

在 frequency-domain view 中，主 replica 與 deramp filter 相乘後的關鍵 cancellation 可寫成

$$
\color{red}{
\exp\left(
-j\psi_{\mathrm{main}}(f_\eta)
\right)
\cdot
H_{\mathrm{de}}(f_\eta) =
\exp\left(
-j\left[
\psi_{0,\mathrm{main}}
+\psi_{1,\mathrm{main}}(f_\eta-f_{\mathrm{ref}})
\right]
\right)
}
$$

因此主 replica 的殘餘 phase 變成

$$
\color{red}{
\psi_{\mathrm{after}}(f_\eta) =
\psi_{0,\mathrm{main}}
+\psi_{1,\mathrm{main}}(f_\eta-f_{\mathrm{ref}})
}
$$

所以

$$
\color{red}{
\frac{d^2\psi_{\mathrm{after}}(f_\eta)}{df_\eta^2} = 0
}
$$

這就是「被拉平 / 被拉直」最直接的數學證據。

對主 replica 而言，因為 $\psi_{2,\mathrm{ref}}=\psi_{2,m_0}$，所以其殘餘二次項近似為零。這就是 deramping 真正的物理作用：它不是把 replicas 消失，而是把主 replica 的 quadratic curvature 拿掉。

在 time-domain view 中，對應判據可寫成

$$
\color{red}{
f_{\eta,\mathrm{new}}(\eta) = 0
}
$$

表示主能量脊線的 instantaneous Doppler slope 被展平。

詳細推導可見：

- [azimuth_deramp_LPF.md](./azimuth_deramp_LPF.md)
- [freq_time_deramping.md](./freq_time_deramping.md)

### 3.4. Why Does Azimuth Time Folding Occur?

azimuth compression 之後，若以有限長 FFT / IFFT 實作卷積，數學上對應的是 circular convolution，而不是 linear convolution。

最關鍵的式子是

$$
\color{red}{
I_{\mathrm{circ}}(\eta) =
\sum_{m=-\infty}^{\infty}
I_{\mathrm{lin}}(\eta-mT_{\mathrm{window}})
}
$$

這個式子說明：線性卷積的結果會以 $T_{\mathrm{window}}$ 為週期被複製，超出主窗口的部分會折回主窗口，因此形成 azimuth time folding / wrap-around / aliasing。

詳細推導可見：

- [azimuth_time_folding.md 第 6 節](./azimuth_time_folding.md#6-circular-convolution-and-wrap-around)
- [azimuth_time_folding.md 第 7 節](./azimuth_time_folding.md#7-wrap-around-location-formula)

## 4. Frequency-Domain Mainline

這一節保留 frequency-domain 的主鏈，用來回答：

- folding 是怎麼從 sampling comb 出來的
- mosaicking 在 frequency axis 上做了什麼
- deramp / LPF / reramp 如何串起來

### 4.1. Signal Model And Azimuth Sampling

起點仍然是 raw data 與 range compression 後的 slow-time sampling comb：

$$
s_1(\tau,\eta) \cdot
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
$$

這裡的 $n$ 是 time-domain slow-time sample index。

### 4.2. Azimuth FFT And Frequency Folding

接著做 azimuth FFT：

$$
\color{red}{
S_{1,c}(\tau,f_\eta;\omega_s) =
\mathcal{F}_\eta\left\{s_1(\tau,\eta)\right\}
}
$$

此時 time-domain 的 index $n$ 不再顯式出現，它的效果改寫成 frequency-domain replicas 的 index $k$。

### 4.3. Frequency-Domain Mosaicking

frequency mosaicking 的關鍵輸出是

$$
\color{red}{
S_3(\tau,f_\eta)
}
$$

它表示 folded replicas 已經被排到 extended frequency axis 上。

### 4.4. Frequency-Domain Deramping

主 replica 的 quadratic phase 在這一步被 reference deramp filter 抵消，核心判據仍然是

$$
\color{red}{
\frac{d^2\psi_{\mathrm{after}}(f_\eta)}{df_\eta^2} = 0
}
$$

### 4.5. Frequency-Domain LPF

deramping 之後，主 replica 的 curvature 被拿掉，因此可用固定頻寬 LPF 保留目標 replica。

### 4.6. Frequency-Domain Reramping

reramping 的目的是在保留 main replica 後，再補回 reference phase law，讓後續 azimuth compression 的相位模型回到一致座標。

### 4.7. Azimuth Compression And Return To Time Domain

完成 frequency-domain UFR 後，經過 reramp 與 azimuth compression，回到 azimuth-time domain，銜接到下一段 time-domain folding / time-domain UFR。

## 5. Time-Domain Mainline

這一節保留 time-domain / UFR 的主鏈，用來回答：

- circular convolution 為什麼導致 time folding
- time mosaicking 如何把 wrapped replicas 排開
- 為什麼 time-domain 也有對應的 `deramp -> LPF -> reramp`

### 5.1. Circular Convolution And Azimuth Time Folding

azimuth time folding 的核心不是幾何，而是有限長 FFT 所對應的 circular convolution。

$$
\color{red}{
I_{\mathrm{circ}}(\eta) =
\sum_{m=-\infty}^{\infty}
I_{\mathrm{lin}}(\eta-mT_{\mathrm{window}})
}
$$

### 5.2. Time-Domain UFR / Time Mosaicking

time-domain mosaicking 的關鍵輸出是

$$
\color{red}{
I_8(\tau,\eta)
}
$$

它表示原本 wrap 回主窗口的 time replicas，已經被重新展開到 extended azimuth-time axis。

### 5.3. Time-Domain Deramping

在 time-domain view 中，對主 replica 做 local chirp deramping 後，可把主能量脊線展平。

對應的判據是

$$
\color{red}{
f_{\eta,\mathrm{new}}(\eta) = 0
}
$$

### 5.4. Time-Domain LPF

當主 replica 被展平後，可用固定 keep window / LPF 去保留主支持區。

### 5.5. Time-Domain Reramping

time-domain reramping 的作用與 frequency-domain reramping 對稱：在保留主 replica 後，補回 reference phase law。

### 5.6. Final Focused Expression

最終輸出是

$$
\color{red}{
I_{\mathrm{focus}}(\tau,\eta)
}
$$

## 6. Reference Guide

若要追完整細節，可直接分流到以下文件：

- frequency folding： [azimuth_freq_folding.md](./azimuth_freq_folding.md)
- deramp + LPF： [azimuth_deramp_LPF.md](./azimuth_deramp_LPF.md)
- frequency/time dual-view deramping： [freq_time_deramping.md](./freq_time_deramping.md)
- azimuth compression： [azimuth_compression.md](./azimuth_compression.md)
- time folding： [azimuth_time_folding.md](./azimuth_time_folding.md)

