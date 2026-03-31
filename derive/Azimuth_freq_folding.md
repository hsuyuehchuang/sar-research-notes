**重點摘要**

* TOPSAR 掃描率 $\omega_s$ 則透過天線方向圖 $w_a$ 改變方位向能量分布。
* 頻譜折疊的直接數學來源是離散取樣；但在 TOPSAR 中，$\omega_s$ 會先展寬連續頻譜 $W_a(f_\eta;\omega_s)$，使其更容易在取樣後形成 folded 頻譜 $W_{fold}(f_\eta;\omega_s)$。
* 因此最後的 folded 頻譜公式必須保留 $\omega_s$，才能清楚分開兩件事：一是取樣造成頻譜折返，二是波束掃描率決定連續頻譜是否被展寬到足以發生折返。

---

**1. 完整時域模型：同時保留 $\omega_s$ 與取樣效應**

設脈衝重複週期為 $T_p = 1/\text{PRF}$，則單點目標的方位向離散回波可寫為：

$$
\begin{aligned}
s_{1,d}(\tau,\eta) =& A_1 \, \text{sinc}[B_r(\tau-\frac{2R(\eta)}{c})]
\cdot w_a(\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta)
\exp\{-j\frac{4\pi f_0 R(\eta)}{c}\}
{\color{red}
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
\end{aligned}
$$

其中

$$
R(\eta)=\sqrt{R_0^2+V_r^2(\eta-\eta_0)^2}
$$

決定幾何距離、空間相位與距離徙動。若以 $s_c(\eta)$ 表示連續慢時間訊號，則其離散取樣後可寫為

$$
{\color{red}
s_d(\eta)=s_c(\eta)\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
$$

因此

$$
{\color{red}
\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)
}
$$

是慢時間取樣算子的數學表示，而非額外的物理散射項。

對小角度近似 $\sin\theta\approx\theta$，若方位向天線長度為 $L_a$，則雙程天線方向圖可近似為：

$$
\begin{aligned}
w_a(\theta)=\text{sinc}^2(\frac{L_a}{\lambda}\theta)
\end{aligned}
$$

其中 $\frac{L_a}{\lambda}\theta$ 的來源見 Appendix A。

在 TOPSAR 中，波束中心以角速度 $\omega_s$ 進行方位掃描，因此方向圖必須以目標相對波束中心的有效離軸角改寫為慢時間函數。其幾何推導見 Appendix B，結果為

$$
\theta_{eff}(\eta)=\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
$$

故 TOPSAR 的時域照明函數為

$$
\begin{aligned}
w_a(\eta;\omega_s)
=\text{sinc}^2[\frac{L_a}{\lambda}(\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta)]
\end{aligned}
$$

此式表明：$\omega_s$ 先透過 beam steering 改變時域照明函數 $w_a(\eta;\omega_s)$，再經由傅立葉轉換進入連續方位頻域包絡 $W_a(f_\eta;\omega_s)$。

**2. 連續方位頻譜：由 $w_a(\eta;\omega_s)$ 導出 $W_a(f_\eta;\omega_s)$**

先忽略離散取樣 comb，只考慮其對應的連續訊號：

$$
\begin{aligned}
s_{1,c}(\tau,\eta) =& A_1 \, \text{sinc}[B_r(\tau-\frac{2R(\eta)}{c})] \\
&\cdot w_a(\eta;\omega_s)
\exp\{-j\frac{4\pi f_0 R(\eta)}{c}\}
\end{aligned}
$$

對 $\eta$ 做方位向傅立葉轉換，並利用駐位相位原理 (POSP)，可得：

$$
\begin{aligned}
S_{1,c}(\tau,f_\eta;\omega_s)
= A_2 \, \text{sinc}[B_r(\tau-\frac{2R_0}{cD(f_\eta,V_r)})]
W_a(f_\eta;\omega_s)
\exp\{\Phi_{az}(f_\eta)\}
\end{aligned}
$$

其中

$$
D(f_\eta,V_r)=\sqrt{1-\frac{c^2f_\eta^2}{4V_r^2f_0^2}}
$$

以及

$$
\Phi_{az}(f_\eta)=-j\frac{4\pi R_0f_0}{c}D(f_\eta,V_r)-j2\pi f_\eta\eta_0
$$

而頻域包絡明確定義為

$$
W_a(f_\eta;\omega_s)=\mathcal{F}_{\eta}\{w_a(\eta;\omega_s)\}
$$

因此，$W_a$ 並不是額外引入的新量，而是時域波束函數 $w_a(\eta;\omega_s)$ 的頻域對應。由於 $w_a$ 含有 $\omega_s$，連續頻域包絡 $W_a$ 也自然保留 $\omega_s$。這正是 TOPSAR 掃描率影響連續方位頻寬的數學來源。若令 $\omega_s=0$，則系統退回不含 beam steering 的情形，頻譜通常不再出現 TOPSAR 額外展寬；但是否完全沒有頻譜折返，仍取決於該連續頻寬相對於 PRF 的大小，而不是因為 $\omega_s=0$ 就自動保證。

**3. 離散取樣後的 folded 頻譜：得到 $W_{fold}(f_\eta;\omega_s)$**

現在把慢時間取樣項乘回來。根據 Poisson Sum Formula，Dirac comb 的頻域表示為：

$$
\begin{aligned}
\mathcal{F}\{\sum_{n=-\infty}^{\infty}\delta(\eta-nT_p)\}
= \text{PRF}\sum_{k=-\infty}^{\infty}\delta(f_\eta-k\cdot\text{PRF})
\end{aligned}
$$

因此時域相乘等效於頻域摺積，離散後的方位頻譜為：

$$
\begin{aligned}
S_1(\tau,f_\eta;\omega_s)
&= S_{1,c}(\tau,f_\eta;\omega_s)
*[\text{PRF}\sum_{k=-\infty}^{\infty}\delta(f_\eta-k\cdot\text{PRF})] \\
&= \text{PRF}\sum_{k=-\infty}^{\infty}S_{1,c}(\tau,f_\eta-k\cdot\text{PRF};\omega_s)
\end{aligned}
$$

將上一節結果代入，可得

$$
\begin{aligned}
S_1(\tau,f_\eta;\omega_s)
&\approx A_2\,\text{sinc}[\dots]\,\exp\{\Phi_{az}(f_\eta)\}
\underbrace{[\sum_{k=-\infty}^{\infty}W_a(f_\eta-k\cdot\text{PRF};\omega_s)]}_{{\color{red}W_{fold}(f_\eta;\omega_s)}}
\end{aligned}
$$

亦即

$$
{\color{red}
W_{fold}(f_\eta;\omega_s)
=\sum_{k=-\infty}^{\infty}W_a(f_\eta-k\cdot\text{PRF};\omega_s)
}
$$

此處的 $W_{fold}$ 建議理解為 folded 後的頻譜，也就是連續頻譜在 PRF 取樣後所形成的週期性折返結果，而不是一般訊號處理語境下那種不可逆的隨機混疊。對 TOPS 而言，這些平移項是由 LFM 型方位訊號與固定 PRF 共同決定的結構化複製，因此更接近 deterministic folding。

這裡保留了完整的依賴鏈：

$$
\omega_s
\;\Longrightarrow\; w_a(\eta;\omega_s)
\;\Longrightarrow\; W_a(f_\eta;\omega_s)
\;\Longrightarrow\; W_{fold}(f_\eta;\omega_s)
$$

因此，整體邏輯應理解為：TOPS 掃描率 $\omega_s$ 先透過 $w_a(\eta;\omega_s)$ 改變時域照明，再經傅立葉轉換形成連續頻域包絡 $W_a(f_\eta;\omega_s)$；最後，PRF 取樣才把這個連續頻譜週期性複製並折回主頻帶，形成 $W_{fold}(f_\eta;\omega_s)$。換言之，$\omega_s$ 控制的是連續頻譜的展寬，而 folded 行為本身則來自離散取樣。由於 TOPS 方位向訊號本質上是 LFM 訊號，這些 folded 副本具有可預測的相位結構，而不是彼此無關的隨機頻率成分。若沒有 beam steering，系統可退回近似 stripmap 的情形，此時少了 TOPSAR 額外展寬機制，通常較容易避免這類 folded 頻譜；但數學上是否發生折返，仍須由「連續頻寬是否超過取樣所允許的無混疊範圍」來判定。

---

**Appendix A. $\frac{L_a}{\lambda}\theta$ 的來源**

設方位向天線孔徑沿座標 $x\in[-L_a/2,L_a/2]$ 均勻照明，則其孔徑分布可寫為

$$
a(x)=
\begin{cases}
1, & |x|\le L_a/2 \\
0, & \text{otherwise}
\end{cases}
$$

在遠場條件下，方向圖為孔徑分布的空間傅立葉轉換，因此單程場型振幅可寫為

$$
E(\theta)\propto \int_{-L_a/2}^{L_a/2}\exp(jk x\sin\theta)\,dx
$$

其中波數 $k=2\pi/\lambda$。直接積分得

$$
\begin{aligned}
E(\theta)
&\propto
\Bigl.
\frac{\exp(jk x\sin\theta)}{jk\sin\theta}
\Bigr|_{-L_a/2}^{L_a/2} \\
&=
\frac{\exp(jk\frac{L_a}{2}\sin\theta)-\exp(-jk\frac{L_a}{2}\sin\theta)}{jk\sin\theta} \\
&=
\frac{2\sin(k\frac{L_a}{2}\sin\theta)}{k\sin\theta}
\end{aligned}
$$

代入 $k=2\pi/\lambda$ 可得

$$
E(\theta)\propto
L_a\,
\frac{\sin(\pi\frac{L_a}{\lambda}\sin\theta)}
\pi\frac{L_a}{\lambda}\sin\theta
$$

因此其歸一化單程方向圖為

$$
G_{1\text{-way}}(\theta)=
\mathrm{sinc}(\frac{L_a}{\lambda}\sin\theta)
$$

在小角度近似 $\sin\theta\approx\theta$ 下，即得

$$
{\color{red}
G_{1\text{-way}}(\theta)\approx
\mathrm{sinc}(\frac{L_a}{\lambda}\theta)
}
$$

對單站雷達而言，發射與接收方向圖相乘，因此雙程方向圖為

$$
{\color{red}
w_a(\theta)=|G_{1\text{-way}}(\theta)|^2
\approx
\mathrm{sinc}^2(\frac{L_a}{\lambda}\theta)
}
$$

故 $\frac{L_a}{\lambda}\theta$ 並不是任意寫入的無因次變數，而是孔徑長度相對波長的歸一化空間相位差。當觀測角度偏移 $\theta$ 時，孔徑兩端的路徑差約為 $L_a\sin\theta\approx L_a\theta$，再除以波長 $\lambda$ 後，即得到控制主瓣與旁瓣結構的無因次相位量。

---

**Appendix B. $w_a(\eta;\omega_s)$ 的幾何來源**

$w_a$ 最原始是角度 $\theta$ 的函數，而不是慢時間 $\eta$ 的函數。因此，寫成 $w_a(\eta)$ 或 $w_a(\eta;\omega_s)$ 的關鍵，在於先將目標相對波束中心的離軸角表示為慢時間函數，再代回方向圖函數。

設目標最近通過時刻為 $\eta_0$，最近斜距為 $R_0$，平台沿方位向的等效速度為 $V_r$。則在慢時間 $\eta$ 下，平台與目標於方位向的相對位移為

$$
x(\eta)=V_r(\eta-\eta_0)
$$

因此目標相對雷達視線的瞬時方位角為

$$
\theta_{tar}(\eta)=\tan^{-1}(\frac{x(\eta)}{R_0})
$$

在小角度近似下，

$$
\theta_{tar}(\eta)\approx \frac{x(\eta)}{R_0}
=\frac{V_r}{R_0}(\eta-\eta_0)
$$

若不考慮 beam steering，則方向圖可直接寫為

$$
w_a(\eta)=w_a(\theta_{tar}(\eta))
=w_a(\frac{V_r}{R_0}(\eta-\eta_0))
$$

在 TOPSAR 模式下，波束中心會隨慢時間進行掃描。若其等效掃描角速度為 $\omega_s$，則波束中心瞬時指向角可近似表示為

$$
\theta_{beam}(\eta)=\omega_s\eta
$$

因此，目標相對掃描波束中心的有效離軸角為

$$
\theta_{eff}(\eta)=\theta_{tar}(\eta)-\theta_{beam}(\eta)
$$

代入後得

$$
\boxed{
\theta_{eff}(\eta)=\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
}
$$

故 TOPSAR 的方向圖項可寫為

$$
w_a\bigl(\theta_{eff}(\eta)\bigr)
=
w_a(\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta)
$$

再代入 Appendix A 中的雙程方向圖近似式

$$
w_a(\theta)\approx \mathrm{sinc}^2(\frac{L_a}{\lambda}\theta)
$$

即可得到

$$
\boxed{
w_a(\eta;\omega_s)
=\mathrm{sinc}^2[
\frac{L_a}{\lambda}
(
\frac{V_r}{R_0}(\eta-\eta_0)-\omega_s\eta
)
]
}
$$

此式顯示：TOPS beam steering 並不是直接作用於頻譜，而是先改變時域照明函數的有效離軸角，再透過傅立葉轉換影響連續方位頻域包絡 $W_a(f_\eta;\omega_s)$。

---

**Appendix C. folded 頻譜可還原性的相位模型**

TOPS 中 folded phenomenon 之所以原理上可還原，關鍵不在於取樣後資訊自動保留，而在於原始方位向訊號具有可建模的 LFM 相位律。對單點目標，經 POSP 後的連續方位頻譜可近似寫為

$$
S_{1,c}(\tau,f_\eta;\omega_s)
\propto
W_a(f_\eta;\omega_s)
\exp\{-j\pi \frac{(f_\eta-f_{dc})^2}{K_a}\}
$$

其中 $K_a$ 為等效方位 FM rate，$f_{dc}$ 為 Doppler centroid。故第 $k$ 個 folded 副本可表示為

$$
W_a(f_\eta-k\cdot\mathrm{PRF};\omega_s)
\exp\{-j\pi \frac{(f_\eta-k\cdot\mathrm{PRF}-f_{dc})^2}{K_a}\}
$$

此式表明，各 folded 副本並非彼此無關，而是共享同一組二次相位 chirp law。若施加對應的逆二次相位，即 deramping / deskew 操作

$$
\boxed{
H_{de}(f_\eta)
=
\exp\{+j\pi \frac{(f_\eta-f_{ref})^2}{K_{ref}}\}
}
$$

則 folded 副本可被映射至近似展平的表示。於是其逆運算可概念化為：先去除已知 chirp phase，再將各 folded 副本依其索引搬回原始位置，最後重新合成連續頻譜。因此其可還原性的核心可寫為

$$
\boxed{
W_{fold}
\xrightarrow{\ \text{known chirp phase law}\ }
\{\tilde{W}_k\}
\xrightarrow{\ \text{unfold / shift}\ }
\hat{W}_{true}
}
$$

因此，folding 本身由取樣公式刻畫，而可還原性則來自 folded 副本之間共享且可逆的 LFM 相位結構。若該相位律不存在，或 $K_a$、$f_{dc}$、$\omega_s$ 無法被穩定估計，則 folded 頻譜將退化為一般 aliasing 的形式，而不再保證可逆。
