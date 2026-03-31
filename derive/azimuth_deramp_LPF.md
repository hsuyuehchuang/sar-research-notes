**Azimuth Deramping And LPF Derivation**

**先講結論**

在完成 mosaicking 之後，訊號 $S_2(\tau,f_\eta)$ 雖然已經被展開到 extended azimuth-frequency axis 上，但每一個 replica 仍然帶有二次 chirp phase。這些 replica 在頻域上雖然被排開了，卻還沒有被「拉平」。deramping filter 的作用，就是把每個 replica 共同具有的主要二次相位拿掉，使其能量從原本沿著 chirp law 擴展的樣子，轉成接近集中在較窄 baseband 內的表示。這時後續的 LPF 才能用一個固定頻寬窗，把欲保留的 unfolded 主分量濾出來。

因此，deramping 不是直接消除 aliasing，而是先把可預測的 chirp phase 結構展平；LPF 則利用展平之後的頻譜集中性，將不需要的 replicas 或超出目標 band 的成分切除。

---

本文件延續 [azimuth_freq_ufr.md](/home/hsuyueh.chuang/Desktop/vscode/github/sar_tops_mode/derive/azimuth_freq_ufr.md) 中的 mosaicked 頻譜 $S_2(\tau,f_\eta)$，進一步推導：

- deramping filter 的數學形式
- deramping 之後的頻譜為什麼會變得可被 LPF 分離
- 後續 LPF 為什麼能保留欲重建的 unfolded 主頻帶

---

**1. 起點：mosaicked azimuth-frequency signal**

由前一份文件可得，mosaicking 後的訊號可寫為

$$ S_2(\tau,f_\eta) = \sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}} S_{2,m}(\tau,f_\eta) $$

其中第 $m$ 個 replica 為

$$
S_{2,m}(\tau,f_\eta) =
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
$$

$$
\cdot
\exp\left[
-j\frac{4\pi R_0f_0}{c}D_m(f_\eta)
-j2\pi\left(f_\eta-m\cdot\mathrm{PRF}\right)\eta_c
\right]
$$

且

$$
D_m(f_\eta)=D\left(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}}\right)
$$

上式中真正使 replica 呈現 chirp 結構的，是相位項

$$
\phi_m(f_\eta) =
-\frac{4\pi R_0f_0}{c}D_m(f_\eta)
-2\pi\left(f_\eta-m\cdot\mathrm{PRF}\right)\eta_c
$$

---

**2. 二次相位近似：為什麼它像一個 chirp**

在目標頻帶附近，可將 $D_m(f_\eta)$ 於某一參考頻率 $f_{\mathrm{ref},m}$ 附近做二階展開：

$$
D_m(f_\eta)
\approx
D_m(f_{\mathrm{ref},m})
+D_m'(f_{\mathrm{ref},m})(f_\eta-f_{\mathrm{ref},m})
+\frac{1}{2}D_m''(f_{\mathrm{ref},m})(f_\eta-f_{\mathrm{ref},m})^2
$$

代回相位項後，可得

$$
\phi_m(f_\eta)
\approx
\phi_{0,m}
+\phi_{1,m}(f_\eta-f_{\mathrm{ref},m})
+\phi_{2,m}(f_\eta-f_{\mathrm{ref},m})^2
$$

因此第 $m$ 個 replica 的主要相位可近似為一個二次相位項，也就是 chirp phase：

$$
\exp\left[j\phi_m(f_\eta)\right]
\approx
\exp\left(j\phi_{0,m}\right)
\cdot
\exp\left[j\phi_{1,m}(f_\eta-f_{\mathrm{ref},m})\right]
\cdot
\exp\left[j\phi_{2,m}(f_\eta-f_{\mathrm{ref},m})^2\right]
$$

其中：

- 常數項 $\phi_{0,m}$ 只改變整體相位
- 一次項 $\phi_{1,m}$ 對應 centroid shift 或群延遲
- 二次項 $\phi_{2,m}$ 才是造成頻譜被展開、傾斜、難以直接用固定頻寬窗截取的主因

所以後續 deramping 的核心目標，就是拿掉這個二次項。

---

**3. Deramping filter 的定義**

令參考 deramping filter 為

$$
H_{\mathrm{de}}(f_\eta) =
\exp\left[
-j\phi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right]
$$

若使用常見的等效 azimuth FM 寫法，也可寫成

$$
H_{\mathrm{de}}(f_\eta) =
\exp\left[
+j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
\right]
$$

這裡的符號約定是：

- $K_{\mathrm{ref}}$ 為用來 deskew / deramp 的參考 chirp rate
- $f_{\mathrm{ref}}$ 為 deramping 所使用的 reference center

經 deramping 後，

$$ S_3(\tau,f_\eta) = S_2(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta) $$

亦即

$$ S_3(\tau,f_\eta) = \sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}} S_{2,m}(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta) $$

---

**4. Deramping 後每個 replica 會變成什麼**

考慮第 $m$ 個 replica 經 deramping 後的相位：

$$
\phi^{(\mathrm{de})}_m(f_\eta) =
\phi_m(f_\eta)+\phi_{\mathrm{de}}(f_\eta)
$$

若 $H_{\mathrm{de}}$ 的二次項設計為近似抵消主要 chirp curvature，則有

$$
\phi^{(\mathrm{de})}_m(f_\eta)
\approx
\tilde{\phi}_{0,m}
+\tilde{\phi}_{1,m}(f_\eta-f_{\mathrm{ref}})
+\tilde{\phi}_{2,m}(f_\eta-f_{\mathrm{ref}})^2
$$

其中

$$
\tilde{\phi}_{2,m} =
\phi_{2,m}-\phi_{2,\mathrm{ref}}
$$

若參考率選得好，則在欲保留的主 replica 上會有

$$
\tilde{\phi}_{2,m}\approx 0
$$

因此主 replica 會從原本具有明顯 chirp curvature 的頻譜，變成近似只有常數相位與一次相位的展平表示：

$$
S_{3,m}(\tau,f_\eta)
\approx
\tilde{A}_m(\tau,f_\eta)
\cdot
\exp\left[
j\tilde{\phi}_{0,m}
+j\tilde{\phi}_{1,m}(f_\eta-f_{\mathrm{ref}})
\right]
$$

也就是說，頻譜能量不再因為二次相位而沿著較寬的頻率區間展開，而會集中到一個較窄且近似線性的 baseband support。

這就是「為什麼 deramping 之後 LPF 變得可行」的核心。

---

**5. 為什麼 LPF 在 deramping 後可以把主頻譜濾出來**

在未 deramp 之前，各 replica 雖然已被 mosaicking 排開，但其內部仍帶有 chirp curvature，因此：

- 單一 replica 的能量分布較分散
- replica 與 replica 之間的邊界不夠「平」
- 用固定 cutoff 的 LPF 不容易只保留欲取的主分量

但在 deramping 之後，主 replica 會被映射到較窄的近 baseband 區域，因此可設一個 low-pass filter

$$
H_{\mathrm{LPF}}(f_\eta) =
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

得到

$$ S_4(\tau,f_\eta) = S_3(\tau,f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta) $$

若展平之後的主 replica 大部分能量都落在 $B_{\mathrm{LPF}}$ 內，而其他 replicas 因為：

- 中心位置不同
- 線性相位不同
- 殘餘二次相位不同

而落在 LPF 通帶之外，則 LPF 就能有效保留主 replica，抑制不需要的 unfolded blocks。

因此整個邏輯是

$$
\text{mosaicking}
\;\Longrightarrow\;
\text{replicas explicitly tiled}
\;\Longrightarrow\;
\text{deramping removes main quadratic phase}
\;\Longrightarrow\;
\text{desired replica becomes compact in baseband}
\;\Longrightarrow\;
\text{LPF can isolate it}
$$

---

**6. 用解析式看 LPF 為何有效**

將 $S_3=S_2\cdot H_{\mathrm{de}}$ 代入後，可得

$$ S_4(\tau,f_\eta) = \sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}} S_{2,m}(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta) $$

若 $m=m_0$ 為欲保留之主 replica，且對此 replica 有

$$
\phi_{2,m_0}\approx \phi_{2,\mathrm{ref}}
$$

則

$$ S_{2,m_0}(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta) $$

會近似成一個被壓縮到 baseband 內的分量，因此

$$ S_4(\tau,f_\eta) \approx S_{2,m_0}(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta) $$

而對 $m\neq m_0$ 的 replicas，由於其中心位置或殘餘 chirp rate 不匹配，即使經過相同 deramping，也往往不能同時被拉平成和主 replica 相同的位置與頻寬，因此會在 LPF 之後被抑制。

---

**7. 以實作角度理解**

如果從程式實作角度看：

- mosaicking：把各個 alias blocks 依序鋪展到 extended 頻率軸
- deramping：對整條展開後的 spectrum 乘上一個 reference quadratic phase
- LPF：只保留被 deramping 後集中到 central baseband 的那一塊

所以程式上看到的現象通常是：

1. mosaicking 後，signal 變長、頻帶展開
2. deramping 後，原本斜的 / 展開的頻譜變得比較平、比較集中
3. LPF 後，只剩下中央主要那一塊 unfolded 頻譜

這正對應數學上

$$ S_2 \;\xrightarrow{\ H_{\mathrm{de}}\ }\; S_3 \;\xrightarrow{\ H_{\mathrm{LPF}}\ }\; S_4 $$

---

**8. 可直接放進筆記的最終版本**

若要用最精簡的方式記錄，可直接寫成：

$$ S_3(\tau,f_\eta) = S_2(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta) $$

$$
H_{\mathrm{de}}(f_\eta) =
\exp\left[
+j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
\right]
$$

$$ S_4(\tau,f_\eta) = S_3(\tau,f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta) $$

$$
H_{\mathrm{LPF}}(f_\eta) =
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

其中 deramping 的作用是抵消主 replica 的二次 chirp phase，使其在頻域上集中到較窄的 baseband；LPF 則利用這種集中性，把目標 unfolded 主頻譜保留下來。
