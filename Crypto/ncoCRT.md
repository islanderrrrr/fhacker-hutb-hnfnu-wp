# 挑战简介 
一个非互质中国剩余定理的题

# 思路 
python解密函数中拥有能够解开中过剩余定理的题
```
#!/usr/bin/python3
from Crypto.Util.number import *
from sympy.ntheory.modular import crt

# 给定值
p = [1921232050179818686537976490035, 2050175089402111328155892746480, 1960810970476421389691930930824, 1797713136323968089432024221276, 2326915607951286191807212748022]
c = [1259284928311091851012441581576, 1501691203352712190922548476321, 1660842626322200346728249202857, 657314037433265072289232145909, 2056630082529583499248887436721]

try:
    # 使用 sympy 的 crt 函数
    m, mod = crt(p, c)
    print(f"恢复的整数: {m}")
    
    # 转换回字节并提取 flag
    flag_with_padding = long_to_bytes(m)
    print(f"恢复的字节: {flag_with_padding}")
    
    # 查找并提取 flag
    if b'SYC{' in flag_with_padding:
        start = flag_with_padding.find(b'SYC{')
        end = flag_with_padding.find(b'}', start)
        if end != -1:
            flag = flag_with_padding[start:end+1]
            print(f"恢复的 flag: {flag.decode()}")
        else:
            print("找不到 flag 结束位置")
    else:
        print("无法在恢复的数据中找到 flag 格式")
except Exception as e:
    print(f"使用 sympy.crt 时出错: {e}")
    print("可能的问题是模数不互质")

```
