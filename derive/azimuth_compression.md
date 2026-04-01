# Azimuth Compression

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Previous: [Azimuth Frequency UFR](./azimuth_freq_ufr.md)
- Next: [Azimuth Time UFR](./azimuth_time_ufr.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Input Signal](#1-input-signal)
- [2. Azimuth Matched Filter](#2-azimuth-matched-filter)
- [3. Azimuth-Compressed Output](#3-azimuth-compressed-output)
- [Physical Meaning](#physical-meaning)
- [Final Result](#final-result)

## Summary

- azimuth compression 的輸入是 frequency-UFR output $S_6(\tau,f_\eta)$。
- 這一步的目標是讓主 replica 在 azimuth time 上聚焦成窄主瓣。
- 若 FFT window 有限，時間域結果會以 multiple replicas 的形式表現出 wrap-around，這正是下一步 time-UFR 要處理的輸入。

## Problem Definition

本文件要把 $S_6(\tau,f_\eta)$ 經過 azimuth matched filtering 後，寫成 azimuth-compressed output $s_7(\tau,\eta)$ 的 fully expanded closed form。

## Derivation Highlights

- 先定義 azimuth matched filter。
- 再把主 replica 與週期延拓 replicas 的壓縮結果一起寫成顯式形式。
- 最後把 $s_7(\tau,\eta)$ 和後續 time-UFR 的關係講清楚。

## Symbols And Assumptions

- $S_6(\tau,f_\eta)$：frequency-UFR output
- $H_{\mathrm{az}}(f_\eta)$：azimuth matched filter
- $s_7(\tau,\eta)$：azimuth-compressed output
- $B_{\mathrm{az},m}$：第 $m$ 個 replica 的等效 azimuth bandwidth
- $\eta_{c,m}$：第 $m$ 個 compressed replica 的中心位置
- $\chi_m(\eta)$：第 $m$ 個時間域 replica 的 phase function

## 1. Input Signal

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

## 2. Azimuth Matched Filter

標準 reference matched filter 寫成

$$
{\color{red}
H_{\mathrm{az}}(f_\eta) =
\exp\left(
+j\phi_{\mathrm{az,ref}}(f_\eta)
\right)
}
$$

## 3. Azimuth-Compressed Output

將 $S_6(\tau,f_\eta)$ 乘上 $H_{\mathrm{az}}(f_\eta)$ 後做 inverse FFT，可得

$$
s_7(\tau,\eta) =
\mathcal{F}_{f_\eta}^{-1}\left[
S_6(\tau,f_\eta)\,
H_{\mathrm{az}}(f_\eta)
\right]
$$

其對應的 fully expanded closed form 可寫成

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

若只保留主 replica $m=m_0$，則

$$
{\color{red}
s_{7,\mathrm{main}}(\tau,\eta) \approx
A_{7,\mathrm{main}}\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c}
\right)
\right] \cdot
\mathrm{sinc}\left[
B_{\mathrm{az,main}}(\eta-\eta_c)
\right]
}
$$

## Physical Meaning

- 這一步把 frequency-UFR 後留下來的主 replica 聚焦成時間域主瓣
- 若 FFT window 足夠大，$s_7(\tau,\eta)$ 幾乎只剩主 replica
- 若 FFT window 不足，則時間域仍會以 $\eta_{c,m}$ 為中心出現 wrap-around replicas

## Final Result

$$
{\color{red}
S_6(\tau,f_\eta)
\xrightarrow{\text{azimuth compression}}
s_7(\tau,\eta)
}
$$
