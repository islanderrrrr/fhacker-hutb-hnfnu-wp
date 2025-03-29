# 挑战简介
椭圆曲线加密，flag前缀为SYC{xx}

# 思路
对于其加密过程，如下
1. 密钥生成：
选择椭圆曲线E和基点G  
选择随机数k作为私钥  
计算K = k × G作为公钥  

2. 加密：
将消息编码为曲线上的点m  
选择随机数r  
计算密文对：c1 = m + r × K，c2 = r × G

3. 解密：
计算s = k × c2 = k × (r × G) = r × (k × G) = r × K  
恢复消息：m = c1 - s

**所以**，我们秩序两个步骤就可以解出flag
- 找回点m：通过解密c1和c2
- 从m恢复flag：使用m的坐标和cipher_left、cipher_right来计算flag的两部分

```
#!/usr/bin/python3
from Crypto.Util.number import long_to_bytes

def extended_gcd(a, b):
    """扩展欧几里得算法，用于计算模逆元"""
    if a == 0:
        return b, 0, 1
    else:
        gcd, x, y = extended_gcd(b % a, a)
        return gcd, y - (b // a) * x, x

def modinv(a, m):
    """计算a在模m下的逆元"""
    a = a % m
    gcd, x, y = extended_gcd(a, m)
    if gcd != 1:
        raise Exception(f'模逆元不存在 (gcd={gcd}, a={a}, m={m})')
    else:
        return x % m

def point_add(P, Q, a, p):
    """在椭圆曲线上添加两点，正确处理所有特殊情况"""
    # 处理无穷远点
    if P is None:
        return Q
    if Q is None:
        return P
    
    x1, y1 = P
    x2, y2 = Q
    
    # 如果点互为负元 (x坐标相同但y坐标不同)
    if x1 == x2 and y1 != y2:
        return None  # 返回无穷远点
    
    # 如果是点加倍 (两点相同)
    if x1 == x2 and y1 == y2:
        # 检查点是否在子群中有限的阶
        if y1 == 0:
            return None  # 如果y=0，那么2P = O
        
        # 点加倍公式
        lam = ((3 * x1 * x1 + a) * modinv((2 * y1), p)) % p
    else:
        # 标准点加法
        lam = ((y2 - y1) * modinv((x2 - x1) % p, p)) % p
    
    x3 = (lam * lam - x1 - x2) % p
    y3 = (lam * (x1 - x3) - y1) % p
    
    return (x3, y3)

def scalar_mult(k, P, a, p):
    """标量乘法：k * P，使用双倍加法法"""
    if k == 0 or P is None:  # 处理k=0或P=O的情况
        return None
    
    result = None
    addend = P
    
    while k:
        if k & 1:
            # 如果当前位为1，添加当前点
            result = point_add(result, addend, a, p)
        # 点加倍
        addend = point_add(addend, addend, a, p)
        k >>= 1
    
    return result

def point_neg(P, p):
    """计算椭圆曲线上点的负元"""
    if P is None:
        return None
    x, y = P
    return (x, (-y) % p)

def point_subtract(P, Q, a, p):
    """点减法：P - Q = P + (-Q)"""
    neg_Q = point_neg(Q, p)
    return point_add(P, neg_Q, a, p)

# 给定的参数
p = 93202687891394085633786409619308940289806301885603002539703165565954917915237
a = 93822086754590882682502837744000915992590989006575416134628106376590825652793
b = 80546187587527518012258369984400999843218609481640396827119274116524742672463
k = 58946963503925758614502522844777257459612909354227999110879446485128547020161

# 给定的点
c1 = (40485287784577105052142632380297282223290388901294496494726004092953216846111, 
      81688798450940847410572480357702533480504451191937977779652402489509511335169)
c2 = (51588540344302003527882762117190244240363885481651104291377049503085003152858, 
      77333747801859674540077067783932976850711668089918703995609977466893496793359)

# 给定的密文
cipher_left = 34210996654599605871773958201517275601830496965429751344560373676881990711573
cipher_right = 62166121351090454316858010748966403510891793374784456622783974987056684617905

# 验证点是否在曲线上
def is_on_curve(point, a, b, p):
    if point is None:
        return True  # 无穷远点总是在曲线上
    x, y = point
    left = (y * y) % p
    right = (x * x * x + a * x + b) % p
    return left == right

print(f"c1在曲线上: {is_on_curve(c1, a, b, p)}")
print(f"c2在曲线上: {is_on_curve(c2, a, b, p)}")

try:
    # 步骤1：计算 k * c2 = r * k * G
    print("计算 k * c2...")
    r_k_G = scalar_mult(k, c2, a, p)
    print(f"k * c2 = {r_k_G}")
    
    # 步骤2：恢复点 m = c1 - r * k * G
    print("恢复点 m...")
    m = point_subtract(c1, r_k_G, a, p)
    print(f"恢复的点 m = {m}")
    print(f"m在曲线上: {is_on_curve(m, a, b, p)}")
    
    if m is None:
        print("错误：恢复的点是无穷远点，无法恢复消息")
    else:
        # 步骤3：恢复flag的两部分
        m_x, m_y = m
        
        # 计算模逆元
        print("计算模逆元...")
        m_x_inv = modinv(m_x, p)
        m_y_inv = modinv(m_y, p)
        
        # 恢复flag的两部分
        print("恢复flag...")
        flag_left_value = (cipher_left * m_x_inv) % p
        flag_right_value = (cipher_right * m_y_inv) % p
        
        # 步骤4：转换回字节并拼接
        flag_left = long_to_bytes(flag_left_value)
        flag_right = long_to_bytes(flag_right_value)
        
        flag = flag_left + flag_right
        print(f"恢复的flag: {flag}")
        
except Exception as e:
    print(f"出错: {e}")
    
    # 尝试另一种方法，使用SageMath风格的椭圆曲线操作
    print("\n尝试备用方法...")
    try:
        # 定义备用方法
        def ec_add(P, Q, a, b, p):
            """椭圆曲线点加法 - 备用实现"""
            if P is None:
                return Q
            if Q is None:
                return P
            
            x1, y1 = P
            x2, y2 = Q
            
            if x1 == x2:
                if (y1 + y2) % p == 0:
                    return None  # P + (-P) = O
                else:  # y1 == y2
                    m = (3 * x1 * x1 + a) * pow(2 * y1, p - 2, p) % p
            else:
                m = (y2 - y1) * pow(x2 - x1, p - 2, p) % p
                
            x3 = (m*m - x1 - x2) % p
            y3 = (m * (x1 - x3) - y1) % p
            
            return (x3, y3)
            
        def ec_scalar_mult(k, P, a, b, p):
            """椭圆曲线标量乘法 - 备用实现"""
            result = None
            addend = P
            
            k = k % p  # 减少k的大小
            
            while k > 0:
                if k & 1:
                    result = ec_add(result, addend, a, b, p)
                addend = ec_add(addend, addend, a, b, p)
                k >>= 1
                
            return result
            
        # 使用备用方法计算
        r_k_G_alt = ec_scalar_mult(k, c2, a, b, p)
        print(f"备用方法 k * c2 = {r_k_G_alt}")
        
        # 计算 c1 - r_k_G_alt
        neg_r_k_G = (r_k_G_alt[0], (-r_k_G_alt[1]) % p)
        m_alt = ec_add(c1, neg_r_k_G, a, b, p)
        print(f"备用方法恢复的点 m = {m_alt}")
        
        # 恢复flag
        m_x_alt, m_y_alt = m_alt
        
        # 使用费马小定理计算模逆元
        m_x_inv_alt = pow(m_x_alt, p - 2, p)
        m_y_inv_alt = pow(m_y_alt, p - 2, p)
        
        flag_left_value_alt = (cipher_left * m_x_inv_alt) % p
        flag_right_value_alt = (cipher_right * m_y_inv_alt) % p
        
        flag_left_alt = long_to_bytes(flag_left_value_alt)
        flag_right_alt = long_to_bytes(flag_right_value_alt)
        
        flag_alt = flag_left_alt + flag_right_alt
        print(f"备用方法恢复的flag: {flag_alt}")
        
    except Exception as e2:
        print(f"备用方法也出错: {e2}")
```
