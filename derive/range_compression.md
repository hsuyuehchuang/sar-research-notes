# Range Compression

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Next: [Azimuth Frequency UFR](./azimuth_freq_ufr.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Raw Echo Model](#1-raw-echo-model)
- [2. Range Matched Filter](#2-range-matched-filter)
- [3. Range-Compressed Signal](#3-range-compressed-signal)
- [Physical Meaning](#physical-meaning)
- [Final Result](#final-result)

## Summary

- range compression 把 raw fast-time chirp 轉成距離向的壓縮脈衝。
- 對後續 TOPS azimuth chain 而言，真正的輸入不是 raw data，而是 range-compressed output $s_1(\tau,\eta)$。
- 這一步之後，range envelope 變成 $\mathrm{sinc}$，但 azimuth illumination 與 azimuth phase 都被完整保留下來。

## Problem Definition

本文件要從 raw data $s_0(\tau,\eta)$ 出發，推到 range-compressed signal $s_1(\tau,\eta)$，並且把輸入與輸出都寫成 fully expanded closed form。

## Derivation Highlights

- 先把瞬時斜距 $R(\eta)$ 與 raw chirp model 寫清楚。
- 再定義 range matched filter $h_r(\tau)$。
- 最後把卷積結果寫成 explicit 的 $\mathrm{sinc}$ 形式，而不是只停在 operator form。

## Symbols And Assumptions

- $\tau$：range fast time
- $\eta$：azimuth slow time
- $R(\eta)=\sqrt{R_0^2+V_r^2(\eta-\eta_0)^2}$：瞬時斜距
- $K_r$：range chirp rate
- $T_r$：range pulse duration
- $B_r$：range bandwidth
- $w_a(\eta;\omega_s)$：TOPS azimuth illumination
- $T_p=1/\mathrm{PRF}$：slow-time sampling interval

## 1. Raw Echo Model

單點目標的 raw echo fully expanded closed form 為

$$
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
$$

## 2. Range Matched Filter

range matched filter 為

$$
h_r(\tau) =
\exp\left(
-j\pi K_r\tau^2
\right)
$$

## 3. Range-Compressed Signal

卷積結果

$$
s_1(\tau,\eta) =
s_0(\tau,\eta) *_\tau h_r(\tau)
$$

其 fully expanded closed form 為

$$
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
$$

## Physical Meaning

- raw range chirp 被壓縮成一個距離向主瓣與旁瓣結構
- TOPS 的 azimuth illumination $w_a(\eta;\omega_s)$ 沒有被這一步改變
- 後續 azimuth folding 與 UFR 都是從這個 $s_1(\tau,\eta)$ 開始

## Final Result

$$
s_0(\tau,\eta)
\xrightarrow{\text{range compression}}
s_1(\tau,\eta)
$$
