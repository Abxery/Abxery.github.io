---
title: RSA-Wiener_attack
date: 2024-11-2 21:40:18
top_img: https://s2.loli.net/2024/10/31/4wesp5mXlYvoNyG.jpg
cover: https://s2.loli.net/2024/10/31/kCoIQ1d9bNfaKy8.jpg
tags:
  - Crypto
categories:
  - Crypto
  - RSA

---
# RSA-Wiener_attack
## 特点：先选定d，再根据d求e, e大d小
源码:
```python
from Crypto.Util.number import *
from gmpy2 import *

flag = b'NSSCTF{******}'

p = getPrime(256)
q = getPrime(256)

n = p*q
d = getPrime(128)
e = inverse(d, (p-1)*(q-1))
m = bytes_to_long(flag)

c = powmod(m, e, n)

print(f'n = {n}')
print(f'e = {e}')
print(f'c = {c}')

'''
n = 6969872410035233098344189258766624225446081814953480897731644163180991292913719910322241873463164232700368119465476508174863062276659958418657253738005689
e = 3331016607237504021038095412236348385663413736904453330557803644384818257225138777641344877202234881627514102078530507171735156112302207979925588113589669
c = 1754994938947260364311041300467524420957926989584983693004487724099773647229373820465164193428679197813476633649362998772470084452129370353136199193923837
'''
```

## 分析
对于e和d，有
$$
e \cdot d \equiv 1 \mod {\phi(n)}
$$
则
$$
e\cdot d = k\cdot \phi(n) + 1
$$
两边同时除以$d\cdot \phi(n)$得
$$
\frac{e}{\phi(n)} = \frac{k}{d} + \frac{1}{d\cdot \phi(n)}
$$
$\because d 和 \phi(n)$都非常大
$\therefore \frac{1}{d\cdot \phi(n)} \rightarrow 0$
$\because \phi(n) = (p-1)(q-1) = n - (p+q) + 1 $
$\because p, q$都是256为素数
$\therefore n >> p+q$
$\therefore \phi(n) \approx n$
上式变为：
$$
\frac{e}{n} \approx \frac{k}{d}
$$
对$\frac{e}{n}$进行[连分数展开](https://abxery.cn/2024/11/11/lian/)，得到的每一个逼近分数就是$\frac{k}{d}$
带入逼近分数中的分母，也就是d，直接作为私钥解密c，看解出的m中有没有flag头

> 至此我们需要知道的是，当发现 d 比较小时可以使用wiener攻击，但有些时候我们并不知道 d 的大小，不过我们能够知道此时 e 会很大 (因为ed - 1 = k$\cdot \phi (n)$,  一个小另一个就会大) 。所有当我们发现 e 很大或者说接近于 n 时，便可以考虑使用连分数展开的方式， 遍历每一个逼近分数， 测试是否是解题中需要用到的关键因子


## exp
```python
from Crypto.Util.number import *
from gmpy2 import *

n = 6969872410035233098344189258766624225446081814953480897731644163180991292913719910322241873463164232700368119465476508174863062276659958418657253738005689
e = 3331016607237504021038095412236348385663413736904453330557803644384818257225138777641344877202234881627514102078530507171735156112302207979925588113589669
c = 1754994938947260364311041300467524420957926989584983693004487724099773647229373820465164193428679197813476633649362998772470084452129370353136199193923837

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
        p_minus_1, p_0 = 1, self.numberlist[0]  # 初始p_-1 = 1, p_0 = a_0
        q_minus_1, q_0 = 0, 1  # 初始q_-1 = 0, q_0 = 1
        self.fractionlist.append([p_0, q_0])  # 将第0个逼近分数，也就是a_0,添加到列表
        
        for a_n in self.numberlist[1:]:  # 计算第1个及之后的逼近分数
            p_n = a_n * p_0 + p_minus_1
            q_n = a_n * q_0 + q_minus_1
            
            self.fractionlist.append([p_n, q_n])  # 将当前逼近分数添加到列表
            
            p_0, p_minus_1 = p_n, p_0  # 更新p 
            q_0, q_minus_1 = q_n, q_0  # 更新q


a = ContinuedFraction(e, n)  # 生成所有逼近分数
            
for k, d in a.fractionlist:  # 遍历每一张逼近分数对应的k,d
    m = powmod(c, d, n)
    flag = long_to_bytes(m)
    
    if b'NSSCTF' in flag:
        print(flag)
        break
    
# b'NSSCTF{e_is_so_huge}' 
```
