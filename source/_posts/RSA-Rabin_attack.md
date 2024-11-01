---
title: RSA-Rabin_attack
date: 2024-11-1 09:36:16
top_img: https://s2.loli.net/2024/10/31/ZGFeBz8jOTUHrN1.jpg
mathjax: true
cover: https://s2.loli.net/2024/10/29/Mqtih9YAUSmvXFV.jpg
tags:
  - Crypto
categories:
  - Crypto
  - RSA

---
# RSA-Rabin_attack

## 适用于
e=2, p,q已知且满足%4=3。此时我们可以通过rabin算法解出四个明文，再判断哪个正确

源码：
```python
from Crypto.Util.number import *
from gmpy2 import *

flag = b'NSSCTF{******}'

p = getPrime(256)
q = getPrime(256)

assert p%4 == 3 and q%4 == 3

n = p*q
e = 2
m = bytes_to_long(flag)

c = powmod(m, e, n)

print(f'p = {p}')
print(f'q = {q}')
print(f'e = {e}')
print(f'c = {c}')

'''
p = 67711062621608175960173275013534737889372437946924512522469843485353704013203
q = 91200252033239924238625443698357031288749612243099728355449192607988117291739
e = 2
c = 5251890478898826530186837207902117236305266861227697352434308106457554098811792713226801824100629792962861125855696719512180887415808454466978721678349614
'''
```

e非常小，但c不是完全平方数，因此不能用低加密指数攻击
同时$e和\phi也不互素$，因此求不出逆元
已知$p \equiv q \equiv 3 \pmod 4$, 且e=2，满足rabin算法

加密：
$$
c \equiv m^2 \pmod n
$$
解密：求解
$$
m^2 \equiv c \pmod n
$$
$\because p,q|n$
$\therefore m^2 = kn + c = kpq + c$
$$
\begin{cases}
m^2 \equiv c \pmod p\\
m^2 \equiv c \pmod q
\end{cases}
$$
$\therefore c是模p的二次剩余$
由[二次剩余+欧拉准则知](https://abxery.cn/2024/11/01/%E4%BA%8C%E6%AC%A1%E5%89%A9%E4%BD%99+%E6%AC%A7%E6%8B%89%E5%87%86%E5%88%99/)
$$
c^\frac{p-1}{2} \equiv 1 \pmod p
$$

$$
\therefore m^2 \equiv c \equiv c*c^\frac{p-1}{2} \equiv c^\frac{p+1}{2} \pmod p
$$
$\because m^2 = kp + c^\frac{p+1}{2}$
$\therefore m = k^\frac{1}{2} p+ c^\frac{p+1}{4}$
开方得：
$$
\begin{cases}
m_1 \equiv c^\frac{p+1}{4} \pmod p\\
m_2 \equiv p-c^\frac{p+1}{4} \pmod p
\end{cases}
$$
同理，c是模q的二次剩余
$$
\begin{cases}
m_3 \equiv c^\frac{q+1}{4} \pmod q\\
m_4 \equiv 1-c^\frac{q+1}{4} \pmod q
\end{cases}
$$

运用[中国剩余定理](https://abxery.cn/2024/11/01/%E4%B8%AD%E5%9B%BD%E5%89%A9%E4%BD%99%E5%AE%9A%E7%90%86/)，将模p和模q组合起来得到n，对应的x就是m
$$
\begin{cases}
m_1 + m_3\\
m_1 + m_4\\
m_2 + m_3\\
m_2 + m_4
\end{cases}
$$

## exp:
```python
from Crypto.Util.number import *
from gmpy2 import *

p = 67711062621608175960173275013534737889372437946924512522469843485353704013203
q = 91200252033239924238625443698357031288749612243099728355449192607988117291739
e = 2
c = 5251890478898826530186837207902117236305266861227697352434308106457554098811792713226801824100629792962861125855696719512180887415808454466978721678349614

def rabin_attack(c, n, p, q):
    c1 = pow(c, (p+1)//4, p)
    c2 = pow(c, (q+1)//4, q)
    cp1 = p - c1
    cp2 = q - c2
    
    t1 = inverse(q, p)
    t2 = inverse(p, q)
    
    m1 = (q*t1*c1 + p*t2*c2) % n
    m2 = (q*t1*c1 + p*t2*cp2) % n
    m3 = (q*t1*cp1 + p*t2*c2) % n
    m4 = (q*t1*cp1 + p*t2*cp2) % n
    
    return m1,m2,m3,m4

ms = rabin_attack(c, p*q, p, q)

for m in ms:
    print(long_to_bytes(m))
```
## 珍贵手稿
![](https://s2.loli.net/2024/11/01/4NSRoM3YvpbVyUH.jpg)