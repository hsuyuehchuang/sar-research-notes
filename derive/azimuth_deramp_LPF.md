**重點摘要**

* mosaicking 之後的訊號 $S_2(\tau,f_\eta)$ 雖然已經把 replicas 展開到 extended azimuth-frequency axis 上，但每個 replica 仍帶有由 $D_m(f_\eta)$ 決定的二次 phase curvature。
* deramping 的核心不是消除 aliasing，而是拿掉主 replica 的 reference quadratic phase，使其頻域能量由彎曲展寬的 chirp-like 結構，轉成較集中的近 baseband 表示。
* LPF 之所以有效，不是因為它本身能辨認主 replica，而是因為主 replica 已先被 deramping 拉平並壓縮到狹窄通帶內。
* 若把第 $m$ 個 replica 的 phase 近似寫成 $\psi_{0,m}+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2$，則 deramping 後的殘餘二次項就是 $\psi_{2,m}-\psi_{2,\mathrm{ref}}$。
* 因此整個處理鏈的關鍵輸出可寫成
  $$
  {\color{red}
  S_3(\tau,f_\eta) =
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
  }
  $$
* LPF 後的輸出則是在上式外再乘上通帶窗，因此每一步的輸出訊號都可以寫成 fully expanded closed form，而不能只停在 operator shorthand。

---

**問題定義**

本文件要證明的是：

1. 為什麼在 UFR / mosaicking 之後，仍必須再做 deramping 才能有效使用 LPF。
2. deramping filter 應該抵消哪一個 phase term。
3. 每一步之後的訊號 $S_2$、$S_3$、$S_4$ 分別變成什麼 fully expanded closed form。

---

**推導重點**

* 以 [azimuth_freq_ufr.md](/home/hsuyueh.chuang/Desktop/vscode/github/sar_tops_mode/derive/azimuth_freq_ufr.md) 的 mosaicked spectrum 為起點。
* 將第 $m$ 個 replica 的相位抽出為 $\psi_m(f_\eta)$。
* 在主 replica 所關注的局部頻帶內，把 $\psi_m(f_\eta)$ 近似成常數項、一次項、二次項。
* 用 reference quadratic curvature $\psi_{2,\mathrm{ref}}$ 建立 deramping filter。
* 把 deramping 與 LPF 都乘回原訊號，逐步寫出 $S_3(\tau,f_\eta)$ 與 $S_4(\tau,f_\eta)$ 的 fully expanded closed form。

---

**符號與假設**

* $S_2(\tau,f_\eta)$：mosaicking 後的 azimuth-frequency signal
* $S_{2,m}(\tau,f_\eta)$：第 $m$ 個 mosaicked replica
* $S_3(\tau,f_\eta)$：deramping 後的訊號
* $S_4(\tau,f_\eta)$：LPF 後的訊號
* $m_0$：欲保留之主 replica 索引
* $f_{\mathrm{ref}}$：主 replica 的 reference frequency
* $\psi_m(f_\eta)$：第 $m$ 個 replica exponent 中的實數 phase function
* $\psi_{0,m},\psi_{1,m},\psi_{2,m}$：$\psi_m(f_\eta)$ 在 $f_{\mathrm{ref}}$ 附近的局部係數
* $\psi_{2,\mathrm{ref}}$：reference quadratic curvature
* $H_{\mathrm{de}}(f_\eta)$：deramping filter
* $H_{\mathrm{LPF}}(f_\eta)$：low-pass filter
* $D_m(f_\eta)=D\left(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}}\right)$

假設如下：

* 在主 replica 的局部頻帶內，二階 phase model 足以描述主要 curvature。
* 用單一 $f_{\mathrm{ref}}$ 與單一 $\psi_{2,\mathrm{ref}}$ 補償主 replica 的 phase curvature。
* LPF 通帶選擇得足以保留 deramping 後的主 replica，並抑制其他 replicas 的主要能量。

---

**1. 起點：mosaicked signal**

由 [azimuth_freq_ufr.md](/home/hsuyueh.chuang/Desktop/vscode/github/sar_tops_mode/derive/azimuth_freq_ufr.md) 可得

$$ S_2(\tau,f_\eta) = \sum_{m=-N_{s,\mathrm{neg}}}^{N_{s,\mathrm{pos}}} S_{2,m}(\tau,f_\eta) $$

其中第 $m$ 個 replica 的 fully expanded closed form 為

$$
S_{2,m}(\tau,f_\eta) =
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
\cdot
\exp\left(
-j\psi_m(f_\eta)
\right)
$$

其中 phase function 為

$$
\psi_m(f_\eta) =
\frac{4\pi R_0f_0}{c}D_m(f_\eta)
+2\pi\left(
f_\eta-m\cdot\mathrm{PRF}
\right)\eta_c
$$

且

$$ D_m(f_\eta) = D\left(f_\eta-m\cdot\mathrm{PRF},V_{\mathrm{eff}}\right) $$

因此本步結束後，總訊號的 fully expanded closed form 為

$$
S_2(\tau,f_\eta) =
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
\cdot
\exp\left(
-j\psi_m(f_\eta)
\right)
$$

---

**2. 局部二次 phase model**

為了建立 deramping filter，將 $\psi_m(f_\eta)$ 在 $f_{\mathrm{ref}}$ 附近做二階近似：

$$
\psi_m(f_\eta) \approx
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
+\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2
$$

其中

$$ \psi_{0,m} = \psi_m(f_{\mathrm{ref}}) $$

$$ \psi_{1,m} = \psi_m'(f_{\mathrm{ref}}) $$

$$ \psi_{2,m} = \frac{\psi_m''(f_{\mathrm{ref}})}{2} $$

因此第 $m$ 個 replica 的局部 fully expanded closed form 變為

$$
S_{2,m}(\tau,f_\eta) \approx
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
\cdot
\exp\left(
-j\left[
\psi_{0,m}
+\psi_{1,m}(f_\eta-f_{\mathrm{ref}})
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
\right]
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

這一步的關鍵是：二次 phase curvature 已被顯式抽成 $\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2$，因此後續 deramping 的抵消目標已清楚可見。

---

**3. Deramping**

取 reference quadratic curvature 為

$$ \psi_{2,\mathrm{ref}} = \psi_{2,m_0} $$

則 deramping filter 定義為

$$
H_{\mathrm{de}}(f_\eta) =
\exp\left(
+j\psi_{2,\mathrm{ref}}(f_\eta-f_{\mathrm{ref}})^2
\right)
$$

若改寫成常見 chirp-rate 記號，則

$$
H_{\mathrm{de}}(f_\eta) =
\exp\left(
+j\pi\frac{(f_\eta-f_{\mathrm{ref}})^2}{K_{\mathrm{ref}}}
\right)
$$

deramping 後的第 $m$ 個 replica 為

$$
S_{3,m}(\tau,f_\eta) =
S_{2,m}(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta)
$$

把 reference quadratic phase 直接乘回去後，可得第 $m$ 個 replica 的 fully expanded closed form

$$
S_{3,m}(\tau,f_\eta) \approx
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

因此 deramping 後的總訊號 fully expanded closed form 為

$$
{\color{red}
S_3(\tau,f_\eta) \approx
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
}
$$

對主 replica 而言，因為 $\psi_{2,\mathrm{ref}}=\psi_{2,m_0}$，所以其殘餘二次項近似變成零，也就是說主 replica 會由原本帶有明顯 curvature 的表示，轉成近似只剩常數項與一次項的較平坦頻譜。

---

**4. LPF**

LPF 定義為

$$
H_{\mathrm{LPF}}(f_\eta) =
\mathrm{rect}\left(
\frac{f_\eta-f_{\mathrm{LPF}}}{B_{\mathrm{LPF}}}
\right)
$$

LPF 後的第 $m$ 個 replica 為

$$
S_{4,m}(\tau,f_\eta) =
S_{3,m}(\tau,f_\eta)\cdot H_{\mathrm{LPF}}(f_\eta)
$$

把通帶窗直接乘回去後，第 $m$ 個 replica 的 fully expanded closed form 為

$$
S_{4,m}(\tau,f_\eta) \approx
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

因此 LPF 後的總訊號 fully expanded closed form 為

$$
{\color{red}
S_4(\tau,f_\eta) \approx
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
}
$$

若只保留主 replica $m=m_0$ 的近似，則輸出可再寫成

$$
S_4(\tau,f_\eta) \approx
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

也就是說，LPF 能保留主 replica 的條件，是它在 deramping 之後已先被壓縮到狹窄通帶內；若不先 deramp，固定通帶窗就很難只保留單一 replica 而不造成失真。

---

**物理意義**

* mosaicking 只是在 extended azimuth-frequency axis 上把 replicas 攤開，並不會消除每塊 replica 的內部 phase curvature。
* phase 的二次項 $\psi_{2,m}(f_\eta-f_{\mathrm{ref}})^2$ 是主 replica 難以直接用固定窗截取的核心原因。
* deramping 的物理作用，是把主 replica 的 reference curvature 拿掉，讓其頻譜變平、變窄。
* LPF 的物理作用，是在這個已展平的 domain 上只保留目標通帶。

---

**最終結果**

起點訊號：

$$
S_2(\tau,f_\eta) =
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
\cdot
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
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
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
\right]
\cdot
\mathrm{rect}\left(
\frac{f_\eta-m\cdot\mathrm{PRF}-k_s\eta_c}{B_{\max}}
\right)
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

---

**實作對應**

在程式上通常對應為：

* 先對 mosaicked spectrum 乘上 reference quadratic phase 的共軛補償，這對應 $H_{\mathrm{de}}(f_\eta)$。
* 再對 deramped spectrum 乘上固定通帶窗，這對應 $H_{\mathrm{LPF}}(f_\eta)$。
* 實作中雖然常寫成 `signal *= filter`，但理論文件裡不能只停在這種 shorthand，而必須把乘完之後的訊號真正展開寫出來。

---

**限制與適用範圍**

* 本文使用局部二階 phase model，適用於主 replica 附近的有限頻帶；若通帶過寬，三階以上項可能不可忽略。
* 單一 deramping filter 只會精確匹配單一 reference curvature，因此它是為了分離某一個指定的主 replica，而不是同時展平所有 replicas。
* LPF 是否有效，取決於 deramping 後主 replica 與其他 replicas 是否真的在 baseband 上可分離。
