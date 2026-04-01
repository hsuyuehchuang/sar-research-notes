# Azimuth Compression

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Previous processing step: [Azimuth Frequency UFR](./azimuth_freq_ufr.md)
- Next phenomenon: [Azimuth Time Folding](./azimuth_time_folding.md)
- Next: [Azimuth Time UFR](./azimuth_time_ufr.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Input Signal](#1-input-signal)
- [2. Azimuth Matched Filter](#2-azimuth-matched-filter)
- [3. Paper Parameterization Of The Reference Chirp](#3-paper-parameterization-of-the-reference-chirp)
- [4. Azimuth-Compressed Output](#4-azimuth-compressed-output)
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
- 再用 paper notation 的 $k_a$、$k_{rot}$、$k_t$ 重寫 reference chirp 參數。
- 再把 $S_6(\tau,f_\eta)H_{\mathrm{az}}(f_\eta)$ 寫成 fully expanded closed form。
- 再把主 replica 與週期延拓 replicas 的壓縮結果一起寫成顯式形式。
- 最後把 $s_7(\tau,\eta)$ 和後續 time-UFR 的關係講清楚。

## Symbols And Assumptions

- $S_6(\tau,f_\eta)$：frequency-UFR output
- $H_{\mathrm{az}}(f_\eta)$：azimuth matched filter
- $\phi_{\mathrm{az},m}(f_\eta)$：第 $m$ 個 replica 的 azimuth phase law
- $\phi_{\mathrm{az,ref}}(f_\eta)$：reference azimuth phase law
- $f_{\mathrm{DC}}$：reference Doppler centroid
- $k_a$：平台幾何造成的 azimuth FM rate
- $k_{rot}$：beam rotation 引入的 Doppler-rate term
- $\alpha$：TOPS stretch factor
- $k_t$：paper notation 中的 effective TOPS chirp rate
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

若在 frequency-UFR 結束後，主 replica 已被 reramp 回 reference curvature，則其 reference azimuth phase law 可局部寫成

$$
{\color{red}
\phi_{\mathrm{az,ref}}(f_\eta) =
\phi_{\mathrm{az},0}
+\phi_{\mathrm{az},1}(f_\eta-f_{\mathrm{ref}})
+\phi_{\mathrm{az},2}(f_\eta-f_{\mathrm{ref}})^2
}
$$

---

## 3. Paper Parameterization Of The Reference Chirp

若改用 TOPSAR paper 常見的參數化，則 reference azimuth chirp 可以用 effective chirp rate $k_t$ 來表達。

先回顧 carried-forward relation：

$$
k_a \approx -\frac{2v_p^2}{\lambda R_0},
\qquad
k_{rot} \approx \frac{2v_p\omega_{rot}}{\lambda}
$$

以及

$$
{\color{red}
\alpha = 1-\frac{k_{rot}}{k_a},
\qquad
k_t = \frac{k_{rot}}{\alpha} =
\frac{k_a k_{rot}}{k_a-k_{rot}}
}
$$

在 local quadratic-FM approximation 下，reference replica 的 azimuth-time phase 可寫成

$$
{\color{red}
\phi_{\mathrm{ref}}(\eta) \approx
2\pi f_{\mathrm{DC}}(\eta-\eta_c)
-\pi k_t(\eta-\eta_c)^2
}
$$

對應地，其 frequency-domain matched filter 可寫成

$$
{\color{red}
H_{\mathrm{az}}(f_\eta) \approx
\exp\left(
+j\pi\frac{(f_\eta-f_{\mathrm{DC}})^2}{k_t}
\right)
}
$$

這裡省略了與聚焦主瓣位置無關的常數幅度與常數相位。重要的是：TOPS azimuth compression filter 的 reference curvature，現在可以直接由 $k_t$ 來記憶，而 $k_t$ 又由 $k_a$ 與 $k_{rot}$ 組合而成。

因此每個 replica 在 matched filtering 前的 local phase 可寫成

$$
{\color{red}
\phi_{\mathrm{az},m}(f_\eta) =
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
}
$$

將 $S_6(\tau,f_\eta)$ 乘上 $H_{\mathrm{az}}(f_\eta)$ 之後，可得壓縮前的 fully expanded filtered spectrum

$$
{\color{red}
S_{6,\mathrm{mf}}(\tau,f_\eta) =
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
\phi_{\mathrm{az},m}(f_\eta)-\phi_{\mathrm{az,ref}}(f_\eta)
\right]
\right)
}
$$

若將 phase mismatch 也顯式展開，則

$$
{\color{red}
S_{6,\mathrm{mf}}(\tau,f_\eta) =
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
\Delta\phi_{0,m}
+\Delta\phi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\Delta\phi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
}
$$

其中

$$
\Delta\phi_{0,m} = \psi_{0,m}-\phi_{\mathrm{az},0}
$$

$$
\Delta\phi_{1,m} = \psi_{1,m}-\phi_{\mathrm{az},1}
$$

$$
\Delta\phi_{2,m} = \psi_{2,m}-\phi_{\mathrm{az},2}
$$

## 4. Azimuth-Compressed Output

將 filtered spectrum $S_{6,\mathrm{mf}}(\tau,f_\eta)$ 做 inverse FFT，可得

$$
s_7(\tau,\eta) =
\mathcal{F}_{f_\eta}^{-1}\left[
S_{6,\mathrm{mf}}(\tau,f_\eta)
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
-j\left[
\chi_{0,m}
+\chi_{1,m}(\eta-\eta_{c,m})
+\chi_{2,m}(\eta-\eta_{c,m})^2
\right]
\right)
}
$$

其中主 replica 若滿足 matched-filter 完全對準，則

$$
\Delta\phi_{0,m_0} \approx 0,\qquad
\Delta\phi_{1,m_0} \approx 0,\qquad
\Delta\phi_{2,m_0} \approx 0
$$

因此其壓縮結果退化為窄主瓣；而其他 replicas 若仍保有 residual phase mismatch，則會表現在 $\chi_{1,m}$ 與 $\chi_{2,m}$ 所描述的位置偏移與殘留二次相位上。

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
- azimuth compression filter 的 closed form 本質上就是 reference phase 的共軛，而在 paper notation 下它可由 $k_t$ 直接參數化
- `matched filtering -> inverse FFT` 之後，頻域的 quadratic phase 會轉成時間域主瓣與殘留 mismatch 項
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
