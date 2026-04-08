# Azimuth Deramp LPF

## Hierarchy

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- Main Flow
  - [1. Starting Point: Mosaicked Signal](#1-starting-point-mosaicked-signal)
  - [2. Local Quadratic Phase Model](#2-local-quadratic-phase-model)
  - [3. Deramping](#3-deramping)
  - [4. Ideal LPF Model](#4-ideal-lpf-model)
  - [5. FFT-Based LPF Implementation](#5-fft-based-lpf-implementation)
- [Physical Meaning](#physical-meaning)
- [Final Result](#final-result)

## Summary

* mosaicking 之後的 $S_2(\tau,f_\eta)$ 只把 replicas 攤開到 extended azimuth-frequency axis，並不會自動拿掉每個 replica 內部的二次 phase curvature。
* deramping 的核心作用，是對主 replica 乘上一個 reference quadratic phase 的共軛補償，使其殘餘二次項由 $\psi_{2,m}$ 變成 $\psi_{2,m}-\psi_{2,\mathrm{ref}}$。
* LPF 之所以有效，不是因為它會自己辨認主 replica，而是因為主 replica 已先在 deramping 後被展平成接近 baseband 的窄頻表示。
* 理想數學模型可以寫成固定通帶的 `rect` 窗，但實作上採用 FFT-based sharp-cut filtering：Forward FFT、將不需要的 bins 直接設為 0、Inverse FFT。
* 整個處理鏈必須拆成起點訊號、局部 phase model、deramping、理想 LPF 模型、FFT-based LPF 實作五個模組，並且每一步都寫出 fully expanded closed form。
* 這裡真正 carried-forward 的結果是 deramping 後的 $S_3(\tau,f_\eta)$ 與 LPF 後的 $S_4(\tau,f_\eta)$。

摘要中最重要的關鍵公式為

$$
S_3(\tau,f_\eta) \approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
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
$$

以及

$$
S_4(\tau,f_\eta) \approx
S_3(\tau,f_\eta) \cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

---

## Problem Definition

本文件要證明三件事：

1. 為什麼 UFR / mosaicking 之後仍必須再做 deramping，才能讓 LPF 有物理意義地保留主 replica。
2. deramping filter 應該抵消哪一個 phase term，抵消後第 $m$ 個 replica 的 phase 會變成什麼。
3. 在起點 $S_2(\tau,f_\eta)$、deramping 後 $S_3(\tau,f_\eta)$、LPF 後 $S_4(\tau,f_\eta)$ 這三步，訊號各自的 fully expanded closed form 是什麼。

---

## Derivation Highlights

* 從 mosaicked azimuth-frequency signal $S_2(\tau,f_\eta)$ 出發，先把每個 replica 的完整振幅與 phase 寫清楚。
* 再把第 $m$ 個 replica 的 phase function $\psi_m(f_\eta)$ 在主 replica 的參考頻率 $f_{\mathrm{ref}}$ 附近展成局部二次模型。
* 然後用主 replica 的 quadratic curvature $\psi_{2,\mathrm{ref}}$ 建立 deramping filter，逐 replica 地把殘餘二次項改寫成 $\psi_{2,m}-\psi_{2,\mathrm{ref}}$。
* 最後把固定通帶窗乘回 deramped spectrum，得到 LPF 後的 fully expanded closed form。

---

## Symbols And Assumptions

* $S_2(\tau,f_\eta)$：mosaicking 後的 azimuth-frequency signal
* $S_{2,m}(\tau,f_\eta)$：第 $m$ 個 mosaicked replica
* $S_3(\tau,f_\eta)$：deramping 後的總訊號
* $S_{3,m}(\tau,f_\eta)$：第 $m$ 個 deramped replica
* $S_4(\tau,f_\eta)$：LPF 後的總訊號
* $S_{4,m}(\tau,f_\eta)$：第 $m$ 個 LPF 後 replica
* $m_0$：欲保留之主 replica 索引
* $f_{\mathrm{ref}}$：主 replica 的 reference frequency
* $\psi_m(f_\eta)$：第 $m$ 個 replica exponent 中的實數 phase function
* $\psi_{0,m},\psi_{1,m},\psi_{2,m}$：$\psi_m(f_\eta)$ 在 $f_{\mathrm{ref}}$ 附近的局部係數
* $\psi_{2,\mathrm{ref}}$：reference quadratic curvature
* $H_{\mathrm{de}}(f_\eta)$：deramping filter
* $H_{\mathrm{LPF}}(f_\eta)$：ideal low-pass filter
* $D_m(f_\eta)=D\left(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}}\right)$：第 $m$ 個 replica 的幾何因子
* $B_{\max}$：mosaicked replica 的有效頻寬
* $B_{\mathrm{LPF}}$：LPF 通帶寬度
* $\widetilde{S}_3(\tau,\nu_u)$：沿目前離散處理軸 $u$ 對 $S_3(\tau,u)$ 做 FFT 後的表示
* $\widetilde{M}_{\mathrm{LPF}}(\nu_u)$：FFT-domain binary keep mask

假設如下：

* 在主 replica 的局部頻帶內，二階 phase model 足以描述主要 curvature。
* deramping 使用單一 reference curvature $\psi_{2,\mathrm{ref}}$，其設計目標是展平主 replica，而不是同時展平所有 replicas。
* LPF 通帶選得足以容納 deramping 後主 replica 的主要能量，並抑制其他 replicas。
* 本文只對 phase 做局部二階近似；振幅支撐項仍以原本的 $\mathrm{sinc}$ 與 $\mathrm{rect}$ 結構保留。

---

## 1. Starting Point: Mosaicked Signal

由 mosaicking 後的 extended azimuth-frequency representation，可先寫成 replica summation

$$
S_2(\tau,f_\eta) =
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
S_{2,m}(\tau,f_\eta)
$$

其中第 $m$ 個 replica 的 fully expanded closed form 為

$$
S_{2,m}(\tau,f_\eta) =
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right) \cdot
\exp\left(
-j\psi_m(f_\eta)
\right)
$$

其中 phase function 為

$$
\psi_m(f_\eta) =
\frac{4\pi R_0f_0}{c}D_m(f_\eta) + 2\pi\left(
f_\eta-m\cdot\mathrm{PRF}
\right)\eta_c
$$

以及

$$
D_m(f_\eta) = D\left(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}}\right)
$$

因此本步結束後，總訊號的 fully expanded closed form 為

$$
S_2(\tau,f_\eta) =
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right) \cdot
\exp\left(
-j\psi_m(f_\eta)
\right)
$$

這一步只回答：mosaicking 後的訊號長什麼樣。它還沒有把 replica 內部的 phase curvature 拿掉。

---

## 2. Local Quadratic Phase Model

為了建立 deramping filter，必須先把第 $m$ 個 replica 的 phase function 在主 replica 所關注的局部頻帶內展成二次式：

$$
\psi_m(f_\eta) \approx
\psi_{0,m} + \psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
$$

其中

$$
\psi_{0,m} = \psi_m(f_{\mathrm{ref}})
$$

$$
\psi_{1,m} = \psi_m'(f_{\mathrm{ref}})
$$

$$
\psi_{2,m} = \frac{\psi_m''(f_{\mathrm{ref}})}{2}
$$

把這個局部 phase model 代回第 $m$ 個 replica，可得局部近似後的 fully expanded closed form

$$
S_{2,m}(\tau,f_\eta) \approx
A_2\,
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
\psi_{0,m} + \psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
$$

因此本步結束後，總訊號的局部 fully expanded closed form 為

$$
S_2(\tau,f_\eta) \approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
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
\psi_{0,m} + \psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
$$

這一步的重點是把每個 replica 的 curvature 明確抽成 $\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2$，否則後面無法明講 deramping 究竟抵消了什麼。

---

## 3. Deramping

取 reference quadratic curvature 為主 replica 的局部二次係數

$$
\psi_{2,\mathrm{ref}} = \psi_{2,m_0}
$$

則 deramping filter 定義為

$$
H_{\mathrm{de}}(f_\eta) =
\exp\left(
+j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

若改寫成常見 chirp-rate 記號，也可寫成

$$
H_{\mathrm{de}}(f_\eta) =
\exp\left(
+j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
\right)
$$

第 $m$ 個 replica 經過 deramping 後為

$$
S_{3,m}(\tau,f_\eta) =
S_{2,m}(\tau,f_\eta) \cdot H_{\mathrm{de}}(f_\eta)
$$

把 filter 乘回去後，第 $m$ 個 replica 的 fully expanded closed form 變成

$$
S_{3,m}(\tau,f_\eta) \approx
A_2\,
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
\psi_{0,m} + \psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
$$

因此 deramping 後的總訊號 fully expanded closed form 為

$$
S_3(\tau,f_\eta) \approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
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
\psi_{0,m} + \psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
$$

對主 replica 而言，因為 $\psi_{2,\mathrm{ref}}=\psi_{2,m_0}$，所以其殘餘二次項近似為零。這就是 deramping 真正的物理作用：它不是把 replicas 消失，而是把主 replica 的 quadratic curvature 拿掉。

---

## 4. Ideal LPF Model

理想 LPF 定義為

$$
H_{\mathrm{LPF}}(f_\eta) =
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

第 $m$ 個 deramped replica 經過 LPF 後為

$$
S_{4,m}(\tau,f_\eta) =
S_{3,m}(\tau,f_\eta) \cdot H_{\mathrm{LPF}}(f_\eta)
$$

把通帶窗乘回去後，第 $m$ 個 replica 的 fully expanded closed form 為

$$
S_{4,m}(\tau,f_\eta) \approx
A_2\,
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
\psi_{0,m} + \psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)(f_\eta-f_{\mathrm{ref}})^2
\right]
\right) \cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

因此 LPF 後的總訊號 fully expanded closed form 為

$$
S_4(\tau,f_\eta) \approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
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
\psi_{0,m} + \psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)(f_\eta-f_{\mathrm{ref}})^2
\right]
\right) \cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

若只保留主 replica $m=m_0$ 的近似，則輸出可再寫成

$$
S_4(\tau,f_\eta) \approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_{m_0}(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-m_0\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right) \cdot
\exp\left(
-j\left[
\psi_{0,m_0} + \psi_{1,m_0}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m_0}-\psi_{2,\mathrm{ref}}
\right)(f_\eta-f_{\mathrm{ref}})^2
\right]
\right) \cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

這一步說明：理想 LPF 不是主角。主角是 deramping 先把主 replica 變窄，LPF 才有可能用固定通帶把它留住。

---

## 5. FFT-Based LPF Implementation

雖然上一節的數學模型寫成

$$
H_{\mathrm{LPF}}(f_\eta) =
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

但在實作上，sharp-cut LPF 採用 FFT-based 方式完成。若以 $u$ 表示目前 UFR 的離散處理軸，則先做

$$
\widetilde{S}_3(\tau,\nu_u) =
\mathcal{F}_{u}\left[
S_3(\tau,u)
\right]
$$

然後用 binary keep mask 將不需要的 bins 直接設為 0：

$$
\widetilde{S}_4(\tau,\nu_u) =
\widetilde{S}_3(\tau,\nu_u)\,
\widetilde{M}_{\mathrm{LPF}}(\nu_u)
$$

最後做 inverse FFT：

$$
S_4(\tau,u) =
\mathcal{F}_{u}^{-1}\left[
\widetilde{S}_3(\tau,\nu_u)\,
\widetilde{M}_{\mathrm{LPF}}(\nu_u)
\right]
$$

因此這裡要明確區分兩層：

- 理想模型：`rect` 通帶，用來定義 desired keep region
- 實作方法：Forward FFT -> zero out unwanted bins -> Inverse FFT

LPF 完成後，再進入 resampling。resampling 是 LPF 之後的下一步，不與 LPF 本身混寫。

---

## Physical Meaning

* mosaicking 只是在 extended azimuth-frequency axis 上把 replicas 攤開，並不會消除每塊 replica 的內部 quadratic phase curvature。
* phase 的二次項 $\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2$，正是主 replica 難以直接用固定頻帶窗截取的原因。
* deramping 的物理作用，是把主 replica 的 reference curvature 拿掉，使其從彎曲展寬的 chirp-like 表示轉成較平坦的表示。
* LPF 的物理作用，則是在這個已展平的 domain 上保留主 replica 的主要能量，並抑制其他 replicas。
* 在數學上，這件事可由理想 `rect` 通帶表示；在程式實作上，則以 FFT-domain zeroing 來實現 sharp cutoff。

---

## Final Result

起點 mosaicked signal：

$$
S_2(\tau,f_\eta) =
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right] \cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right) \cdot
\exp\left(
-j\psi_m(f_\eta)
\right)
$$

deramping filter：

$$
H_{\mathrm{de}}(f_\eta) =
\exp\left(
+j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

deramping 後：

$$
S_3(\tau,f_\eta) \approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
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
\psi_{0,m} + \psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
$$

LPF：

$$
H_{\mathrm{LPF}}(f_\eta) =
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

LPF 後：

$$
S_4(\tau,f_\eta) \approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
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
\psi_{0,m} + \psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)(f_\eta-f_{\mathrm{ref}})^2
\right]
\right) \cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

---

## Implementation Mapping

* 程式上通常先對 mosaicked spectrum 乘上 reference quadratic phase 的共軛補償，這對應 $H_{\mathrm{de}}(f_\eta)$。
* 再對 deramped spectrum 乘上固定通帶窗，這對應 $H_{\mathrm{LPF}}(f_\eta)$。
* 實作裡常寫成 `signal *= filter`，但理論文件不能只停在這種 shorthand，必須把乘完之後的輸出訊號真正展開寫出來。

---

## Limits And Applicability

* 本文使用局部二階 phase model，適用於主 replica 附近的有限頻帶；若通帶過寬，三階以上項可能不可忽略。
* 單一 deramping filter 只會精確匹配單一 reference curvature，因此它是為了分離某一個指定主 replica，而不是同時展平所有 replicas。
* LPF 是否真的有效，取決於 deramping 後主 replica 與其他 replicas 是否已在 baseband 上可分離。
