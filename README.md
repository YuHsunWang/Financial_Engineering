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
    C --> D{開始迴圈\nfor month in 0..12t}
    D --> E[計算利息\ninterest = p × r / 12]
    E --> F{最後一期？}
    F -- 是 --> G[payment = 剩餘本金 p\n結清]
    F -- 否 --> H[payment = ceil 本金 / 總期數]
    G --> I[更新 total、p\n寫入各 List]
    H --> I
    I --> D
    D -- 迴圈結束 --> J[將五個 List 轉成 DataFrame]
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
    B --> C[計算每期付息\nPMT = Par × Coupon / Payment]
    C --> D["利用等比級數公式建立方程式\nBond_Price = Σ PMT/(1+YTM)^t + Par/(1+YTM)^n"]
    D --> E["sympy solve() 求 YTM\n篩選正實數解"]
    E --> F([輸出 YTM])

    B --> G["Bootstrap 求 Spot Rate"]
    G --> H["期 1：(PMT + Par) / (1 + S₁) = Price\nsolve() 求 S₁"]
    H --> I["期 2：PMT/(1+S₁) + (PMT+Par)/(1+S₂)² = Price\nsolve() 求 S₂"]
    I --> J[依此類推至第 n 期]
    J --> K([輸出 Spot Rate 列表])

    K --> L["計算 Forward Rate\nf(0,1) = S(1)\nf(0,2) = S(2)\nf(1,2) = S(2)/S(1)  ..."]
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
    B --> C["計算 R 與風險中立機率 P\nR = e^r\nP = (R - d) / (u - d)"]
    C --> D["建立股價樹（正向）\nS[i][j] = S × u^(i-j) × d^j\n同時計算各節點機率（二項式定理）"]

    D --> E["計算最後一期損益\nCall[n][j] = max(S[n][j] - X, 0)\nPut[n][j]  = max(X - S[n][j], 0)"]
    E --> F["倒推法（Backward-induction）\nCall[i][j] = (P×C_u + (1-P)×C_d) / R\nPut[i][j]  = (P×P_u + (1-P)×P_d) / R"]
    F --> G["計算避險比率 Δ 與債券倉位 B\nΔ = (C_u - C_d) / (S_u - S_d)\nB = (u×P_d - d×P_u) / ((u-d)×R)"]
    G --> H(["Print 股價樹（含機率）\nPrint 買權表（含 Δ）\nPrint 賣權表（含 B）"])
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
    A([匯入套件\nscipy / pandas / numpy]) --> B["輸入參數\nS（股價）、K（履約價）\ndividend_list、dividend_month\nsigma（波動度）、r（利率）、due（到期月數）"]
    B --> C["計算股利現值 D\nD = Σ dᵢ × e^(−r × tᵢ)\n（以 zip 遍歷金額與月份）"]
    C --> D["調整股價\nŜ = S − D"]
    D --> E["計算 d₁ 與 d₂\nd₁ = [ln(Ŝ/K) + (r + σ²/2)×T] / (σ√T)\nd₂ = d₁ − σ√T"]
    E --> F["查常態分佈 CDF\nN(d₁), N(d₂), N(−d₁), N(−d₂)"]
    F --> G["代入 BS 公式\nCall = Ŝ×N(d₁) − K×e^(−rT)×N(d₂)\nPut  = K×e^(−rT)×N(−d₂) − Ŝ×N(−d₁)"]
    G --> H(["Print Call & Put 價格"])
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
    A([匯入套件\nQuantLib / numpy / matplotlib]) --> B["輸入參數\nT、dt、σ、S₀、K\na（mean-reversion）、forward_rate\ntimestep、num_paths"]
    B --> C["建立 Hull-White 模型\n設定 flat forward curve\n初始化 HullWhiteProcess"]
    C --> D["生成 num_paths 條短率路徑\n每條路徑長度 = timestep + 1"]
    D --> E["以短率路徑作為 μ\n代入 GBM 生成股價路徑\nS(t) = S₀ × exp((μ − σ²/2)t + σW(t))"]
    E --> F["計算每條路徑的到期損益\nCall payoff = max(S_T − K, 0)\nPut payoff  = max(K − S_T, 0)"]
    F --> G["取 payoff 期望值\nE[Call payoff]、E[Put payoff]"]
    G --> H["折現至現值\nPrice = E[payoff] / (1 + r/12)^(12T)"]
    H --> I(["Print Call Price & Put Price"])
```

**輸出：**
- Call ≈ 24.37
- Put ≈ 0.004

---

*Repository: [yuhsunwang/financial_engineering](https://github.com/yuhsunwang/financial_engineering)*
