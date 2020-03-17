# 財務工程第一次作業
  

Week2 
=======
(一)Day count 計算方式
======
    1. Actual / Actual <- 實際天數
    2. 30/360 <- 每月都是30天
======
(二)Bond 計價
======
    1. Omega = 下次付款日/一期時間
    2. AI = C * (1-Omega)
    3. Clean Price + AI = sum(C之折現) + F折現
       C折現之分母為(1+r/m)^(w+i)
       F折現之分母為(1+r/m)^(w+n-1)
======
(三)Duration 計算
    1.MD = (1/P)*[sum((i*C/((1+y)^i)))+(nF/((1+y)^n))]
======    
