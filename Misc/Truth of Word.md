# 挑战简介
给了你一个docm文件，一些word文档小知识

# 思路
flag总共分为3部分，考点如下
- word文字隐写
- word宏启用
- word文件的本质

## 一
word文字被隐藏了，成了一个等于号的存在![2d58ed8892b7bdd1749f8e80162c683](https://github.com/user-attachments/assets/fb49cf70-ef80-4816-9bf8-0058ad729ff5)
鼠标放上去就会将隐藏内容显示出来，显而易见了

## 二
宏定义首先需要你将此文件属性内解除锁定，以应用宏定义，解除了宏定义，一个ALT+F8，所有宏就都出现了，flag2就在里面![image](https://github.com/user-attachments/assets/fab78d4e-1039-4a50-a5a0-1756b42833e0)

## 三
其实docm文件本质是一个压缩包，不信你改它为zip后缀试试，里面有个文件就是flag3的内容

