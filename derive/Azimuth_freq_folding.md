**重點摘要**

* TOPSAR 掃描率 $\omega_s$ 並不是直接作用在 folded spectrum 上，而是先改變時域照明函數 $w_a(\eta;\omega_s)$，再經傅立葉轉換形成連續頻域包絡 $W_a(f_\eta;\omega_s)$。
* folded phenomenon 的直接數學來源是慢時間離散取樣，也就是 Dirac comb $\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)$ 所導致的頻域週期性複製。
* 因此 TOPSAR 的 folded spectrum 應寫成連續頻譜與取樣 comb 卷積後的結果，而不是把 $\omega_s$ 與 aliasing 混成同一件事。
* 對 TOPS 而言，最重要的 carried-forward quantity 是
  $$
  {\color{red}
  W_{fold}(f_\eta;\omega_s)
  =
  \sum_{k=-\infty}^{\infty}
  W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
  }
  $$
* 因此完整依賴鏈應理解為
  $$
  {\color{red}
  \omega_s
  \Longrightarrow
  w_a(\eta;\omega_s)
  \Longrightarrow
  W_a(f_\eta;\omega_s)
  \Longrightarrow
  W_{fold}(f_\eta;\omega_s)
  }
  $$

---

**問題定義**

本文件要證明的是：

1. 為什麼 TOPSAR 的 folded spectrum 必須同時保留 beam steering rate $\omega_s$ 與離散取樣算子。
2. 為什麼 folded spectrum 的直接數學來源是慢時間取樣，而不是 beam steering 本身。
3. 為什麼在 TOPSAR 中，$\omega_s$ 仍然必須保留在最終 folded 頻譜表示內。
4. 如何從完整時域模型一路推導到 folded 頻域表示
   $$
   S_1(\tau,f_\eta;\omega_s)
   $$
   與
   $$
   {\color{red}
   W_{fold}(f_\eta;\omega_s)
   }
   $$

---

**推導重點**

* 先從完整時域模型出發，同時保留距離項、天線方向圖與慢時間取樣 comb。
* 再把 TOPSAR beam steering 幾何寫成有效離軸角 $\theta_{eff}(\eta)$，得到 $w_a(\eta;\omega_s)$。
* 接著忽略 comb，先建立連續方位頻譜 $S_{1,c}(\tau,f_\eta;\omega_s)$ 與連續包絡 $W_a(f_\eta;\omega_s)$。
* 最後把 comb 乘回來，利用 Poisson sum formula 將時域取樣轉成頻域週期性複製，得到 folded spectrum。
* Appendix A 說明 $\frac{L_a}{\lambda}\theta$ 的來源，Appendix B 說明 $w_a(\eta;\omega_s)$ 的幾何來源，Appendix C 說明 folded 副本為什麼仍可藉由 chirp phase 結構被還原。

---

**符號與假設**

* $\tau$：距離向快時間
* $\eta$：方位向慢時間
* $f_\eta$：方位向頻率
* $T_p=1/\mathrm{PRF}$：慢時間取樣週期
* $R_0$：最近斜距
* $\eta_0$：目標最近通過時刻
* $V_r$：等效方位相對速度
* $\omega_s$：TOPSAR beam steering 的等效方位掃描角速度
* $L_a$：方位向天線長度
* $\lambda$：波長
* $f_0$：載波頻率
* $B_r$：距離向頻寬
* $w_a(\theta)$：雙程天線方向圖
* $w_a(\eta;\omega_s)$：以慢時間表示的 TOPSAR 照明函數
* $W_a(f_\eta;\omega_s)$：$w_a(\eta;\omega_s)$ 的連續方位頻域包絡
* $W_{fold}(f_\eta;\omega_s)$：離散取樣後 folded 的頻域包絡
* $D(f_\eta,V_r)$：由 POSP 推導得到的幾何因子

假設：

* 採用小角度近似 $\sin\theta\approx\theta$
* 遠場條件成立，天線方向圖可由孔徑分布傅立葉轉換得到
* 方位向頻域推導採用駐位相位原理
* folded phenomenon 只討論由固定 PRF 慢時間取樣所導致的週期性複製

---

**1. 完整時域模型：同時保留 $\omega_s$ 與取樣效應**

設單點目標的完整方位向離散回波為

$$
s_{1,d}(\tau,\eta)
=
A_1\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R(\eta)}{c}
\right)
\right]
\cdot
w_a\left(
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
\right)
$$

$$
\cdot
\exp\left(
-j\frac{4\pi f_0R(\eta)}{c}
\right)
\cdot
{\color{red}
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
$$

其中幾何距離為

$$
R(\eta)
=
\sqrt{
R_0^2+V_r^2(\eta-\eta_0)^2
}
$$

若以 $s_c(\eta)$ 表示連續慢時間訊號，則離散取樣可寫為

$$
{\color{red}
s_d(\eta)
=
s_c(\eta)
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
$$

因此

$$
{\color{red}
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
$$

是慢時間取樣算子的數學表示，而不是額外的物理散射項。

本步結束後的輸入訊號已經清楚分解成三部分：

* 幾何距離項 $R(\eta)$
* TOPSAR 照明項 $w_a(\eta;\omega_s)$
* 離散取樣 comb $\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)$

---

**2. TOPSAR 照明函數：由 beam steering 幾何得到 $w_a(\eta;\omega_s)$**

在小角度近似下，雙程天線方向圖可寫為

$$
w_a(\theta)
=
\mathrm{sinc}^2\left(
\frac{L_a}{\lambda}\theta
\right)
$$

其中 $\frac{L_a}{\lambda}\theta$ 的來源見 Appendix A。

在 TOPSAR 中，目標相對掃描波束中心的有效離軸角為

$$
{\color{red}
\theta_{eff}(\eta)
=
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
}
$$

因此 TOPSAR 的時域照明函數可寫為

$$
{\color{red}
w_a(\eta;\omega_s)
=
\mathrm{sinc}^2\left[
\frac{L_a}{\lambda}
\left(
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
\right)
\right]
}
$$

這一步的物理意義是：$\omega_s$ 先改變時域照明函數，而不是直接作用在 folded 頻譜上。

因此第 1 步的完整時域模型也可重寫為

$$
s_{1,d}(\tau,\eta)
=
A_1\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R(\eta)}{c}
\right)
\right]
\cdot
w_a(\eta;\omega_s)
$$

$$
\cdot
\exp\left(
-j\frac{4\pi f_0R(\eta)}{c}
\right)
\cdot
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
$$

---

**3. 連續方位頻譜：由 $w_a(\eta;\omega_s)$ 導出 $W_a(f_\eta;\omega_s)$**

先忽略離散取樣 comb，只考慮其對應的連續訊號：

$$
s_{1,c}(\tau,\eta)
=
A_1\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R(\eta)}{c}
\right)
\right]
\cdot
w_a(\eta;\omega_s)
\cdot
\exp\left(
-j\frac{4\pi f_0R(\eta)}{c}
\right)
$$

對 $\eta$ 做方位向傅立葉轉換，並利用駐位相位原理，可得

$$
S_{1,c}(\tau,f_\eta;\omega_s)
=
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta,V_r)}
\right)
\right]
\cdot
W_a(f_\eta;\omega_s)
$$

$$
\cdot
\exp\left(
\Phi_{az}(f_\eta)
\right)
$$

其中

$$
D(f_\eta,V_r)
=
\sqrt{
1-\frac{c^2f_\eta^2}{4V_r^2f_0^2}
}
$$

以及

$$
\Phi_{az}(f_\eta)
=
-j\frac{4\pi R_0f_0}{c}D(f_\eta,V_r)
-j2\pi f_\eta\eta_0
$$

而連續頻域包絡定義為

$$
W_a(f_\eta;\omega_s)
=
\mathcal{F}_{\eta}\left[
w_a(\eta;\omega_s)
\right]
$$

因此本步結束後的連續方位頻譜 closed form 為

$$
S_{1,c}(\tau,f_\eta;\omega_s)
=
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta,V_r)}
\right)
\right]
\cdot
W_a(f_\eta;\omega_s)
\cdot
\exp\left(
\Phi_{az}(f_\eta)
\right)
$$

這裡必須保留 $\omega_s$，因為它已經被寫進 $w_a(\eta;\omega_s)$，並進一步傳到 $W_a(f_\eta;\omega_s)$。

---

**4. 離散取樣後的 folded 頻譜：由 comb 得到週期性複製**

根據 Poisson sum formula，

$$
\mathcal{F}\left[
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
\right]
=
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
\delta(f_\eta-k\cdot\mathrm{PRF})
$$

因此時域相乘等效於頻域卷積，離散後的方位頻譜為

$$
S_1(\tau,f_\eta;\omega_s)
=
S_{1,c}(\tau,f_\eta;\omega_s)
*
\left[
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
\delta(f_\eta-k\cdot\mathrm{PRF})
\right]
$$

$$
=
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
S_{1,c}(\tau,f_\eta-k\cdot\mathrm{PRF};\omega_s)
$$

若將連續頻譜表示代入，則 folded 後的完整 closed form 可寫為

$$
S_1(\tau,f_\eta;\omega_s)
=
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta-k\cdot\mathrm{PRF},V_r)}
\right)
\right]
$$

$$
\cdot
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
\cdot
\exp\left(
\Phi_{az}(f_\eta-k\cdot\mathrm{PRF})
\right)
$$

若只強調 folded 包絡本身，則可將 envelope 部分整理成

$$
{\color{red}
W_{fold}(f_\eta;\omega_s)
=
\sum_{k=-\infty}^{\infty}
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

在 envelope 與 phase 緩變近似下，也可進一步寫成

$$
S_1(\tau,f_\eta;\omega_s)
\approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta,V_r)}
\right)
\right]
\cdot
\exp\left(
\Phi_{az}(f_\eta)
\right)
$$

$$
\cdot
\underbrace{
\left[
\sum_{k=-\infty}^{\infty}
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
\right]
}_{{\color{red}W_{fold}(f_\eta;\omega_s)}}
$$

這裡最重要的是：

* folded 行為本身來自慢時間離散取樣
* $\omega_s$ 決定的是連續頻域包絡 $W_a$ 的展寬
* 因此最終 folded 頻譜必須保留 $\omega_s$

---

**5. 物理判讀：為什麼 TOPSAR 特別容易出現 folded spectrum**

這裡保留完整依賴鏈：

$$
{\color{red}
\omega_s
\Longrightarrow
w_a(\eta;\omega_s)
\Longrightarrow
W_a(f_\eta;\omega_s)
\Longrightarrow
W_{fold}(f_\eta;\omega_s)
}
$$

正確的物理理解是：

* $\omega_s$ 改變的是 beam steering 下的時域照明
* 該照明函數經傅立葉轉換後，決定連續頻域包絡 $W_a(f_\eta;\omega_s)$ 的寬度
* 當這個連續頻譜相對於 PRF 過寬時，離散取樣才會把它折回主頻帶

因此 folded phenomenon 的直接數學來源是取樣，但 TOPSAR 的 $\omega_s$ 會把系統推到更容易發生 folded 的條件。

---

**物理意義**

* $R(\eta)$：決定幾何距離、相位歷程與距離徙動
* $w_a(\eta;\omega_s)$：描述 beam steering 下目標被照射的時域包絡
* $W_a(f_\eta;\omega_s)$：時域照明函數對應的連續方位頻域包絡
* $\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)$：慢時間離散取樣算子
* $W_{fold}(f_\eta;\omega_s)$：連續頻域包絡被 PRF 週期性複製後的 folded 表示

本質上：

* $\omega_s$ 決定連續頻譜是否被展寬
* PRF 取樣決定展寬後的頻譜是否被折回

---

**最終結果**

完整時域模型：

$$
s_{1,d}(\tau,\eta)
=
A_1\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R(\eta)}{c}
\right)
\right]
\cdot
w_a(\eta;\omega_s)
\cdot
\exp\left(
-j\frac{4\pi f_0R(\eta)}{c}
\right)
\cdot
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
$$

TOPSAR 時域照明函數：

$$
{\color{red}
w_a(\eta;\omega_s)
=
\mathrm{sinc}^2\left[
\frac{L_a}{\lambda}
\left(
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
\right)
\right]
}
$$

連續方位頻譜：

$$
S_{1,c}(\tau,f_\eta;\omega_s)
=
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta,V_r)}
\right)
\right]
\cdot
W_a(f_\eta;\omega_s)
\cdot
\exp\left(
\Phi_{az}(f_\eta)
\right)
$$

folded 包絡：

$$
{\color{red}
W_{fold}(f_\eta;\omega_s)
=
\sum_{k=-\infty}^{\infty}
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

folded 方位頻譜：

$$
{\color{red}
S_1(\tau,f_\eta;\omega_s)
=
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
S_{1,c}(\tau,f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

近似 envelope form：

$$
S_1(\tau,f_\eta;\omega_s)
\approx
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta,V_r)}
\right)
\right]
\cdot
\exp\left(
\Phi_{az}(f_\eta)
\right)
\cdot
W_{fold}(f_\eta;\omega_s)
$$

---

**實作對應**

在程式上通常對應為：

* 先建立連續或準連續的方位包絡 $W_a(f_\eta;\omega_s)$
* 再用以 $\mathrm{PRF}$ 為間隔的 shift index $k$ 對它做週期性複製
* 若程式是把不同 block 依序 append 或 concatenate 到延展頻率軸上，則那是後續 unfolded / mosaicked 表示；它不是 folded 頻譜本身，而是 folded replicas 的重排

因此：

* 取樣 comb 對應的是理論上的 periodic replication operator
* `append` / `concatenate` 對應的是後續重排或 unfolding 實作

---

**限制與適用範圍**

* 本文件把 folded phenomenon 的主要來源鎖定在慢時間取樣，未展開更多與有限 aperture、window truncation、burst slicing 有關的高階效應。
* 連續頻譜到近似 envelope form 的過程使用了 POSP 與緩變近似，因此更適合用來說明結構，而不是做最終高精度誤差分析。
* 若沒有可靠的 chirp phase model，folded 副本雖然仍可被寫成週期性複製，但不一定保證後續可逆還原。

---

**Appendix A. $\frac{L_a}{\lambda}\theta$ 的來源**

設方位向天線孔徑沿座標 $x\in[-L_a/2,L_a/2]$ 均勻照明，則孔徑分布可寫為

$$
a(x)
=
\begin{cases}
1, & |x|\le L_a/2 \\
0, & \text{otherwise}
\end{cases}
$$

在遠場條件下，單程場型振幅為孔徑分布的空間傅立葉轉換：

$$
E(\theta)
\propto
\int_{-L_a/2}^{L_a/2}
\exp\left(
jkx\sin\theta
\right)\,dx
$$

其中 $k=2\pi/\lambda$。直接積分可得

$$
E(\theta)
\propto
\frac{
\exp\left(
jk\frac{L_a}{2}\sin\theta
\right)
-
\exp\left(
-jk\frac{L_a}{2}\sin\theta
\right)
}{
jk\sin\theta
}
$$

$$
=
\frac{
2\sin\left(
k\frac{L_a}{2}\sin\theta
\right)
}{
k\sin\theta
}
$$

代入 $k=2\pi/\lambda$，可得

$$
E(\theta)
\propto
L_a\,
\frac{
\sin\left(
\pi\frac{L_a}{\lambda}\sin\theta
\right)
}{
\pi\frac{L_a}{\lambda}\sin\theta
}
$$

因此歸一化單程方向圖為

$$
G_{1\text{-way}}(\theta)
=
\mathrm{sinc}\left(
\frac{L_a}{\lambda}\sin\theta
\right)
$$

在小角度近似 $\sin\theta\approx\theta$ 下，

$$
{\color{red}
G_{1\text{-way}}(\theta)
\approx
\mathrm{sinc}\left(
\frac{L_a}{\lambda}\theta
\right)
}
$$

對單站雷達而言，發射與接收方向圖相乘，因此雙程方向圖為

$$
w_a(\theta)
=
\left|
G_{1\text{-way}}(\theta)
\right|^2
\approx
\mathrm{sinc}^2\left(
\frac{L_a}{\lambda}\theta
\right)
$$

因此 $\frac{L_a}{\lambda}\theta$ 不是任意寫入的無因次量，而是孔徑長度相對於波長的歸一化空間相位差。

---

**Appendix B. $w_a(\eta;\omega_s)$ 的幾何來源**

$w_a$ 最原始是角度 $\theta$ 的函數，而不是慢時間 $\eta$ 的函數。要把它寫成 $w_a(\eta;\omega_s)$，必須先把目標相對波束中心的有效離軸角表示為慢時間函數。

設目標最近通過時刻為 $\eta_0$，最近斜距為 $R_0$，平台沿方位向的等效速度為 $V_r$。則方位向相對位移為

$$
x(\eta)=V_r(\eta-\eta_0)
$$

因此目標相對雷達視線的瞬時方位角為

$$
\theta_{tar}(\eta)
=
\tan^{-1}\left(
\frac{x(\eta)}{R_0}
\right)
$$

在小角度近似下，

$$
\theta_{tar}(\eta)
\approx
\frac{x(\eta)}{R_0}
=
\frac{V_r}{R_0}(\eta-\eta_0)
$$

TOPSAR 中，波束中心隨慢時間掃描。若其等效掃描角速度為 $\omega_s$，則

$$
\theta_{beam}(\eta)=\omega_s\eta
$$

因此目標相對波束中心的有效離軸角為

$$
{\color{red}
\theta_{eff}(\eta)
=
\theta_{tar}(\eta)-\theta_{beam}(\eta)
=
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
}
$$

故方向圖項可寫為

$$
w_a\left(
\theta_{eff}(\eta)
\right)
=
w_a\left(
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
\right)
$$

再代入 Appendix A 中的雙程方向圖近似，即得

$$
{\color{red}
w_a(\eta;\omega_s)
=
\mathrm{sinc}^2\left[
\frac{L_a}{\lambda}
\left(
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
\right)
\right]
}
$$

---

**Appendix C. folded 頻譜可還原性的相位模型**

TOPS 中 folded phenomenon 之所以原理上可還原，關鍵不在於取樣後資訊自動保留，而在於原始方位向訊號具有可建模的 LFM 相位律。

對單點目標，經 POSP 後的連續方位頻譜可近似寫為

$$
S_{1,c}(\tau,f_\eta;\omega_s)
\propto
W_a(f_\eta;\omega_s)
\cdot
\exp\left(
-j\pi\frac{(f_\eta-f_{dc})^2}{K_a}
\right)
$$

因此第 $k$ 個 folded 副本可表示為

$$
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
\cdot
\exp\left(
-j\pi\frac{(f_\eta-k\cdot\mathrm{PRF}-f_{dc})^2}{K_a}
\right)
$$

若施加對應的逆二次相位，即 deramping / deskew 操作

$$
H_{de}(f_\eta)
=
\exp\left(
+j\pi\frac{(f_\eta-f_{ref})^2}{K_{ref}}
\right)
$$

則 folded 副本可被映射到近似展平的表示。其可逆流程可概念化為

$$
{\color{red}
W_{fold}
\xrightarrow{\ \text{known chirp phase law}\ }
\{\tilde{W}_k\}
\xrightarrow{\ \text{unfold / shift}\ }
\hat{W}_{true}
}
$$

因此 folding 本身由取樣公式刻畫，而可還原性則來自 folded 副本之間共享且可逆的 LFM phase structure。若該相位律不存在，或 $K_a$、$f_{dc}$、$\omega_s$ 無法被穩定估計，則 folded 頻譜將退化為一般不可逆的 aliasing。
