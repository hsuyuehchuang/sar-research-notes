# TOPS SAR Derivations

## Flowchart

- [Raw Data](#1-raw-data)
- [Range Compression](#2-range-compression)
- [Azimuth Frequency Unfolding And Resampling (UFR)](#3-azimuth-frequency-unfolding-and-resampling-ufr)
  - [Azimuth Frequency Folding (Explain)](#31-azimuth-frequency-folding-explain)
  - [Mosaicking](#32-mosaicking)
  - [Deramping](#33-deramping)
  - [Low Pass Filter](#34-low-pass-filter)
  - [Reramping](#35-reramping)
- [Azimuth Compression](#4-azimuth-compression)
- [Azimuth Time Unfolding And Resampling (UFR)](#5-azimuth-time-unfolding-and-resampling-ufr)
  - [Azimuth Time Folding (Explain)](#51-azimuth-time-folding-explain)
  - [Mosaicking](#52-mosaicking)
  - [Deramping](#53-deramping)
  - [Low Pass Filter](#54-low-pass-filter)
  - [Reramping](#55-reramping)
- [Focused Image](#6-focused-image)

## Hierarchy

- [Summary](#summary)
- [Signal Definitions](#signal-definitions)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- Main Flow
  - [1. Raw Data](#1-raw-data)
  - [2. Range Compression](#2-range-compression)
  - [3. Azimuth Frequency Unfolding And Resampling (UFR)](#3-azimuth-frequency-unfolding-and-resampling-ufr)
    - [3.1. Azimuth Frequency Folding (Explain)](#31-azimuth-frequency-folding-explain)
  - [4. Azimuth Compression](#4-azimuth-compression)
  - [5. Azimuth Time Unfolding And Resampling (UFR)](#5-azimuth-time-unfolding-and-resampling-ufr)
    - [5.1. Azimuth Time Folding (Explain)](#51-azimuth-time-folding-explain)
  - [6. Focused Image](#6-focused-image)
- [Physical Meaning](#physical-meaning)
- [Final Result](#final-result)

## Problem Definition

本文件要把 TOPS SAR 的方位向處理鏈完整寫成一條顯式數學主線，並且回答以下幾個問題：

1. 如何以數學形式證明 azimuth frequency folding (aliasing) 的來源與其機制
2. 如何以數學形式說明 mosaicking 所執行的重排操作
3. 如何以數學形式證明 deramping 對主 replica phase curvature 的影響
4. 如何以數學形式證明 azimuth time folding (aliasing) 的來源與其機制

## Summary

- 這份推導把 TOPS SAR 的完整訊號 從 raw data 一路推到 focused image。
- 每一個 stage signal 都必須寫成自己的 fully expanded closed form，不能只用操作符代替。
- 對應的主鏈訊號依序為 $s_0(\tau,\eta)$、$s_1(\tau,\eta)$、$S_2(\tau,f_\eta)$、$S_3(\tau,f_\eta)$、$S_4(\tau,f_\eta)$、$S_5(\tau,f_\eta)$、$S_6(\tau,f_\eta)$、$s_7(\tau,\eta)$、$I_8(\tau,\eta)$、$I_9(\tau,\eta)$、$I_{10}(\tau,\eta)$、$I_{11}(\tau,\eta)$ 與 $I_{\mathrm{focus}}(\tau,\eta)$。

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

## Symbols And Assumptions

- $\tau$：range fast time
- $\eta$：azimuth slow time
- $f_\eta$：azimuth frequency
- $R(\eta)=\sqrt{R_0^2+V_r^2(\eta-\eta_0)^2}$：瞬時斜距
- $K_r$：range chirp rate
- $B_r$：range bandwidth
- $T_r$：range pulse duration
- $T_p=1/\mathrm{PRF}$：slow-time sampling interval
- $f_{\eta_c}$：reference slow time $\eta_c$ 對應的 azimuth frequency
- $k_s$：frequency-domain local chirp slope，滿足 $f_\eta = k_s\eta + f_{\eta_c}$
- $k_t$：time-domain local chirp slope，滿足 $f_\eta(\eta)=k_t(\eta-\eta_c)+f_{\eta_c}$
- $w_a(\eta;\omega_s)$：TOPS azimuth illumination
- $W_a(f_\eta;\omega_s)$：其 azimuth-frequency envelope
- $\psi_m(f_\eta)$：frequency-UFR 中第 $m$ 個 replica 的 phase
- $\chi_m(\eta)$：time-UFR 中第 $m$ 個 replica 的 phase
- $B_{\mathrm{LPF}}$：frequency-UFR keep band
- $T_{\mathrm{LPF}}$：time-UFR keep window
- $B_{\mathrm{az,keep}}$：最終保留的 azimuth effective bandwidth

## 1. Raw Data

The received signal of TOPS SAR is:

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
\color{red}{
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)}
$$

$\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)$
此式為 slow-time 上的 impulse train，將連續的 slow-time 訊號，以 $T_p$ 的時間間隔進行取樣。
新增此項，是為了後續用數學證明 azimuth frequency folding 是由取樣及 azimuth FFT 所造成。

## 2. Range Compression

range compression 的完整推導可直接參考 [range_compression.md](./range_compression.md)

range matched filter 為

$$
h_r(\tau) =
\exp\left(
-j\pi K_r\tau^2
\right)
$$

After range compression, we obtain

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
\color{red}{
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)}
$$

## 3. Azimuth Frequency Unfolding And Resampling (UFR)

詳細推導在 [Azimuth Frequency Folding](./azimuth_freq_folding.md)。先證明為什麼會有 folded spectrum ，再進入 $mosaicking \rightarrow deramping \rightarrow LPF \rightarrow reramping$ 的處理鏈。

### 3.1. Azimuth Frequency Folding (Explain)

做完range compression的訊號可以改寫成

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
\color{red}{
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)}
$$

$$
s_1(\tau,\eta) =
s_{1,\mathrm{cont}}(\tau,\eta)\cdot
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
$$

where

$$
s_{1,\mathrm{cont}}(\tau,\eta) =
A_1\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R(\eta)}{c}
\right)
\right] \cdot
w_a(\eta;\omega_s) \cdot
\exp\left(
-j\frac{4\pi f_0R(\eta)}{c}
\right)
$$

也就是說，$s_1(\tau,\eta)$ 是把連續 slow-time 訊號 $s_{1,\mathrm{cont}}(\tau,\eta)$ 乘上一個 impulse train 之後得到的 sampled signal。

先對連續訊號 $s_{1,\mathrm{cont}}(\tau,\eta)$ 在 azimuth 方向 $\eta$ 做 Fourier transform：

$$
S_{1,c}(\tau,f_\eta;\omega_s) =
\mathcal{F}_{\eta}\bigl[s_{1,\mathrm{cont}}(\tau,\eta)\bigr]
$$

若把 $S_{1,c}$ 寫開，則

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

接下來把 slow-time sampling 的效果帶進 frequency domain。由於 $s_1(\tau,\eta)$ 是 continuous signal 與 impulse train 的乘積，因此其 azimuth Fourier transform 可逐步寫成

$$
S_{2}(\tau,f_\eta) =
\mathcal{F}_{\eta}\bigl[
s_{1,\mathrm{cont}}(\tau,\eta) \cdot
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
\bigr]
$$

$$
S_{2}(\tau,f_\eta) =
\mathcal{F}_{\eta}\bigl[
s_{1,\mathrm{cont}}(\tau,\eta)
\bigr] *
\mathcal{F}_{\eta}\bigl[
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
\bigr]
$$

$$
S_{2}(\tau,f_\eta) =
S_{1,c}(\tau,f_\eta;\omega_s)
\,
\ast
\left[
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}\delta(f_\eta-k\cdot\mathrm{PRF})
\right]
$$

$$
S_{2}(\tau,f_\eta) =
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
S_{1,c}(\tau,f_\eta-k\cdot\mathrm{PRF};\omega_s)
$$

也就是說，$S_2$ 不是另一個獨立定義出來的訊號，而是把連續 azimuth spectrum $S_{1,c}$ 的所有 `PRF`-spaced replicas 全部加總之後得到的 folded azimuth-frequency signal。

把上式中的 $S_{1,c}$ 完整展開之後，就得到

$$
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
$$

這裡要特別注意：

- $n$ 是 time domain 裡的 slow-time sample index，來自 $\eta=nT_p$
- 做完 azimuth FFT 之後，$n$ 不再顯式出現
- 它的效果會變成 frequency domain 中每隔 `PRF` 出現一次的 spectrum replicas
- 因此後面改用 $k$ 來編號那些 folded copies，而不是再用 $n$

如果要看這件事的完整推導，可直接看：
- [azimuth_freq_folding.md 第 4 節](./azimuth_freq_folding.md#4-continuous-azimuth-spectrum)
- [azimuth_freq_folding.md 第 5 節](./azimuth_freq_folding.md#5-folded-spectrum-from-the-sampling-comb)


這裡的 $k$ 不是 slow-time sample index，而是取樣後在頻域中每隔 `PRF` 出現一次的 **replica index**。也就是說，$k$ 就是 azimuth frequency folding 之後，第 $k$ 個 replica 的編號。

所以
$$
\color{red}{
f_\eta-k\cdot\mathrm{PRF}}
$$
就是把連續 azimuth spectrum 以 `PRF` 為間隔做平移後所得到的第 $k$ 個 spectral replica，也就是第 $k$ 個 folded copy。

這些 folded copies 的來源，是 slow-time 上的離散取樣

$$
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
$$

在做 azimuth FFT 之後，於 frequency domain 變成一個以 `PRF` 為間隔的 impulse train，因而使原本的連續 spectrum $S_{1,c}$ 被週期性複製。

而 folding 可以從下面這個式子直接看出來：

$$
W_{\mathrm{fold}}(f_\eta;\omega_s) =
\sum_{k=-\infty}^{\infty}
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
$$

因為這個式子明確表示同一個連續頻譜 $W_a$ 被複製成許多個 `PRF`-spaced replicas；當這些 replicas 在同一基本頻帶內互相重疊時，就形成 azimuth frequency folding (aliasing)。

另外，這裡之所以還能在後續 UFR 中把它還原，是因為 TOPS azimuth signal 本質上是 chirp-like signal，因此在 time 與 frequency 之間保有明確對應關係。也就是說，不同 replicas 雖然在 folded frequency axis 上彼此重疊，但它們仍然對應到不同的 chirp support / local phase law，所以後面才能透過 mosaicking、deramping、LPF 與 reramping，把這些 folded copies 再重新展開與分離。


### 3.2. Mosaicking

Mosaicking 的核心不是消除 replicas，而是把 $S_2(\tau,f_\eta)$ 中原本 folded 在同一個基本頻帶內的 replicas，重新排到 extended azimuth-frequency axis 上。因此，mosaicking 對應的數學操作不是 filtering，而是 replica-dependent coordinate relabeling 與重新組裝。

在連續表示中，mosaicked signal 可先寫成

$$
{\color{red}{
S_3(\tau,f_\eta) = \sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}} S_{3,m}(\tau,f_\eta)
}}
$$

這裡的重點是，$S_2(\tau,f_\eta)$ 中的 $f_\eta$ 仍然是 folded frequency axis 上的座標；而到了 $S_3(\tau,f_\eta)$，$f_\eta$ 已經要被重新解釋成 extended axis 上的座標。也就是說，mosaicking 真正做的事情，不只是把多個 replicas 分開，而是先依 replica index $m$ 重新定義其對應的 frequency support，再把它們放回 extended axis 的正確位置。

因此，第 $m$ 個 mosaicked replica 可寫成

$$
S_{3,m}(\tau,f_\eta) =
A_3\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-f_{\eta_c}}{B_{\max}}
\right) \cdot
\exp\left(
-j\phi_m(f_\eta)
\right)
$$

其中

$$
\phi_m(f_\eta) =
\frac{4\pi R_0f_0}{c}D(f_\eta-m\cdot\mathrm{PRF},V_r)
+2\pi(f_\eta-m\cdot\mathrm{PRF})\eta_0
$$

上式中的

$$
{\color{red}{
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-f_{\eta_c}}{B_{\max}}
\right)
}}
$$

明確表示第 $m$ 個 replica 已經被放到其在 extended axis 上對應的 support region。這也是為什麼 $S_3(\tau,f_\eta)$ 才是表達 mosaicking 的關鍵 signal：在 $S_2(\tau,f_\eta)$ 中，多個 folded copies 還共用同一個 folded coordinate；在 $S_3(\tau,f_\eta)$ 中，每個 replica 已經擁有自己的 support 與自己的索引 $m$。

接著，為了讓後續 deramping filter 能直接作用在主 replica 的局部 chirp curvature 上，我們不再保留 $\phi_m(f_\eta)$ 的完整 closed form，而是在 $f_{\mathrm{ref}}$ 附近把它寫成局部二次形式：

$$
\phi_m(f_\eta) \approx
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
$$

其中 $\psi_{0,m}$ 是 phase offset，$\psi_{1,m}$ 是局部線性 slope，而 $\psi_{2,m}$ 是第 $m$ 個 replica 的 local quadratic curvature。這一步之所以必要，是因為後面的 deramping 本質上就是消掉某個 replica 的 local quadratic phase curvature，因此必須先把每個 replica 的 phase 改寫成可以直接讀出 quadratic term 的形式。

將這個局部 phase model 代回之後，mosaicked signal 的完整解析式可寫成

$$
{\color{red}{
S_3(\tau,f_\eta) =
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_3\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-f_{\eta_c}}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
}}
$$

在離散實作中，這件事可以直接對應為：把每個 replica 對應的 $(\tau,f_\eta)$ sub-matrix，依照其 extended frequency support 重新排列並組裝成一個較大的 matrix。只要每個 sub-matrix 都真的對應一個明確的 replica index $m$，那麼這個離散的重排與組裝操作，就正是連續數學上形成 $S_3(\tau,f_\eta)$ 的資料結構對應。換句話說，資料結構上能直接做 matrix assembly，成立的前提不是單純「把矩陣相加」，而是每個 replica 的 $f_\eta$ 座標已經先被重新定義到 extended axis 上。

更完整的 mosaicking 與 local phase model，可直接看：
- [azimuth_freq_ufr.md](./azimuth_freq_ufr.md)
- [azimuth_deramp_LPF.md 第 1-2 節](./azimuth_deramp_LPF.md#1-starting-point-mosaicked-signal)

### 3.3. Deramping

要嚴格證明 deramping 會把主 replica 拉平，必須先把主 replica 的 phase
寫到和 filter 相同的 reference center $f_{\eta_c}$：

$$
\phi_{\mathrm{main}}(f_\eta) =
\phi_{0,\mathrm{main}}
+\phi_{1,\mathrm{main}}(f_\eta-f_{\eta_c})
+\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
$$

frequency-domain deramping filter 為

$$
\color{red}{
H_{\mathrm{de},f}(f_\eta) =
\exp\left(
+j\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
\right)
}
$$

主 replica 經過 deramping 後有

$$
\exp\left(
-j\phi_{\mathrm{main}}(f_\eta)
\right)
H_{\mathrm{de},f}(f_\eta) =
\exp\left(
-j\left[
\phi_{0,\mathrm{main}}
+\phi_{1,\mathrm{main}}(f_\eta-f_{\eta_c})
\right]
\right)
$$

$$
\color{red}{
\phi_{\mathrm{after}}(f_\eta) =
\phi_{0,\mathrm{main}}
+\phi_{1,\mathrm{main}}(f_\eta-f_{\eta_c})
}
$$

這個 red-highlighted $equation$ 就是最直接的數學證據：主 replica 的 quadratic term
\[
\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
\]
在乘完 deramp filter 後 **完全 cancellation**，只剩常數項與線性項。

對主 replica 而言，因為 $\psi_{2,\mathrm{ref}}=\psi_{2,m_0}$，所以其殘餘二次項近似為零。這就是 deramping 真正的物理作用：它不是把 replicas 消失，而是把主 replica 的 quadratic curvature 拿掉。

這一段若要看更完整的 phase-model、deramping 與 LPF 銜接細節，可直接看
[azimuth_deramp_LPF.md](./azimuth_deramp_LPF.md)。

因此

$$
\color{red}{
\frac{d^2\phi_{\mathrm{after}}(f_\eta)}{df_\eta^2} = 0
}
$$

二階導數為零代表 phase curvature 已被消除。從 time-frequency diagram 的角度看，
主 replica 的能量脊線會被拉直，在適當座標下會趨近水平或垂直；這正是後面 LPF 能只保留主 replica 的原因。

The deramped spectrum is then

$$
S_4(\tau,f_\eta) =
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_4\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-f_{\eta_c}}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right) \cdot
\exp\left(
+j\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
\right)
$$

### 3.4. Low Pass Filter

frequency-domain keep window 為

$$
H_{\mathrm{LPF},f}(f_\eta) =
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

After LPF, the output becomes

$$
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
\frac{f_\eta-m\cdot\mathrm{PRF}-f_{\eta_c}}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right) \cdot
\exp\left(
+j\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
\right)
$$

LPF 的完整理想模型與 FFT-based 實作，可直接看：
- [azimuth_deramp_LPF.md 第 4 節](./azimuth_deramp_LPF.md#4-ideal-lpf-model)
- [azimuth_deramp_LPF.md 第 5 節](./azimuth_deramp_LPF.md#5-fft-based-lpf-implementation)

### 3.5. Reramping

frequency-domain reramping filter 為

$$
H_{\mathrm{re},f}(f_\eta) =
\exp\left(
-j\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
\right)
$$

After reramping, we obtain

$$
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
\frac{f_\eta-m\cdot\mathrm{PRF}-f_{\eta_c}}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
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

With the main replica restored to the reference curvature, azimuth compression gives

$$
\color{red}{
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

更完整的 azimuth compression 推導，可直接看：
- [azimuth_compression.md](./azimuth_compression.md)

## 5. Azimuth Time Unfolding And Resampling (UFR)

這一段主流程的前置現象推導是 [Azimuth Time Folding](./azimuth_time_folding.md)。也就是先證明 finite-FFT 為什麼會把線性卷積折回成 time-domain wrap-around，再進入 $mosaicking \rightarrow deramping \rightarrow LPF \rightarrow reramping$ 的處理鏈。

### 5.1. Azimuth Time Folding (Explain)

time-domain wrap-around 的現象本身由 [Azimuth Time Folding](./azimuth_time_folding.md) 單獨說明。主流程在這裡把它列成 explain stage，目的不是重複整份推導，而是明確標出：

- azimuth compression 之後，問題已經從 frequency folding 轉成 time folding
- $mosaicking \rightarrow deramping \rightarrow LPF \rightarrow reramping$ 這條 time-UFR 鏈是為了解開這個 wrap-around 現象
- 若不先理解 [Azimuth Time Folding](./azimuth_time_folding.md)，後面的 time-UFR 會只剩操作流程，少掉幾何原因

其中最直接表達 time folding / wrap-around 的 $equation$ 是

$$
\color{red}{
I_{\mathrm{circ}}(\eta) =
\sum_{m=-\infty}^{\infty}
I_{\mathrm{lin}}(\eta-mT_{\mathrm{window}})
}
$$

這個式子的意思是：

- 理想的線性卷積輸出是 $I_{\mathrm{lin}}(\eta)$
- 但有限長 FFT 實作下，實際得到的是它以 $T_{\mathrm{window}}$ 為週期的週期延拓和
- 超出主時間窗口的部分會被折回來，這就是 azimuth-time folding / wrap-around

如果要看完整推導，可直接看
[azimuth_time_folding.md 的 circular convolution 式子](./azimuth_time_folding.md#6-circular-convolution-and-wrap-around)。

### 5.2. Mosaicking

After mosaicking onto the extended time axis, the signal can be written as

這一步真正表達 time-domain mosaicking 的 $equation$，就是下面的 $I_8(\tau,\eta)$。

原因是：

- 在 folding 階段，time replicas 還是以週期延拓的方式折回主時間窗口
- 到了 $I_8$，每個 replica 已經改用 $m$ 編號，並被放到 extended time axis 的對應位置
- 式子中的
  $$
  \mathrm{rect}\left(
  \frac{\eta-mT_{\mathrm{window}}-\eta_c}{T_{\mathrm{keep}}}
  \right)
  $$
  就是在表示第 $m$ 個 time replica 佔據自己被重排後的時間區段

所以最白話地講，$I_8$ 不再是 wrap-around copies 疊在同一個主時間窗口裡，而是每個 time replica 已經被拆開、排開，這就是 time-domain UFR 的 mosaicking。

$$
\color{red}{
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

time-domain folding 與 wrap-around location 的完整說明，可直接看：
- [azimuth_time_folding.md 第 6 節](./azimuth_time_folding.md#6-circular-convolution-and-wrap-around)
- [azimuth_time_folding.md 第 7 節](./azimuth_time_folding.md#7-wrap-around-location-formula)

### 5.3. Deramping

time-domain 要證明 deramping 會把主 replica 拉平，也必須先把主 replica
在 $\eta_c$ 附近寫成 reference chirp：

$$
\phi_{\mathrm{main}}(\eta) =
\phi_{0,\mathrm{main}}
+\pi k_t(\eta-\eta_c)^2
+2\pi f_{\eta_c}\eta
$$

time-domain deramping filter 為

$$
\color{red}{
H_{\mathrm{de},t}(\eta) =
\exp\left(
-j\pi k_t(\eta-\eta_c)^2 - j2\pi f_{\eta_c}\eta
\right)
}
$$

主 replica 經過 deramping 後有

$$
\exp\left(
-j\phi_{\mathrm{main}}(\eta)
\right)
H_{\mathrm{de},t}(\eta) =
\exp\left(
-j\phi_{0,\mathrm{main}}
\right)
$$

$$
\color{red}{
\phi_{\mathrm{after}}(\eta)=\phi_{0,\mathrm{main}}
}
$$

這個 red-highlighted $equation$ 就是 time-domain 的 cancellation 證據：  
主 replica 的 quadratic chirp phase 與 linear carrier term 在乘完 deramp filter 後都被消掉，只剩常數相位。

對主 replica 而言，若把 time-domain 局部 phase 也寫成
\[
\chi_{0,m}+\chi_{1,m}(\eta-\eta_{\mathrm{ref}})
+\chi_{2,m}(\eta-\eta_{\mathrm{ref}})^2
\]
的形式，則可等價地理解成：因為 $\chi_{2,\mathrm{ref}}=\chi_{2,m_0}$，所以主 replica 的殘餘二次項近似為零。這就是 time-domain deramping 真正的物理作用：它不是把 replicas 消失，而是把主 replica 的 chirp curvature 拿掉。

等價地看 instantaneous Doppler，原本主 replica 的局部 Doppler 為

$$
f_{\eta,\mathrm{old}}(\eta)=k_t(\eta-\eta_c)+f_{\eta_c}
$$

filter 所造成的補償量為

$$
\Delta f_\eta(\eta)= -k_t(\eta-\eta_c)-f_{\eta_c}
$$

所以

$$
\color{red}{
f_{\eta,\mathrm{new}}(\eta)=
f_{\eta,\mathrm{old}}(\eta)+\Delta f_\eta(\eta)=0
}
$$

這表示主 replica 的 instantaneous Doppler 已由斜線變成常數。從 time-frequency diagram 的角度看，
原本傾斜的 chirp 軌跡會被展平成近似水平或垂直的主能量帶，這正是後面 time-domain LPF 可以只保留主 replica 的數學原因。

若要看 frequency-domain 與 time-domain deramping 設計的完整對照，可直接看
[freq_time_deramping.md](./freq_time_deramping.md)。

The time-domain deramped signal is

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
+\chi_{2,m}(\eta-\eta_{\mathrm{ref}})^2
\right]
\right) \cdot
\exp\left(
-j\pi k_t(\eta-\eta_c)^2 - j2\pi f_{\eta_c}\eta
\right)
$$

### 5.4. Low Pass Filter

time-domain keep window 為

$$
H_{\mathrm{LPF},t}(\eta) =
\mathrm{rect}\left(
\frac{\eta-\eta_{\mathrm{LPF}}}{T_{\mathrm{LPF}}}
\right)
$$

After time-domain LPF, the output becomes

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
+\chi_{2,m}(\eta-\eta_{\mathrm{ref}})^2
\right]
\right) \cdot
\exp\left(
-j\pi k_t(\eta-\eta_c)^2 - j2\pi f_{\eta_c}\eta
\right)
$$

### 5.5. Reramping

time-domain reramping filter 為

$$
H_{\mathrm{re},t}(\eta) =
\exp\left(
+j\pi k_t(\eta-\eta_c)^2 + j2\pi f_{\eta_c}\eta
\right)
$$

After time-domain reramping, we obtain

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

## 6. Focused Image

If only the main replica $m=m_0$ is retained, the final focused image is

$$
\color{red}{
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
$$
