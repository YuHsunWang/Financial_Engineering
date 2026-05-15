# HW1 — 等額分期還款表（Loan Amortization Schedule）

## 主題

計算每月定額本金還款的完整明細表，包含每期的償還本金、利息、累計金額以及剩餘未還本金。

## 公式說明

**每期償還本金**

$$\text{payment} = \left\lceil \frac{p}{12t} \right\rceil \quad (\text{最後一期直接結清剩餘本金})$$

**每期利息**

$$\text{interest} = \text{round}\!\left( p_{\text{剩餘}} \times \frac{r}{12} \right)$$

**剩餘未還本金**

$$p_{\text{剩餘}} \leftarrow p_{\text{剩餘}} - \text{payment}$$

## 流程圖

```mermaid
flowchart TD
    A([匯入套件\nmath / numpy / pandas]) --> B[設定參數\n本金 p、利率 r、期數 t]
    B --> C[初始化五個 List\n期數 / 償還本金 / 償還利息 / 累計 / 剩餘本金]
    C --> D{開始迴圈\n月份 0 → 12t-1}
    D --> E[計算當期利息]
    E --> F{最後一期？}
    F -- 是 --> G[本期還款額 = 剩餘本金\n一次結清]
    F -- 否 --> H[本期還款額 = 無條件進位\n本金 / 總期數]
    G --> I[更新累計金額與剩餘本金\n寫入各 List]
    H --> I
    I --> D
    D -- 迴圈結束 --> J[將五個 List 組合成 DataFrame]
    J --> K([輸出還款時間表])
```

## 使用方法

開啟 [HW1.ipynb](HW1.ipynb)，在「參數設定」區塊修改以下三個變數後執行全部儲存格：

```python
p = 100000  # 本金
r = 0.05    # 年利率
t = 7       # 還款年數
```

## 學習心得

本題的核心在於熟悉 Python 迴圈與 List 的操作。  
利率是以年利率 / 12 計算月利息，且最後一期需要特別處理，將剩餘本金一次結清，避免浮點數誤差導致尾差。  
最後以 pandas DataFrame 整理輸出，使結果易於閱讀。
