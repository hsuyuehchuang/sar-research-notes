# SAR TOPS Mode Simulator 🛰️

## Summary

This code provides a test for the TOPS (Terrain Observation with Progressive Scans) mode SAR imaging algorithm.
The code is written in a notebook for convenient debugging.
This README discusses the flowchart and physical explanation of this algorithm.

---

## 2. Block Diagram

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