**Azimuth-Frequency UFR / Mosaicking Derivation**

**先講結論**

做完 mosaicking 之後，signal 不再是「所有 aliased replicas 摺疊在單一 PRF 主頻帶內」的 folded 表示，而會變成一個沿 azimuth-frequency 軸被顯式鋪開的 UFR 表示。也就是說，原本重疊在同一個觀測頻帶內的各個 shifted components，會依照 `m\cdot\mathrm{PRF}` 的位移關係，被重新排列到 extended frequency axis 上，形成一塊一塊相鄰的 replica。

若用數學表示，mosaicking 後的 signal 為

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
S_{1,\mathrm{cont}}(\tau,f_\eta-m\cdot\mathrm{PRF})
$$

因此，mosaicking 後的 signal 可以理解為：

- 在頻率軸上變寬，不再侷限於單一 `[-\mathrm{PRF}/2,\mathrm{PRF}/2]` 主帶
- 每一塊 replica 對應一個不同的 `m`
- 每一塊都保留自己的 range response、support window 與 phase term
- 結果是一個 unfolded / tiled 的 azimuth-frequency representation，方便後續做 deramping、LPF、reramping 與 azimuth compression

本文件延續 [Azimuth_freq_folding.md](/home/hsuyueh.chuang/Desktop/vscode/github/sar_tops_mode/derive/Azimuth_freq_folding.md) 中的 folded 方位頻譜
$S_1(\tau,f_\eta;\omega_s)$，進一步推導 azimuth-frequency mosaicking 在 extended aliased band 上的數學解析式。

這裡的推導依據你提供的圖式做整理；其中 `tiling convolution` 的意義可理解為：

- 不是再創造新的物理頻譜
- 而是把原本在主 PRF band 內觀察到的 alias 結構，沿著 `m\cdot\mathrm{PRF}` 週期性平移
- 於 expanded frequency axis 上顯式展開成一個可操作的 mosaic 頻譜

---

**1. 起點：folded azimuth spectrum**

由前一份文件可得

$$
S_1(\tau,f_\eta;\omega_s)
=
\mathrm{PRF}\sum_{k=-\infty}^{\infty}
S_{1,c}(\tau,f_\eta-k\cdot\mathrm{PRF};\omega_s)
$$

其中連續頻譜可寫為

$$
S_{1,c}(\tau,f_\eta;\omega_s)
=
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D(f_\eta,V_r)}
\right)
\right]
\cdot
W_a(f_\eta;\omega_s)
\cdot
\exp\left(\Phi_{az}(f_\eta)\right)
$$

若把 TOPS 掃描造成的有限頻寬包絡近似寫成一個中心在 `k_s\eta_c`、寬度為 `B_{\max}` 的頻域窗，則可進一步寫成

$$
W_a(f_\eta;\omega_s)
\approx
\mathrm{rect}\left(
\frac{f_\eta-k_s\eta_c}{B_{\max}}
\right)
$$

因此可把單一 replica 的連續分量近似表示為

$$
S_{1,\mathrm{cont}}(\tau,f_\eta)
=
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{c\,D(f_\eta,V_{\mathrm{eff}})}
\right)
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-k_s\eta_c}{B_{\max}}
\right)
\cdot
\exp\left[
-j\frac{4\pi R_0f_0}{c}D(f_\eta,V_{\mathrm{eff}})
-j2\pi f_\eta\eta_c
\right]
$$

上式中的 `V_{\mathrm{eff}}` 只是強調此處使用 azimuth FM law 所對應的等效速度；若你前文已固定用 `V_r`，也可全部改寫回 `V_r`。

---

**2. Mosaicking 的數學操作**

azimuth-frequency mosaicking 的核心，不是做 matched filtering，而是把已知的 PRF-shifted replicas 顯式鋪開到 extended axis。

可把 mosaicking operator 定義為一個有限長的 tiling comb：

$$
M(f_\eta)
=
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
\delta\left(f_\eta-m\cdot\mathrm{PRF}\right)
$$

則 expanded 頻域上的 mosaic spectrum 可寫為

$$
S_2(\tau,f_\eta)
=
S_{1,\mathrm{cont}}(\tau,f_\eta)*M(f_\eta)
$$

利用 delta train 的摺積平移性質，可得

$$
S_{2,m}(\tau,f_\eta)
=
S_{1,\mathrm{cont}}(\tau,f_\eta-m\cdot\mathrm{PRF})
$$

其中 `S_{2,m}` 表示第 `m` 個被鋪展到 extended frequency axis 上的 replica。

因此總 mosaicked 頻譜可寫為

$$
S_2(\tau,f_\eta)
=
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
S_{2,m}(\tau,f_\eta)
$$

這就是你圖片中 `tiling convolutions stretch the signal algebraically` 的精確數學形式。

---

**3. 將第 `m` 個 replica 展開後的 closed-form expression**

先把第 `m` 個 replica 寫開，可得

$$
S_{2,m}(\tau,f_\eta)
=
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
\exp\left[
-j\frac{4\pi R_0f_0}{c}D_m(f_\eta)
-j2\pi\left(f_\eta-m\cdot\mathrm{PRF}\right)\eta_c
\right]
$$

其中

$$
D_m(f_\eta)=D\left(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}}\right)
$$

因此總 mosaicked 頻譜為

$$
S_2(\tau,f_\eta)
=
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
S_{2,m}(\tau,f_\eta)
$$

這就得到圖中 mosaicking / UFR representation 的解析形式。

---

**4. 這個式子的物理意義**

上式的三個因子可分開理解：

- `\mathrm{sinc}[\cdots]`
  對應 range-Doppler coupling 下的距離向響應；每個 shifted replica 都要用自己的 `D_m(f_\eta)`。

- `\mathrm{rect}((f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c)/B_{\max})`
  表示第 `m` 個 replica 在 expanded 頻率軸上的有效支撐區間；mosaicking 就是把這些支撐區間一塊一塊鋪開。

- `\exp[\cdots]`
  表示該 replica 對應的二次距離相位與線性 centroid phase；這些相位在後續 deramping / reramping / azimuth compression 中會決定能否正確重建。

因此 mosaicking 的本質可以總結為：

$$
\text{folded spectrum in one PRF band}
\;\Longrightarrow\;
\text{explicit periodic extension over } m\cdot\mathrm{PRF}
\;\Longrightarrow\;
\text{UFR domain representation}
$$

---

**5. 與 `S_1(\tau,f_\eta;\omega_s)` 的關係**

若從 folded spectrum 的觀點來看，

$$
S_1(\tau,f_\eta;\omega_s)
=
\sum_{k=-\infty}^{\infty}
S_{1,\mathrm{cont}}(\tau,f_\eta-k\cdot\mathrm{PRF})
$$

其實已經隱含了所有 replicas 的存在；只是這些 replicas 在主 band 內被 folded 疊加觀察。

而 mosaicking 所做的，是改用一個 extended frequency coordinate 重新編排這些 replicas：

$$
S_1
\;\xrightarrow{\ \text{mosaicking}\ }\;
S_2
$$

其中 `S_1` 是 folded representation，
`S_2` 是 unfolded / tiled representation。

也就是說：

- `folding` 是取樣造成的主 band 內重疊
- `mosaicking` 是利用已知的 PRF shift 結構，把它們顯式搬回 extended axis

---

**6. 建議在論文或筆記中採用的最終版本**

若你想保留和圖片最接近的符號，可直接使用下面這組：

$$
S_{2,m}(\tau,f_\eta)
=
S_{1,\mathrm{cont}}(\tau,f_\eta-m\cdot\mathrm{PRF})
$$

$$
=
A_2
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
\exp\left[
-j\frac{4\pi R_0f_0}{c}D_m(f_\eta)
-j2\pi\left(f_\eta-m\cdot\mathrm{PRF}\right)\eta_c
\right]
$$

$$
S_2(\tau,f_\eta)
=
\sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}}
S_{2,m}(\tau,f_\eta)
$$

$$
D_m(f_\eta)=D\left(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}}\right)
$$

---

**7. 備註**

這裡把 `W_a(f_\eta;\omega_s)` 近似成 `rect` 窗，是為了得到和你提供圖片一致的 closed-form UFR expression。若你要保留更一般的 TOPS 波束包絡，而不做 `rect` 近似，則只需把上式中的

$$
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
$$

替換回

$$
W_a(f_\eta-m\cdot\mathrm{PRF};\omega_s)
$$

即可得到更一般的 mosaicking 表達式。
