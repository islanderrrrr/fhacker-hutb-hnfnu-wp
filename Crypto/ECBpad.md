# 挑战简介
ECB模式是最简单的加密模式，它将明文分成固定大小的块，然后分别对每个块进行加密。
在这个挑战中，AES-ECB使用16字节(128位)的块大小。

# 思路
利用其弱点，我们可以逐字节猜测flag，flag前缀是SYC{  
前十五个字节作填充，最后一个字节用作猜测，爆破每一种可能，直到出现加密后的结果与flag加密的结果相同  
相同则把猜出的字节前移，并猜测下一个字节，以此循环  
```
#!/usr/bin/env python3
import socket
import string
import time

# 服务器配置
HOST = 'nc1.ctfplus.cn'  # 修改为目标服务器地址
PORT = 25198

# 可能的字符集 - 优先考虑常见FLAG字符
CHARSET = string.ascii_letters + string.digits + '{}' + '_' + '!'

def receive_until(s, end_marker=None, timeout=5):
    """接收数据直到遇到特定标记或超时"""
    s.settimeout(timeout)
    data = b''
    start_time = time.time()
    
    try:
        while True:
            if time.time() - start_time > timeout:
                break
                
            chunk = s.recv(1024)
            if not chunk:
                # 连接已关闭
                break
                
            data += chunk
            
            if end_marker and end_marker in data:
                break
    except socket.timeout:
        pass
        
    s.settimeout(None)
    return data

def connect_and_send(plaintext):
    """连接到服务器并发送明文，返回密文"""
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((HOST, PORT))
        
        # 接收初始消息
        welcome = receive_until(s, b'Start?(yes/no)')
        
        # 发送yes开始
        s.sendall(b'yes\n')
        
        # 等待提示符
        prompt = receive_until(s, b'[-] ')
        
        # 发送明文
        s.sendall(plaintext + b'\n')
        
        # 接收密文
        response = receive_until(s, b'\n')
        
        # 关闭连接
        s.close()
        
        # 提取密文
        if b'Your cipher:' in response:
            hex_cipher = response.split(b'Your cipher:')[1].strip()
            cipher = bytes.fromhex(hex_cipher.decode('utf-8'))
            return cipher
        else:
            print(f"[!] 未找到密文标记. 收到: {response}")
            return None
    except Exception as e:
        print(f"[!] 连接/发送过程中出错: {e}")
        if 's' in locals() and s:
            s.close()
        return None

def extract_flag():
    """提取FLAG - 使用最简单直接的ECB攻击方法"""
    # 已知部分FLAG
    known_flag = b"SYC{CRY9T0HACK_A"  # 从您的输出中提取已知部分
    
    # 块大小
    block_size = 16
    
    # 最大尝试长度
    max_length = 50
    
    print("[*] 开始提取FLAG从第 {} 个字符开始...".format(len(known_flag) + 1))
    print(f"[*] 当前已知FLAG: {known_flag.decode('utf-8', errors='ignore')}")
    
    # 逐字符提取
    found_end = False
    
    while not found_end and len(known_flag) < max_length:
        # 当前要猜测的字符位置
        current_pos = len(known_flag)
        
        # 构造基础填充 - 使得目标字符在块边界上
        pad_blocks = block_size - 1 - (current_pos % block_size)
        if pad_blocks < 0:
            pad_blocks += block_size
            
        padding = b'A' * pad_blocks
        
        print(f"[*] 尝试第 {current_pos + 1} 个字符，填充长度: {pad_blocks}")
        
        # 从密文中获取目标字符位置所在的块
        target_block_start = ((pad_blocks + current_pos) // block_size) * block_size
        target_block_end = target_block_start + block_size
        
        print(f"[*] 目标块索引: {target_block_start//block_size}, 范围 {target_block_start}-{target_block_end-1}")
        
        # 获取基准密文 (仅填充)
        base_cipher = connect_and_send(padding)
        if not base_cipher or len(base_cipher) < target_block_end:
            print("[!] 无法获取有效的基准密文")
            time.sleep(1)
            continue
            
        base_block = base_cipher[target_block_start:target_block_end]
        
        # 字符是否找到的标志
        found = False
        
        # 遍历所有可能的字符
        for c in CHARSET:
            test_char = bytes([ord(c)])
            
            # 构造测试明文: 填充 + 已知FLAG部分 + 测试字符
            test_plaintext = padding + known_flag + test_char
            
            # 发送并获取密文
            test_cipher = connect_and_send(test_plaintext)
            
            if not test_cipher or len(test_cipher) < target_block_end:
                print(f"[!] 测试字符 '{c}' 时无法获取有效密文")
                continue
                
            # 获取测试密文对应块
            relevant_block_start = ((pad_blocks + len(known_flag)) // block_size) * block_size
            relevant_block_end = relevant_block_start + block_size
            
            if relevant_block_end > len(test_cipher):
                print(f"[!] 测试字符 '{c}' 时密文长度不足，跳过")
                continue
                
            test_block = test_cipher[relevant_block_start:relevant_block_end]
            
            # 比较块
            if base_block == test_block:
                known_flag += test_char
                found = True
                print(f"[+] 找到字符: '{c}'")
                
                # 如果找到右大括号，表示FLAG结束
                if c == '}':
                    found_end = True
                
                break
                
            # 添加短暂延迟避免服务器限流
            time.sleep(0.1)
            
        if not found:
            print(f"[!] 无法找到第 {current_pos + 1} 个字符，尝试不同的方法")
            
            # 尝试额外的方法，例如更改字符集或调整块计算
            
            # 如果仍然找不到，可以尝试跳过这个字符
            # known_flag += b'?'  # 占位符，表示未知字符
            break
            
    print(f"\n[+] 最终提取的FLAG: {known_flag.decode('utf-8', errors='ignore')}")
    return known_flag

if __name__ == "__main__":
    extract_flag()
```
