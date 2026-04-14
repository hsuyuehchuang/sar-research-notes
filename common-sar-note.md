# Common SAR note:

## Azimuth Resolution：

### Summary

- 依據訊號處理原理，空間解析度正比於速度、反比於都卜勒頻寬：

$$
\delta_a = \frac{V_p}{B_a}
$$

- 在理想的條帶模式 (Stripmap SAR) 下，推導出的物理極限為：

$$
\delta_a = \frac{L}{2}
$$

（其中 $V_p$ 為載台速度， $B_a$ 為都卜勒頻寬， $L$ 為實體天線方位向長度）。

- 實體天線 $L$ 越短，波束（實體天線照射波束）越寬，反而能「看目標看越久」，累積更大的都卜勒頻寬，最終得到更細的極限解析度。
- 「看越久越細」：看越久代表合成孔徑時間 ( $T_a$ ) 越長，能合成出的「虛擬天線陣列」越長，波束（ 合成孔徑成像後的等效主瓣 / 解析能力）越窄，解析度自然越高。
- 「都卜勒頻寬越大越細」:此處的頻寬指都卜勒頻寬 ( $B_a$ )。根據傅立葉轉換性質，頻域上的頻寬越寬，時域（或空間域）上壓縮後的 Sinc 函數主瓣就越窄，解析度越精細。

---

### 詳細物理意義與數學推導

- 在 SAR 系統中，方位向的壓縮本質上是對目標產生的都卜勒頻率歷程 (Doppler History)進行匹配濾波 (Matched Filtering)。

#### 1. 為什麼「(Doppler) Bandwidth 越大，解析度越細」？

- 在方位向 (慢時間 $\eta$ ) 上，接收到的訊號經過匹配濾波後，其包絡線會呈現 Sinc 函數的形式。
- Sinc 函數的主瓣寬度 (即方位時間解析度 $\Delta \eta$ ) 反比於訊號的頻寬：


Proof:
在訊號處理中，最標準的矩形函數 (Rect function) 與 Sinc 函數的傅立葉轉換對 (Fourier Transform Pair) 形式如下：

$$
\mathrm{F} \biggl[ \mathrm{rect} \biggl( \frac{t}{\tau} \biggr) \biggr] = \tau \cdot \mathrm{sinc}(f\tau)
$$

Pulse長度越短（$\tau$ 越小），頻寬就越寬（Sinc 主瓣變寬），反之亦然

若方位向頻譜近似為寬度 $B_a$ 的矩形函數，則其反傅立葉轉換可寫成

$$
s(\eta) = B_a \mathrm{sinc}(B_a \eta)
$$

其中 $\mathrm{sinc}(x) = \frac{\sin(\pi x)}{\pi x}$ 

而 $\mathrm{sinc}(B_a \eta)$ 的第一個零點滿足

$$
B_a \eta = \pm 1
\quad \Rightarrow \quad
\eta = \pm \frac{1}{B_a}
$$

因此主瓣寬度大約為

$$
\Delta \eta \sim \frac{2}{B_a}
$$

可得時間解析度與頻寬成反比。

$$\Delta \eta \approx \frac{1}{B_a}$$

將時間解析度乘上載台的等效速度 $V_p$ ，即可得到空間中的方位向解析度 $\delta_a$ ：

$$\delta_a = V_p \cdot \Delta \eta = \frac{V_p}{B_a}$$

都卜勒頻寬 $B_a$ 越大，Sinc 函數越窄，解析度越細。

#### 2. 為什麼「看越久，解析度越細」？

- 「看越久」在數學上對應的是方位向照射時間 (Illumination Time, $T_a$ ) 的增加。

- 當雷達波束掃過一個點目標時，目標與雷達的相對距離 $R(\eta)$ 會產生非線性變化，進而產生都卜勒頻移 $f_d(\eta)$ 。
- 在小角度近似下，都卜勒頻移隨時間呈線性變化 (即線性調頻訊號 LFM)：

$$f_d(\eta) \approx K_a \cdot \eta$$

其中 $K_a$ 為都卜勒調頻率 (Doppler Frequency Rate)：

$$K_a = -\frac{2 V_p^2}{\lambda R_0}$$

$\lambda$ 為波長， $R_0$ 為最短斜距。

- 目標被波束照射的總時間 $T_a$ (看多久) 決定了我們能收集到多寬的都卜勒頻率範圍，這就是都卜勒頻寬 $B_a$ ：

$$B_a = |K_a| \cdot T_a = \frac{2 V_p^2}{\lambda R_0} \cdot T_a$$

- 當我們「看越久」( $T_a$ 越大)，累積的都卜勒頻率變化範圍就越大 ( $B_a$ 越大)。再代回前面的公式 $\delta_a = \frac{V_p}{B_a}$ ，即可證明解析度越細。這也是為什麼 Spotlight 模式 (聚束模式) 透過控制波束一直盯著目標，延長了 $T_a$ ，從而能獲得比 Stripmap 模式更高的解析度。

#### 3. 理論極限：為何最終公式是 $\delta_a = L/2$？

- 如果我們將上述公式展開，將實體天線的物理限制帶入：
- 實體天線的波束寬度 (Beamwidth)： $\theta_{bw} \approx \frac{\lambda}{L}$ 
- 合成孔徑長度： $L_s = \theta_{bw} \cdot R_0 = \frac{\lambda R_0}{L}$ 
- 方位向照射時間可以表示為： $T_a = \frac{L_s}{V_p} = \frac{\lambda R_0}{V_p L}$ 

我們將 $T_a$ 代入都卜勒頻寬的公式中：

$$
B_a = |K_a| \cdot T_a 
= \biggl( \frac{2 V_p^2}{\lambda R_0} \biggr) \cdot \biggl( \frac{\lambda R_0}{V_p L} \biggr) 
= \frac{2 V_p}{L}
$$

最後，代回方位向解析度的基本定義：

$$
\delta_a = \frac{V_p}{B_a} = \frac{V_p}{\frac{2 V_p}{L}} = \frac{L}{2}
$$

- 這是一個非常漂亮的反直覺結果：在傳統雷達中，天線越長波束越窄、解析度越好；但在 SAR 的方位向處理中，
- 實體天線 $L$ 越短，波束越寬，反而能「看目標看越久」，累積更大的都卜勒頻寬，最終得到更細的極限解析度。
