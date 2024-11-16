---
title: XOR
date: 2024-11-3 00:32:36
top_img: https://s2.loli.net/2024/10/31/nVLPkAGRsEHjvW1.jpg
cover: https://s2.loli.net/2024/10/31/sQoB5rIj98AJMxh.jpg
tags:
  - Crypto
categories:
  - Crypto
  - 定理

---
## XOR Starter
XOR 是按位运算符，如果位相同则返回 0，否则返回 1。在教科书中，XOR 运算符用 ⊕ 表示，但在大多数挑战和编程语言中，您会看到使用插入符号^代替。
![](https://s2.loli.net/2024/11/03/D2o1saRVJSkZ3jn.png)

对于更长的二进制数，我们逐位进行异或： 0110 ^ 1010 = 1100 。我们可以通过首先将整数从十进制转换为二进制来对整数进行异或。对于字符形式，首先将每个字符转换为表示 Unicode 字符的整数来对字符串进行异或。

1. ==**^运算符**== 是位运算符，通常用于按位异或（XOR） 操作。它要求操作数是整数类型或可以转换为整数类型的数据。

+ 数据类型要求
  **整数类型（int）**：标准用法是在两个整数之间进行按位异或。例如：
  a = 5  # 二进制：0101
  b = 3  # 二进制：0011
  result = a ^ b  # 结果为 6（二进制：0110）
  **布尔类型（bool）**：在 Python 中，布尔值可以用 ^ 进行异或操作，因为布尔值 True 和 False 实际上是整数 1 和 0 的别名。例如：
  result = True ^ False  # 结果为 True
  **^ 运算符不支持字符串、浮点数、字节等非整数类型**。如果对非整数类型使用 ^ 运算符，会抛出 TypeError。

+ 如果你需要对其他类型（如字符串或字节）进行 XOR 操作，需要将它们转换为整数或使用迭代的方式，例如对字节序列进行异或运算：
```
byte1 = b'a'[0]  # 转换为整数：97
byte2 = b'b'[0]  # 转换为整数：98
result = byte1 ^ byte2  # 结果为 3
```
对于较长的字节序列，可以使用列表推导或 bytearray 来逐字节异或。

2. ==**pwntools 的 xor() 函数**== 用于对数据进行按位异或运算。它常用于二进制数据操作。

+ **xor() 函数的参数**
  data（必需）：需要进行异或的输入数据。可以是 bytes、bytearray 或 str 类型。
  char（可选）：异或运算的单个字节或字节序列。可以是 int、bytes、bytearray 或 str。如果是单个字符（字节），则会对 data 的每个字节进行异或。

+ **返回值**
  返回一个 bytes 类型的对象，其中每个字节都是 data 和 char 逐字节异或的结果。
+ **用法示例**
```python
from pwn import xor

# 将 'hello' 与 0x42 进行逐字节异或
result = xor('hello', 0x42)
print(result)  #输出为 b'*\x0e\x0e\x0e\x0d'

# 将 b'hello' 与 b'key' 进行异或
result = xor(b'hello', b'key')
print(result)  # 输出异或后的字节序列

# 如果 'char' 比 'data' 短，且 cut=False，则 'char' 会重复使用
result = xor(b'hello world', b'key')
print(result)  # b'\x03\x0f\x02\x0e\x0c\x0b\x0e\x0e\x02\x01\x0d'
```
+ **细节说明**
  xor() 会自动处理输入类型，如果输入是字符串 (str)，会自动转换为 bytes。
  如果 char 是单个字节，则会对 data 的每个字节执行相同的异或。
  如果 char 是字节序列，则逐字节与 data 中的字节进行异或，支持循环重复。
+ **返回值类型**
  返回值是 bytes，需要根据具体情况解码或进一步处理，如转换为 str 等。

## XOR Properties
![](https://s2.loli.net/2024/11/03/nliDHbYy6vca2jw.png)

让我们来分解一下。可交换意味着 XOR 运算的顺序并不重要。关联意味着一系列操作可以无序地进行（我们不需要担心括号）。身份是 0，因此与 0 进行异或“什么也不做”，最后与自身进行异或的结果返回零。

### challenge1:   Properties
```
KEY1 = a6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313
KEY2 ^ KEY1 = 37dcb292030faa90d07eec17e3b1c6d8daf94c35d4c9191a5e1e
KEY2 ^ KEY3 = c1545756687e7573db23aa1c3452a098b71a7fbf0fddddde5fc1
FLAG ^ KEY1 ^ KEY3 ^ KEY2 = 04ee9855208a2cd59091d04767ae47963170d1660df7f56f5faf
```

注意到都是十六进制，要先转换为字节，使用```bytes.fromhex()```
*记得修改异或运算后的结果名称*

exp:
```python
from pwn import xor
from Crypto.Util.number import *


KEY1 = "a6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313"
KEY2_KEY1 = "37dcb292030faa90d07eec17e3b1c6d8daf94c35d4c9191a5e1e"
KEY2_KEY3 = "c1545756687e7573db23aa1c3452a098b71a7fbf0fddddde5fc1"
FLAG_KEY1_KEY3_KEY2 = "04ee9855208a2cd59091d04767ae47963170d1660df7f56f5faf"

key1 = bytes.fromhex(KEY1)
key2 = xor(key1, bytes.fromhex(KEY2_KEY1))
key3 = xor(key2, bytes.fromhex(KEY2_KEY3))

key1_2_3 = xor(bytes.fromhex(KEY2_KEY1), key3)
flag = xor(bytes.fromhex(FLAG_KEY1_KEY3_KEY2), key1_2_3)
print(flag)

#b'crypto{x0r_i5_ass0c1at1v3}'
```

### challenge2:  Favourite byte
我使用 XOR 与单个字节隐藏了一些数据，但该字节是一个秘密。不要忘记首先从十六进制解码。
```
73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d
```

既然是单字节，8比特$\rightarrow $范围是0-255， 直接爆破
```python
from pwn import xor
from Crypto.Util.number import *

a = bytes.fromhex("73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d")
for i in range(256):
    result = xor(a, i).decode()
    if "crypto" in result:
        print(result)
        break
  
# crypto{0x10_15_my_f4v0ur173_by7e}
```
要注意xor()参数的顺序，前面是被异或的对象，可以是 bytes、bytearray 或 str 类型。后面是去给对象异或的值，可以是 int、bytes、bytearray 或 str。所以这里不需要再把int型的i转换为字节


### challenge3:   You either know, XOR you don't
我已经用我的密钥加密了该标志，你永远无法猜到它。
```
0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104
```
secret_key ^ flag = message
已知message和flag的前7位“crypto{”，   则可以用message的前7为和flag的前7位异或得到secret_key的前7位
```python
from pwn import xor
from Crypto.Util.number import *

message = bytes.fromhex("0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104")
partical_secret_key = xor(message[:7], "crypto{")
print(partical_secret_key)

# b'myXORke'
```

由partical_secret_key的结果myXORke推测，secret_key可能是"myXORkey", 此时char为字节序列，需要与data循环异或，来保证char和data长度一致

```python
from pwn import xor
from Crypto.Util.number import *

message = bytes.fromhex("0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104")
secret_key = "myXORkey"
flag = xor(message, secret_key.encode())
print(flag)

# b'crypto{1f_y0u_Kn0w_En0uGH_y0u_Kn0w_1t_4ll}'
```
