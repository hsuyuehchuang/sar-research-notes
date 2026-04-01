# Azimuth Frequency UFR

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Previous: [Range Compression](./range_compression.md)
- Next: [Azimuth Compression](./azimuth_compression.md)
- Support notes:
  - [Azimuth Frequency Folding](./azimuth_freq_folding.md)
  - [Frequency-Time Deramping](./freq_time_deramping.md)
  - [Azimuth Deramp LPF](./azimuth_deramp_LPF.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Input Signal](#1-input-signal)
- [2. Azimuth Frequency Folding](#2-azimuth-frequency-folding)
- [3. Mosaicking](#3-mosaicking)
- [4. Deramping](#4-deramping)
- [5. Low Pass Filter](#5-low-pass-filter)
- [6. Reramping](#6-reramping)
- [Physical Meaning](#physical-meaning)
- [Final Result](#final-result)

## Summary

- azimuth frequency UFR 的完整流程是 `folding -> mosaicking -> deramping -> LPF -> reramping`。
- folded spectrum 的直接來源是 slow-time sampling；mosaicking 只把 replicas 攤到 extended $f_\eta$ axis。
- 主 replica 真正變得可被固定通帶保留，是因為 deramping 先把 reference curvature 拿掉。
- reramping 不能省略，因為後續 azimuth compression 需要主 replica 回到標準 reference phase law。

## Problem Definition

本文件要從 range-compressed input $s_1(\tau,\eta)$ 出發，一路推到 frequency-UFR output $S_6(\tau,f_\eta)$，並對每個中間 stage 都寫出 fully expanded closed form。

## Derivation Highlights

- 先把連續 azimuth spectrum 與 folded spectrum 分開。
- 再把 folded replicas 重排成 extended-frequency mosaicked signal。
- 接著以主 replica 的二次 curvature 建立 deramping filter。
- 然後用固定頻寬窗口做 LPF。
- 最後用 reramping 把保留下來的主 replica 補回 reference curvature。

## Symbols And Assumptions

- $s_1(\tau,\eta)$：range-compressed input
- $S_2(\tau,f_\eta)$：folded azimuth-frequency signal
- $S_3(\tau,f_\eta)$：mosaicked signal
- $S_4(\tau,f_\eta)$：deramped signal
- $S_5(\tau,f_\eta)$：LPF output
- $S_6(\tau,f_\eta)$：reramped output
- $W_a(f_\eta;\omega_s)$：continuous azimuth envelope
- $D_m(f_\eta)=D(f_\eta-m\cdot\mathrm{PRF},V_r)$：第 $m$ 個 replica 的幾何因子
- $\psi_{0,m},\psi_{1,m},\psi_{2,m}$：第 $m$ 個 replica 的局部 phase 係數
- $\psi_{2,\mathrm{ref}}$：主 replica 的 reference quadratic curvature

## 1. Input Signal

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

## 2. Azimuth Frequency Folding

連續 azimuth spectrum 為

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

因此 folded signal 的 fully expanded closed form 為

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

## 3. Mosaicking

將 replicas 攤到 extended-frequency axis 後，

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

## 4. Deramping

frequency-domain deramping filter 為

$$
H_{\mathrm{de},f}(f_\eta) =
\exp\left(
+j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

乘上後得到

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

## 5. Low Pass Filter

frequency-domain LPF 為

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

## 6. Reramping

frequency-domain reramping filter 為

$$
H_{\mathrm{re},f}(f_\eta) =
\exp\left(
-j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

因此 frequency-UFR output 的 fully expanded closed form 為

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

## Physical Meaning

- $S_2$：folded replicas 還疊在 PRF 週期內
- $S_3$：replicas 被攤開，但主 replica 還是彎的
- $S_4$：主 replica 的 curvature 被拿掉
- $S_5$：固定通帶只保留已展平的主 replica
- $S_6$：主 replica 被補回正確 reference phase，供後續 azimuth compression 使用

## Final Result

$$
{\color{red}
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
}
$$
