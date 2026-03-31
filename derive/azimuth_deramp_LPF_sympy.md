**Azimuth Deramping, LPF, And Reramping Derivation By SymPy**

**結論**

- mosaicking 後的 $S_2(\tau,f_\eta)$ 雖然已經把 replicas 在 extended azimuth-frequency axis 上展開，但每個 replica 仍帶有由 $D_m(f_\eta)$ 決定的非線性相位。
- 對主 replica 的相位在 reference frequency $f_{\mathrm{ref}}$ 附近做二階泰勒展開後，可把它寫成常數項、一次項與二次項的和；其中二次項就是 deramping 要抵消的目標。
- SymPy 可直接驗證：若把 exponent 中的實數 phase function 記為 $\psi_m(f_\eta)$，則其二次曲率係數等於 $\psi_{2,m}=\psi_m''(f_{\mathrm{ref}})/2$；因此 deramping filter 本質上就是 reference quadratic phase 的共軛補償。
- deramping 後，主 replica 的二次曲率近似被拿掉，能量在 baseband 上變集中；此時 LPF 才能有效地把目標 replica 濾出來。
- reramping 則把 reference quadratic phase 乘回去，使保留下來的主 replica 回到後續 azimuth compression 所需的相位模型。

---

**問題定義**

本文件要回答的是：

1. 從 mosaicked spectrum $S_2(\tau,f_\eta)$ 出發，主 replica 的相位如何嚴謹地近似成二次形式？
2. deramping filter 應該抵消哪一個相位項？
3. 為什麼必須先 deramp，再做 LPF？
4. 為什麼 LPF 後還需要 reramping？

---

**推導重點**

- 以 [azimuth_freq_ufr.md](/home/hsuyueh.chuang/Desktop/vscode/github/sar_tops_mode/derive/azimuth_freq_ufr.md) 的 $S_2(\tau,f_\eta)$ 為起點。
- 抽出第 $m$ 個 replica exponent 中的實數 phase function $\psi_m(f_\eta)$。
- 用 SymPy 對 $\psi_m(f_\eta)$ 在 $f_{\mathrm{ref}}$ 附近做二階泰勒展開。
- 從符號微分結果明確定義二次曲率係數 $\psi_{2,m}$。
- 以 $\psi_{2,m_0}$ 為 reference curvature 建立 deramping、LPF、reramping 三步鏈式表示。

---

**符號與假設**

- $S_2(\tau,f_\eta)$：mosaicking 後的 UFR spectrum
- $S_{2,m}(\tau,f_\eta)$：第 $m$ 個 replica
- $m_0$：欲保留的主 replica 索引
- $f_{\mathrm{ref}}$：主 replica 的 reference frequency
- $D_m(f_\eta)=D(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}})$
- $\psi_m(f_\eta)$：第 $m$ 個 replica exponent 中的實數 azimuth phase function
- $H_{\mathrm{de}}(f_\eta)$：deramping filter
- $H_{\mathrm{LPF}}(f_\eta)$：low-pass filter
- $H_{\mathrm{re}}(f_\eta)$：reramping filter

假設：

- 在主 replica 的局部頻帶內，二階泰勒展開已足以描述主要 phase curvature。
- 主 replica 可由單一 reference frequency $f_{\mathrm{ref}}$ 與單一 reference curvature 近似補償。
- LPF 通帶選擇得足以保留 deramping 後的主 replica，並排除其他 replicas 的主要能量。

---

**逐步推導**

**1. 起點：mosaicked spectrum**

由 [azimuth_freq_ufr.md](/home/hsuyueh.chuang/Desktop/vscode/github/sar_tops_mode/derive/azimuth_freq_ufr.md) 可得

$$
S_2(\tau,f_\eta)
=
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
S_{2,m}(\tau,f_\eta)
$$

其中第 $m$ 個 replica 可寫成

$$
S_{2,m}(\tau,f_\eta)
=
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right]
$$

$$
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
\cdot
\exp\left(
-j\psi_m(f_\eta)
\right)
$$

將 exponent 寫成 $-j\psi_m(f_\eta)$，則

$$
\psi_m(f_\eta)
=
\frac{4\pi R_0f_0}{c}D_m(f_\eta)
+2\pi\left(f_\eta-m\cdot\mathrm{PRF}\right)\eta_c
$$

本步結束後的訊號 closed form 為

$$
S_2(\tau,f_\eta)
=
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right]
$$

$$
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
\cdot
\exp\left(
-j\psi_m(f_\eta)
\right)
$$

---

**2. 用 SymPy 對相位做局部展開**

令

$$
u_m=f_\eta-m\cdot\mathrm{PRF}
$$

以及

$$
D_m(f_\eta)
=
\sqrt{
1-\frac{c^2u_m^2}{4V_{\mathrm{eff}}^2f_0^2}
}
$$

則實數 phase function 可寫成

$$
\psi_m(f_\eta)
=
\frac{4\pi R_0f_0}{c}D_m(f_\eta)
+2\pi u_m\eta_c
$$

因此原始 exponent 為

$$
\exp\left(
-j\psi_m(f_\eta)
\right)
$$

對 $\psi_m(f_\eta)$ 在 $f_{\mathrm{ref}}$ 附近做二階泰勒展開：

$$
\psi_m(f_\eta)
\approx
\psi_m(f_{\mathrm{ref}})
+\psi_m'(f_{\mathrm{ref}})(f_\eta-f_{\mathrm{ref}})
+\frac{\psi_m''(f_{\mathrm{ref}})}{2}(f_\eta-f_{\mathrm{ref}})^2
$$

因此可定義

$$
\psi_{0,m}
=
\psi_m(f_{\mathrm{ref}})
$$

$$
\psi_{1,m}
=
\psi_m'(f_{\mathrm{ref}})
$$

$$
\psi_{2,m}
=
\frac{\psi_m''(f_{\mathrm{ref}})}{2}
$$

使得

$$
\psi_m(f_\eta)
\approx
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
$$

因此第 $m$ 個 replica 在局部頻帶內可近似為

$$
S_{2,m}(\tau,f_\eta)
\approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right]
$$

$$
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
\cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
$$

本步結束後的訊號 closed form 為

$$
S_2(\tau,f_\eta)
\approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
S_{2,m}(\tau,f_\eta)
$$

$$
S_{2,m}(\tau,f_\eta)
\approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
$$

$$
\cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
$$

---

**3. SymPy 驗證的一次與二次係數**

對 range-Doppler 主要 phase 部分

$$
\psi_m(f_\eta)
=
\frac{4\pi R_0f_0}{c}D_m(f_\eta)
+2\pi\left(f_\eta-m\cdot\mathrm{PRF}\right)\eta_c
$$

做符號微分，SymPy 可驗證

$$
\psi_m'(f_\eta)
=
-\frac{\pi R_0c}{V_{\mathrm{eff}}^2f_0}
\frac{u_m}{D_m(f_\eta)}
+2\pi\eta_c
$$

以及

$$
\psi_m''(f_\eta)
=
-\frac{\pi R_0c}{V_{\mathrm{eff}}^2f_0}
\frac{1}{D_m^3(f_\eta)}
$$

因此二次曲率係數可寫成

$$
\psi_{2,m}
=
-\frac{\pi R_0c}{2V_{\mathrm{eff}}^2f_0}
\frac{1}{D_m^3(f_{\mathrm{ref}})}
$$

這個結果很關鍵，因為它說明主 replica 的 phase curvature 並不是任意假設出來的，而是直接來自 $D_m(f_\eta)$ 的二階導數。

因此第 $m$ 個 replica 的局部 closed form 可再明確寫成

$$
S_{2,m}(\tau,f_\eta)
\approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
$$

$$
\cdot
\exp\left(
-j\left[
\psi_{0,m}
+\left(
-\frac{\pi R_0c}{V_{\mathrm{eff}}^2f_0}
\frac{u_m}{D_m(f_{\mathrm{ref}})}
+2\pi\eta_c
\right)
(f_\eta-f_{\mathrm{ref}})
-\frac{\pi R_0c}{2V_{\mathrm{eff}}^2f_0}
\frac{(f_\eta-f_{\mathrm{ref}})^2}{D_m^3(f_{\mathrm{ref}})}
\right]
\right)
$$

本步結束後的訊號 closed form 仍為近似展開後的 $S_2(\tau,f_\eta)$，只是其中每個 replica 的一次項與二次項係數已由 SymPy 明確求出。

---

**4. Deramping 的數學來源**

若欲保留的主 replica 為 $m=m_0$，則最自然的 reference quadratic phase 應取自其局部二次項：

$$
\psi_{2,\mathrm{ref}}
=
\psi_{2,m_0}
$$

因此 deramping filter 定義為主 replica 二次項的共軛補償：

$$
H_{\mathrm{de}}(f_\eta)
=
\exp\left(
+j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

若採常見 chirp-rate 表示，亦可寫成

$$
H_{\mathrm{de}}(f_\eta)
=
\exp\left(
+j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
\right)
$$

其中 $K_{\mathrm{ref}}$ 與 $\psi_{2,\mathrm{ref}}$ 只是不同記號下的同一個 reference curvature。

經 deramping 後，

$$
S_3(\tau,f_\eta)
=
S_2(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta)
$$

因此第 $m$ 個 replica 變成

$$
S_{3,m}(\tau,f_\eta)
=
S_{2,m}(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta)
$$

其殘餘二次曲率為

$$
\tilde{\psi}_{2,m}
=
\psi_{2,m}-\psi_{2,\mathrm{ref}}
$$

對主 replica 而言，

$$
\tilde{\psi}_{2,m_0}
\approx
0
$$

也就是說，主 replica 在 deramping 後近似只剩常數項與一次項，因此頻域能量會變得集中。

更明確地，第 $m$ 個 replica 在 deramping 後變成

$$
S_{3,m}(\tau,f_\eta)
\approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
$$

$$
\cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)
(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
$$

因此本步結束後的訊號 closed form 為

$$
S_3(\tau,f_\eta)
\approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
$$

$$
\cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)
(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
$$

---

**5. 為什麼 LPF 必須放在 deramping 之後**

定義 LPF 為

$$
H_{\mathrm{LPF}}(f_\eta)
=
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

經 LPF 後，

$$
S_4(\tau,f_\eta)
=
S_3(\tau,f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta)
$$

若不先 deramp，主 replica 本身仍帶有顯著二次 curvature，其能量不會集中於狹窄 baseband，固定頻寬 LPF 很難只擷取單一 replica 而不造成失真。

相反地，在 deramping 後，主 replica 已近似展平，因此有

$$
S_4(\tau,f_\eta)
\approx
S_{3,m_0}(\tau,f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta)
$$

而其餘 replicas 由於：

- 中心位置不同
- 曲率補償不匹配
- 殘餘二次項 $\tilde{\psi}_{2,m}$ 不同

故不會同時集中到相同的 baseband，於是可被 LPF 抑制。

因此本步結束後的訊號 closed form 為

$$
S_4(\tau,f_\eta)
\approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
$$

$$
\cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m}-\psi_{2,\mathrm{ref}}
\right)
(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
\cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

在只保留主 replica 的近似下，上式可寫成

$$
S_4(\tau,f_\eta)
\approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_{m_0}(f_\eta)}
\right)
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m_0\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
$$

$$
\cdot
\exp\left(
-j\left[
\psi_{0,m_0}
+\psi_{1,m_0}(f_\eta-f_{\mathrm{ref}})
+\left(
\psi_{2,m_0}-\psi_{2,\mathrm{ref}}
\right)
(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
\cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

---

**6. Reramping**

LPF 之後保留下來的是已被展平的主 replica。若後續 azimuth compression 仍需要原本的 quadratic phase 結構，則必須把 reference curvature 乘回去。

定義 reramping filter 為

$$
H_{\mathrm{re}}(f_\eta)
=
H_{\mathrm{de}}^{-1}(f_\eta)
$$

因此

$$
H_{\mathrm{re}}(f_\eta)
=
\exp\left(
-j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

若採 chirp-rate 表示，則

$$
H_{\mathrm{re}}(f_\eta)
=
\exp\left(
-j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
\right)
$$

reramping 後可得

$$
S_5(\tau,f_\eta)
=
S_4(\tau,f_\eta)\cdot H_{\mathrm{re}}(f_\eta)
$$

在理想近似下，

$$
S_5(\tau,f_\eta)
\approx
S_{2,m_0}(\tau,f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta)
$$

也就是說，主 replica 已被單獨保留，且其 reference quadratic phase 已恢復。

若把 reramping 明確乘回去，則本步結束後的訊號 closed form 為

$$
S_5(\tau,f_\eta)
\approx
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_m(f_\eta)}
\right)
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
$$

$$
\cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
\right]
\right)
\cdot
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

也就是說，reramping 把 deramping 時拿掉的 $\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2$ 乘回去，因此每個 replica 的二次項重新變回 $\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2$。

對只保留主 replica 的情況，上式可寫成

$$
S_5(\tau,f_\eta)
\approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D_{m_0}(f_\eta)}
\right)
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m_0\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
$$

$$
\cdot
\exp\left(
-j\left[
\psi_{0,m_0}
+\psi_{1,m_0}(f_\eta-f_{\mathrm{ref}})
 +\psi_{2,m_0}(f_\eta-f_{\mathrm{ref}})^2
 \right]
 \right)
 \cdot
 \mathrm{rect}\left(
 \frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
 \right)
$$

---

**物理意義**

- mosaicking 只是在 extended azimuth-frequency axis 上把 replicas 攤開，並不會消除每塊 replica 的內部 chirp curvature。
- SymPy 驗證的二階展開指出：造成 replica 不易直接用固定窗分離的核心，就是 $\psi_{2,m}$ 所對應的 phase curvature。
- deramping 的物理作用，就是把主 replica 的 curvature 拿掉，讓它在頻域上變平、變窄。
- LPF 的物理作用，是在這個「已展平」的 domain 上只保留主 replica。
- reramping 的物理作用，是恢復後續聚焦或 matched filtering 所需的 phase model。

---

**最終結果**

可直接使用的鏈式表示為

$$
S_3(\tau,f_\eta)
=
S_2(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta)
$$

$$
H_{\mathrm{de}}(f_\eta)
=
\exp\left(
+j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

$$
S_4(\tau,f_\eta)
=
S_3(\tau,f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta)
$$

$$
H_{\mathrm{LPF}}(f_\eta)
=
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

$$
S_5(\tau,f_\eta)
=
S_4(\tau,f_\eta)\cdot H_{\mathrm{re}}(f_\eta)
$$

$$
H_{\mathrm{re}}(f_\eta)
=
H_{\mathrm{de}}^{-1}(f_\eta)
$$

因此整體鏈可寫成

$$
S_2
\xrightarrow{\ H_{\mathrm{de}}\ }
S_3
\xrightarrow{\ H_{\mathrm{LPF}}\ }
S_4
\xrightarrow{\ H_{\mathrm{re}}\ }
S_5
$$

---

**實作對應**

在程式上通常對應為：

1. 先對 mosaicked spectrum 乘上 deramp phase
2. 在 deramped domain 上做低通、裁切或 mask
3. 對保留下來的主 block 乘回 reramp phase

也就是說：

- 對主 replica 的二次項做共軛補償，對應 $H_{\mathrm{de}}$
- 只保留已展平且集中到通帶內的主 block，對應 $H_{\mathrm{LPF}}$
- 把 reference curvature 乘回去，對應 $H_{\mathrm{re}}$

---

**限制與適用範圍**

- 本推導依賴主 replica 在 $f_{\mathrm{ref}}$ 附近的二階泰勒展開；若處理頻帶太寬，三階以上項可能不可忽略。
- 單一 deramping filter 只會精確匹配單一 reference curvature，因此它主要是為了分離某一個指定的主 replica，而不是同時展平全部 replicas。
- LPF 是否有效，取決於 deramping 後主 replica 與其他 replicas 是否真的在 baseband 上可分離。
- 若後續處理直接在 deramped domain 完成，則 reramping 可視需求省略或改寫。
