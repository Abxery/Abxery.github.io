---
title: 连分数展开
date: 2024-11-12 01:28:14
top_img: https://s2.loli.net/2024/10/31/OcBR861ui3q9hNa.jpg
cover: https://s2.loli.net/2024/10/31/zXfKBoZFvxnCTe7.jpg
tags:
  - Crypto
categories:
  - Crypto
  - 姿势

---

# 连分数展开
## 连分数
![](https://s2.loli.net/2024/11/12/hQ9ixRVjXa1qInH.png)

## 逼近分数
+ 初始条件
  我们从分数 $\frac{e}{n}$ 开始。连分数展开的目标是把它写成如下的形式：
  ![](https://s2.loli.net/2024/11/12/lQptCnI6oMfBLH2.png)
其中$a_0,a_1,a_2....$是一系列的整数

+ 计算逼近分数
> 固定值：
$p_{-1} = 1 ,  p_0 = 0  , q_{-1} = 0   , q_0 = 1$
> 
> 计算公式：  
$$
p_n = a_n \cdot p_{n-1} + p_{n-2} \\
q_n = a_n \cdot q_{n-1} + q_{n-2}
$$

## exp
```python
class ContinuedFraction():
    def __init__(self, numerator, denominator):
        self.numberlist = [] # 保存连分数展开的系数
        self.fractionlist = [] # 保存逼近分数
        self.GenerateNumberList(numerator, denominator) # 生成连分数展开的系数
        self.GenerateFractionList() # 生成逼近分数
        
    def GenerateNumberList(self, numerator, denominator):
        while denominator != 0:
            quotient = numerator // denominator  # 商
            self.numberlist.append(quotient)  # 将商，也就是连分数展开的系数添加到系数列表
            numerator, denominator = denominator, numerator % denominator  # 产生新的将要进行连分数分解的分子分母
            
    def GenerateFractionList(self):
        p_minus_1, p_0 = 1, 0  # 初始p_-1 = 1, p_0 = a_0
        q_minus_1, q_0 = 0, 1  # 初始q_-1 = 0, q_0 = 1
        self.fractionlist.append([p_0, q_0])  # 将第0个逼近分数，也就是a_0,添加到列表
        
        for a_n in self.numberlist[1:]:  # 计算第1个及之后的逼近分数
            p_n = a_n * p_0 + p_minus_1
            q_n = a_n * q_0 + q_minus_1
            
            self.fractionlist.append([p_n, q_n])  # 将当前逼近分数添加到列表
            
            p_0, p_minus_1 = p_n, p_0  # 更新p 
            q_0, q_minus_1 = q_n, q_0  # 更新q


a = ContinuedFraction(e, n)  # 生成所有逼近分数
```