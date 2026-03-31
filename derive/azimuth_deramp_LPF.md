**Azimuth Deramping, LPF, And Reramping Derivation**

**結論**

- mosaicking 之後的 $S_2(\tau,f_\eta)$ 已經把 replicas 展開，但每個 replica 仍保留二次 chirp phase，尚不適合直接用固定頻寬窗分離。
- deramping 的作用是乘上一個 reference quadratic phase，使欲保留的主 replica 在頻域上被展平並壓縮到較窄的 baseband。
- LPF 的作用不是去除 chirp，而是在 deramping 之後只保留被壓縮到通帶內的主 replica，抑制其他 replicas 與帶外殘留。
- reramping 則把先前拿掉的 reference quadratic phase 乘回去，使保留下來的主 replica 回到後續 azimuth compression 所需要的相位模型。
- 因此 $deramping \rightarrow LPF \rightarrow reramping$ 的整體作用，是「先展平、再截取、最後恢復相位結構」。

---

**問題定義**

本文件要回答的是：

1. 為什麼 mosaicking 後還需要 deramping？
2. 為什麼 deramping 之後 LPF 才有效？
3. 為什麼在 LPF 之後還要 reramping？
4. 如何把這三個操作寫成一組連續的數學表示？

---

**推導重點**

- 以 mosaicked spectrum $S_2(\tau,f_\eta)$ 為起點。
- 將每個 replica 的主要相位近似為二次 chirp phase。
- 定義 deramping filter 去抵消主 replica 的二次項。
- 證明主 replica 在 deramping 後會變得集中，因此 LPF 可以分離它。
- 定義 reramping filter，將 LPF 後保留下來的主 replica 恢復到後續壓縮所需的 chirp phase。

---

**符號與假設**

- $S_2(\tau,f_\eta)$：mosaicking 後的 UFR spectrum
- $S_{2,m}(\tau,f_\eta)$：第 $m$ 個 replica
- $H_{\mathrm{de}}(f_\eta)$：deramping filter
- $H_{\mathrm{LPF}}(f_\eta)$：low-pass filter
- $H_{\mathrm{re}}(f_\eta)$：reramping filter
- $f_{\mathrm{ref}}$：reference frequency
- $K_{\mathrm{ref}}$：reference chirp rate
- $m_0$：欲保留的主 replica 索引

假設：

- 在欲處理的局部頻帶內，每個 replica 的主要相位可用二階近似表示
- 主 replica 的二次相位可由單一 reference chirp 近似補償
- LPF 通帶足以保留展平後的主 replica，並抑制其他 replicas 的主要能量

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

其中

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
\phi_m(f_\eta)
\right)
$$

且相位定義為

$$
\phi_m(f_\eta)
=
-j\frac{4\pi R_0f_0}{c}D_m(f_\eta)
-j2\pi\left(f_\eta-m\cdot\mathrm{PRF}\right)\eta_c
$$

**2. 局部二次相位近似**

在主 replica 附近，可把 $\phi_m(f_\eta)$ 近似為

$$
\phi_m(f_\eta)
\approx
\phi_{0,m}
+\phi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\phi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
$$

其中：

- $\phi_{0,m}$ 是常數相位
- $\phi_{1,m}$ 是一次相位
- $\phi_{2,m}$ 是使 replica 呈現 chirp curvature 的二次項

主問題在於二次項使單一 replica 的能量沿頻域擴展，因此不適合直接用固定頻寬 LPF 截取。

**3. Deramping**

定義 deramping filter 為

$$
H_{\mathrm{de}}(f_\eta)
=
\exp\left(
-j\phi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

或等價地寫成

$$
H_{\mathrm{de}}(f_\eta)
=
\exp\left(
+j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
\right)
$$

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

其殘餘二次項為

$$
\tilde{\phi}_{2,m}
=
\phi_{2,m}-\phi_{2,\mathrm{ref}}
$$

若對主 replica $m=m_0$ 選得好，則有

$$
\tilde{\phi}_{2,m_0}\approx 0
$$

也就是說，主 replica 會被展平到近似只剩常數項與一次項的形式。

**4. LPF**

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

由於主 replica 已在 deramping 後變得集中，LPF 可近似保留

$$
S_4(\tau,f_\eta)
\approx
S_{3,m_0}(\tau,f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta)
$$

而對 $m\neq m_0$ 的 replicas，由於：

- 中心位置不同
- 殘餘二次項不同
- 展平後能量不落在同一 baseband

因此會被 LPF 抑制。

**5. Reramping**

LPF 之後保留下來的主 replica 雖然已被分離，但其相位已是「被展平後」的形式。若後續 azimuth compression 需要回到原本的 chirp 模型，則必須乘回 reference quadratic phase。

定義 reramping filter 為

$$
H_{\mathrm{re}}(f_\eta)
=
H_{\mathrm{de}}^{-1}(f_\eta)
$$

也就是

$$
H_{\mathrm{re}}(f_\eta)
=
\exp\left(
-j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
\right)
$$

因此 reramping 後，

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

但現在它只保留了主 replica，且已回到與後續方位壓縮相容的相位框架。

---

**物理意義**

- mosaicking：把 replicas 鋪開，但不改變其內部 chirp phase。
- deramping：把主 replica 的二次相位拿掉，讓它在頻域上變平、變集中。
- LPF：在展平之後，只保留目標 replica 所在的 baseband。
- reramping：把 reference chirp phase 乘回去，恢復後續 matched filtering 所需的相位模型。

所以這三步的物理分工非常明確：

- deramping 解決「不易分離」
- LPF 解決「只保留哪一塊」
- reramping 解決「怎麼回到後續聚焦模型」

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
+j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
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
=
\exp\left(
-j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
\right)
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

1. 對 mosaicked spectrum 乘上 deramp phase
2. 在展平後的 baseband 上做低通或裁切
3. 對保留下來的主 block 乘回 reramp phase

也就是說：

- multiply by deramp phase 對應 $H_{\mathrm{de}}$
- crop / LPF / mask 對應 $H_{\mathrm{LPF}}$
- multiply by reramp phase 對應 $H_{\mathrm{re}}$

---

**限制與適用範圍**

- 本推導假設主 replica 可由單一 reference chirp rate 良好近似。
- 若不同 replicas 的二次項差異過大，則單一 deramp filter 可能不能同時展平所有 replicas。
- LPF 是否有效，取決於 deramping 後主 replica 是否真的集中到與其他 replicas 可分離的 baseband。
- reramping 的必要性取決於後續處理鏈是否需要回到原本的 chirp phase 模型；若後續直接在 deramped domain 處理，則可省略或改寫。
