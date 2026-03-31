**Azimuth-Frequency UFR / Mosaicking Derivation**

**結論**

- mosaicking 之後的 signal，不再是單一 PRF 主頻帶內的 folded 表示，而是沿著 extended azimuth-frequency axis 顯式鋪開的 UFR 表示。
- 每一個 replica 都可視為同一個連續頻譜分量，沿 $m\cdot\mathrm{PRF}$ 做平移後所得到的結果。
- 因此 mosaicking 的數學本質不是新的成像操作，而是把原本隱含在 folded 頻譜中的 shifted replicas 重新排列到可分離的頻率座標上。
- 這個展開後的表示，才使後續 deramping、LPF、reramping 與 azimuth compression 能夠針對單一 replica 進行操作。

---

**問題定義**

本文件要回答的是：

1. 從 folded azimuth spectrum $S_1(\tau,f_\eta;\omega_s)$ 出發，mosaicking 後的 signal 會變成什麼形式？
2. 為什麼程式中「把 block 一塊一塊 append 到 extended 軸上」的作法，對應到一個嚴格的數學表示？
3. 如何把這個動作寫成可供後續推導使用的 closed-form UFR expression？

---

**推導重點**

- 先從 folded spectrum 的週期複製形式出發。
- 將單一連續 replica 定義為 $S_{1,\mathrm{cont}}(\tau,f_\eta)$。
- 用有限長 tiling comb 定義 mosaicking operator。
- 證明該 operator 對應到各 replica 在 $m\cdot\mathrm{PRF}$ 上的平移與重排。
- 寫出第 $m$ 個 replica 與總 UFR spectrum 的閉式表示。

---

**符號與假設**

- $\tau$：距離時間
- $\eta$：方位慢時間
- $f_\eta$：方位頻率
- $\mathrm{PRF}$：pulse repetition frequency
- $R_0$：最近斜距
- $f_0$：載波頻率
- $V_{\mathrm{eff}}$：等效方位速度；若前文統一用 $V_r$，可直接替換
- $k_s\eta_c$：方位頻域包絡中心
- $B_{\max}$：mosaicked 後單一 replica 的有效頻寬近似
- $N_{s,\mathrm{neg}},N_{s,\mathrm{pos}}$：往負頻與正頻方向保留的 replica 數目

假設：

- 單一 replica 的 beam envelope 可近似為有限支撐窗
- mosaicking 只做頻率座標上的重排，不改變單一 replica 的物理相位模型
- 各 replicas 的中心相隔 $\mathrm{PRF}$

---

**逐步推導**

**1. folded spectrum 的起點**

由 [Azimuth_freq_folding.md](/home/hsuyueh.chuang/Desktop/vscode/github/sar_tops_mode/derive/Azimuth_freq_folding.md) 可得 folded azimuth spectrum

$$
S_1(\tau,f_\eta;\omega_s)
=
\mathrm{PRF}\sum_{k=-\infty}^{\infty}
S_{1,c}(\tau,f_\eta-k\cdot\mathrm{PRF};\omega_s)
$$

這個式子已經說明：在主頻帶中觀察到的 folded spectrum，本質上是許多沿 $k\cdot\mathrm{PRF}$ 平移後的連續頻譜副本之和。

**2. 定義單一連續 replica**

令單一連續頻譜副本寫為

$$
S_{1,\mathrm{cont}}(\tau,f_\eta)
=
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D(f_\eta,V_{\mathrm{eff}})}
\right)
\right]
$$

$$
\cdot
W_a(f_\eta;\omega_s)
\cdot
\exp\left(
-j\frac{4\pi R_0f_0}{c}D(f_\eta,V_{\mathrm{eff}})
-j2\pi f_\eta\eta_c
\right)
$$

若將 beam envelope 以有限頻寬窗近似，則可寫成

$$
W_a(f_\eta;\omega_s)
\approx
\mathrm{rect}\left(
\frac{f_\eta-k_s\eta_c}{B_{\max}}
\right)
$$

因此

$$
S_{1,\mathrm{cont}}(\tau,f_\eta)
\approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D(f_\eta,V_{\mathrm{eff}})}
\right)
\right]
$$

$$
\cdot
\mathrm{rect}\left(
\frac{f_\eta-k_s\eta_c}{B_{\max}}
\right)
\cdot
\exp\left(
-j\frac{4\pi R_0f_0}{c}D(f_\eta,V_{\mathrm{eff}})
-j2\pi f_\eta\eta_c
\right)
$$

**3. 定義 mosaicking operator**

用有限長的 tiling comb 表示 mosaicking：

$$
M(f_\eta)
=
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
\delta\left(f_\eta-m\cdot\mathrm{PRF}\right)
$$

則 UFR 表示可寫為

$$
S_2(\tau,f_\eta)
=
S_{1,\mathrm{cont}}(\tau,f_\eta)*M(f_\eta)
$$

利用 delta 函數的平移摺積性質，可得第 $m$ 個 replica 為

$$
S_{2,m}(\tau,f_\eta)
=
S_{1,\mathrm{cont}}(\tau,f_\eta-m\cdot\mathrm{PRF})
$$

因此總 mosaicked spectrum 為

$$
S_2(\tau,f_\eta)
=
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
S_{2,m}(\tau,f_\eta)
$$

**4. 將第 $m$ 個 replica 展開**

把 $f_\eta$ 換成 $f_\eta-m\cdot\mathrm{PRF}$，得到

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
$$

$$
\cdot
\exp\left(
-j\frac{4\pi R_0f_0}{c}D_m(f_\eta)
-j2\pi\left(f_\eta-m\cdot\mathrm{PRF}\right)\eta_c
\right)
$$

其中

$$
D_m(f_\eta)
=
D\left(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}}\right)
$$

這就是 UFR / mosaicking 的 closed-form replica expression。

---

**物理意義**

- $\mathrm{sinc}[\cdots]$ 對應 range response；每個 replica 都要以自己的 $D_m(f_\eta)$ 來描述 range-Doppler coupling。
- $\mathrm{rect}[\cdots]$ 表示該 replica 在 extended 頻率軸上的有效支撐區間。
- 指數項對應該 replica 的 propagation phase 與 centroid phase。
- $m$ 不是新的物理目標，而是同一組連續頻譜在 $\mathrm{PRF}$ 週期下的 replica 索引。

因此，mosaicking 的作用不是改變 signal 的物理內容，而是把 folded 表示中的隱含 replicas，顯式展開到一條更寬的頻率軸上。

---

**最終結果**

最終可直接使用的方程組為

$$
S_2(\tau,f_\eta)
=
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
S_{2,m}(\tau,f_\eta)
$$

$$
S_{2,m}(\tau,f_\eta)
=
S_{1,\mathrm{cont}}(\tau,f_\eta-m\cdot\mathrm{PRF})
$$

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
-j\frac{4\pi R_0f_0}{c}D_m(f_\eta)
-j2\pi\left(f_\eta-m\cdot\mathrm{PRF}\right)\eta_c
\right)
$$

$$
D_m(f_\eta)
=
D\left(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}}\right)
$$

---

**實作對應**

若程式中是把 signal block 一塊一塊 append 到 extended 頻率軸上，則它對應的正是離散化的 mosaicking operator。

- 數學上：$S_2=\sum_m S_{2,m}$
- 實作上：$block_m$ 依照 $m\cdot\mathrm{PRF}$ 的順序被拼接到更長的頻率軸

所以 append 並不是額外的近似，而是 $S_{2,m}$ 在離散頻域座標下的實作型態。

---

**限制與適用範圍**

- 若 $W_a(f_\eta;\omega_s)$ 不能以 $\mathrm{rect}$ 近似，則只需把 $\mathrm{rect}$ 換回原本的 beam envelope。
- 若各 replicas 在實作上存在重疊或插值，則離散程式不再是單純 append，而需要額外的 index mapping 或 resampling。
- 本推導針對的是 UFR / mosaicking representation，本身尚未處理 deramping、LPF、reramping 與最終方位壓縮。
