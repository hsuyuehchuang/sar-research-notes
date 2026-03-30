# SAR TOPS Mode Simulator 🛰️

## 1. 專案摘要 (Executive Summary)
本專案實作合成孔徑雷達 (SAR) 的 **TOPS (Terrain Observation with Progressive Scans)** 模式成像演算法。
TOPS 模式透過主動旋轉天線波束，克服了傳統 ScanSAR 的 Scalloping 效應，提供更均勻的信噪比與寬廣的測繪帶。

* **核心價值：** 實作方位向解斜頻 (Azimuth Deramp) 處理，解決 TOPS 模式特有的多普勒頻率走動問題。
* **開發環境：** Python 3.x (NumPy, SciPy, Matplotlib)。

---

## 2. 系統架構圖 (System Architecture)
使用 Mermaid 語法定義的演算法處理鏈：

```mermaid
graph TD
    %% 輸入
    In1[/RAW Radar Data/] --> P1
    In2[/Orbit & Attitude/] --> P1

    %% 演算法流程
    subgraph Algorithm_Core [TOPS Processing Chain]
        P1[Data Preprocessing] --> P2[Range Compression]
        P2 --> P3[Azimuth Deramp]
        P3 --> P4[RCMC]
        P4 --> P5[Azimuth Compression]
        P5 --> P6[Image Formation]
    end

    %% 輸出
    P6 --> Out1[/Focused SAR Image/]
    P6 --> Out2[/Quality Analysis/]