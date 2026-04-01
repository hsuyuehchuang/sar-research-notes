# Explain UFR4: Focused-Time Inflation In TOPS Vs Stripmap

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Companion note: [Explain UFR3](./explain_ufr3.md)
- Related notes:
  - [Azimuth Compression](./azimuth_compression.md)
  - [Azimuth Time Folding](./azimuth_time_folding.md)
  - [Azimuth Time UFR](./azimuth_time_ufr.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Common Raw-Signal Model](#1-common-raw-signal-model)
- [2. Case A: Stripmap-Like Reference](#2-case-a-stripmap-like-reference)
- [3. Case B: TOPS-Like Scan-Dependent Exposure](#3-case-b-tops-like-scan-dependent-exposure)
- [4. Focused-Time Inflation Formula](#4-focused-time-inflation-formula)
- [Final Result](#final-result)

## Summary

- `explain_UFR4.py` 的重點不是完整 UFR，而是比較兩種 exposure geometry。
- Case A 用 `$t_{\mathrm{expo}} = t_c$`，代表 stripmap-like reference。
- Case B 用 `$t_{\mathrm{expo}} = \frac{k_a}{k_a-k_s} t_c$`，代表 TOPS-like scan-dependent exposure。
- 兩個 case 共用同一條 `raw -> spectrum -> matched filtering -> focused output` 鏈，所以差異只會來自 `$t_{\mathrm{expo}}$` 的幾何定義。
- 這份文件同樣採用 `圖 -> 數學 -> 程式碼 -> 物理`，讓 focused-time inflation 的來源可以一眼看懂。

## Problem Definition

本文件要回答兩件事：

1. `explain_UFR4.py` 的兩個 case 在數學上差在哪裡。
2. 為什麼 `$t_{\mathrm{expo}} \neq t_c$` 會導致 focused-time inflation。

## Symbols And Assumptions

- $\eta$：azimuth slow time
- $f_\eta$：azimuth frequency
- $T_{\mathrm{burst}}$：raw burst duration
- $\mathrm{PRF}$：azimuth sampling rate
- $k_a$：target azimuth FM rate
- $k_s$：scan-induced Doppler-centroid rate
- $T_{\mathrm{dwell}}$：illumination dwell time
- $t_c$：target focus-center label
- $t_{\mathrm{expo}}$：raw slow-time 中的 exposure center
- $s_{\mathrm{raw}}(\eta)$：synthetic raw azimuth signal
- $S_1(f_\eta)$：raw signal 的 azimuth spectrum
- $H_{\mathrm{az}}(f_\eta)$：matched filter
- $s_{\mathrm{final}}(\eta)$：focus 後的時間域輸出

## 1. Common Raw-Signal Model

<p align="center">
  <img src="./figures/explain_ufr4/ufr4_caseA_overview.png" width="900">
</p>

Figure Caption:
這張總覽圖顯示兩個 case 共用的一維 azimuth chain：`raw_signal -> S_1 -> matched filtering -> s_final`。

Mathematical Step:

$$
s_{\mathrm{raw},p}(\eta) =
\mathrm{rect}\left(
\frac{\eta-t_{\mathrm{expo},p}}{T_{\mathrm{dwell}}}
\right)
\exp\left(
j\pi k_a(\eta-t_{c,p})^2
\right)
$$

$$
{\color{red}
s_{\mathrm{raw}}(\eta) =
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
S_1(f_\eta) = \mathcal{F}_\eta\left[s_{\mathrm{raw}}(\eta)\right]
$$

$$
H_{\mathrm{az}}(f_\eta) =
\exp\left(
j\pi \frac{f_\eta^2}{k_a}
\right)
$$

$$
{\color{red}
s_{\mathrm{final}}(\eta) =
\mathcal{F}^{-1}_\eta\left[
S_1(f_\eta)H_{\mathrm{az}}(f_\eta)
\right]
}
$$

Code Mapping:

```python
raw_signal = np.zeros(Naz, dtype=complex)
for tc in tc_array:
    t_expo = exposure_fn(tc)
    window = np.abs(eta - t_expo) <= (T_dwell / 2)
    target_phase = np.exp(1j * np.pi * ka * (eta - tc) ** 2)
    raw_signal[window] += target_phase[window]

S1 = np.fft.fftshift(np.fft.fft(raw_signal))
H_az = np.exp(1j * np.pi * (1.0 / ka) * f_eta**2)
S2 = S1 * H_az
s_final = np.fft.ifft(np.fft.ifftshift(S2))
```

Strict Math <-> Code Mapping:

- `$s_{\mathrm{raw}}(\eta)$` <-> `raw_signal`
- `$S_1(f_\eta)$` <-> `S1`
- `$H_{\mathrm{az}}(f_\eta)$` <-> `H_az`
- focused output <-> `s_final`
- `$t_{\mathrm{expo}}$` <-> `t_expo`
- `$t_c$` <-> `tc`

Physical Meaning:
兩個 case 的處理鏈完全一樣，所以任何 focused-time 差異都只能來自 exposure geometry。

Why This Leads To The Next Figure:
既然共用同一條處理鏈，就先看 stripmap-like reference case。

## 2. Case A: Stripmap-Like Reference

<p align="center">
  <img src="./figures/explain_ufr4/ufr4_caseA_raw_tf.png" width="900">
</p>

Figure Caption:
這張圖是 stripmap-like 參考情況的 raw time-frequency view。各條 chirp traces 的 illumination center 與 focus-center label 對齊。

Mathematical Step:

$$
{\color{red}
t_{\mathrm{expo}} = t_c
}
$$

$$
s_{\mathrm{raw},A}(\eta) =
\sum_p
\mathrm{rect}\left(
\frac{\eta-t_{c,p}}{T_{\mathrm{dwell}}}
\right)
\exp\left(
j\pi k_a(\eta-t_{c,p})^2
\right)
$$

Code Mapping:

```python
_plot_latex_ufr_case(
    ...,
    exposure_fn=lambda tc: 1.0 * tc,
)
```

Strict Math <-> Code Mapping:

- `$t_{\mathrm{expo}} = t_c$` <-> `exposure_fn=lambda tc: 1.0 * tc`
- `$s_{\mathrm{raw},A}(\eta)$` <-> `raw_signal` in Case A

Physical Meaning:
這個 case 中，raw signal 中目標出現的時間與後來應該聚焦的位置一致，因此 raw burst support 映到 focused domain 時不會被額外拉伸。

Why This Leads To The Next Figure:
有了 raw 參考圖之後，就可以看它 focus 後是否仍維持相同幾何尺度。

<p align="center">
  <img src="./figures/explain_ufr4/ufr4_caseA_focused_tf.png" width="900">
</p>

Figure Caption:
這張圖顯示 stripmap-like 參考情況下的 focused time-frequency view。能量被壓縮，但 focused support 並沒有額外膨脹。

Mathematical Step:

$$
{\color{red}
s_{\mathrm{final},A}(\eta) =
\mathcal{F}^{-1}_\eta\left[
\mathcal{F}_\eta\left[s_{\mathrm{raw},A}(\eta)\right]
H_{\mathrm{az}}(f_\eta)
\right]
}
$$

Code Mapping:

```python
S1 = np.fft.fftshift(np.fft.fft(raw_signal))
S2 = S1 * H_az
s_final = np.fft.ifft(np.fft.ifftshift(S2))
```

Strict Math <-> Code Mapping:

- `$s_{\mathrm{final},A}(\eta)$` <-> `s_final` in Case A
- focused spectrum product <-> `S2`

Physical Meaning:
這是沒有 focused-time inflation 時的基準圖。

Why This Leads To The Next Figure:
有了基準之後，再看 TOPS-like case 時才知道差異到底從哪裡來。

## 3. Case B: TOPS-Like Scan-Dependent Exposure

<p align="center">
  <img src="./figures/explain_ufr4/ufr4_caseB_raw_tf.png" width="900">
</p>

Figure Caption:
這張圖是 TOPS-like case 的 raw time-frequency view。和 Case A 相比，chirp traces 的中心位置與可見範圍已被 scan-dependent exposure 改變。

Mathematical Step:

$$
{\color{red}
t_{\mathrm{expo}} = \frac{k_a}{k_a-k_s}t_c
}
$$

$$
{\color{red}
s_{\mathrm{raw},B}(\eta) =
\sum_p
\mathrm{rect}\left(
\frac{\eta-\frac{k_a}{k_a-k_s}t_{c,p}}{T_{\mathrm{dwell}}}
\right)
\exp\left(
j\pi k_a(\eta-t_{c,p})^2
\right)
}
$$

Code Mapping:

```python
_plot_latex_ufr_case(
    ...,
    exposure_fn=lambda tc: (ka / (ka - ks)) * tc,
)
```

Strict Math <-> Code Mapping:

- `$t_{\mathrm{expo}} = \frac{k_a}{k_a-k_s}t_c$` <-> `exposure_fn=lambda tc: (ka / (ka - ks)) * tc`
- `$k_a$` <-> `ka`
- `$k_s$` <-> `ks`
- `$s_{\mathrm{raw},B}(\eta)$` <-> `raw_signal` in Case B

Physical Meaning:
TOPS-like case 改變的不是 matched filter，而是目標在 raw burst 內被照亮的時刻。beam scanning 先扭斜了 exposure geometry，再間接造成 focused-time inflation。

Why This Leads To The Next Figure:
一旦 raw geometry 被扭斜，就要看 focus 後這個扭斜如何變成時間支撐的拉長。

<p align="center">
  <img src="./figures/explain_ufr4/ufr4_caseB_focused_tf.png" width="900">
</p>

Figure Caption:
這張圖顯示 TOPS-like case 的 focused time-frequency view。目標仍然可以被聚焦，但能量支撐相對於 raw burst geometry 已被拉長。

Mathematical Step:

$$
{\color{red}
s_{\mathrm{final},B}(\eta) =
\mathcal{F}^{-1}_\eta\left[
\mathcal{F}_\eta\left[s_{\mathrm{raw},B}(\eta)\right]
H_{\mathrm{az}}(f_\eta)
\right]
}
$$

Code Mapping:

```python
S1 = np.fft.fftshift(np.fft.fft(raw_signal))
S2 = S1 * H_az
s_final = np.fft.ifft(np.fft.ifftshift(S2))
```

Strict Math <-> Code Mapping:

- `$s_{\mathrm{final},B}(\eta)$` <-> `s_final` in Case B
- focused-spectrum multiplication <-> `S2`

Physical Meaning:
這張圖說明問題不是「不能聚焦」，而是「聚焦之後的時間支撐比原本 burst window 更寬」。

Why This Leads To The Next Figure:
既然現象已經在圖上看見，就可以把它整理成 focused-time inflation 的公式。

## 4. Focused-Time Inflation Formula

<p align="center">
  <img src="./figures/explain_ufr4/ufr4_caseB_overview.png" width="900">
</p>

Figure Caption:
這張總覽圖提醒你：Case B 的所有變化都來自 `$t_{\mathrm{expo}}$` 與 `$t_c$` 的映射關係，而不是來自不同的 matched filter。

Mathematical Step:

$$
{\color{red}
t_c = \frac{k_a-k_s}{k_a} t_{\mathrm{expo}}
}
$$

若 raw burst 的 exposure support 為

$$
|t_{\mathrm{expo}}| \le \frac{T_{\mathrm{burst}}}{2}
$$

則 focused support 近似滿足

$$
|t_c|
\le
\left|
\frac{k_a-k_s}{k_a}
\right|
\frac{T_{\mathrm{burst}}}{2}
$$

因此

$$
{\color{red}
T_{\mathrm{focused}}
\approx
\left|
\frac{k_a-k_s}{k_a}
\right|
T_{\mathrm{burst}}
}
$$

Code Mapping:

```python
PRF = 1000.0
T_burst = 4.0
ks = 100.0
ka = -500.0
exposure_fn = lambda tc: (ka / (ka - ks)) * tc
```

Strict Math <-> Code Mapping:

- `$T_{\mathrm{burst}}$` <-> `T_burst`
- `$k_a$` <-> `ka`
- `$k_s$` <-> `ks`
- exposure mapping <-> `exposure_fn`

Physical Meaning:
只要 `$k_s \neq 0$` 且與 `$k_a$` 反號，TOPS-like case 的 focused-time span 就會比 raw burst window 更長。這就是 focused-time inflation，也是後續 time wrap-around 與 time UFR 必須存在的幾何根源。

Why This Leads To The Next Figure:
這已經是最後的整理公式，沒有下一步。

## Final Result

`explain_UFR4.py` 用兩個最小對照 case 證明：

- 若 `$t_{\mathrm{expo}} = t_c$`，則 focused-time span 不會因 exposure geometry 額外拉長
- 若 `$t_{\mathrm{expo}} = \frac{k_a}{k_a-k_s}t_c$`，則固定的 raw burst window 會映射成更長的 focused support

這份文件的重點就是讓你在看圖的同時，也立刻看到對應的數學、程式碼和嚴格的符號對應。
