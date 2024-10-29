---
title: ctfshow 爆破 web21-28
date: 2024-10-28 19:21:11
top_img: https://s2.loli.net/2024/10/29/WdqxoyKh8AYXf93.png
mathjax: true
cover: https://s2.loli.net/2024/10/29/ZYrUAPp4FcvNj1R.jpg
tags:
  - Web
categories:
  - Web
  - ctfshow

---

# web21
打开发现要登录用户名和密码,随机输入admin  111
![](https://s2.loli.net/2024/10/29/IVQg96Jk3eArRH5.png)

Burp抓包,奇怪的是没有看到username 和 password参数,但是发现有一个请求头Authorization
![](https://s2.loli.net/2024/10/29/1iBnTQwKamlJ6d9.png)
> Authorization
> 用于提供服务器验证用户代理身份的凭据，允许访问受保护的资源。该标头表示哪些身份验证的方案可用于访问资源（以及客户端使用它们时需要的额外的信息）。

简要概括，就是检查输入的身份能不能访问接下来的内容
将Basic后面的那串字符发送到decoder, 用Base64解码刚好是我们输入的内容
![](https://s2.loli.net/2024/10/29/BsPX4RvSl81iapQ.png)

这下就好办了，直接对这串内容进行爆破
值得注意的是,解码后的格式是admin:111, 对应的是 -> admin:password
所以爆破的时候要用 自定义迭代器Custom iterator 类型的payload

选择这串字符，发送到爆破模块
![](https://s2.loli.net/2024/10/29/uljAvyBsMDHTVEo.png)

选中,添加payload位置
![](https://s2.loli.net/2024/10/29/W7pZiNCmUJBguaK.png)

选择自定义迭代器, 在位置1输入admin,点击添加
![](https://s2.loli.net/2024/10/29/n7omTHAu5JkzcVW.png)

**在位置2输入 : 点击添加** (因为上文推断出的解码后的格式)
![](https://s2.loli.net/2024/10/29/bR7O8ci13xqtrVS.png)

位置3是需要爆破的部分,导入字典
![](https://s2.loli.net/2024/10/29/1PYtk9CQEFIwS7m.png)

下面的payload处理,由上面可知需要进行base64编码,点“添加”选择Base64-encode
![](https://s2.loli.net/2024/10/29/8yQRSDZkxFuNfKL.png)

由于是header里的,没有url编码,所以把payload编码的√取消
![](https://s2.loli.net/2024/10/29/TNxBGHvWsOdqnYU.png)

所有设置完成, 点击右上角的开始攻击，结果太多一个个看得虾:( 直接在过滤中选择成功的
![](https://s2.loli.net/2024/10/29/Az81QhDogEYZCNK.png)

爆破完毕，响应包中直接就有flag啦~
![](https://s2.loli.net/2024/10/29/1zE5Iaq78uChZsB.png)

# web22
域名已经更换了，所以写不了。但就是正常的dns爆破

# web23
源码：
```php
<?php
error_reporting(0);

include('flag.php');
if(isset($_GET['token'])){
    $token = md5($_GET['token']);
    if(substr($token, 1,1)===substr($token, 14,1) && substr($token, 14,1) ===substr($token, 17,1)){
        if((intval(substr($token, 1,1))+intval(substr($token, 14,1))+substr($token, 17,1))/substr($token, 1,1)===intval(substr($token, 31,1))){
            echo $flag;
        }
    }
}else{
    highlight_file(__FILE__);

}
?>
```
> **substr(1，2，3)函数**
类似于切片，
1 -> 要处理的字符串
2 -> 开始处，如果是负就表示从最后开始，默认为0
3 -> 规定要返回的字符串的长度，默认到结尾

需要token的md5值的第1位=第14位=第17位
且md5值的(第1位转成整数+第14位转成整数+第17位)/第1位 = 第31位转成整数

直接爆破
**exp1:**  python数字+字母
```python
import hashlib 

a = '1234567890qwertyuiopasdffghjklzxcvbnm'
for i in a:
    for j in a:
        b = (str(i) + str(j)).encode()
        token = hashlib.md5(b).hexdigest() #hexdigest() 的作用是将 MD5 哈希结果转换成可读的十六进制字符串
        if token[1] == token[14] and token[14] == token[17]:
            if (int(token[1])+int(token[14])+int(token[17]))/int(token[1]) == int(token[31]):
                print(b)
# b'3j'
```
**exp2:**   python纯数字
```python
import hashlib

for i in range(1000):
    token = hashlib.md5(str(i).encode('utf-8')).hexdigest()
    # 检查 token[1]、token[14] 和 token[17] 是否为数字
    if token[1].isdigit() and token[14].isdigit() and token[17].isdigit():
        # 判断 token[1]、token[14] 和 token[17] 是否相等
        if token[1] == token[14] and token[14] == token[17]:
            # 检查 token[31] 是否为数字，然后再进行运算
            if token[31].isdigit() and (int(token[1]) + int(token[14]) + int(token[17])) / int(token[1]) == int(token[31]):
                print(i)
# 422
```
**exp3:**    php直接爆破
```php
<?php 
error_reporting(0); 
 
$a="asdfghjklqwertyuiopzxcvbnm1234567890";
for($i=0;$i<36;$i++){
    for($j=0;$j<36;$j++){
        $token=$a[$i].$a[$j];    
        $token = md5($token); 
        if(substr($token, 1,1)===substr($token, 14,1) && substr($token, 14,1) ===substr($token, 17,1)){ 
            if((intval(substr($token, 1,1))+intval(substr($token, 14,1))+substr($token, 17,1))/substr($token, 1,1)===intval(substr($token, 31,1))){ 
                echo $a[$i].$a[$j];
                exit(0);
            } 
        } 
    }
} 
?> 
```
![](https://s2.loli.net/2024/10/29/ot8VGAnPLEO6dHD.png)

传入token的值，3j或422 得到flag
*在 PHP 中不区分字符串和字节字符串的概念*

# web 24
源码：
```php
<?php
error_reporting(0);
include("flag.php");
if(isset($_GET['r'])){
    $r = $_GET['r'];
    mt_srand(372619038);
    if(intval($r)===intval(mt_rand())){
        echo $flag;
    }
}else{
    highlight_file(__FILE__);
    echo system('cat /proc/version');
}
```
**随机数生成mt_rand()和播种mt_srand()**
> **mt_srand()**
播下一个更好的随机数种子，当有了随机数种子的时候，那么每次运行得到的随机数也是固定的
比如种子 `372619038` 得到固定随机数为 `1155388967`

> **mt_rand()**
生成随机数，如果前面播了随机数种子，那么my_rand()的值固定，否则随机
由源码mt_srand(372619038); -> 已播种，则后面mt_rand()生成的随机数是固定的，本地测试一下固定的随机数是什么


```php
<?php
mt_srand(372619038);
echo mt_rand();
```
![](https://s2.loli.net/2024/10/29/N2FfKLviTXSPbad.png)
直接传入r=1155388967,得到flag

# web25
源码：
```php
<?php
error_reporting(0);
include("flag.php");
if(isset($_GET['r'])){
    $r = $_GET['r'];   # get变量r
    mt_srand(hexdec(substr(md5($flag), 0,8)));  
    # 将flag的前8个字母作为随机种子
    $rand = intval($r)-intval(mt_rand());
    if((!$rand)){  # rand=0才会继续执行
        if($_COOKIE['token']==(mt_rand()+mt_rand())){ # 叫token的cookie=两个随机数相加
            echo $flag;
        }
    }else{
        echo $rand;
    }
}else{
    highlight_file(__FILE__);
    echo system('cat /proc/version');
}
```
> **hexdec()**
> 将十六进制转为十进制

**hint**
+ 种子固定，所以每次生成的随机数相同
+ r=0时，rand = -mt_rand(), 由输出的rand可以得到mt_rand(), 由php_mt_seed4.0逆序推出种子
+ 获取该随机种子的第二个、第三个随机数进行相加得到token

**exp**:
1. get传入r=0, 得到固定的随机数mt_rand()
   ![](https://s2.loli.net/2024/10/29/Exw7dyFOJIUTBgL.png)
   mt_rand = 1052183770

2. kali中使用php_mt_seed4.0
```
./php_mt_seed  1052183770(就是生成的随机数)
```
查看php版本
![](https://s2.loli.net/2024/10/29/QadP4pRZwXhib8t.png)
PHP/7.3.11
所以对应的种子也应该是PHP/7.3.11
![](https://s2.loli.net/2024/10/29/aSAP3gm7OYGBHRN.png)

3. 分别计算3个种子生成的随机数，然后计算叫token的cookie的值，带入进去，看能不能出flag
```php
<?php
mt_srand(1389196252);  #1623441608  2475778407
echo mt_rand()."\n";  # 要带入r的，让$rand=0
echo mt_rand()+mt_rand();  #token的值
```
![](https://s2.loli.net/2024/10/29/eVIkzCw6xrbJOln.png)
> 为什么每一次的mt_rand()+mt_rand()不是第一次的随机数相加？？ 因为生成的随机数可以说是一个线性变换（实际上非常复杂）的每一次的确定的但是每一次是不一样的，所以不能 进行第一次*2就得到mt_rand()+mt_rand()

# web26
题目打不开.....

# web27
打开题目，是一个网站的登录系统
![](https://s2.loli.net/2024/10/29/1dY395Hj4ZJ6X8c.png)

分别查看录取名单和查询系统
![](https://s2.loli.net/2024/10/29/1XegD7IvOSjUb92.png)

查询系统需要输入姓名和身份证号，录取名单中的身份证号码缺失出生的年月日，挑个喜欢的输入，缺失部分随机填，直接爆破
![](https://s2.loli.net/2024/10/29/patfZWR52F8T9oX.png)

由于姓名是确定的，只需要爆破身份证号码
![](https://s2.loli.net/2024/10/29/falFXcb9mRstIy6.png)

缺失部分是年月日，所有这里采用日期payload, 时间跨度适中
![](https://s2.loli.net/2024/10/29/7TVfCGMWsOoUDpJ.png)
**注意格式为20060807这种，设置为yyyyMMdd**

爆破出好多200的结果，一个个看又要虾，
> **tips**
将长度从大到小排
![](https://s2.loli.net/2024/10/30/Oi82PTVg9K46qHF.png)

输入正确的身份证号码，弹窗给出学号和密码
![](https://s2.loli.net/2024/10/30/dgXikrztfLCQ5Mb.png)

回到登录页面，输入信息，得到flag
![](https://s2.loli.net/2024/10/30/OzWUjY1ipxZr5Ao.png)

# web28
![](https://s2.loli.net/2024/10/30/3fndFiJMzPl4GEV.png)
目录里有0和1，尝试对它们进行爆破，爆破的时候需要将2.txt给删了
![](https://s2.loli.net/2024/10/30/hwNCmMz1oLPbR38.png)

> **集束炸弹**
>+ 多个攻击点：与其他攻击类型不同，集束炸弹允许在单个请求中设置多个攻击点。
>+ 有效载荷集的组合：为每个攻击点定义一个有效载荷集。Intruder 会组合这些有效载荷，以生成不同的请求。
>+ 遍历所有组合：Intruder 遍历每个有效载荷集中的每个项，为所有可能的组合创建请求。

payload类型为数值，爆破0-100
![](https://s2.loli.net/2024/10/30/rf1YZtzH2lkxjEG.png)

![](https://s2.loli.net/2024/10/30/84oyHYNz75TVljv.png)

在响应里找到flag
![](https://s2.loli.net/2024/10/30/7kr5HlFRQMpnfby.png)