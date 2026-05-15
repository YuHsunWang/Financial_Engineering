# 財務工程作業集 — 王裕勛

**學號：** R08323031 ｜ **系所：** 經碩一

---

## 目錄

| 作業 | 主題 | 核心方法 | Notebook |
|------|------|----------|----------|
| [HW1](#hw1) | 等額分期還款表 | 迴圈計算、DataFrame | [HW1.ipynb](HW1/HW1.ipynb) |
| [HW2](#hw2) | 殖利率 / 即期利率 / 遠期利率 | 數值求解（SymPy Solve）、Bootstrap | [HW2.ipynb](HW2/HW2.ipynb) |
| [HW3](#hw3) | 二項式選擇權定價（BOPM） | 倒推法、二項樹 | [HW3.ipynb](HW3/HW3.ipynb) |
| [HW4](#hw4) | Black-Scholes 含股利 | BS 公式、常態分佈 | [HW4.ipynb](HW4/HW4.ipynb) |
| [HW5](#hw5) | Hull-White + Monte Carlo 選擇權定價 | 短期利率模擬、幾何布朗運動 | [HW5.ipynb](HW5/HW5.ipynb) |

---

## HW1

### 等額分期還款表（Loan Amortization Schedule）

計算每月還款金額，拆解為償還本金、利息、累計總額及剩餘本金，並以 DataFrame 呈現完整還款時間表。

**技術重點：** 等額本金分期公式、Python 迴圈、pandas DataFrame

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

**輸出範例：**

| 期數 | 償還本金 | 償還利息 | 本金利息累計 | 剩餘未還本金 |
|------|---------|---------|------------|------------|
| 1 | 1,191 | 417 | 1,608 | 98,809 |
| … | … | … | … | … |
| 84 | 1,147 | 5 | 117,702 | 0 |

---

## HW2

### 殖利率 / 即期利率 / 遠期利率（YTM / Spot Rate / Forward Rate）

從債券市場價格出發，先求 YTM，再以 Bootstrap 逐期推導即期利率，最後計算遠期利率。

**技術重點：** 等比級數折現公式、SymPy `solve()`、Bootstrap 逐期求解

```mermaid
flowchart TD
    A([匯入套件\npandas / numpy / sympy]) --> B[輸入參數\nBond Price、Par Value\nCoupon Rate、Maturity、Payment]
    B --> C[計算每期付息 PMT]
    C --> D[以等比級數化簡折現公式\n建立 YTM 方程式]
    D --> E[sympy solve 求根\n篩選正實數解]
    E --> F([輸出 YTM])

    C --> G[Bootstrap 逐期求 Spot Rate]
    G --> H[第 1 期：單期折現方程式\nsolve 求 S₁]
    H --> I[第 2 期：代入 S₁ 建立方程式\nsolve 求 S₂]
    I --> J[依此類推至第 n 期]
    J --> K([輸出 Spot Rate 列表])

    K --> L[由相鄰 Spot Rate 推算\n各期 Forward Rate]
    L --> M([輸出 Forward Rate])
```

---

## HW3

### 二項式選擇權定價模型（Binomial Option Pricing Model）

建立完整的股價二項樹，利用風險中立機率與倒推法計算買權與賣權價格，同時輸出避險比率（Delta）與債券倉位（B）。

**技術重點：** 風險中立定價、Backward-induction、二維陣列

```mermaid
flowchart TD
    A([匯入套件\nnumpy / scipy]) --> B[輸入參數\nS、u、d、r、X、n]
    B --> C[計算連續複利因子 R\n與風險中立機率 P]
    C --> D[正向建立股價二項樹\n同時計算各節點機率\n二項式定理]
    D --> E[計算到期損益\nCall 與 Put 各取 max 與 0]
    E --> F[倒推法\ni 從 n-1 反向推回 0\n計算 Call 與 Put 價格樹]
    F --> G[計算各節點避險比率 Δ]
    G --> H[計算各節點債券倉位 B]
    H --> I([Print 股價樹 / 買權表 / 賣權表])
```

**輸出範例（買權，節點格式：價格（Δ））：**

```
108.842 (0.903)
173.932 (0.941)   29.526 (0.633)
276.476 (0.972)   50.616 (0.723)    0.0 (0.0)
...
```

---

## HW4

### Black-Scholes 含離散股利（Black-Scholes with Discrete Dividends）

以 Black-Scholes 公式計算含已知現金股利的歐式選擇權價格，先將股利折現從股價扣除，再代入 BS 公式。

**技術重點：** Black-Scholes 公式、現金股利折現調整、常態分佈 CDF

```mermaid
flowchart TD
    A([匯入套件\nscipy / pandas / numpy]) --> B[輸入參數\nS、K、dividend_list\ndividend_month、sigma、r、due]
    B --> C[zip 遍歷股利金額與月份\n計算各筆現值並加總得 D]
    C --> D[從股價扣除股利現值\n得調整後股價 S_hat]
    D --> E[計算 d₁]
    E --> F[計算 d₂]
    F --> G[查標準常態分佈 CDF\n取得四個累積機率值]
    G --> H[代入 Black-Scholes 公式\n計算 Call 與 Put]
    H --> I([Print Call & Put 價格])
```

**輸出：**
- Call = 12.80
- Put = 2.85

---

## HW5

### Hull-White + 幾何布朗運動 Monte Carlo 選擇權定價

以 Hull-White 模型模擬隨機短期利率，再將短期利率作為漂移項代入幾何布朗運動（GBM），透過 Monte Carlo 模擬大量路徑，取到期損益的期望值後折現，得到選擇權價格。

**技術重點：** Hull-White 短率模型、GBM、Monte Carlo 模擬、QuantLib

```mermaid
flowchart TD
    A([匯入套件\nQuantLib / numpy / matplotlib]) --> B[輸入參數\nT、σ、S₀、K、a\nforward_rate、timestep、num_paths]
    B --> C[建立 Hull-White 模型\nFlat Forward Curve\nGaussianPathGenerator]
    C --> D[模擬 num_paths 條短率路徑\n每條含 timestep+1 個時步]
    D --> E[繪製短率模擬路徑圖\n視覺化檢查]
    E --> F[以短率作為漂移項\n代入幾何布朗運動\n模擬對應股價路徑]
    F --> G[計算每條路徑的到期損益\nCall 與 Put 分別取 max]
    G --> H[對所有路徑取損益期望值]
    H --> I[折現至現值]
    I --> J([Print Call Price & Put Price])
```

**輸出：**
- Call ≈ 24.37
- Put ≈ 0.004

---

*Repository: [yuhsunwang/financial_engineering](https://github.com/yuhsunwang/financial_engineering)*
