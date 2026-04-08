# Frequency-Time Deramping

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Main flow:
  [Azimuth Frequency UFR](./azimuth_freq_ufr.md),
  [Azimuth Time UFR](./azimuth_time_ufr.md)
- Related support note: [Azimuth Deramp LPF](./azimuth_deramp_LPF.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Local Frequency-Time Mapping](#1-local-frequency-time-mapping)
- [2. Frequency-Domain Deramping Filter Design](#2-frequency-domain-deramping-filter-design)
- [3. Time-Domain Deramping Filter Design](#3-time-domain-deramping-filter-design)
- [4. Relation To The TOPS SAR Main Flow](#4-relation-to-the-tops-sar-main-flow)
- [Physical Meaning](#physical-meaning)
- [Final Result](#final-result)

## Summary

* 這份文件的重點不是再證明「為什麼要 deramp」，而是明確回答「TOPS SAR 的 deramping filter 應該怎麼設計」。
* 若某個主 replica 在局部可用線性關係 $f_\eta = k_s\eta + f_{\eta_c}$ 描述，則 frequency-domain deramping filter 應設計成一個 quadratic phase，讓由該 chirp 對應的群延遲被拉回 reference 位置。
* 若同一個 replica 在 slow-time domain 可視為 azimuth chirp，則 time-domain deramping filter 應設計成其 conjugate quadratic phase，使 instantaneous Doppler 由斜線變成常數。
* frequency-domain deramping 與 time-domain deramping 本質上是在不同 domain 做同一件事：消去主 replica 的 reference chirp slope。
* 這份設計筆記應與 [azimuth_freq_folding.md](./azimuth_freq_folding.md) 及 [azimuth_deramp_LPF.md](./azimuth_deramp_LPF.md) 一起讀，前者說明 replicas 如何形成，後者說明 deramp 之後為什麼 LPF 才有效。

摘要中最重要的關鍵公式為

$$
H_{\mathrm{deramp},f}(f_\eta) =
\exp\left(
+j\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
\right)
$$

以及

$$
H_{\mathrm{deramp},t}(\eta) =
\exp\left(
-j\pi k_t(\eta-\eta_c)^2 - j2\pi f_{\eta_c}\eta
\right)
$$

---

## Problem Definition

本文件要回答四件事：

1. 若主 replica 的局部 chirp slope 已知，frequency-domain deramping filter 應該如何設計。
2. 這個 frequency-domain filter 為什麼能把原本的斜線關係 $f_\eta(\eta)$ 拉平成 reference time。
3. 若改在 slow-time domain 設計 deramping filter，時間域的 filter 形式應該是什麼。
4. frequency-domain 與 time-domain 兩種設計方式之間，物理意義上如何對應。

---

## Derivation Highlights

* 先以主 replica 的局部線性關係 $f_\eta = k_s\eta + f_{\eta_c}$ 建立 frequency-time mapping。
* 再從 group delay compensation 的角度推導 frequency-domain deramping filter。
* 接著從 instantaneous Doppler compensation 的角度推導 time-domain deramping filter。
* 最後把兩者和 [azimuth_deramp_LPF.md](./azimuth_deramp_LPF.md) 裡的 reference quadratic curvature 概念對齊。

---

## Symbols And Assumptions

* $\eta$：方位向 slow time
* $f_\eta$：azimuth frequency
* $f_{\eta_c}$：reference slow time $\eta_c$ 對應的 azimuth frequency
* $\eta_c$：reference slow time
* $k_s$：在 frequency-time mapping 下的局部 slope，滿足 $f_\eta = k_s\eta + f_{\eta_c}$
* $k_t$：time-domain azimuth chirp 的局部 slope
* $H_{\mathrm{deramp},f}(f_\eta)$：frequency-domain deramping filter
* $H_{\mathrm{deramp},t}(\eta)$：time-domain deramping filter
* $\phi_f(f_\eta)$：frequency-domain deramping filter 的 phase
* $\phi_t(\eta)$：time-domain deramping filter 的 phase
* $\Delta\eta(f_\eta)$：frequency-domain filter 對群延遲所造成的補償量
* $\Delta f_\eta(\eta)$：time-domain filter 對 instantaneous Doppler 所造成的補償量

假設如下：

* 主 replica 在局部頻帶內可用單一 chirp slope 近似。
* 本文關注的是 filter 設計法則，因此只保留局部一階 frequency-time mapping 與局部二次 phase。
* 對主 replica 而言，$k_s$ 與 $k_t$ 都代表 reference chirp slope，只是分別在 frequency-domain 與 time-domain 表示。

---

## 1. Local Frequency-Time Mapping

若主 replica 在局部頻帶內可視為一條線性 chirp，則其 azimuth frequency 與 slow time 的關係可寫成

$$
f_\eta = k_s\eta + f_{\eta_c}
$$

因此 slow time 可反解為

$$
\eta = \frac{1}{k_s}(f_\eta-f_{\eta_c})
$$

這一步的意義是：每一個 $f_\eta$ 都對應到一個由 chirp slope 決定的 reference slow time。若要在 frequency-domain 上把這條斜線拉平，就必須設計一個 filter，使這個對應的 time offset 被抵消掉。

---

## 2. Frequency-Domain Deramping Filter Design

設 frequency-domain deramping filter 的 phase 為

$$
\phi_f(f_\eta) =
\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
$$

則對應的 filter 為

$$
H_{\mathrm{deramp},f}(f_\eta) =
\exp\left(
+j\phi_f(f_\eta)
\right) =
\exp\left(
+j\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
\right)
$$

frequency-domain 相位乘法所對應的群延遲補償量為

$$
\Delta\eta(f_\eta) =
-\frac{1}{2\pi}\frac{d\phi_f(f_\eta)}{df_\eta}
$$

直接微分可得

$$
\frac{d\phi_f(f_\eta)}{df_\eta} =
2\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})
$$

因此

$$
\Delta\eta(f_\eta) =
-\frac{1}{k_s}(f_\eta-f_{\eta_c})
$$

又因為原本的 slow-time mapping 是

$$
\eta_{\mathrm{old}} =
\frac{1}{k_s}(f_\eta-f_{\eta_c})
$$

所以經過 filter 後的新位置為

$$
\eta_{\mathrm{new}} =
\eta_{\mathrm{old}} + \Delta\eta(f_\eta)
= \frac{1}{k_s}(f_\eta-f_{\eta_c}) - \frac{1}{k_s}(f_\eta-f_{\eta_c})
= 0
$$

因此本步的 fully expanded closed-form 結論是：

$$
\eta_{\mathrm{new}} = 0
$$

也就是說，frequency-domain deramping filter 的設計原理，就是把原本由 chirp slope 對應出的 time offset 完整拉回 reference location。

---

## 3. Time-Domain Deramping Filter Design

若改在 slow-time domain 看同一個主 replica，其局部 instantaneous Doppler 可寫成

$$
f_\eta(\eta) = k_t(\eta-\eta_c) + f_{\eta_c}
$$

這表示對應的 time-domain replica 可視為一個以 $\eta_c$ 為 reference center 的 azimuth chirp。其 phase 可寫成

$$
\phi_{\mathrm{sig}}(\eta) =
\pi k_t(\eta-\eta_c)^2 + 2\pi f_{\eta_c}\eta
$$

因此對應的 time-domain deramping filter 應取其 conjugate quadratic phase：

$$
H_{\mathrm{deramp},t}(\eta) =
\exp\left(
-j\phi_{\mathrm{sig}}(\eta)
\right) =
\exp\left(
-j\pi k_t(\eta-\eta_c)^2 - j2\pi f_{\eta_c}\eta
\right)
$$

若將 filter phase 記為

$$
\phi_t(\eta) =
-\pi k_t(\eta-\eta_c)^2 - 2\pi f_{\eta_c}\eta
$$

則該 filter 造成的 instantaneous Doppler 補償量為

$$
\Delta f_\eta(\eta) =
\frac{1}{2\pi}\frac{d\phi_t(\eta)}{d\eta}
$$

直接微分可得

$$
\frac{d\phi_t(\eta)}{d\eta} =
-2\pi k_t(\eta-\eta_c) - 2\pi f_{\eta_c}
$$

因此

$$
\Delta f_\eta(\eta) =
-k_t(\eta-\eta_c) - f_{\eta_c}
$$

原本的 instantaneous Doppler 為

$$
f_{\eta,\mathrm{old}}(\eta) =
k_t(\eta-\eta_c) + f_{\eta_c}
$$

經過 time-domain deramping 後的新 instantaneous Doppler 為

$$
f_{\eta,\mathrm{new}}(\eta) =
f_{\eta,\mathrm{old}}(\eta) + \Delta f_\eta(\eta)
= k_t(\eta-\eta_c) + f_{\eta_c} - k_t(\eta-\eta_c) - f_{\eta_c}
= 0
$$

因此本步的 fully expanded closed-form 結論是：

$$
f_{\eta,\mathrm{new}}(\eta) = 0
$$

也就是說，time-domain deramping 的設計原理，是把原本線性變化的 instantaneous Doppler 直接展平成常數。

---

## 4. Relation To The TOPS SAR Main Flow

[azimuth_freq_folding.md](./azimuth_freq_folding.md) 說明了 TOPS 為什麼會在 azimuth-frequency domain 出現 folded replicas。  
[azimuth_deramp_LPF.md](./azimuth_deramp_LPF.md) 則說明了在 mosaicking 之後，為什麼必須再做 deramping，才能讓 LPF 有效保留主 replica。

把那份文件的 quadratic curvature notation 對到這裡，可以得到：

$$
\psi_{2,\mathrm{ref}} \quad \Longleftrightarrow \quad -\pi\frac{1}{k_s}
$$

這裡的精確正負號取決於你採用的 Fourier transform sign convention，但物理意義不變：

* frequency-domain deramping：用一個 quadratic phase 來抵消主 replica 對應的 group-delay slope
* time-domain deramping：用一個 conjugate chirp 來抵消主 replica 對應的 instantaneous-Doppler slope

因此它們都是在設計同一個 reference curvature 的補償器，只是作用 domain 不同。

---

## Physical Meaning

* $k_s$ 或 $k_t$ 代表主 replica 在局部頻帶內的 chirp slope。
* frequency-domain deramp filter 不直接「消掉 aliasing」，它是把主 replica 的群延遲斜率拉回 reference location。
* time-domain deramp filter 不直接「做 LPF」，它是先把 chirp phase 拿掉，使訊號變成近似 constant-frequency representation。
* 之所以後續 LPF 能有效，是因為主 replica 已先被這個 deramping filter 展平。

---

## Final Result

局部 frequency-time mapping：

$$
f_\eta = k_s\eta + f_{\eta_c},
\qquad
\eta = \frac{1}{k_s}(f_\eta-f_{\eta_c})
$$

frequency-domain deramping filter：

$$
H_{\mathrm{deramp},f}(f_\eta) =
\exp\left(
+j\pi\frac{1}{k_s}(f_\eta-f_{\eta_c})^2
\right)
$$

其對應的群延遲補償量：

$$
\Delta\eta(f_\eta) =
-\frac{1}{k_s}(f_\eta-f_{\eta_c})
$$

補償後的新 slow-time location：

$$
\eta_{\mathrm{new}} = 0
$$

time-domain deramping filter：

$$
H_{\mathrm{deramp},t}(\eta) =
\exp\left(
-j\pi k_t(\eta-\eta_c)^2 - j2\pi f_{\eta_c}\eta
\right)
$$

其對應的 Doppler 補償量：

$$
\Delta f_\eta(\eta) =
-k_t(\eta-\eta_c) - f_{\eta_c}
$$

補償後的新 instantaneous Doppler：

$$
f_{\eta,\mathrm{new}}(\eta) = 0
$$

---

## Implementation Mapping

* 若你手上已有主 replica 在 frequency-domain 的局部 slope estimate，則最直接的設計法是用 $H_{\mathrm{deramp},f}(f_\eta)$。
* 若你手上已有時間域參考 chirp，則最直接的設計法是用其 conjugate chirp 當作 $H_{\mathrm{deramp},t}(\eta)$。
* 在實作中通常只要求主 replica 在 reference band 上被展平，因此 $k_s$ 或 $k_t$ 往往是局部參數，不必強求全頻帶精確成立。

---

## Limits And Applicability

* 本文的設計法則建立在局部線性 chirp 假設上；若主 replica 的 slope 在通帶內變化太大，單一 quadratic phase 可能不足。
* 這裡的 frequency-domain 與 time-domain 濾波器是理論上的 reference design；實際系統中可能還需乘上窗函數、做相位中心平移或把 reference shift 併入 mosaicking。
* 精確的正負號與常數項，會依你的 Fourier sign convention 與 reference center 定義而改變，但「filter phase 必須是主 replica chirp phase 的 reference 補償」這個原則不變。
