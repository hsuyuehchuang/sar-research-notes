# Azimuth Time UFR

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Previous: [Azimuth Compression](./azimuth_compression.md)
- Support note: [Azimuth Time Folding](./azimuth_time_folding.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Input Signal](#1-input-signal)
- [2. Mosaicking](#2-mosaicking)
- [3. Deramping](#3-deramping)
- [4. Low Pass Filter](#4-low-pass-filter)
- [5. Reramping](#5-reramping)
- [6. Focused Output](#6-focused-output)
- [Physical Meaning](#physical-meaning)
- [Final Result](#final-result)

## Summary

- azimuth time UFR 的輸入是 azimuth-compressed output $s_7(\tau,\eta)$。
- 這一段和 frequency-UFR 同型，也是 `mosaicking -> deramping -> LPF -> reramping`。
- 差別只在於 replicas 現在出現在 azimuth time，而不是 azimuth frequency。
- 最終 focused image 是在 time-UFR 保留主 replica 並補回 reference curvature 之後得到的。

## Problem Definition

本文件要把 azimuth-compressed signal 中的 time-domain replicas 展開、展平、裁切、補回，最後得到 $I_{\mathrm{focus}}(\tau,\eta)$ 的 fully expanded closed form。

## Derivation Highlights

- 先把壓縮後的 time replicas 顯式寫成 $s_7(\tau,\eta)$。
- 再把它重排到 extended azimuth-time axis，得到 $I_8(\tau,\eta)$。
- 接著用 reference time curvature 做 deramping，得到 $I_9(\tau,\eta)$。
- 再用理想 keep-window 模型定義 LPF，並以 FFT-based zeroing 實作 sharp-cut filtering，得到 $I_{10}(\tau,\eta)$。
- 最後 reramp，得到 $I_{11}(\tau,\eta)$ 與 focused image。

## Symbols And Assumptions

- $s_7(\tau,\eta)$：azimuth-compressed input
- $I_8(\tau,\eta)$：mosaicked time-domain signal
- $I_9(\tau,\eta)$：deramped time-domain signal
- $I_{10}(\tau,\eta)$：LPF output
- $I_{11}(\tau,\eta)$：reramped output
- $I_{\mathrm{focus}}(\tau,\eta)$：final focused image
- $\chi_{0,m},\chi_{1,m},\chi_{2,m}$：第 $m$ 個時間 replica 的局部 phase 係數
- $\chi_{2,\mathrm{ref}}$：主 replica 的 reference curvature
- $T_{\mathrm{window}}$：FFT 對應的基本時間窗口
- $T_{\mathrm{keep}}$：單一 time replica 的保持窗口
- $T_{\mathrm{LPF}}$：time-domain keep window
- $\widetilde{I}_9(\tau,\nu_\eta)$：沿目前離散處理軸對 $I_9(\tau,\eta)$ 做 FFT 後的表示
- $\widetilde{M}_{\mathrm{LPF}}(\nu_\eta)$：FFT-domain keep mask

## 1. Input Signal

azimuth-compressed input 的 fully expanded closed form 為

$$
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
$$

## 2. Mosaicking

將 time replicas 攤到 extended azimuth-time axis 後，得到

$$
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
$$

## 3. Deramping

time-domain deramping filter 為

$$
H_{\mathrm{de},t}(\eta) =
\exp\left(
+j\chi_{2,\mathrm{ref}}(\eta-\eta_{\mathrm{ref}})^2
\right)
$$

因此 deramped signal 的 fully expanded closed form 為

$$
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
$$

## 4. Low Pass Filter

理想 time-domain keep window 為

$$
H_{\mathrm{LPF},t}(\eta) =
\mathrm{rect}\left(
\frac{\eta-\eta_{\mathrm{LPF}}}{T_{\mathrm{LPF}}}
\right)
$$

在數學上，上一式定義了要保留的主 replica support。

但在實作上，sharp-cut filtering 採用 FFT-based 方式完成。若以目前 time-UFR 的離散處理軸仍記為 $\eta$，則

$$
\widetilde{I}_9(\tau,\nu_\eta) =
\mathcal{F}_{\eta}\left[
I_9(\tau,\eta)
\right]
$$

$$
\widetilde{I}_{10}(\tau,\nu_\eta) =
\widetilde{I}_9(\tau,\nu_\eta)\,
\widetilde{M}_{\mathrm{LPF}}(\nu_\eta)
$$

$$
I_{10}(\tau,\eta) =
\mathcal{F}_{\eta}^{-1}\left[
\widetilde{I}_9(\tau,\nu_\eta)\,
\widetilde{M}_{\mathrm{LPF}}(\nu_\eta)
\right]
$$

因此這一步也要分成兩層：

- 理想模型：`rect` keep window
- 實作方法：Forward FFT -> zero out unwanted bins -> Inverse FFT

LPF 完成後，再進入 resampling；resampling 不是 LPF 本身的一部分。

因此 LPF output 的 fully expanded closed form 為

$$
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
$$

## 5. Reramping

time-domain reramping filter 為

$$
H_{\mathrm{re},t}(\eta) =
\exp\left(
-j\chi_{2,\mathrm{ref}}(\eta-\eta_{\mathrm{ref}})^2
\right)
$$

因此 reramped output 的 fully expanded closed form 為

$$
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
$$

## 6. Focused Output

若只保留主 replica $m=m_0$，則 focused image 的 fully expanded closed form 為

$$
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
$$

## Physical Meaning

- $I_8$：所有時間 replicas 都被攤到 extended time axis
- $I_9$：主 replica 被展平
- $I_{10}$：固定 keep window 保留主 replica，而實作上對應 FFT-domain zeroing 後再 inverse FFT 的結果
- $I_{11}$：主 replica 被補回 reference curvature
- $I_{\mathrm{focus}}$：只保留主 replica 的最終 focused image

## Final Result

$$
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
$$
