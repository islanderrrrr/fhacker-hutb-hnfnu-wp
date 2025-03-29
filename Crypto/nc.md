# 挑战简介
nc一个地址，会有一些回显  
服务器创建一个20字符的随机字符串 proof  
计算 proof 的SHA-256哈希值  
向客户端发送除前4个字符外的部分，以及完整字符串的哈希值  
客户端需要找到缺失的前4个字符，使得拼接后的字符串哈希值与服务器提供的相同  

# 思路
因此，我们的解题思路  
1. 首先需要破解PoW(Proof of Work)挑战

2. 获取完整flag：

flag格式通常是SYC{...}，长度未知但肯定不超过32个字符  
我们有35次查询机会，可以通过输入1-32的数字，逐个获取flag的每个字符  
需要按顺序查询，例如输入1得到S，输入2得到Y，以此类推  
```
#!/usr/bin/python3
import socket
import re
import hashlib
import string
import itertools

def solve_pow(challenge):
    # 从挑战中提取信息
    match = re.search(r'sha256\(XXXX\+(.*)\) == (.*)', challenge)
    if not match:
        return None
    
    suffix = match.group(1)
    target_hash = match.group(2)
    
    print(f"需要找到XXXX，使得sha256(XXXX+{suffix}) == {target_hash}")
    
    # 尝试所有可能的4字符前缀
    chars = string.ascii_letters + string.digits
    for prefix_tuple in itertools.product(chars, repeat=4):
        prefix_str = ''.join(prefix_tuple)
        test_str = prefix_str + suffix
        current_hash = hashlib.sha256(test_str.encode()).hexdigest()
        if current_hash == target_hash:
            print(f"找到解决方案: {prefix_str}")
            return prefix_str
        # 可选：添加进度指示
        if sum(ord(c) for c in prefix_str) % 100000 == 0:
            print(f"正在尝试: {prefix_str}")
    
    return None

def get_flag():
    HOST = 'nc1.ctfplus.cn'
    PORT = 26201
    
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((HOST, PORT))
        
        # 接收工作量证明挑战
        data = b""
        while b"sha256" not in data and b"Plz tell me XXXX" not in data:
            chunk = s.recv(1024)
            if not chunk:
                break
            data += chunk
            if b'\n' in chunk:
                break
        
        pow_challenge = data.decode()
        print("服务器初始消息:", pow_challenge)
        
        # 解决工作量证明
        # 从实际接收到的挑战中提取信息
        suffix_match = re.search(r'sha256\(XXXX\+(.*?)\)', pow_challenge)
        hash_match = re.search(r'== (.*?)$', pow_challenge)
        
        if suffix_match and hash_match:
            suffix = suffix_match.group(1)
            target_hash = hash_match.group(1)
            
            # 重新构建挑战字符串以适应solve_pow函数
            formatted_challenge = f"sha256(XXXX+{suffix}) == {target_hash}"
            solution = solve_pow(formatted_challenge)
        else:
            print("无法解析工作量证明挑战")
            return
        
        if not solution:
            print("无法解决工作量证明")
            return
        
        # 发送工作量证明答案
        prompt = s.recv(1024)  # 接收提示符 "[+] Plz tell me XXXX: "
        print(f"提示: {prompt.decode()}")
        
        print(f"发送工作量证明答案: {solution}")
        s.sendall(f"{solution}\n".encode())
        
        # 接收工作量证明响应
        response = s.recv(1024).decode()
        print("工作量证明响应:", response)
        
        # 如果通过工作量证明，开始获取flag
        flag = ""
        for i in range(1, 33):
            # 可能需要接收提示符 [+]
            prompt = s.recv(1024)
            if prompt:
                print(f"提示: {prompt.decode().strip()}")
            
            # 发送数字
            print(f"发送: {i}")
            s.sendall(f"{i}\n".encode())
            
            # 接收对应位置的字符
            response = s.recv(1024).decode().strip()
            print(f"接收: {response}")
            
            # 从响应中提取字符
            try:
                char = response.split('] ')[1].strip()
                flag += char
                print(f"得到第{i}个字符: {char}, 当前flag: {flag}")
            except IndexError:
                print(f"无法从响应中提取字符: {response}")
        
        print(f"完整的flag是: {flag}")
        return flag

if __name__ == "__main__":
    print("开始破解flag...")
    get_flag()
```
