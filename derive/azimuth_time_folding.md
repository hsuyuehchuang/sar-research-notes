# Azimuth Time Folding

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Previous main step: [Azimuth Compression](./azimuth_compression.md)
- Next main step: [Azimuth Time UFR](./azimuth_time_ufr.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Derivation Highlights](#derivation-highlights)
- [Symbols And Assumptions](#symbols-and-assumptions)
- [1. Geometric Starting Point: Ground Motion Of The Beam Footprint](#1-geometric-starting-point-ground-motion-of-the-beam-footprint)
- [2. From Ground Span To Equivalent Azimuth-Time Span](#2-from-ground-span-to-equivalent-azimuth-time-span)
- [3. Paper Parameterization Of The Stretch Factor](#3-paper-parameterization-of-the-stretch-factor)
- [4. Required Length For Linear Convolution](#4-required-length-for-linear-convolution)
- [5. Unambiguous Time Window In FFT Implementation](#5-unambiguous-time-window-in-fft-implementation)
- [6. Why Finite-Length FFT Becomes Circular Convolution](#6-why-finite-length-fft-becomes-circular-convolution)
- [7. Wrap-Around Location Formula](#7-wrap-around-location-formula)
- [Physical Meaning](#physical-meaning)
- [Final Result](#final-result)

## Summary

* TOPS 的 beam steering 不直接生成 time-domain ghost；它先把單一 burst 內被照射到的目標集合拉成更長的有效方位時間跨度 $T_{\mathrm{focus,TOPS}}$。
* 若這個有效時間跨度再加上 azimuth matched-filter 長度後，超過 FFT block 所對應的無模糊時間窗口 $T_{\mathrm{window}}$，則 FFT 壓縮不再等價於線性卷積，而會退化為 circular convolution。
* azimuth-time wrap-around 的直接數學來源，不是 beam steering 本身，而是有限長 DFT 所隱含的週期延拓。
* 這個問題必須拆成幾何跨度、等效時間跨度、線性卷積需求長度、FFT 有效窗口、circular convolution 五個模組來看，否則會把「頻譜展寬」與「時間折回」混成同一件事。
* 真正需要 carried forward 的結果是 $T_{\mathrm{focus,TOPS}}$、$L_{\mathrm{req,TOPS}}$、$T_{\mathrm{window}}$ 與 wrap-around location formula。

摘要中最重要的關鍵公式為

$$
{\color{red}
T_{\mathrm{focus,TOPS}} =
T_b\left(
1+\frac{v_s}{v_p}
\right)
}
$$

以及

$$
{\color{red}
I_{\mathrm{circ}}(\eta) =
\sum_{m=-\infty}^{\infty}
I_{\mathrm{lin}}(\eta-mT_{\mathrm{window}})
}
$$

---

## Problem Definition

本文件要回答三件事：

1. 為什麼 TOPS 在單一 burst 內會形成比 stripmap 更長的有效目標時間跨度。
2. 為什麼這個更長的時間跨度，在以 FFT 做方位壓縮時，會轉成 azimuth-time wrap-around。
3. wrap-around 的 ghost location 應該如何以 $T_{\mathrm{window}}$ 的週期性折回公式來寫。

---

## Derivation Highlights

* 先從 beam footprint 的地面移動出發，把 stripmap 與 TOPS 的地面跨度分別寫成 closed form。
* 再把地面跨度映射回等效方位慢時間跨度，明確得到 $T_{\mathrm{focus,strip}}$ 與 $T_{\mathrm{focus,TOPS}}$。
* 接著把方位 matched filtering 所需的線性卷積長度寫成 $L_{\mathrm{req}}=T_{\mathrm{focus}}+T_{\mathrm{ref}}$。
* 然後用 FFT block size 與 PRF 寫出無模糊時間窗口 $T_{\mathrm{window}}$，並說明何時會失去線性卷積條件。
* 最後把 FFT 實作下的 circular convolution 顯式寫成週期和，推出 wrap-around 的 ghost location。

---

## Symbols And Assumptions

* $\eta$：方位向慢時間
* $T_b$：單一 burst 的原始慢時間長度
* $v_p$：平台沿方位向的等效速度
* $v_s$：beam steering 在地面上對應的等效掃描速度
* $v_{\mathrm{scan}}$：paper notation 中的 scan speed，與本文的 $v_s$ 對應
* $\omega_{rot}$：beam rotation rate
* $y_{\mathrm{strip}}(\eta)$：stripmap 模式下 beam center 的地面方位位置
* $y_{\mathrm{TOPS}}(\eta)$：TOPS 模式下 beam center 的地面方位位置
* $\Delta y_{\mathrm{strip}}$：單一 burst 內 stripmap footprint 掃過的地面跨度
* $\Delta y_{\mathrm{TOPS}}$：單一 burst 內 TOPS footprint 掃過的地面跨度
* $T_{\mathrm{focus,strip}}$：stripmap 的等效目標時間跨度
* $T_{\mathrm{focus,TOPS}}$：TOPS 的等效目標時間跨度
* $T_{\mathrm{ref}}$：方位 matched-filter 的時間長度
* $L_{\mathrm{req}}$：實現線性卷積所需的總時間長度
* $\alpha$：TOPS focus-time stretching factor
* $k_a$：平台幾何造成的 azimuth FM rate
* $k_{rot}$：beam rotation 引入的 Doppler-rate term
* $k_t$：paper notation 中的 effective TOPS chirp rate
* $N_a$：方位向 FFT 點數
* $\mathrm{PRF}$：脈衝重複頻率
* $T_{\mathrm{window}}=N_a/\mathrm{PRF}$：FFT 對應的基本時間窗口
* $I_{\mathrm{lin}}(\eta)$：理想線性卷積輸出
* $I_{\mathrm{circ}}(\eta)$：有限長 FFT 所對應的 circular convolution 輸出

假設如下：

* 本文只關注 azimuth-time folding / wrap-around 的核心機制，不展開更高階的 beam pattern truncation 與 burst-edge weighting。
* $v_s$ 用來代表 beam steering 在地面上的等效掃描速度；這裡重點是 footprint 被拉長，而不是機電控制的精確定義。
* 方位壓縮以有限長 FFT 實作，因此若無足夠 zero-padding，就必然對應 circular convolution。

---

## 1. Geometric Starting Point: Ground Motion Of The Beam Footprint

若波束不掃描，stripmap 模式下 beam center 的地面方位位置為

$$
y_{\mathrm{strip}}(\eta) = v_p\eta
$$

在 TOPS 模式下，beam center 還有額外的方位掃描，因此可寫成

$$
y_{\mathrm{TOPS}}(\eta) = \left(
v_p+v_s
\right)\eta
$$

對長度為 $T_b$ 的單一 burst 而言，stripmap 與 TOPS 在地面上掃過的跨度分別為

$$
\Delta y_{\mathrm{strip}} =
y_{\mathrm{strip}}(T_b)-y_{\mathrm{strip}}(0) =
v_pT_b
$$

$$
{\color{red}
\Delta y_{\mathrm{TOPS}} =
y_{\mathrm{TOPS}}(T_b)-y_{\mathrm{TOPS}}(0) =
\left(
v_p+v_s
\right)T_b
}
$$

因此本步結束後的 closed-form 結論是：在同一個 burst 內，TOPS 比 stripmap 多掃過了額外的地面距離 $v_sT_b$。這件事本身還不是 wrap-around，但它已經把後面會被聚焦的目標集合拉長了。

---

## 2. From Ground Span To Equivalent Azimuth-Time Span

地面方位座標 $y$ 與平台慢時間的對應關係可寫成

$$
\eta_{\mathrm{tar}} = \frac{y}{v_p}
$$

因此地面跨度可以直接改寫成等效的目標時間跨度。

對 stripmap，

$$
T_{\mathrm{focus,strip}} =
\frac{\Delta y_{\mathrm{strip}}}{v_p} =
\frac{v_pT_b}{v_p} =
T_b
$$

對 TOPS，

$$
{\color{red}
T_{\mathrm{focus,TOPS}} =
\frac{\Delta y_{\mathrm{TOPS}}}{v_p} =
\frac{\left(
v_p+v_s
\right)T_b}{v_p} =
T_b\left(
1+\frac{v_s}{v_p}
\right)
}
$$

因此

$$
T_{\mathrm{focus,TOPS}} > T_{\mathrm{focus,strip}}
\quad \text{whenever} \quad
v_s > 0
$$

這一步回答的是幾何問題：TOPS 並不是直接製造 ghost，而是先把同一個 burst 內必須被完整聚焦的目標時間範圍拉長。

---

## 3. Paper Parameterization Of The Stretch Factor

若用 paper notation 把 TOPS 的時間展寬寫成 stretch factor，則

$$
{\color{red}
\alpha \triangleq
\frac{T_{\mathrm{focused}}}{T_{\mathrm{burst}}}
}
$$

由前兩步的幾何結果可直接得到

$$
{\color{red}
\alpha =
\frac{v_p+v_{\mathrm{scan}}}{v_p}
=
1+\frac{v_{\mathrm{scan}}}{v_p}
}
$$

若再改用 paper 中的 rate relation，並代入

$$
k_a \approx -\frac{2v_p^2}{\lambda R_0},
\qquad
k_{rot} \approx \frac{2v_p\omega_{rot}}{\lambda}
$$

則

$$
\frac{k_{rot}}{k_a} =
\frac{\frac{2v_p\omega_{rot}}{\lambda}}
-\frac{2v_p^2}{\lambda R_0}
=
-\frac{R_0\omega_{rot}}{v_p}
=
-\frac{v_{\mathrm{scan}}}{v_p}
$$

因此 stretch factor 也可寫成

$$
{\color{red}
\alpha =
1-\frac{k_{rot}}{k_a}
=
\frac{k_a-k_{rot}}{k_a}
}
$$

在 TOPS mode 中，通常 $k_a k_{rot} < 0$，因此

$$
{\color{red}
\alpha > 1
}
$$

這就把幾何版的「focus time 變長」與 paper 版的「rotation rate 改寫等效 chirp law」接起來了。

若再把 effective TOPS chirp rate 寫成 paper 常用形式，則

$$
{\color{red}
k_t = \frac{k_{rot}}{\alpha}
=
\frac{k_{rot}}{1-\frac{k_{rot}}{k_a}}
=
\frac{k_a k_{rot}}{k_a-k_{rot}}
}
$$

因此本步結束後，有兩個必須 carried forward 的 paper-style 結果：

$$
\alpha = 1-\frac{k_{rot}}{k_a}
$$

$$
k_t = \frac{k_a k_{rot}}{k_a-k_{rot}}
$$

這兩個式子分別對應：
- TOPS burst 被拉長多少
- TOPS 在 paper notation 下的等效 chirp rate 應該怎麼寫

---

## 4. Required Length For Linear Convolution

設方位 matched-filter 的時間長度為 $T_{\mathrm{ref}}$。若要對整批分布在慢時間上的目標做正確的線性卷積，總處理長度至少必須滿足

$$
L_{\mathrm{req}} = T_{\mathrm{focus}} + T_{\mathrm{ref}}
$$

因此對 stripmap 與 TOPS，所需長度分別為

$$
L_{\mathrm{req,strip}} =
T_{\mathrm{focus,strip}} + T_{\mathrm{ref}} =
T_b + T_{\mathrm{ref}}
$$

$$
{\color{red}
L_{\mathrm{req,TOPS}} =
T_{\mathrm{focus,TOPS}} + T_{\mathrm{ref}} =
\alpha T_b + T_{\mathrm{ref}}
}
$$

因此本步的重點不是 matched-filter 的精確 closed form，而是：TOPS 先把 $T_{\mathrm{focus}}$ 變長，然後這個變長的結果直接推高線性卷積所需的總窗口。

---

## 5. Unambiguous Time Window In FFT Implementation

若方位壓縮用 $N_a$ 點 FFT 實作，且慢時間取樣率為 $\mathrm{PRF}$，則 FFT 基本時間窗口為

$$
{\color{red}
T_{\mathrm{window}} = \frac{N_a}{\mathrm{PRF}}
}
$$

要讓 FFT-based matched filtering 仍等價於線性卷積，至少要滿足

$$
T_{\mathrm{window}} \ge L_{\mathrm{req}}
$$

對 TOPS 而言，真正需要的條件是

$$
T_{\mathrm{window}} \ge
T_b\left(
1+\frac{v_s}{v_p}
\right) + T_{\mathrm{ref}}
$$

一旦反過來出現

$$
{\color{red}
T_{\mathrm{window}} <
\alpha T_b + T_{\mathrm{ref}}
}
$$

就表示 FFT block 已不足以承載正確的線性卷積輸出。wrap-around 的必要條件就在這一步被寫出來了。

---

## 6. Why Finite-Length FFT Becomes Circular Convolution

若用 FFT 做 matched filtering，運算形式為

$$
I_{\mathrm{circ}}(\eta) =
\mathrm{IFFT}\left[
\mathrm{FFT}\left[s(\eta)\right] \cdot
\mathrm{FFT}\left[h(\eta)\right]
\right]
$$

在有限長 DFT 下，這個結果不等於無條件的線性卷積，而是等於理想線性卷積的週期延拓和：

$$
{\color{red}
I_{\mathrm{circ}}(\eta) =
\sum_{m=-\infty}^{\infty}
I_{\mathrm{lin}}(\eta-mT_{\mathrm{window}})
}
$$

這裡

$$
I_{\mathrm{lin}}(\eta) =
\int_{-\infty}^{\infty}
s(\xi)h(\eta-\xi)\,d\xi
$$

因此只要 $I_{\mathrm{lin}}(\eta)$ 的有效支撐超出長度 $T_{\mathrm{window}}$ 的基本區間，超出的部分就會被以 $T_{\mathrm{window}}$ 為週期折回來。wrap-around 的直接數學來源就是這個週期和，而不是 beam steering 本身。

---

## 7. Wrap-Around Location Formula

設某個目標在理想線性卷積下的聚焦中心位於 $\eta_c$。若它落在 FFT 的基本時間窗口之外，則實際 circular convolution 輸出中的 ghost 位置滿足

$$
\eta_{\mathrm{ghost}} = \eta_c - mT_{\mathrm{window}}
$$

其中 $m$ 是某個整數，並且必須選到使得

$$
{\color{red}
\eta_{\mathrm{ghost}} \in
\left[
-\frac{T_{\mathrm{window}}}{2},
\frac{T_{\mathrm{window}}}{2}
\right]
}
$$

若寫成條件句，就是：

$$
\left|
\eta_c
\right| >
\frac{T_{\mathrm{window}}}{2}
\quad \Longrightarrow \quad
\text{response wraps back into the principal window}
$$

因此 wrap-around 不是「新的假訊號被生成」，而是原本應出現在窗口外的線性卷積響應，被 DFT 的週期性強制搬回主區間。

---

## Physical Meaning

* $v_s$ 的作用是拉長同一個 burst 內被照射到的地面目標集合，因此先把幾何上的 footprint 變大。
* 幾何 footprint 變大後，等效目標時間跨度 $T_{\mathrm{focus,TOPS}}$ 也同步變大。
* 在 paper notation 中，這件事等價於說 $\alpha = 1-k_{rot}/k_a > 1$，因此 TOPS 的 effective focused span 一定比 burst 本身更長。
* 同時，$k_t = k_{rot}/\alpha$ 提供了後續 paper-style azimuth compression / deramping 表示法所需的有效 chirp-rate 參數。
* FFT block 只能表示一個基本區間長度為 $T_{\mathrm{window}}$ 的時間軸，因此當線性卷積輸出比這個窗口更長時，超出的部分只能被週期折回。
* 所以 TOPS 比 stripmap 更容易出現 azimuth-time folding / wrap-around，不是因為它有什麼神祕新機制，而是因為它更容易讓 $L_{\mathrm{req}}$ 超出 $T_{\mathrm{window}}$。

---

## Final Result

TOPS 的等效目標時間跨度：

$$
{\color{red}
T_{\mathrm{focus,TOPS}} =
\alpha T_b =
T_b\left(
1+\frac{v_s}{v_p}
\right) =
T_b\left(
1-\frac{k_{rot}}{k_a}
\right)
}
$$

paper-style stretch factor：

$$
{\color{red}
\alpha =
1+\frac{v_{\mathrm{scan}}}{v_p}
=
1-\frac{k_{rot}}{k_a}
}
$$

paper-style effective chirp rate：

$$
{\color{red}
k_t = \frac{k_{rot}}{\alpha} =
\frac{k_a k_{rot}}{k_a-k_{rot}}
}
$$

TOPS 的線性卷積需求長度：

$$
{\color{red}
L_{\mathrm{req,TOPS}} =
\alpha T_b + T_{\mathrm{ref}}
}
$$

FFT 的無模糊時間窗口：

$$
{\color{red}
T_{\mathrm{window}} = \frac{N_a}{\mathrm{PRF}}
}
$$

wrap-around 的必要條件：

$$
{\color{red}
T_{\mathrm{window}} < L_{\mathrm{req,TOPS}}
}
$$

FFT 壓縮對應的 circular convolution：

$$
{\color{red}
I_{\mathrm{circ}}(\eta) =
\sum_{m=-\infty}^{\infty}
I_{\mathrm{lin}}(\eta-mT_{\mathrm{window}})
}
$$

ghost location formula：

$$
{\color{red}
\eta_{\mathrm{ghost}} = \eta_c - mT_{\mathrm{window}},
\qquad
\eta_{\mathrm{ghost}} \in
\left[
-\frac{T_{\mathrm{window}}}{2},
\frac{T_{\mathrm{window}}}{2}
\right]
}
$$

---

## Implementation Mapping

* 在程式實作上，若方位向 block 長度固定，則 $N_a$ 與 $\mathrm{PRF}$ 直接決定了 $T_{\mathrm{window}}$。
* 若沒有額外 zero-padding 或 overlap-save / overlap-add 類處理，FFT-based matched filtering 就會自動落入 circular convolution。
* 因此 suppress wrap-around 的本質做法，不是事後修 ghost，而是事前讓 $T_{\mathrm{window}}$ 足夠大，或把長卷積拆成正確的線性卷積實作。

---

## Limits And Applicability

* 本文用 $v_s$ 表示 beam steering 的等效地面掃描速度，重點在因果鏈，不在機電幾何的最細節定義。
* 本文將 matched-filter 長度寫成 $T_{\mathrm{ref}}$，只要求它能代表 azimuth compression kernel 的有效時間支撐，未展開到更細的 chirp parameter closed form。
* 若實作中已採用充分 zero-padding、分段線性卷積或其他 anti-wrap-around 設計，則本文的 wrap-around 條件可以被工程上避免，但其數學來源不變。
