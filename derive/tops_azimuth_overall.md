# TOPS SAR Azimuth Overall

## Navigation

- [Flowchart](#flowchart)
- [Reading Order](#reading-order)
- [Table of Contents](#table-of-contents)
- Stage notes:
  - [Range Compression](./range_compression.md)
  - [Azimuth Frequency UFR](./azimuth_freq_ufr.md)
  - [Azimuth Compression](./azimuth_compression.md)
  - [Azimuth Time UFR](./azimuth_time_ufr.md)

## Flowchart

- [Raw Data](#1-raw-data)
- [Range Compression](#2-range-compression)
- [Azimuth Frequency Unfolding And Resampling (UFR)](#3-azimuth-frequency-unfolding-and-resampling-ufr)
  - [Azimuth Frequency Folding](#31-azimuth-frequency-folding)
  - [Mosaicking](#32-mosaicking)
  - [Deramping](#33-deramping)
  - [Low Pass Filter](#34-low-pass-filter)
  - [Reramping](#35-reramping)
- [Azimuth Compression](#4-azimuth-compression)
- [Azimuth Time Unfolding And Resampling (UFR)](#5-azimuth-time-unfolding-and-resampling-ufr)
  - [Mosaicking](#51-mosaicking)
  - [Deramping](#52-deramping)
  - [Low Pass Filter](#53-low-pass-filter)
  - [Reramping](#54-reramping)
- [Focused Image](#6-focused-image)

## Reading Order

1. [Range Compression](./range_compression.md)
2. [Azimuth Frequency Folding](./azimuth_freq_folding.md)
3. [Azimuth Frequency UFR](./azimuth_freq_ufr.md)
4. [Azimuth Compression](./azimuth_compression.md)
5. [Azimuth Time Folding](./azimuth_time_folding.md)
6. [Azimuth Time UFR](./azimuth_time_ufr.md)
7. Support derivations:
   [Frequency-Time Deramping](./freq_time_deramping.md),
   [Azimuth Deramp LPF](./azimuth_deramp_LPF.md)

## Table of Contents

- [Summary](#summary)
- [Signal Definitions](#signal-definitions)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Raw Data](#1-raw-data)
- [2. Range Compression](#2-range-compression)
- [3. Azimuth Frequency Unfolding And Resampling (UFR)](#3-azimuth-frequency-unfolding-and-resampling-ufr)
- [4. Azimuth Compression](#4-azimuth-compression)
- [5. Azimuth Time Unfolding And Resampling (UFR)](#5-azimuth-time-unfolding-and-resampling-ufr)
- [6. Focused Image](#6-focused-image)
- [Physical Meaning](#physical-meaning)
- [Final Result](#final-result)

## Summary

- 這份文件把 TOPS SAR 的完整 azimuth chain 從 raw data 一路推到 focused image。
- 這條主線必須分成 `range compression -> azimuth frequency UFR -> azimuth compression -> azimuth time UFR`，否則 folded spectrum、deramp-LPF、time wrap-around 會被混成同一件事。
- 這裡的規則是：每一個 stage signal 都必須寫成自己的 fully expanded closed form，不能只用操作符代替。
- 對應的主鏈訊號依序為 $s_0(\tau,\eta)$、$s_1(\tau,\eta)$、$S_2(\tau,f_\eta)$、$S_3(\tau,f_\eta)$、$S_4(\tau,f_\eta)$、$S_5(\tau,f_\eta)$、$S_6(\tau,f_\eta)$、$s_7(\tau,\eta)$、$I_8(\tau,\eta)$、$I_9(\tau,\eta)$、$I_{10}(\tau,\eta)$、$I_{11}(\tau,\eta)$ 與 $I_{\mathrm{focus}}(\tau,\eta)$。

摘要中最重要的關鍵公式為

$$
{\color{red}
S_6(\tau,f_\eta) \approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_6\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right) \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
}
$$

以及

$$
{\color{red}
I_{\mathrm{focus}}(\tau,\eta) \approx
A_f\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c}
\right)
\right] \cdot
\mathrm{sinc}\left[
B_{\mathrm{az,keep}}(\eta-\eta_c)
\right]
}
$$

## Signal Definitions

- $s_0(\tau,\eta)$：raw data
- $s_1(\tau,\eta)$：range-compressed data
- $S_2(\tau,f_\eta)$：folded azimuth-frequency signal
- $S_3(\tau,f_\eta)$：mosaicked azimuth-frequency signal
- $S_4(\tau,f_\eta)$：frequency-domain deramped signal
- $S_5(\tau,f_\eta)$：frequency-domain LPF output
- $S_6(\tau,f_\eta)$：frequency-domain reramped output
- $s_7(\tau,\eta)$：azimuth-compressed output
- $I_8(\tau,\eta)$：mosaicked azimuth-time signal
- $I_9(\tau,\eta)$：time-domain deramped signal
- $I_{10}(\tau,\eta)$：time-domain LPF output
- $I_{11}(\tau,\eta)$：time-domain reramped output
- $I_{\mathrm{focus}}(\tau,\eta)$：focused image

## Problem Definition

本文件要把 TOPS SAR 的方位向處理鏈完整寫成一條顯式數學主線，並且回答三件事：

1. raw data 經過哪幾個 stage 之後才會變成 focused image。
2. 兩次 UFR 為什麼都必須經過 `mosaicking -> deramping -> LPF -> reramping`。
3. 為什麼每個 stage 都必須單獨保留 fully expanded closed form。

## Derivation Highlights

- 先把 raw range chirp 壓縮成距離向 $\mathrm{sinc}$，得到 $s_1(\tau,\eta)$。
- 再把 slow-time sampling 導致的 azimuth frequency folding 顯式寫成 $S_2(\tau,f_\eta)$。
- 接著在 frequency-UFR 中把 folded replicas 攤開、展平、裁切、再補回 reference curvature，得到 $S_6(\tau,f_\eta)$。
- 然後用 azimuth matched filter 把主 replica 聚焦成 $s_7(\tau,\eta)$。
- 最後處理 finite-FFT 導致的 time-domain wrap-around，得到最終 $I_{\mathrm{focus}}(\tau,\eta)$。

## Symbols And Assumptions

- $\tau$：range fast time
- $\eta$：azimuth slow time
- $f_\eta$：azimuth frequency
- $R(\eta)=\sqrt{R_0^2+V_r^2(\eta-\eta_0)^2}$：瞬時斜距
- $K_r$：range chirp rate
- $B_r$：range bandwidth
- $T_r$：range pulse duration
- $T_p=1/\mathrm{PRF}$：slow-time sampling interval
- $w_a(\eta;\omega_s)$：TOPS azimuth illumination
- $W_a(f_\eta;\omega_s)$：其 azimuth-frequency envelope
- $\psi_m(f_\eta)$：frequency-UFR 中第 $m$ 個 replica 的 phase
- $\chi_m(\eta)$：time-UFR 中第 $m$ 個 replica 的 phase
- $\psi_{2,\mathrm{ref}}$：frequency-UFR reference curvature
- $\chi_{2,\mathrm{ref}}$：time-UFR reference curvature
- $B_{\mathrm{LPF}}$：frequency-UFR keep band
- $T_{\mathrm{LPF}}$：time-UFR keep window
- $B_{\mathrm{az,keep}}$：最終保留的 azimuth effective bandwidth

假設如下：

- range compression 已完成 standard matched filtering。
- azimuth 頻域與時間域的 replica 都只在局部通帶內用二次 phase 近似。
- 主 replica 的設計目標是被保留並回復成 reference phase law；其他 replicas 只作為干擾項處理。

## 1. Raw Data

raw TOPS burst data 的 fully expanded closed form 為

$$
{\color{red}
s_0(\tau,\eta) =
A_0\,
\mathrm{rect}\left(
\frac{\tau-\frac{2R(\eta)}{c}}{T_r}
\right) \cdot
w_a(\eta;\omega_s) \cdot
\exp\left(
+j\pi K_r\left(
\tau-\frac{2R(\eta)}{c}
\right)^2
\right) \cdot
\exp\left(
-j\frac{4\pi f_0R(\eta)}{c}
\right) \cdot
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
$$

## 2. Range Compression

range matched filter 為

$$
h_r(\tau) =
\exp\left(
-j\pi K_r\tau^2
\right)
$$

與之對應的 range-compressed output 的 fully expanded closed form 為

$$
{\color{red}
s_1(\tau,\eta) =
A_1\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R(\eta)}{c}
\right)
\right] \cdot
w_a(\eta;\omega_s) \cdot
\exp\left(
-j\frac{4\pi f_0R(\eta)}{c}
\right) \cdot
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
$$

## 3. Azimuth Frequency Unfolding And Resampling (UFR)

這一段主流程的前置現象推導是 [Azimuth Frequency Folding](./azimuth_freq_folding.md)。也就是先證明 folded spectrum 為什麼會出現，再進入 `mosaicking -> deramping -> LPF -> reramping` 的處理鏈。

### 3.1. Azimuth Frequency Folding

連續 azimuth spectrum 先寫成

$$
S_{1,c}(\tau,f_\eta;\omega_s) =
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta,V_r)}
\right)
\right] \cdot
W_a(f_\eta;\omega_s) \cdot
\exp\left(
-j\frac{4\pi R_0f_0}{c}D(f_\eta,V_r) - j2\pi f_\eta\eta_0
\right)
$$

因此 folded azimuth-frequency signal 的 fully expanded closed form 為

$$
{\color{red}
S_2(\tau,f_\eta) =
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta-k\cdot\mathrm{PRF},V_r)}
\right)
\right] \cdot
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s) \cdot
\exp\left(
-j\frac{4\pi R_0f_0}{c}D(f_\eta-k\cdot\mathrm{PRF},V_r) - j2\pi(f_\eta-k\cdot\mathrm{PRF})\eta_0
\right)
}
$$

### 3.2. Mosaicking

把 folded replicas 攤到 extended axis 後，可寫成

$$
{\color{red}
S_3(\tau,f_\eta) =
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_3\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
}
$$

### 3.3. Deramping

frequency-domain deramping filter 為

$$
H_{\mathrm{de},f}(f_\eta) =
\exp\left(
+j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

因此 deramped signal 的 fully expanded closed form 為

$$
{\color{red}
S_4(\tau,f_\eta) =
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_4\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
}
$$

### 3.4. Low Pass Filter

frequency-domain keep window 為

$$
H_{\mathrm{LPF},f}(f_\eta) =
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

因此 LPF output 的 fully expanded closed form 為

$$
{\color{red}
S_5(\tau,f_\eta) =
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_5\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right) \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
}
$$

### 3.5. Reramping

frequency-domain reramping filter 為

$$
H_{\mathrm{re},f}(f_\eta) =
\exp\left(
-j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

因此 reramped output 的 fully expanded closed form 為

$$
{\color{red}
S_6(\tau,f_\eta) =
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_6\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right) \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
}
$$

## 4. Azimuth Compression

這一步的輸入來自 [Azimuth Frequency UFR](./azimuth_freq_ufr.md)，而它的輸出 wrap-around 現象則由 [Azimuth Time Folding](./azimuth_time_folding.md) 進一步解釋。

令 azimuth matched filter 為

$$
H_{\mathrm{az}}(f_\eta) =
\exp\left(
+j\phi_{\mathrm{az,ref}}(f_\eta)
\right)
$$

則在主 replica 已回到標準 reference curvature 的條件下，azimuth-compressed output 的 fully expanded closed form 可寫成

$$
{\color{red}
s_7(\tau,\eta) \approx
\sum_{m=-N_{t,\mathrm{neg}}}^{N_{t,\mathrm{pos}}}
A_7\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c}
\right)
\right] \cdot
\mathrm{sinc}\left[
B_{\mathrm{az},m}\left(
\eta-\eta_{c,m}
\right)
\right] \cdot
\exp\left(
-j\chi_m(\eta)
\right)
}
$$

這裡 $\eta_{c,m}$ 表示因 circular convolution 或窗口週期延拓而出現的時間 replica center。

## 5. Azimuth Time Unfolding And Resampling (UFR)

這一段主流程的前置現象推導是 [Azimuth Time Folding](./azimuth_time_folding.md)。也就是先證明 finite-FFT 為什麼會把線性卷積折回成 time-domain wrap-around，再進入 `mosaicking -> deramping -> LPF -> reramping` 的處理鏈。

### 5.1. Mosaicking

把 azimuth-time replicas 攤到 extended time axis 後，可寫成

$$
{\color{red}
I_8(\tau,\eta) =
\sum_{m=-N_{t,\mathrm{neg}}}^{N_{t,\mathrm{pos}}}
A_8\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{\eta-mT_{\mathrm{window}}-\eta_c}{T_{\mathrm{keep}}}
\right) \cdot
\mathrm{sinc}\left[
B_{\mathrm{az},m}\left(
\eta-mT_{\mathrm{window}}-\eta_c
\right)
\right] \cdot
\exp\left(
-j\left[
\chi_{0,m}
+\chi_{1,m}(\eta-\eta_{\mathrm{ref}})
+\chi_{2,m}(\eta-\eta_{\mathrm{ref}})^2
\right]
\right)
}
$$

### 5.2. Deramping

time-domain deramping filter 為

$$
H_{\mathrm{de},t}(\eta) =
\exp\left(
+j\chi_{2,\mathrm{ref}}(\eta-\eta_{\mathrm{ref}})^2
\right)
$$

因此 time-domain deramped signal 的 fully expanded closed form 為

$$
{\color{red}
I_9(\tau,\eta) =
\sum_{m=-N_{t,\mathrm{neg}}}^{N_{t,\mathrm{pos}}}
A_9\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{\eta-mT_{\mathrm{window}}-\eta_c}{T_{\mathrm{keep}}}
\right) \cdot
\mathrm{sinc}\left[
B_{\mathrm{az},m}\left(
\eta-mT_{\mathrm{window}}-\eta_c
\right)
\right] \cdot
\exp\left(
-j\left[
\chi_{0,m}
+\chi_{1,m}(\eta-\eta_{\mathrm{ref}})
+\left(
\chi_{2,m}-\chi_{2,\mathrm{ref}}
\right)(\eta-\eta_{\mathrm{ref}})^2
\right]
\right)
}
$$

### 5.3. Low Pass Filter

time-domain keep window 為

$$
H_{\mathrm{LPF},t}(\eta) =
\mathrm{rect}\left(
\frac{\eta-\eta_{\mathrm{LPF}}}{T_{\mathrm{LPF}}}
\right)
$$

因此 LPF output 的 fully expanded closed form 為

$$
{\color{red}
I_{10}(\tau,\eta) =
\sum_{m=-N_{t,\mathrm{neg}}}^{N_{t,\mathrm{pos}}}
A_{10}\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{\eta-\eta_{\mathrm{LPF}}}{T_{\mathrm{LPF}}}
\right) \cdot
\mathrm{rect}\left(
\frac{\eta-mT_{\mathrm{window}}-\eta_c}{T_{\mathrm{keep}}}
\right) \cdot
\mathrm{sinc}\left[
B_{\mathrm{az},m}\left(
\eta-mT_{\mathrm{window}}-\eta_c
\right)
\right] \cdot
\exp\left(
-j\left[
\chi_{0,m}
+\chi_{1,m}(\eta-\eta_{\mathrm{ref}})
+\left(
\chi_{2,m}-\chi_{2,\mathrm{ref}}
\right)(\eta-\eta_{\mathrm{ref}})^2
\right]
\right)
}
$$

### 5.4. Reramping

time-domain reramping filter 為

$$
H_{\mathrm{re},t}(\eta) =
\exp\left(
-j\chi_{2,\mathrm{ref}}(\eta-\eta_{\mathrm{ref}})^2
\right)
$$

因此 reramped output 的 fully expanded closed form 為

$$
{\color{red}
I_{11}(\tau,\eta) =
\sum_{m=-N_{t,\mathrm{neg}}}^{N_{t,\mathrm{pos}}}
A_{11}\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{\eta-\eta_{\mathrm{LPF}}}{T_{\mathrm{LPF}}}
\right) \cdot
\mathrm{rect}\left(
\frac{\eta-mT_{\mathrm{window}}-\eta_c}{T_{\mathrm{keep}}}
\right) \cdot
\mathrm{sinc}\left[
B_{\mathrm{az},m}\left(
\eta-mT_{\mathrm{window}}-\eta_c
\right)
\right] \cdot
\exp\left(
-j\left[
\chi_{0,m}
+\chi_{1,m}(\eta-\eta_{\mathrm{ref}})
+\chi_{2,m}(\eta-\eta_{\mathrm{ref}})^2
\right]
\right)
}
$$

## 6. Focused Image

若只保留主 replica $m=m_0$，則最終 focused image 的 fully expanded closed form 為

$$
{\color{red}
I_{\mathrm{focus}}(\tau,\eta) \approx
A_f\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c}
\right)
\right] \cdot
\mathrm{sinc}\left[
B_{\mathrm{az,keep}}(\eta-\eta_c)
\right]
}
$$

## Physical Meaning

- $s_0 \rightarrow s_1$：把 raw range chirp 變成 range-compressed pulse。
- $s_1 \rightarrow S_6$：把 azimuth frequency folded replicas 攤開、展平、裁切，再補回 reference curvature。
- $S_6 \rightarrow s_7$：做主聚焦。
- $s_7 \rightarrow I_{11}$：把壓縮後 time-domain 的 replicas 再做一次 UFR。
- $I_{11} \rightarrow I_{\mathrm{focus}}$：保留主 replica，得到 focused image。

## Final Result

$$
{\color{red}
s_0(\tau,\eta)
\rightarrow
s_1(\tau,\eta)
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
