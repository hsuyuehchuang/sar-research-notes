**重點摘要**

* TOPSAR 的掃描率 $\omega_s$ 不會直接生成 folded spectrum；它先改變慢時間照明函數 $w_a(\eta;\omega_s)$，再經傅立葉轉換形成連續方位包絡 $W_a(f_\eta;\omega_s)$。
* folded phenomenon 的直接數學來源是慢時間離散取樣，也就是 Dirac comb $\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)$ 所導致的頻域週期性複製。
* folded spectrum 的正確因果鏈是 $\omega_s \Longrightarrow w_a(\eta;\omega_s) \Longrightarrow W_a(f_\eta;\omega_s) \Longrightarrow W_{fold}(f_\eta;\omega_s)$。
* 這個問題必須分成幾何、時域照明、連續頻域包絡、取樣複製、物理判讀五個模組來推導，否則會把 beam steering 與 aliasing 混成同一件事。
* 最重要的 carried-forward results 是 folded 包絡 $W_{fold}(f_\eta;\omega_s)$ 與 folded 頻譜 $S_1(\tau,f_\eta;\omega_s)$。

摘要中最重要的關鍵公式為

$$
{\color{red}
W_{fold}(f_\eta;\omega_s) =
\sum_{k=-\infty}^{\infty}
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

以及

$$
{\color{red}
S_1(\tau,f_\eta;\omega_s) =
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
S_{1,c}(\tau,f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

---

**問題定義**

本文件要證明三件事：

1. 為什麼 TOPSAR folded spectrum 不能只寫成一般 aliasing，而必須保留 $\omega_s$。
2. 為什麼 folded spectrum 的直接數學來源是慢時間取樣，而不是 beam steering 本身。
3. 如何從完整時域模型一路推導到連續頻域包絡 $W_a(f_\eta;\omega_s)$ 與 folded 頻譜 $S_1(\tau,f_\eta;\omega_s)$。

---

**推導重點**

* 先從完整時域模型出發，同時保留幾何距離項、天線照明項與慢時間取樣 comb。
* 再把 TOPSAR beam steering 幾何寫成有效離軸角 $\theta_{eff}(\eta)$，將方向圖改寫成 $w_a(\eta;\omega_s)$。
* 接著忽略 comb，先建立連續方位訊號 $s_{1,c}(\tau,\eta)$ 與其方位頻譜 $S_{1,c}(\tau,f_\eta;\omega_s)$。
* 然後把取樣 comb 乘回來，用 Poisson sum formula 把時域取樣轉成頻域週期性複製。
* 最後把結果整理成 folded 包絡、完整 folded 頻譜與其物理意義。

---

**符號與假設**

* $\tau$：距離向快時間
* $\eta$：方位向慢時間
* $f_\eta$：方位向頻率
* $T_p=1/\mathrm{PRF}$：慢時間取樣週期
* $R_0$：目標最近斜距
* $\eta_0$：目標最近通過時刻
* $V_r$：等效方位相對速度
* $L_a$：方位向天線孔徑長度
* $\lambda$：波長
* $f_0$：載波頻率
* $B_r$：距離向頻寬
* $\omega_s$：TOPSAR beam steering 的等效角速度
* $R(\eta)$：慢時間下的瞬時斜距
* $w_a(\theta)$：雙程天線方向圖
* $w_a(\eta;\omega_s)$：改寫成慢時間座標後的 TOPSAR 時域照明函數
* $W_a(f_\eta;\omega_s)$：連續方位頻域包絡
* $W_{fold}(f_\eta;\omega_s)$：PRF 週期性複製後的 folded 頻域包絡
* $D(f_\eta,V_r)$：POSP 下的幾何因子
* $\Phi_{az}(f_\eta)$：連續頻域的 azimuth phase term

假設如下：

* 採用小角度近似 $\sin\theta\approx\theta$
* 採用遠場條件，方向圖可由孔徑分布的空間傅立葉轉換得到
* 方位頻域推導使用駐位相位原理
* 本文只討論由固定 PRF 慢時間取樣導致的 folded phenomenon
* 在 folded 包絡的近似寫法中，只對 envelope 與 phase 的緩變結構做說明，不把它當成高精度誤差分析公式

---

**1. 幾何與完整時域模型**

設單點目標的瞬時斜距為

$$
R(\eta) =
\sqrt{
R_0^2+V_r^2(\eta-\eta_0)^2
}
$$

若保留距離包絡、天線照明與慢時間取樣 comb，則離散慢時間回波可寫成

$$
s_{1,d}(\tau,\eta) =
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
\cdot
\exp\left(
-j\frac{4\pi f_0R(\eta)}{c}
\right)
\cdot
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
$$

若以 $s_c(\eta)$ 表示對應的連續慢時間訊號，則離散取樣模型為

$$
{\color{red}
s_d(\eta) =
s_c(\eta)
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
$$

因此本步結束後的輸出訊號就是完整的離散慢時間回波 $s_{1,d}(\tau,\eta)$。這一步只建立 folded 的來源算子，還沒有把 folded 頻譜寫出來。

---

**2. TOPSAR 照明函數**

在小角度近似下，雙程方向圖可寫為

$$
w_a(\theta) =
\mathrm{sinc}^2\left(
\frac{L_a}{\lambda}\theta
\right)
$$

其中 $\frac{L_a}{\lambda}\theta$ 的來源見 Appendix A。

在 TOPSAR 幾何下，目標相對掃描波束中心的有效離軸角為

$$
{\color{red}
\theta_{eff}(\eta) =
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
}
$$

將 $\theta=\theta_{eff}(\eta)$ 代入雙程方向圖，可得 TOPSAR 時域照明函數

$$
{\color{red}
w_a(\eta;\omega_s) =
\mathrm{sinc}^2\left[
\frac{L_a}{\lambda}
\left(
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
\right)
\right]
}
$$

因此第 1 步的完整時域模型也可改寫成

$$
s_{1,d}(\tau,\eta) =
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

因此本步結束後的關鍵輸出是顯式寫成慢時間函數的 $w_a(\eta;\omega_s)$。這一步澄清：$\omega_s$ 先改變的是時域照明，而不是直接生成 folded spectrum。

---

**3. 連續方位訊號與連續頻域包絡**

為了把 beam steering 的影響與取樣效應分開，先忽略 Dirac comb。對應的連續慢時間訊號為

$$
s_{1,c}(\tau,\eta) =
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

此時連續方位頻域包絡定義為

$$
W_a(f_\eta;\omega_s) =
\mathcal{F}_{\eta}\left[
w_a(\eta;\omega_s)
\right]
$$

利用駐位相位原理，連續方位頻譜可寫成

$$
S_{1,c}(\tau,f_\eta;\omega_s) =
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

其中

$$
D(f_\eta,V_r) =
\sqrt{
1-\frac{c^2f_\eta^2}{4V_r^2f_0^2}
}
$$

以及

$$
\Phi_{az}(f_\eta) =
-j\frac{4\pi R_0f_0}{c}D(f_\eta,V_r)
-j2\pi f_\eta\eta_0
$$

因此本步結束後的 fully expanded outputs 為：

$$
s_{1,c}(\tau,\eta) =
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

$$
S_{1,c}(\tau,f_\eta;\omega_s) =
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

這一步最重要的結論是：$\omega_s$ 已經透過 $w_a(\eta;\omega_s)$ 進入 $W_a(f_\eta;\omega_s)$，因此在後續 folded 頻譜中仍必須被保留。

---

**4. 由取樣 comb 得到 folded 頻譜**

慢時間取樣 comb 的傅立葉轉換為

$$
\mathcal{F}_{\eta}\left[
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
\right] =
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
\delta(f_\eta-k\cdot\mathrm{PRF})
$$

因此時域相乘在頻域中對應為卷積，離散訊號的方位頻譜可先寫成

$$
S_1(\tau,f_\eta;\omega_s) =
S_{1,c}(\tau,f_\eta;\omega_s)
*
\left[
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
\delta(f_\eta-k\cdot\mathrm{PRF})
\right]
$$

將卷積與 impulse train 展開後，可得 folded 頻譜的 compact closed form

$$
{\color{red}
S_1(\tau,f_\eta;\omega_s) =
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
S_{1,c}(\tau,f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

再把第 3 步的連續頻譜代入，可得 folded 頻譜的 fully expanded closed form

$$
{\color{red}
S_1(\tau,f_\eta;\omega_s) =
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta-k\cdot\mathrm{PRF},V_r)}
\right)
\right]
\cdot
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
\cdot
\exp\left(
\Phi_{az}(f_\eta-k\cdot\mathrm{PRF})
\right)
}
$$

若只抽出 envelope 結構，則 folded 包絡為

$$
{\color{red}
W_{fold}(f_\eta;\omega_s) =
\sum_{k=-\infty}^{\infty}
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

因此在 envelope 與 phase 緩變近似下，folded 頻譜也可寫成

$$
S_1(\tau,f_\eta;\omega_s) \approx
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

因此本步結束後，兩個必須保留的輸出訊號就是：

$$
W_{fold}(f_\eta;\omega_s) =
\sum_{k=-\infty}^{\infty}
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
$$

$$
S_1(\tau,f_\eta;\omega_s) =
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta-k\cdot\mathrm{PRF},V_r)}
\right)
\right]
\cdot
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
\cdot
\exp\left(
\Phi_{az}(f_\eta-k\cdot\mathrm{PRF})
\right)
$$

---

**5. 為什麼 TOPSAR 特別容易出現 folded spectrum**

這裡的因果順序必須保持清楚：

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

也就是說：

* $\omega_s$ 改變的是 beam steering 下的時域照明
* 時域照明函數經傅立葉轉換後，決定連續方位包絡 $W_a(f_\eta;\omega_s)$ 的寬度
* 當連續頻譜相對於 PRF 過寬時，取樣 comb 才會把其週期性複製回主頻帶

因此 folded phenomenon 的直接數學來源是取樣，而 TOPSAR 的特殊性在於它透過 $\omega_s$ 改變連續頻譜的寬度，讓 folded 更容易發生。

---

**物理意義**

* $R(\eta)$：控制幾何距離、二次相位歷程與方位壓縮基礎
* $w_a(\eta;\omega_s)$：描述 beam steering 下目標被照射多久、如何被照射
* $W_a(f_\eta;\omega_s)$：時域照明經傅立葉轉換後的連續方位頻域包絡
* $\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)$：慢時間離散取樣算子
* $W_{fold}(f_\eta;\omega_s)$：連續頻域包絡被 PRF 週期性複製後的 folded 表示

本質上不是「$\omega_s$ 等於 aliasing」，而是「$\omega_s$ 先改變連續頻譜，取樣再把這個頻譜折回」。

---

**最終結果**

完整時域模型：

$$
s_{1,d}(\tau,\eta) =
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
w_a(\eta;\omega_s) =
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
S_{1,c}(\tau,f_\eta;\omega_s) =
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
W_{fold}(f_\eta;\omega_s) =
\sum_{k=-\infty}^{\infty}
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

folded 方位頻譜：

$$
{\color{red}
S_1(\tau,f_\eta;\omega_s) =
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
S_{1,c}(\tau,f_\eta-k\cdot\mathrm{PRF};\omega_s)
}
$$

fully expanded folded 方位頻譜：

$$
{\color{red}
S_1(\tau,f_\eta;\omega_s) =
\mathrm{PRF}
\sum_{k=-\infty}^{\infty}
A_2\,
\mathrm{sinc}\left[
B_r\left(
\tau-\frac{2R_0}{cD(f_\eta-k\cdot\mathrm{PRF},V_r)}
\right)
\right]
\cdot
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
\cdot
\exp\left(
\Phi_{az}(f_\eta-k\cdot\mathrm{PRF})
\right)
}
$$

近似 folded envelope form：

$$
S_1(\tau,f_\eta;\omega_s) \approx
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

在程式上通常可分成兩個層次：

* 理論 folded operator：以 $\mathrm{PRF}$ 為週期，對連續頻譜或連續包絡做 periodic replication
* 後處理的 unfolded / mosaicked 實作：把不同 replicas 依索引平移後重新排列到延展頻率軸

因此：

* 取樣 comb 對應的是理論上的週期性複製算子
* `append`、`concatenate`、`shift by m * PRF` 對應的是 folded replicas 的重排，不是 folding 本身

---

**限制與適用範圍**

* 本文將 folded phenomenon 的直接來源限定為固定 PRF 的慢時間取樣，未展開 burst slicing、有限 aperture truncation 與加窗效應的高階修正。
* 連續頻譜表示依賴 POSP 與緩變近似，因此更適合說明結構與因果鏈，而不是做最終高精度誤差分析。
* folded 副本的「可還原性」不是由 sampling theorem 自動保證，而是依賴原始訊號具有可建模且可逆的 chirp phase structure。

---

**Appendix A. $\frac{L_a}{\lambda}\theta$ 的來源**

設方位向天線孔徑沿座標 $x\in[-L_a/2,L_a/2]$ 均勻照明，則孔徑分布為

$$
a(x) =
\begin{cases}
1, & |x|\le L_a/2 \\
0, & \text{otherwise}
\end{cases}
$$

在遠場條件下，單程場型振幅為孔徑分布的空間傅立葉轉換：

$$
E(\theta) \propto
\int_{-L_a/2}^{L_a/2}
\exp\left(
+jkx\sin\theta
\right)\,dx
$$

其中 $k=2\pi/\lambda$。直接積分可得

$$
E(\theta) \propto
\frac{
\exp\left(
+jk\frac{L_a}{2}\sin\theta
\right)
-
\exp\left(
-jk\frac{L_a}{2}\sin\theta
\right)
}{
+jk\sin\theta
}
$$

進一步整理為

$$
E(\theta) \propto
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
E(\theta) \propto
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
G_{1\text{-way}}(\theta) =
\mathrm{sinc}\left(
\frac{L_a}{\lambda}\sin\theta
\right)
$$

在小角度近似 $\sin\theta\approx\theta$ 下，

$$
{\color{red}
G_{1\text{-way}}(\theta) \approx
\mathrm{sinc}\left(
\frac{L_a}{\lambda}\theta
\right)
}
$$

對單站雷達而言，發射與接收方向圖相乘，因此雙程方向圖為

$$
w_a(\theta) =
\left|
G_{1\text{-way}}(\theta)
\right|^2
\approx
\mathrm{sinc}^2\left(
\frac{L_a}{\lambda}\theta
\right)
$$

因此 $\frac{L_a}{\lambda}\theta$ 不是任意湊出的無因次量，而是孔徑長度相對於波長的歸一化空間相位差。

---

**Appendix B. $w_a(\eta;\omega_s)$ 的幾何來源**

$w_a$ 最原始是角度 $\theta$ 的函數。要把它改寫成 $w_a(\eta;\omega_s)$，必須先把目標相對於掃描波束中心的離軸角表示為慢時間函數。

設目標最近通過時刻為 $\eta_0$，最近斜距為 $R_0$，平台等效方位速度為 $V_r$。則目標的方位向相對位移為

$$
x(\eta) = V_r(\eta-\eta_0)
$$

故目標相對雷達視線的瞬時角度為

$$
\theta_{tar}(\eta) =
\tan^{-1}\left(
\frac{x(\eta)}{R_0}
\right)
$$

在小角度近似下，

$$
\theta_{tar}(\eta) \approx
\frac{x(\eta)}{R_0} =
\frac{V_r}{R_0}(\eta-\eta_0)
$$

TOPSAR 中，波束中心隨慢時間掃描。若其等效掃描角速度為 $\omega_s$，則

$$
\theta_{beam}(\eta) = \omega_s\eta
$$

因此目標相對波束中心的有效離軸角為

$$
{\color{red}
\theta_{eff}(\eta) =
\theta_{tar}(\eta)-\theta_{beam}(\eta) =
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
}
$$

把此結果代回 Appendix A 的雙程方向圖，可得

$$
{\color{red}
w_a(\eta;\omega_s) =
\mathrm{sinc}^2\left[
\frac{L_a}{\lambda}
\left(
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
\right)
\right]
}
$$

---

**Appendix C. folded 頻譜的可還原性為什麼來自 phase structure**

folding 本身由取樣公式決定，但 folded 副本之所以在 TOPS 中常可被重排與還原，關鍵在於連續訊號具有可建模的 chirp phase structure。

對單點目標，經 POSP 後的連續頻譜可近似寫為

$$
S_{1,c}(\tau,f_\eta;\omega_s) \propto
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

若施加對應的逆二次相位，也就是 deramping / deskew 操作

$$
H_{de}(f_\eta) =
\exp\left(
+j\pi\frac{(f_\eta-f_{ref})^2}{K_{ref}}
\right)
$$

則副本可被映射到近似展平的表示。概念上可寫成

$$
{\color{red}
W_{fold}
\xrightarrow{\ \text{known chirp phase law}\ }
\{\tilde{W}_k\}
\xrightarrow{\ \text{unfold / shift}\ }
\hat{W}_{true}
}
$$

因此：

* folding 由取樣 comb 決定
* 可還原性由 folded 副本共享的 LFM phase structure 決定

若 $K_a$、$f_{dc}$、$\omega_s$ 或參考相位模型無法被穩定估計，則 folded 頻譜就會退化為一般不可逆的 aliasing。
