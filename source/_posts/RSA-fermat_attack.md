---
title: RSA-fermat_attack
date: 2024-10-30 0:13:6
top_img: https://s2.loli.net/2024/10/29/vuzlNgm4LexbsQR.jpg
mathjax: true
cover: https://s2.loli.net/2024/10/28/sZDounlLCOUewFS.png
tags:
  - Crypto
categories:
  - Crypto
  - RSA

---

# fermat_attack
**源码**
```python
from Crypto.Util.number import *
import gmpy2
flag = b'NSSCTF{******}'

p = getPrime(512)
q = gmpy2.next_prime(p - getPrime(256))
n = p*q
e = 65537
phi = (p-1)*(q-1)
m = bytes_to_long(flag)
c = pow(m, e, n)

print(f'n = {n}')
print(f'e = {e}')
print(f'c = {c}')
'''
n = 148841588941490812589697505975986386226158446072049530534135525236572105309550985274214825612079495930267744452266230141871521931612761645600600201983605957650711248808703757693378777706453580124982526368706977258199152469200838211055230241296139605912607613807871432800586045262879581100319519318390454452117
e = 65537
c = 69038543593219231496623016705860610154255535760819426453485115089535439537440188692852514795648297200067103841434646958466720891016026061658602312900242658759575613625726750416539176437174502082858413122020981274672260498423684555063381678387696096811975800995242962853092582362805345713900308205654744774932
'''
```
生成了一个512位的素数，随后用```next_prime()```函数获取$p-r$的下一个素数作为q,其中r为一个随机的256位的素数

由于512位的素数远大于256位的素数，我们可以认为p和q是两个比较接近的数
但求解$sn=\sqrt{n}$后， $q$不是$sn$的下一个素数

## <mark>**费马分解**</mark>

$\begin{gathered}
\frac{(p+q)^2}{4} - n = \frac{p^2 + 2pq + q^2}{4} - pq = \frac{(p-q)^2}{4}
\end{gathered}$

$\because p和q比较接近$  

$\therefore p - q 较小$  

$\therefore \frac{(p-q)^2}{4} 较小$ 

$\therefore \frac{(p+q)^2}{4} \approx n 且略大于 n$

+ 遍历大于 $\sqrt{n}$ 的每一个整数 $x$，若满足  
  $\begin{gathered}
  x^2 - n = y^2
  \end{gathered}$  
  其中  
  $\begin{gathered}
  x^2 = \frac{(p+q)^2}{4}, \quad y^2 = \frac{(p-q)^2}{4}
  \end{gathered}$

+ 解上述式子，即可得到 $p, q$



## exp
```python
from gmpy2 import *
from Crypto.Util.number import *

def fermat_attack(n):
    a = iroot(n, 2)[0]
    while 1:
        a += 1
        B2 = pow(a, 2) - n
        if is_square(B2):
            b = iroot(B2, 2)[0]
            p = a+b
            q = a-b
            assert n == p*q
            return p, q

n = 148841588941490812589697505975986386226158446072049530534135525236572105309550985274214825612079495930267744452266230141871521931612761645600600201983605957650711248808703757693378777706453580124982526368706977258199152469200838211055230241296139605912607613807871432800586045262879581100319519318390454452117
e = 65537
c = 69038543593219231496623016705860610154255535760819426453485115089535439537440188692852514795648297200067103841434646958466720891016026061658602312900242658759575613625726750416539176437174502082858413122020981274672260498423684555063381678387696096811975800995242962853092582362805345713900308205654744774932

p, q = fermat_attack(n)
phi = (p-1)*(q-1)
d = inverse(e, phi)
m = pow(c, d, n)
print(long_to_bytes(m))
        
# NSSCTF{fermat_factor}
```

### 珍贵手稿  
~~别管我的丑字~~
![](https://s2.loli.net/2024/10/30/c7CqrhWEJYB8DOA.jpg)
![](https://s2.loli.net/2024/10/30/zhp7C18PHekfbnj.jpg)


