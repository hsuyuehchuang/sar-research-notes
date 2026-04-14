# DPCA for SAR Imaging

## Outline

- [1. DPCA Concept and Motivation](#1-dpca-concept-and-motivation)
  - [1.1. Why DPCA](#11-why-dpca-is-introduced-in-sar-imaging)
  - [1.2. Displaced Phase Center and Effective PRF](#12-displaced-phase-center-and-effective-along-track-sampling)
  - [1.3. Imaging Enhancement](#13-equivalent-prf-and-imaging-enhancement)
- [2. Geometric Conditions](#2-geometric-conditions)
  - [2.1. Geometric Condition](#21-geometric-condition-between-platform-motion-and-phase-centers)
  - [2.2. Mismatched Spacing Condition](#22-mismatched-spacing-condition)
- [3. Conclusions and Trade-offs](#3-conclusions-and-trade-offs)
  - [3.1. Trade-off between Resolution, NESZ](#31-resolution-nesz-and-panel-trade-off)
  - [3.2. Summary](#32-engineering-design-summary)
  - [3.3. Pros and Cons](#33-dpca-pros-and-cons)
- [4. Follow-up Topics](#4-follow-up-topics)

## Summary

- 為了達成HRWS成像
- DPCA用以增加方位向取樣率（等效PRF），可以讓方位解析度變好並且不會減少swath width
- 但是因為每個resolution cell內可累積的能量下降，SNR變低，NESZ變大
- 所以DPCA的trade-off是用NESZ換azimuth resolution

## 1. DPCA Concept and Motivation

### 1.1. Why Displacement Phase Center Antenna (DPCA)

- **High-resolution (azimuth resolution) and wide-swath (HRWS) are contradictions** with the conventional single-channel spaceborne SAR systems.
- Higher PRF is needed for high-resolution, but it reduces the (unambiguous) swath width.
- DPCA has been successfully applied in the advanced SAR systmes.
- DPCA improves azimuth sampling without requiring the true PRF to increase by the same factor, so the swath-width penalty can be relaxed.

| Goal | Requires | Costs / Limits |
|---|---|---|
| High azimuth resolution | High PRF | Reduced swath width |
| Wide swath width | Low PRF | Reduced azimuth resolution |
| HRWS | DPCA | Added system complexity |

### 1.2. Displaced Phase Center and Effective PRF

- Displaced phase center creates **multiple phase centers** along the flight direction (along-track / azimuth direction), which effectively increase the sampling rate in azimuth.
    - Phase center is the effective transmit/receive center of the antenna aperture.
- Two main DPCA configurations are:
    - **Ping-pong mode**: 
        - The aft-antenna collects the data from the same points as the fore-antenna with a time delay.
        - The condition of baseline $d = v_p T$ is required for the ping-pong mode to achieve the desired effective sampling.
    - **SIMO mode**: 
        - Use one TX and multiple RX to create multiple phase centers.
        - The condition of baseline $d = 2v_p T$ is required for the SIMO mode to achieve the desired effective sampling.

![DPCA Ping-Pong Mode](./figure/dpca-ping-pong.png)
![DPCA SIMO Mode](./figure/dpca-simo.png)

- Once the effective sampling condition is satisfied, the effective PRF is doubled, which can improve the azimuth resolution without reducing the swath width.
- **For example, with two phase centers, the effective PRF doubles.**

### 1.3. Imaging Enhancement

- DPCA increases the effective sampling rate in the along-track direction (PRF), which can be interpreted as an increase in effective PRF.
- This is because the multiple phase centers create additional spatial samples within one pulse interval ( $T = \mathrm{PRI}$ ).

For a conventional single-channel SAR system, the along-track sampling spacing $(\Delta y)$ is

$$
\Delta y = v_p \cdot \mathrm{PRI} = \frac{v_p}{\mathrm{PRF}}
$$

where $v_p$ is the platform velocity and $\mathrm{PRF}$ is the pulse repetition frequency.

If DPCA provides $N$ effective phase centers and the geometric condition is properly matched, the effective sampling spacing becomes

$$ 
\Delta y_{\mathrm{eff}} \approx \frac{v_p}{N \cdot \mathrm{PRF}} \approx \frac{v_p}{\mathrm{PRF}_{\mathrm{eff}}}
$$

Thus, the effective PRF is increased by a factor of $N$

## 2. Geometric Conditions

### 2.1. Geometric Condition

- The effective-sampling interpretation of DPCA is only valid when the **platform motion** and the **phase-center spacing** are properly matched.
- During one pulse repetition interval $T$, the platform moves

$$
\Delta y = v_p T
$$

where $v_p$ is the platform velocity.

- For the ping-pong mode, the matched condition is

$$
d = v_p T
$$

- For the 1-TX/2-RX SIMO case discussed here, the matched condition is

$$
d = 2 v_p T
$$

- Under the mismatched condition, the additional samples are no longer uniformly distributed in azimuth.
-   Therefore, the validity of the effective-PRF interpretation depends on whether the phase-center geometry is matched to the platform motion.

### 2.2. Mismatched Spacing Condition

- In the mismatched case, the phase-center spacing does not satisfy the required geometric relation, so the additional azimuth samples are not placed at the intended along-track positions.
- As a result, the effective sampling grid becomes nonuniform, which means the Doppler-domain samples are no longer supported on the ideal uniform sampling structure.
- Physically, this makes the effective-PRF interpretation less accurate and the expected imaging benefit weaker.
- 還是可以改善解析度，但效果沒有matched case好。

## 3. Conclusions and Trade-offs 

### 3.1. Trade-off between Resolution, NESZ

- Noise-Equivalent Sigma Zero (NESZ，等效雜訊後向散射係數) 是衡量 SAR 系統雜訊水平的核心指標。
    - 定義：NESZ 是雷達系統本身在『未接收到外部信號時』，內部硬體產生的『背景噪聲轉換為後向散射係數 $\sigma^0$ 的數值』，通常以分貝（dB）表示。
      - 後向散射係數（Backscattering Coefficient）是量化目標物將雷達波束反射回雷達接收器的能力之指標。它代表單位面積的反射率，通常以分貝（dB）表示。
    - NESZ 代表雷達內部雜訊對應到的最小可探測地物散射強度。
    - 其數值越低表示雷達系統靈敏度越高，越能分辨低反射率的目標，成像品質越好。
    - 後向散射係數（ $\sigma^0$ Backscattering Coefficient）是量化目標物將雷達波束反射回雷達接收器的能力之指標。它代表單位面積的反射率，通常以分貝（dB）表示。

- Relation between NESZ and SNR: 

$$
\mathrm{SNR} \propto \frac{\sigma^0}{\mathrm{NESZ}}
\propto \frac{1}{\mathrm{NESZ}}
$$

- The bigger Doppler bandwidth, the better (smaller) azimuth resolution.

$$
\delta_a = \frac{V_p}{B_a}
$$

- The result of higher effective PRF:
  - 每個 resolution cell 內可累積的能量下降，或等效處理增益下降
  - SNR 變低
  - NESZ 變大、變差
  - azimuth resolution 變好

- 這裡的 trade-off 是：**用 NESZ 換 azimuth resolution**。

### 3.2. Summary

- From an engineering viewpoint, DPCA should be understood as a system-level trade-off.

- DPCA is beneficial when:
    - HRWS imaging requires better azimuth sampling without proportionally increasing the true PRF.

- The critical condition is:
    - the phase-center geometry must match the platform motion and PRI.

- The main cost is:
    - higher hardware complexity
    - calibration burden
    - multichannel processing complexity

- The main risk is:
    - degraded image quality if reconstruction quality or channel consistency is poor.

- The main trade-off in DPCA-based SAR imaging is between azimuth resolution improvement and NESZ degradation.

### 3.3. Pros and Cons

| Pros | Cons |
|---|---|
| Improves effective azimuth sampling | Increases hardware complexity |
| Helps relax the HRWS conflict between resolution and swath width | Requires channel calibration and multichannel processing |
| Can improve azimuth resolution without proportionally increasing the true PRF | Imaging benefit becomes weaker under geometry mismatch or channel inconsistency |

## 4. Follow-up Topics

- DPCA-based GMTI (Ground Moving Target Indication) and its trade-offs.
- If the channel consistency is poor, the reconstructed azimuth signal is degraded and the expected DPCA benefit becomes weaker.
