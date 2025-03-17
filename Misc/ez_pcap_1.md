# 挑战简介
一个pcapng文件，使用wireshark分析

# 思路
打开后，追踪TCP流，发现全是加密文字  
因此尝试打开协议分级，发现smb占比很大  
![image](https://github.com/user-attachments/assets/9324226b-3c87-406e-bd0c-fd1c9a60846e)  
我们选择分析smb信息，果不其然，发现flag文件，直接导出对象 选择smb，导出那个fl4g即可  
![image](https://github.com/user-attachments/assets/09927003-2a6a-4410-a706-f0a5633b3545)

