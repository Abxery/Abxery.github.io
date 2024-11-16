---
title: BaseCTF2024_crypto wp
date: 2024-11-16 09:19:55
top_img: https://s2.loli.net/2024/10/31/ez6OPckFvxHqlZ2.jpg
cover: https://s2.loli.net/2024/10/31/dKX3xuQVOkRSIja.jpg
tags:
  - Crypto
categories:
  - wp

---
# 你会算md5吗
## 分析
将flag逐字md5加密，得到output
单字符md5，可以理解为单字节加密，没有行列转换混淆的话，一般都可以直接爆破，得到每个原字符
（如果答案不对一般就是字符集的问题，整数32-126就是可见字符范围）

## exp
```python
import hashlib

output = ['9d5ed678fe57bcca610140957afab571', '0cc175b9c0f1b6a831c399e269772661', '03c7c0ace395d80182db07ae2c30f034', 'e1671797c52e15f763380b45e841ec32', '0d61f8370cad1d412f80b84d143e1257', 'b9ece18c950afbfa6b0fdbfa4ff731d3', '800618943025315f869e4e1f09471012', 'f95b70fdc3088560732a5ac135644506', '0cc175b9c0f1b6a831c399e269772661', 'a87ff679a2f3e71d9181a67b7542122c', '92eb5ffee6ae2fec3ad71c777531578f', '8fa14cdd754f91cc6554c9e71929cce7', 'a87ff679a2f3e71d9181a67b7542122c', 'eccbc87e4b5ce2fe28308fd9f2a7baf3', '0cc175b9c0f1b6a831c399e269772661', 'e4da3b7fbbce2345d7772b0674a318d5', '336d5ebc5436534e61d16e63ddfca327', 'eccbc87e4b5ce2fe28308fd9f2a7baf3', '8fa14cdd754f91cc6554c9e71929cce7', '8fa14cdd754f91cc6554c9e71929cce7', '45c48cce2e2d7fbdea1afc51c7c6ad26', '336d5ebc5436534e61d16e63ddfca327', 'a87ff679a2f3e71d9181a67b7542122c', '8f14e45fceea167a5a36dedd4bea2543', '1679091c5a880faf6fb5e6087eb1b2dc', 'a87ff679a2f3e71d9181a67b7542122c', '336d5ebc5436534e61d16e63ddfca327', '92eb5ffee6ae2fec3ad71c777531578f', '8277e0910d750195b448797616e091ad', '0cc175b9c0f1b6a831c399e269772661', 'c81e728d9d4c2f636f067f89cc14862c', '336d5ebc5436534e61d16e63ddfca327', '0cc175b9c0f1b6a831c399e269772661', '8fa14cdd754f91cc6554c9e71929cce7', 'c9f0f895fb98ab9159f51fd0297e236d', 'e1671797c52e15f763380b45e841ec32', 'e1671797c52e15f763380b45e841ec32', 'a87ff679a2f3e71d9181a67b7542122c', '8277e0910d750195b448797616e091ad', '92eb5ffee6ae2fec3ad71c777531578f', '45c48cce2e2d7fbdea1afc51c7c6ad26', '0cc175b9c0f1b6a831c399e269772661', 'c9f0f895fb98ab9159f51fd0297e236d', '0cc175b9c0f1b6a831c399e269772661', 'cbb184dd8e05c9709e5dcaedaa0495cf']

flag = []
for i in output:
    for j in range(32, 127):
        my_md5 = hashlib.md5()  # 创建一个md5对象
        my_md5.update(chr(j).encode())  
        if my_md5.hexdigest() == i:
            flag.append(chr(j))
            break
        
print(f"flag: {''.join(flag)}")
# BaseCTF{a4bf43a5-3ff9-4764-bda2-af8ee4db9a8a}
``` 

  

# 十七倍
源码：

``` C
#include <stdio.h>

int main() {
    unsigned char flag[] = "BaseCTF{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}";
    
    /**
     * 由 (unsigned) char 决定，每个元素在内存中占 1 字节，即 8 位（8 个 0 或 1）
     * 在内存中，“字符”保存的是其在“字母表”中是第几个字符
     * 例如：
     * B 即  66 在内存中存的是 01000010
     * a 即  97 在内存中存的是 01100001
     * s 即 115 在内存中存的是 01110011
     * e 即 101 在内存中存的是 01100101
     */

    int i;
    for (i = 0; i < 40; i++) {
        flag[i] = flag[i] * 17;
    }
    if (flag[0] != 98) {  /* 下标是从 0 开始的 */
        printf("CPU Error???\n");
        return 1;
    }

    /**
     * 66 * 17 = 1122
     * 如果在内存中保存 1122，会是 00000100 01100010
     * 但是 unsigned char 决定了只能存 8 位，CPU 硬件会自动取低 8 位，即 01100010
     * 01100010 即 98，所以 66 * 17 = 98
     * 
     * 注意到 8 个 0 或 1 有 256 种可能，即 0~255
     * 且取低 8 位即取模（取余数）256
     * 你可以验证：1122 除以 256 商为 4 余数为 98
     */

    unsigned char cipher[] = {
         98, 113, 163, 181, 115, 148, 166,  43,   9,  95,
        165, 146,  79, 115, 146, 233, 112, 180,  48,  79,
         65, 181, 113, 146,  46, 249,  78, 183,  79, 133,
        180, 113, 146, 148, 163,  79,  78,  48, 231,  77
    };
    for (i = 0; i < 40; i++) {
        if (flag[i] != cipher[i]) {
            printf("flag[%d] is wrong, expect %d, got %d.\n", i, cipher[i], flag[i]);
            return 1;
        }
    }

    /**
     * 如果 flag 是正确的，运算后会得到上面的数据。
     * 如果是实数域运算，flag[i] * 17 = cipher[i]，那么 flag[i] = cipher[i] / 17
     * 模了 256 后又是怎么样呢？学一下“模运算乘法逆元”吧。
     */

    return 0;
}
```
## 分析
flag每个字符转换成ASCII码后，unsigned char 决定了对应的二进制只保留低8位， 2^8， 相当于模256
$$
cipher[i] = flag[i] * 17 \mod 256
$$
模运算中，有公式
$$
a \cdot b \mod c =  ((a \mod c) * (b \mod c)) \mod c
$$
方便起见， 令 y = cipher[i] ， x = flag[i]
那么
$$
x * 17 \mod 256 = ((x \mod 256) * (17 \mod 256)) \mod 256
$$
要得到x， 就要使后面的 $17 \mod 256 = 1$
设$17 * a \equiv 1 \mod 256$
a 就是 17 的逆元
```a = inverse(17, 256) ```  
得到a=241

等式两边同时乘241
$$
y * 241 = (x * 17 * 241) \mod 256 = ((x \mod 256) * (17*241 \mod 256)) \mod 256 = x \mod 256
$$
可以得到 x 就是 y * 241 模256 的结果
$$
x \equiv y*241 \mod 256
$$
遍历cipher,还原flag

## exp
```python
from Crypto.Util.number import *

cipher = [
         98, 113, 163, 181, 115, 148, 166,  43,   9,  95,
        165, 146,  79, 115, 146, 233, 112, 180,  48,  79,
         65, 181, 113, 146,  46, 249,  78, 183,  79, 133,
        180, 113, 146, 148, 163,  79,  78,  48, 231,  77
]

ascii_flag = [] # 存放解密后的flag[i]的值，是ASCII码

for i in cipher:
    x = i*241%256  # 求出flag[i]
    ascii_flag.append(x)  
    
print(''.join(chr(x) for x in ascii_flag), end = '')  # 将 ASCII 值转换为对应字符

# BaseCTF{yoUr_CrYpt0_1earNinG_5tarTs_n0w}
```

# mid_math