# 挑战简介
基础的RSA解密
```
from Crypto.Util.number import bytes_to_long, getPrime
from secret import flag
p = getPrime(128)
q = getPrime(128)
n = p*q
e = 65537
m = bytes_to_long(flag)
c = pow(m, e, n)
print(f"n = {n}")
print(f"p = {p}")
print(f"q = {q}")
print(f"c = {c}")

'''
n = 33108009203593648507706487693709965711774665216872550007309537128959455938833
p = 192173332221883349384646293941837353967
q = 172282016556631997385463935089230918399
c = 5366332878961364744687912786162467698377615956518615197391990327680664213847
'''
```

# 步骤
对于RSA解密顺序，有如下规则
```
生成两个128位的质数p和q
计算n = p*q
设置公钥指数e = 65537（这是RSA中常用的值）
将flag（秘密消息）转换为整数m
使用RSA加密公式 c = m^e mod n 计算密文c
```
1. 计算欧拉函数φ(n)：

2. φ(n) = (p-1)(q-1)
计算私钥d：

3. d = e^(-1) mod φ(n)，即e的模反元素
使用扩展欧几里得算法计算：d ≡ e^(-1) (mod φ(n))
解密密文：

4. 使用私钥d解密：m = c^d mod n
将解密后的整数m转换回字节，得到flag

解密脚本
```
#!/usr/bin/python3
from Crypto.Util.number import long_to_bytes

# 已知的值
p = 192173332221883349384646293941837353967
q = 172282016556631997385463935089230918399
e = 65537
c = 5366332878961364744687912786162467698377615956518615197391990327680664213847

# 计算私钥
n = p * q
phi = (p - 1) * (q - 1)

# 计算私钥d (使用扩展欧几里得算法)
def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, x, y = egcd(b % a, a)
        return (g, y - (b // a) * x, x)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('模逆不存在')
    else:
        return x % m

d = modinv(e, phi)

# 解密
m = pow(c, d, n)
flag = long_to_bytes(m)

print(f"解密后的flag: {flag}")
```
