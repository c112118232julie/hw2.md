# 專案管理分析 - PERT/CPM與甘特圖

## 任務清單

| 任務 | 說明 | 需時(天) | 前置任務 |
|------|------|----------|----------|
| 1 | 研擬計畫 | 1 | - |
| 2 | 任務分配 | 4 | 1 |
| 3 | 取得硬體 | 17 | 1 |
| 4 | 程式開發 | 70 | 2 |
| 5 | 安裝硬體 | 10 | 3 |
| 6 | 程式測試 | 30 | 4 |
| 7 | 撰寫使用手冊 | 25 | 5 |
| 8 | 轉換檔案 | 20 | 5 |
| 9 | 系統測試 | 25 | 6 |
| 10 | 使用者訓練 | 20 | 7,8 |
| 11 | 使用者測試 | 25 | 9,10 |

## 1. PERT/CPM 網絡圖

```mermaid
graph TD
    Start([開始]) --> T1[1. 研擬計畫<br/>1天]
    T1 --> T2[2. 任務分配<br/>4天]
    T1 --> T3[3. 取得硬體<br/>17天]
    T2 --> T4[4. 程式開發<br/>70天]
    T3 --> T5[5. 安裝硬體<br/>10天]
    T4 --> T6[6. 程式測試<br/>30天]
    T5 --> T7[7. 撰寫使用手冊<br/>25天]
    T5 --> T8[8. 轉換檔案<br/>20天]
    T6 --> T9[9. 系統測試<br/>25天]
    T7 --> T10[10. 使用者訓練<br/>20天]
    T8 --> T10
    T9 --> T11[11. 使用者測試<br/>25天]
    T10 --> T11
    T11 --> End([結束])
    
    classDef critical fill:#ff9999,stroke:#333,stroke-width:3px
    class T1,T2,T4,T6,T9,T11 critical
```

## 2. 甘特圖

```mermaid
gantt
    title 專案時程甘特圖 (總工期: 155天)
    dateFormat YYYY-MM-DD
    axisFormat %m/%d
    
    section 計畫階段
    研擬計畫 (第0-1天)           :crit, t1, 2024-01-01, 1d
    任務分配 (第1-5天)           :crit, t2, after t1, 4d
    取得硬體 (第1-18天)          :t3, 2024-01-02, 17d
    
    section 開發階段
    程式開發 (第5-75天)          :crit, t4, after t2, 70d
    安裝硬體 (第18-28天)         :t5, after t3, 10d
    程式測試 (第75-105天)        :crit, t6, after t4, 30d
    
    section 整合階段
    撰寫使用手冊 (第28-53天)     :t7, after t5, 25d
    轉換檔案 (第28-48天)         :t8, after t5, 20d
    系統測試 (第105-130天)       :crit, t9, after t6, 25d
    
    section 驗收階段
    使用者訓練 (第53-73天)       :t10, after t7 t8, 20d
    使用者測試 (第130-155天)     :crit, t11, after t9 t10, 25d
```

## 3. 關鍵路徑分析

### 關鍵路徑

```mermaid
graph LR
    A[任務1<br/>研擬計畫] --> B[任務2<br/>任務分配]
    B --> C[任務4<br/>程式開發]
    C --> D[任務6<br/>程式測試]
    D --> E[任務9<br/>系統測試]
    E --> F[任務11<br/>使用者測試]
    
    classDef critical fill:#ff6b6b,stroke:#d63031,stroke-width:3px,color:#fff
    class A,B,C,D,E,F critical
```

**關鍵路徑：** `1 → 2 → 4 → 6 → 9 → 11`
