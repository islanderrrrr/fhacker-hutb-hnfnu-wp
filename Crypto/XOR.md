# 挑战简介
一个异或题
```
from Crypto.Util.number import *
from pwn import xor

key = b'...'
flag = b'...'
assert len(key)==4

enc = bytes_to_long(xor(flag,key))

f1 = 4585958212176920650644941909171976689111990
f2 = 3062959364761961602614252587049328627114908
e1 = enc^f1
e2 = e1^f2
print(e2)

"""
10706859949950921239354880312196039515724907
"""
```
# 思路
这段代码的基本流程是：

1. 使用一个长度为4字节的密钥key对flag进行XOR操作
2. 将XOR结果转换为长整数，得到enc
3. 计算enc^f1得到e1
4. 计算e1^f2得到e2
5. 输出e2，值为10706859949950921239354880312196039515724907

所以解密步骤也是异或回去了  
```
#!/usr/bin/python3
from Crypto.Util.number import long_to_bytes
from pwn import xor

# 已知值
e2 = 10706859949950921239354880312196039515724907
f1 = 4585958212176920650644941909171976689111990
f2 = 3062959364761961602614252587049328627114908

# 从 e2 恢复 enc
enc = e2 ^ f2 ^ f1

# 将 enc 转换为字节串
enc_bytes = long_to_bytes(enc)

# 已知 flag 的前缀是 b'SYC{'
flag_prefix = b'SYC{'

# 恢复 key
key = xor(enc_bytes[:4], flag_prefix)

# 恢复完整的 flag
flag = xor(enc_bytes, key * (len(enc_bytes) // len(key) + 1))

print(flag.decode())
```
