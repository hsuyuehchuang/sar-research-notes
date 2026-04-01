# TOPS Azimuth Validation Plan

## Navigation

- [Overall](./tops_azimuth_overall.md)
- Related stage notes:
  - [Azimuth Frequency Folding](./azimuth_freq_folding.md)
  - [Azimuth Frequency UFR](./azimuth_freq_ufr.md)
  - [Azimuth Compression](./azimuth_compression.md)
  - [Azimuth Time Folding](./azimuth_time_folding.md)
  - [Azimuth Time UFR](./azimuth_time_ufr.md)

## Table of Contents

- [Summary](#summary)
- [Problem Definition](#problem-definition)
- [Validation Philosophy](#validation-philosophy)
- [Assumptions And Scope](#assumptions-and-scope)
- [1. Test Scene Design](#1-test-scene-design)
- [2. Stage-By-Stage Validation Items](#2-stage-by-stage-validation-items)
- [3. Recommended Observables](#3-recommended-observables)
- [4. Pass Criteria](#4-pass-criteria)
- [5. Suggested Implementation Structure](#5-suggested-implementation-structure)
- [Decision Log](#decision-log)
- [Final Result](#final-result)

## Summary

- 本文件定義如何驗證 TOPS azimuth chain 的每一個主要 stage 是否在數學與物理上都正確。
- 驗證重點不是先做大型 end-to-end scene，而是先用 synthetic point-target scene 檢查每一步是否呈現預期現象。
- 最優先要驗的 stage 是 `spectrum folding`、`mosaicking`、`deramping`、`FFT-based LPF`、`reramping`、`azimuth compression`、`time wrap-around`。
- LPF 的驗證必須以 FFT-based sharp-cut implementation 為主，也就是 `Forward FFT -> zero out unwanted bins -> Inverse FFT`，而不是只驗理想 `rect` 公式。
- 驗證的核心思想是：每一個 stage 都要有可觀察現象、可量測指標、以及明確的 pass condition。

## Problem Definition

本文件要回答三件事：

1. 如何把目前的數學推導轉成可執行的 simulation-based verification。
2. 對每一個 stage，應該觀察哪些訊號、圖形、頻譜或幾何量。
3. 什麼結果可以算作「這一步數學與物理上是正確的」。

## Validation Philosophy

這裡的驗證分成三層：

1. `Signal-form validation`
   檢查某一步輸出訊號是否符合推導中的 closed form 結構，例如 replica 數量、中心位置、bandwidth、phase curvature。

2. `Phenomenon validation`
   檢查物理現象是否真的出現，例如 folded spectrum、主 replica 被 deramp 後搬到近 baseband、time wrap-around 折回主窗口。

3. `Processing-intent validation`
   檢查該步操作是否達成它的工程目的，例如 LPF 是否只保留 desired energy、reramping 是否把主 replica 補回 reference curvature。

因此這份驗證計畫不是只比對一條公式，而是要求

$$
\text{closed-form structure}
\Longleftrightarrow
\text{observable phenomenon}
\Longleftrightarrow
\text{processing intent}
$$

## Assumptions And Scope

- 第一版驗證只使用 synthetic point-target scenes。
- 第一版不追求與真實 Sentinel-1 工程碼 bitwise 一致。
- 第一版優先驗證 azimuth chain，不把整個成像系統的所有誤差源一起放進來。
- 每一個 stage 至少要保存一份複數訊號與一份可視化圖。
- LPF 依照目前文件，使用 FFT-based sharp-cut filtering，再把 resampling 當作下一步處理。

## 1. Test Scene Design

建議至少準備三種 synthetic scene：

1. `Single point target at nominal center`
   用來驗證最基本的 folded / deramp / LPF / compression 是否正常。

2. `Two point targets with different azimuth positions`
   用來驗證 mosaicking 與 time wrap-around 的位置關係，避免只有單點時看不出 replica 排列是否正確。

3. `Point target near ambiguity boundary`
   用來刻意觸發 folded spectrum 或 wrap-around，驗證 UFR 的必要性與效果。

每個 scene 都建議保存：

- complex signal
- magnitude image
- phase image
- one-dimensional cuts
- FFT-domain views

## 2. Stage-By-Stage Validation Items

### 2.1 Spectrum Folding

驗證目標：
- slow-time sampling 後，連續 azimuth spectrum 是否真的出現 PRF 週期性複製

應觀察：
- $|S_{1,c}(\tau,f_\eta)|$
- $|S_2(\tau,f_\eta)|$
- replica 間距是否等於 `PRF`

正確現象：
- folded replicas 在頻域上呈現等間距排列
- replica 中心間距與理論 PRF 一致

### 2.2 Mosaicking

驗證目標：
- folded replicas 是否被正確重排到 extended axis

應觀察：
- mosaicking 前後的 replica 中心
- replica 排列是否從重疊變成分離

正確現象：
- 原本重疊在主頻帶內的能量被攤開
- 每個 replica 的相對順序與中心位置符合推導

### 2.3 Deramping

驗證目標：
- 主 replica 的 quadratic phase curvature 是否被有效拿掉

應觀察：
- deramp 前後的 phase residual
- phase 二次項擬合係數

正確現象：
- 主 replica 的 residual quadratic coefficient 接近 0
- 其他 replicas 仍保有明顯 residual mismatch

### 2.4 FFT-Based LPF

驗證目標：
- FFT-based sharp-cut LPF 是否只保留 desired-energy region

應觀察：
- Forward FFT 後的 spectrum
- keep mask
- zero-out 前後能量分佈
- inverse FFT 後的訊號

正確現象：
- keep region 內能量保留
- keep region 外 bins 被直接清零
- inverse FFT 後主 replica 保持，非主 replica 顯著抑制

### 2.5 Reramping

驗證目標：
- 主 replica 是否被補回 reference curvature

應觀察：
- reramp 前後的 phase law
- 主 replica 與 reference phase 的差值

正確現象：
- reramping 後主 replica phase law 與後續 compression 所需 reference phase 一致

### 2.6 Azimuth Compression

驗證目標：
- 主 replica 是否在 azimuth time 上聚焦成主瓣

應觀察：
- compression 前後 azimuth impulse response
- mainlobe width
- peak location

正確現象：
- 主 replica 壓縮後主瓣變窄
- 峰值位置與理論 target 位置一致

### 2.7 Time Wrap-Around

驗證目標：
- finite FFT 下是否真的出現 circular-convolution fold-back

應觀察：
- $I_{\mathrm{lin}}(\eta)$ 與 $I_{\mathrm{circ}}(\eta)$ 的差異
- ghost 位置是否符合 $\eta_{\mathrm{ghost}} = \eta_c - mT_{\mathrm{window}}$

正確現象：
- 當 $T_{\mathrm{window}} < L_{\mathrm{req}}$ 時，ghost 折回主窗口
- 折回位置與理論週期關係一致

## 3. Recommended Observables

每一個 stage 至少保存以下觀察量：

- complex signal array
- magnitude plot
- phase plot
- one-dimensional central cut
- FFT-domain cut
- fitted quadratic phase coefficient
- energy inside keep region / outside keep region

特別是 LPF 與 deramping，建議額外保存：

- binary keep mask
- masked-bin ratio
- residual curvature estimate

## 4. Pass Criteria

第一版可以先用下列類型的 pass criteria：

- `Replica spacing error`
  replica 中心間距與理論值的相對誤差必須夠小

- `Residual curvature reduction`
  deramping 後主 replica 的二次 phase 係數必須明顯下降

- `Energy rejection ratio`
  LPF 後 keep region 外的能量必須顯著下降

- `Peak localization error`
  壓縮後主峰位置與理論 target 位置的誤差必須夠小

- `Wrap-around location error`
  time ghost 的位置必須符合 $mT_{\mathrm{window}}$ 週期折回公式

初版不一定要先定死數字門檻，但每個指標都必須先能被量測。

## 5. Suggested Implementation Structure

若之後要寫 code simulation，建議拆成以下模組：

1. `scene_generator`
   產生 synthetic point-target scenes

2. `stage_models`
   每一個 stage 各自一個函式：
   - folding
   - mosaicking
   - deramping
   - FFT-based LPF
   - reramping
   - compression
   - time-UFR

3. `measurements`
   負責量測：
   - replica center
   - bandwidth
   - phase curvature
   - energy ratio
   - peak location

4. `plots`
   統一產出 magnitude / phase / spectrum / comparison figures

5. `validation_cases`
   針對不同 synthetic scenes 定義 test cases

這樣未來每一步都能單獨 debug，不會一開始就卡在 end-to-end 黑盒子。

## Decision Log

- 決定：第一版驗證採用流程級訊號驗證，而不是只做單步公式檢查。
- 決定：LPF 必須以 FFT-based sharp-cut implementation 為主，不只寫理想 `rect` 模型。
- 決定：驗證流程獨立成一份文件，不散落在 stage notes 裡。
- 決定：第一版先使用 synthetic point-target scenes。

## Final Result

這份文件定義了一套從數學推導走向 simulation 驗證的主架構：

$$
\text{scene generation}
\Longrightarrow
\text{stage signal logging}
\Longrightarrow
\text{phenomenon measurement}
\Longrightarrow
\text{pass criteria}
$$

因此下一步若要寫 code，目標不是直接做一個大型 end-to-end simulator，而是先做一個可以逐 stage 輸出訊號、圖、與量測指標的 validation-oriented simulator。
