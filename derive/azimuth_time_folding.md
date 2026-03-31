**Azimuth-Time Folding / Wrap-around Derivation**

**重點摘要**

- TOPS 的 beam steering 使單一 burst 內被照射到的目標，其等效方位時間跨度明顯大於 stripmap。
- 這個「目標時間跨度」若超過後續方位向 FFT 所能承載的無模糊時間窗口，就不可能再以線性卷積方式完整聚焦。
- 一旦以 FFT 實作方位壓縮而未提供足夠 zero-padding，處理結果就會退化為 circular convolution。
- 因此，超出時間窗口的方位響應會以 $T_{\mathrm{window}}=N_a/\mathrm{PRF}$ 為週期折回主影像區域，形成 azimuth-time wrap-around。

---

**1. 問題定義**

本文件要回答的是「為什麼在方位角壓縮之後，影像時域端會出現 wrap-around，也就是頭尾折回主視窗的 ghost response？」

這件事的關鍵不在於單純的頻譜展寬，而在於：

1. TOPS beam steering 使 burst 內的有效目標分布時間跨度變長。
2. 方位壓縮通常以 FFT 實作，FFT 對應的是有限長度、週期延拓的 circular convolution。
3. 當有效目標跨度超過 FFT 對應的無模糊時間窗口時，超出部分必然被週期性折回。

所以本文件將推導拆成兩層：

- 幾何層：為什麼 TOPS 使目標時間跨度變長？
- 訊號處理層：為什麼過長的時間跨度在 FFT 壓縮後會變成 wrap-around？

---

**2. 幾何起點：beam footprint 的地面移動**

設慢時間為 $\eta$，burst 長度為 $T_p$，載台沿方位向的等效速度為 $v_p$。

若波束不掃描，則 stripmap 模式下地面波束中心的軌跡為

$$
y_{\mathrm{strip}}(\eta)=v_p\eta
$$

在 TOPS 模式下，波束還具有額外的方位掃描。若以 $\omega_s$ 表示此掃描在地面上對應的等效線性速度，則地面波束中心的軌跡為

$$
y_{\mathrm{TOPS}}(\eta)=\left(v_p+\omega_s\right)\eta
$$

這裡的關鍵不是 $\omega_s$ 的精確機電定義，而是：

- 在 burst 期間內，波束中心掃過的地面距離由 $v_pT_p$ 增加為 $(v_p+\omega_s)T_p$
- 因此同一個 burst 內被照射到的目標集合，在地面上被拉得更長

---

**3. 從地面跨度轉成等效方位時間跨度**

方位向地面座標 $y$ 可對應回載台慢時間座標

$$
\eta_{\mathrm{tar}}=\frac{y}{v_p}
$$

因此，beam footprint 在 burst 內掃過的地面距離，可等價地解讀成「被照射目標在方位慢時間上的總跨度」。

對 stripmap，

$$
\Delta y_{\mathrm{strip}}=v_pT_p
$$

故其等效目標時間跨度為

$$
T_{\mathrm{focus,strip}}
=
\frac{\Delta y_{\mathrm{strip}}}{v_p}
=
T_p
$$

對 TOPS，

$$
\Delta y_{\mathrm{TOPS}}
=
\left(v_p+\omega_s\right)T_p
$$

故其等效目標時間跨度為

$$
T_{\mathrm{focus,TOPS}}
=
\frac{\Delta y_{\mathrm{TOPS}}}{v_p}
=
T_p\left(1+\frac{\omega_s}{v_p}\right)
$$

因此只要 $\omega_s>0$，就有

$$
T_{\mathrm{focus,TOPS}}>T_{\mathrm{focus,strip}}
$$

若 $\omega_s\gg v_p$，則甚至有

$$
T_{\mathrm{focus,TOPS}}\gg T_p
$$

這就是 TOPS 在同一個 burst 內「塞進更多目標」的嚴格數學表述。

---

**4. 這個跨度為什麼和方位壓縮有關**

方位壓縮本質上是在每一個目標的 Doppler 歷程上做 matched filtering。

若單一目標的參考函數長度記為 $T_{\mathrm{ref}}$，那麼要對一整批分布在時間軸上的目標做線性卷積，處理窗口至少必須能容納：

$$
L_{\mathrm{req}}
=
T_{\mathrm{focus}}+T_{\mathrm{ref}}
$$

因此：

對 stripmap，

$$
L_{\mathrm{req,strip}}
=
T_{\mathrm{focus,strip}}+T_{\mathrm{ref}}
=
T_p+T_{\mathrm{ref}}
$$

對 TOPS，

$$
L_{\mathrm{req,TOPS}}
=
T_{\mathrm{focus,TOPS}}+T_{\mathrm{ref}}
=
T_p\left(1+\frac{\omega_s}{v_p}\right)+T_{\mathrm{ref}}
$$

這裡最重要的不是 $T_{\mathrm{ref}}$ 的精確閉式，而是：

- $T_{\mathrm{focus}}$ 變長會直接推高線性卷積所需的總處理長度
- TOPS 使這個需求遠大於 stripmap

---

**5. FFT 壓縮對應的無模糊時間窗口**

實務上方位壓縮通常透過 FFT 實作。若方位向 FFT 點數為 $N_a$，脈衝重複頻率為 $\mathrm{PRF}$，則 FFT 對應的時間窗口長度為

$$
T_{\mathrm{window}}
=
\frac{N_a}{\mathrm{PRF}}
$$

這表示 IFFT 回來後的方位時間軸，只在一個長度為 $T_{\mathrm{window}}$ 的基本區間內表示結果；超出這個區間的成分會因 DFT 的週期性被視為下一個週期的內容。

因此若要保證方位壓縮仍等價於線性卷積，至少需要

$$
T_{\mathrm{window}}\ge L_{\mathrm{req}}
$$

對 TOPS 而言，若仍沿用與 stripmap 類似的 block 大小，或受限於記憶體與效率而無法把 $N_a$ 做得足夠大，就很容易落入

$$
T_{\mathrm{window}}<L_{\mathrm{req,TOPS}}
$$

這就是 wrap-around 發生的必要條件。

---

**6. 為什麼不足的窗口會變成 circular convolution**

若用 FFT 做匹配濾波，運算形式是

$$
I(\eta)
=
\mathrm{IFFT}
\left[
\mathrm{FFT}\left[s(\eta)\right]\cdot
\mathrm{FFT}\left[h(\eta)\right]
\right]
$$

在有限長 DFT 下，這不是無條件地等於線性卷積，而是等於 circular convolution：

$$
I_{\mathrm{circ}}(\eta)
=
\sum_{m=-\infty}^{\infty}
I_{\mathrm{lin}}(\eta-mT_{\mathrm{window}})
$$

其中 $I_{\mathrm{lin}}$ 是理想的線性卷積結果。

所以只要 $I_{\mathrm{lin}}(\eta)$ 的有效支撐區超出基本區間，超出的部分就會被以 $T_{\mathrm{window}}$ 為週期搬移回來。

也就是說：

- 線性卷積希望所有目標都留在原本的正確方位位置
- circular convolution 會把超出窗口的部分當作週期延拓，折回主區間

這就是 wrap-around 的嚴格訊號處理來源。

---

**7. Wrap-around 的位置公式**

設某個目標在理想線性卷積下的聚焦中心位於 $\eta_c$。

若它落在 FFT 基本時間窗口之外，也就是

$$
\left|\eta_c\right|>\frac{T_{\mathrm{window}}}{2}
$$

則在實際的 circular convolution 輸出中，該響應會出現在某個折回後的位置 $\eta_{\mathrm{ghost}}$，滿足

$$
\eta_{\mathrm{ghost}}
=
\eta_c-mT_{\mathrm{window}}
$$

其中 $m$ 為某個非零整數，且需使

$$
\eta_{\mathrm{ghost}}
\in
\left[-\frac{T_{\mathrm{window}}}{2},\frac{T_{\mathrm{window}}}{2}\right]
$$

因此 wrap-around 不是「新的假訊號被生成」，而是原本正確的線性卷積響應，被 DFT 的週期性強制折回到了主觀測窗口內。

---

**8. TOPS 為什麼比 stripmap 更容易出現這件事**

stripmap 中，

$$
T_{\mathrm{focus,strip}}=T_p
$$

通常較容易透過合理的 zero-padding 與 FFT 長度設計，滿足

$$
T_{\mathrm{window}}\ge T_p+T_{\mathrm{ref}}
$$

但在 TOPS 中，

$$
T_{\mathrm{focus,TOPS}}
=
T_p\left(1+\frac{\omega_s}{v_p}\right)
$$

當 $\omega_s/v_p$ 很大時，所需的線性卷積窗口會顯著增長。因此即使原始 burst 長度仍是 $T_p$，真正需要被完整聚焦的目標集合卻已經跨越了遠大於 $T_p$ 的方位時間範圍。

這正是 TOPS 比 stripmap 更容易在 azimuth-time domain 看到 folding / wrap-around 的根本原因。

---

**9. 最終結論**

整個邏輯鏈可濃縮為

$$
\text{beam steering}
\;\Longrightarrow\;
\text{larger ground footprint in one burst}
\;\Longrightarrow\;
\text{larger }T_{\mathrm{focus,TOPS}}
$$

$$
\Longrightarrow\;
\text{larger }L_{\mathrm{req,TOPS}}
\;\Longrightarrow\;
T_{\mathrm{window}}<L_{\mathrm{req,TOPS}}
\;\Longrightarrow\;
\text{circular convolution}
$$

$$
\Longrightarrow\;
\text{azimuth-time wrap-around}
$$

因此，方位角時域的 wrap-around 並不是一個獨立的數值瑕疵，而是由下列兩件事共同決定的必然結果：

- TOPS 幾何掃描使有效目標時間跨度增加
- FFT 壓縮在有限窗口下只能實現 circular convolution

只要前者把目標響應撐到超出後者的可承載窗口，折回就一定會發生。

---

**10. 一句話版本**

TOPS 先把同一個 burst 內的目標方位時間跨度拉長；若後續方位壓縮的 FFT 時間窗口不足以容納這個拉長後的總卷積長度，則超出部分必然依 DFT 的週期性折回主區間，於 azimuth-time domain 形成 wrap-around。
