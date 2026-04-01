# Explain UFR3: TOPS Time-Frequency Flow

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Related notes:
  - [Azimuth Frequency Folding](./azimuth_freq_folding.md)
  - [Azimuth Frequency UFR](./azimuth_freq_ufr.md)
  - [Azimuth Compression](./azimuth_compression.md)
  - [Azimuth Time UFR](./azimuth_time_ufr.md)
- Companion note: [Explain UFR4](./explain_ufr4.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Input Aliased Signal](#1-input-aliased-signal)
- [2. Frequency Mosaicking](#2-frequency-mosaicking)
- [3. Frequency Deramping](#3-frequency-deramping)
- [4. Pseudo-Time LPF](#4-pseudo-time-lpf)
- [5. Frequency Reramping](#5-frequency-reramping)
- [6. Azimuth Compression](#6-azimuth-compression)
- [7. Time Mosaicking](#7-time-mosaicking)
- [8. Time Deramping](#8-time-deramping)
- [9. Time LPF](#9-time-lpf)
- [10. Final Time Reramping](#10-final-time-reramping)
- [Final Result](#final-result)

## Summary

- `explain_UFR3.py` 把整個 TOPS azimuth chain 轉成一條 figure-driven 的 time-frequency explanation。
- 這條主線是
  `$s_1(\eta) \rightarrow S_1(f_\eta) \rightarrow S_2(f_\eta) \rightarrow S_3(f_\eta) \rightarrow S_4(f_\eta) \rightarrow S_5(f_\eta) \rightarrow s_7(\eta) \rightarrow s_{7,\mathrm{mosaic}}(\eta) \rightarrow s_8(\eta) \rightarrow s_{8,\mathrm{LPF}}(\eta) \rightarrow s_{\mathrm{final}}(\eta)$`。
- 前半段處理 `frequency folding / frequency UFR`，後半段處理 `focused-time wrap-around / time UFR`。
- 這份文件的閱讀方式不是先看完整推導再回頭對圖，而是每一張圖下面立刻放對應數學與物理解釋。
- 因此讀者看到某一張圖時，可以直接知道那一步做了什麼、數學上信號變成什麼、以及它為什麼會導向下一張圖。

## Problem Definition

本文件要回答三件事：

1. `explain_UFR3.py` 每一張圖對應哪一個 stage。
2. 每一個 stage 的 fully expanded closed form 是什麼。
3. 每一張圖所顯示的數學與物理現象是什麼。

## Symbols And Assumptions

- $\eta$：azimuth slow time
- $f_\eta$：azimuth frequency
- $T_{\mathrm{burst}}$：單次 burst 的 slow-time window
- $\mathrm{PRF}$：azimuth sampling rate
- $N_{\mathrm{az}}=\mathrm{PRF}\cdot T_{\mathrm{burst}}$：burst 內取樣點數
- $k_a$：target inherent azimuth FM rate
- $k_s$：antenna steering FM rate 或 Doppler-centroid rate
- $T_{\mathrm{dwell}}$：單一目標的照明時間
- $t_c$：target focus-center label
- $t_{\mathrm{expo}}$：目標在 raw burst 中真正被照明的中心時刻
- $k_t=\frac{k_ak_s}{k_a-k_s}$：time-UFR 使用的等效 time-domain chirp rate

假設如下：

- 本文只模擬單一 range cell，因此所有圖都只解釋 azimuth 向現象。
- `np.tile` 是 folded-replica 展開的離散示意。
- deramp 與 reramp 以程式內的 FFT 慣例為準，因此重點是「一對共軛補償」，不是單看符號正負。

## 1. Input Aliased Signal

![Step 1](./figures/explain_ufr3/ufr3_step1_input_aliased.png)

Figure Caption:
這張圖是原始輸入 `$s_1(\eta)$` 的 time-frequency view。每條斜 chirp trace 代表一個 target 的 azimuth phase history，而可見區域則由照明窗口控制。

Mathematical Step:
每個 target 只在自己的 illumination window 內被加入 raw signal。

Fully Expanded Closed Form:

$$
w_p(\eta) =
\mathrm{rect}\left(
\frac{\eta-t_{\mathrm{expo},p}}{T_{\mathrm{dwell}}}
\right)
$$

$$
s_{1,p}(\eta) =
w_p(\eta)\,
\exp\left(
j\pi k_a(\eta-t_{c,p})^2
\right)
$$

$$
{\color{red}
s_1(\eta) =
\sum_p
\mathrm{rect}\left(
\frac{\eta-t_{\mathrm{expo},p}}{T_{\mathrm{dwell}}}
\right)
\exp\left(
j\pi k_a(\eta-t_{c,p})^2
\right)
}
$$

$$
{\color{red}
t_{\mathrm{expo},p} = \frac{k_a}{k_a-k_s}t_{c,p}
}
$$

Physical Meaning:
目標真正出現在 raw burst 內的時刻不是 `$t_c$`，而是 `$t_{\mathrm{expo}}$`。TOPS 掃描先改變的是照明幾何，這也是後面 folding 與 focused-time inflation 的源頭。

Why This Leads To The Next Figure:
一旦對 `$s_1(\eta)$` 做 FFT，就會進入原始 PRF 限制下的 aliased azimuth spectrum。

## 2. Frequency Mosaicking

![Step 2](./figures/explain_ufr3/ufr3_step2_mosaicking.png)

Figure Caption:
這張圖對應 `$S_2(f_\eta)$`。它不是新的物理頻譜，而是把 folded 在 principal band 內的 replicas 沿 extended frequency axis 攤開。

Mathematical Step:
先對 raw signal 做 FFT，再做 frequency mosaicking。

Fully Expanded Closed Form:

$$
S_1(f_\eta) = \mathcal{F}_\eta\left[s_1(\eta)\right]
$$

$$
{\color{red}
S_2(f_\eta) =
\sum_{m=-1}^{1}
S_1(f_\eta-m\cdot\mathrm{PRF})
}
$$

Physical Meaning:
這一步把原本疊在主頻帶裡的 folded replicas 攤開，讓主 replica 可以被單獨處理。程式裡用 `np.tile`，本質上就是 replica unfolding 的離散示意。

Why This Leads To The Next Figure:
攤開之後，主 replica 的 quadratic curvature 才能被 frequency deramp 單獨展平。

## 3. Frequency Deramping

![Step 3](./figures/explain_ufr3/ufr3_step3_freq_deramp.png)

Figure Caption:
這張圖對應 `$S_3(f_\eta)$`。主 replica 經過 deramp 後，quadratic curvature 被展平，因此在後續 pseudo-time domain 會更集中。

Mathematical Step:
對 mosaicked spectrum 乘上 reference quadratic phase 的共軛補償。

Fully Expanded Closed Form:

$$
{\color{red}
D_{\mathrm{de}}(f_\eta) =
\exp\left(
j\pi \frac{f_\eta^2}{k_s}
\right)
}
$$

$$
{\color{red}
S_3(f_\eta) = S_2(f_\eta)\,D_{\mathrm{de}}(f_\eta)
}
$$

Physical Meaning:
deramp 的真正作用不是改變 bandwidth，而是把主 replica 的 reference curvature 拿掉，讓主能量在下一步 pseudo-time domain 變成接近 central support 的集中能量。

Why This Leads To The Next Figure:
一旦主 replica 被展平，就可以用固定 support 的 LPF 只保留它，而把其餘 clones 丟掉。

## 4. Pseudo-Time LPF

![Step 4](./figures/explain_ufr3/ufr3_step4_pseudotime_lpf.png)

Figure Caption:
這張圖對應 `$S_4(f_\eta)$` 的形成過程。程式先把 `$S_3$` 轉到 pseudo-time domain，再用中央窗口保留主 clone。

Mathematical Step:
先沿 extended frequency axis 做 FFT，再對 pseudo-time support 做 central masking，最後回到 frequency domain。

Fully Expanded Closed Form:

$$
\widetilde{s}_3(\eta') = \mathcal{F}_{f_\eta}\left[S_3(f_\eta)\right]
$$

$$
\widetilde{s}_4(\eta') =
\widetilde{s}_3(\eta')\,
\mathrm{rect}\left(
\frac{\eta'}{T_{\mathrm{keep}}}
\right)
$$

$$
{\color{red}
S_4(f_\eta) =
\mathcal{F}^{-1}_{f_\eta}\left[
\widetilde{s}_4(\eta')
\right]
}
$$

Physical Meaning:
被保留的是與原始 burst support 對齊的主 clone，其餘 clones 在這裡被去掉。這一步就是 frequency-UFR 真正完成「只留下主 replica」的地方。

Why This Leads To The Next Figure:
LPF 後主 replica 已被孤立，但為了讓後續 matched filtering 有正確幾何，還必須把 reference curvature 補回來。

## 5. Frequency Reramping

![Step 5](./figures/explain_ufr3/ufr3_step5_freq_reramp.png)

Figure Caption:
這張圖對應 `$S_5(f_\eta)$`。經過 reramp 之後，主 replica 被補回可供 azimuth compression 使用的 reference curvature。

Mathematical Step:
對 `$S_4$` 乘回與 deramp 共軛的一個 quadratic phase。

Fully Expanded Closed Form:

$$
{\color{red}
D_{\mathrm{re}}(f_\eta) =
\exp\left(
-j\pi \frac{f_\eta^2}{k_s}
\right)
}
$$

$$
{\color{red}
S_5(f_\eta) = S_4(f_\eta)\,D_{\mathrm{re}}(f_\eta)
}
$$

Physical Meaning:
reramp 不是把 clones 放回來，而是只對已保留下來的主 replica 恢復 reference phase law，讓它重新變成可聚焦的 chirp。

Why This Leads To The Next Figure:
此時主 replica 已乾淨且幾何正確，因此可以進入 azimuth matched filtering。

## 6. Azimuth Compression

![Step 6](./figures/explain_ufr3/ufr3_step6_azimuth_compression.png)

Figure Caption:
這張圖對應 `$s_7(\eta)$`。目標已經被壓縮成 focused response，但 focused support 已超出原始 `T_{\mathrm{burst}}`，因此出現時間折返。

Mathematical Step:
對 `$S_5$` 乘上 azimuth matched filter，再回到時間域。

Fully Expanded Closed Form:

$$
{\color{red}
H_{\mathrm{az}}(f_\eta) =
\exp\left(
j\pi \frac{f_\eta^2}{k_a}
\right)
}
$$

$$
S_6(f_\eta) = S_5(f_\eta)H_{\mathrm{az}}(f_\eta)
$$

$$
{\color{red}
s_7(\eta) =
\mathcal{F}^{-1}_\eta\left[S_6(f_\eta)\right]
}
$$

Physical Meaning:
第一個 UFR 問題已經解掉，但第二個問題在這裡出現了：focus 後的有效時間支撐不再被原始 burst window 容納，因此 circular-convolution 邊界導致 time aliasing。

Why This Leads To The Next Figure:
既然時間域開始折返，就必須再做一次時間軸的 mosaicking，把 wrapped responses 攤開。

## 7. Time Mosaicking

![Step 7](./figures/explain_ufr3/ufr3_step7_time_mosaicking.png)

Figure Caption:
這張圖對應 `$s_{7,\mathrm{mosaic}}(\eta)$`。原本每個 `T_{\mathrm{burst}}` 會折返一次的 aliased timeline，被沿時間軸攤開 3 倍。

Mathematical Step:
對 `$s_7(\eta)$` 做時間域 mosaicking。

Fully Expanded Closed Form:

$$
{\color{red}
s_{7,\mathrm{mosaic}}(\eta) =
\sum_{n=-1}^{1}
s_7(\eta-nT_{\mathrm{burst}})
}
$$

Physical Meaning:
這一步把時間折返的 clones 展開，和前面頻域 mosaicking 的角色完全對應，只是展開的軸已經從 `$f_\eta$` 換成 `$\eta$`。

Why This Leads To The Next Figure:
攤開之後，主時間 clone 才能被 time deramp 展平。

## 8. Time Deramping

![Step 8](./figures/explain_ufr3/ufr3_step8_time_deramp.png)

Figure Caption:
這張圖對應 `$s_8(\eta)$`。主時間 clone 的 quadratic curvature 被拿掉，因此主能量會向中央可保留頻帶集中。

Mathematical Step:
先定義等效 time-domain chirp rate，再對時間域做 deramp。

Fully Expanded Closed Form:

$$
{\color{red}
k_t = \frac{k_ak_s}{k_a-k_s}
}
$$

$$
{\color{red}
s_8(\eta) =
s_{7,\mathrm{mosaic}}(\eta)\,
\exp\left(
-j\pi k_t\eta^2
\right)
}
$$

Physical Meaning:
這一步和 frequency deramp 的角色完全平行，只是現在被展平的是時間域中的 quadratic phase curvature。

Why This Leads To The Next Figure:
一旦主時間 clone 被展平，就可以在 frequency domain 用中央 PRF band 把它單獨保留下來。

## 9. Time LPF

![Step 9](./figures/explain_ufr3/ufr3_step9_time_lpf.png)

Figure Caption:
這張圖對應 `$s_{8,\mathrm{LPF}}$`。經過 time-domain LPF 後，只留下 central PRF band 對應的主時間 clone。

Mathematical Step:
先把 `$s_8$` 轉到 frequency domain，再做 central keep-band filtering，最後回到時間域。

Fully Expanded Closed Form:

$$
S_8(f_\eta) = \mathcal{F}_\eta\left[s_8(\eta)\right]
$$

$$
S_{8,\mathrm{LPF}}(f_\eta) =
S_8(f_\eta)\,
\mathrm{rect}\left(
\frac{f_\eta}{B_{\mathrm{keep,time}}}
\right)
$$

$$
{\color{red}
s_{8,\mathrm{unfolded}}(\eta) =
\mathcal{F}^{-1}_\eta\left[
S_{8,\mathrm{LPF}}(f_\eta)
\right]
}
$$

Physical Meaning:
這一步在時間-UFR 中扮演和 pseudo-time LPF 完全對應的角色，也就是只保留主時間 clone，去掉其餘時間折返。

Why This Leads To The Next Figure:
LPF 後主時間 clone 雖已孤立，但仍處於 deramped geometry，因此最後還要把 curvature 補回去。

## 10. Final Time Reramping

![Step 10](./figures/explain_ufr3/ufr3_step10_time_reramp.png)

Figure Caption:
這張圖對應最終輸出 `$s_{\mathrm{final}}(\eta)$`。時間 aliasing 已被解除，主 focused response 回到正確幾何。

Mathematical Step:
對 `$s_{8,\mathrm{unfolded}}$` 乘回 time reramp phase。

Fully Expanded Closed Form:

$$
{\color{red}
s_{\mathrm{final}}(\eta) =
s_{8,\mathrm{unfolded}}(\eta)\,
\exp\left(
j\pi k_t\eta^2
\right)
}
$$

Physical Meaning:
time reramp 並不是重新製造 aliasing，而是只對保留下來的主時間 clone 恢復正確相位幾何，因此最後得到 unaliased 的 focused response。

Why This Leads To The Next Figure:
這已經是最後一張圖，沒有下一步；它代表整個 TOPS frequency-UFR 與 time-UFR chain 的終點。

## Final Result

`explain_UFR3.py` 最重要的不是單一公式，而是整條圖與數學一一對應的鏈：

$$
\text{input aliasing}
\Longrightarrow
\text{frequency UFR}
\Longrightarrow
\text{azimuth compression}
\Longrightarrow
\text{time aliasing}
\Longrightarrow
\text{time UFR}
$$

因此這份文件的定位就是：當你看某一張圖時，立刻就在下面看到該圖對應的 fully expanded closed form 與物理解釋。
