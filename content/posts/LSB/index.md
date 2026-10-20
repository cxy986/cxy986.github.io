---
title: "图片最低位隐写LSB"
date: 2026-10-19
draft: false
---

<img width="271" height="278" alt="magic" src="https://github.com/user-attachments/assets/20661328-74c7-46d8-813a-537d0dcd4878" />


<img width="394" height="203" alt="image" src="https://github.com/user-attachments/assets/b3342d03-1b94-40fc-a3ad-010f8c43eba6" />


先看题，给了个png图片，介于之前做的图片题

我选择先看他的文件结构

文件头看着没啥问题

<img width="670" height="225" alt="image" src="https://github.com/user-attachments/assets/d810398a-71d4-41ae-8e73-6696e308a49f" />

文件尾也没啥问题

<img width="564" height="217" alt="image" src="https://github.com/user-attachments/assets/e783e593-c6b8-4b10-82de-a8ab2e121f4a" />

貌似真是个平平无奇的png文件？

看看像素数据

由此得出今天主题lsb图片最低位隐写

原理为最低位是最不起眼的那一位，只改那个人肉眼几乎看不出来

打开stepsolve工具进行数据分析

<img width="356" height="275" alt="image" src="https://github.com/user-attachments/assets/3c0d014f-99d2-44ff-b1bc-a2c69907460a" />

没有，怪了，后来发现alpha是恒定的，关掉α0

果然得到隐写的东西

<img width="351" height="119" alt="image" src="https://github.com/user-attachments/assets/f427a26c-522f-4fc2-9003-946f8c107519" />


后面有个=长得像base64

解码

<img width="853" height="491" alt="image" src="https://github.com/user-attachments/assets/a1f1f2c6-d266-492c-8536-f573977c0a90" />

？？这是。。二进制？

要ASCII吗？那也不对啊，这间隔这么刻意

那就是前天中国人能飞接触的摩斯电码了

11	--	M	1000	-...	B
11111	-----	0	10000	-....	6
010	.-.	R	00001	....-	4
000	...	S	001101	..--.-	_
00011	...--	3	01111	.----	1
001101	..--.-	_	10	-.	N
00001	....-	4	001101	..--.-	_
0010	..-.	F	0100	.-..	L
1	-	T	000	...	S
00011	...--	3	1000	-...	B
010	.-.	R			
001101	..--.-	_

M0RS3_4FT3R_B64_1N_LSB

OK拿下

这里再说一下我自己对LSB的理解

首先这是一个隐写技术，将想要隐写的东西藏进对图片影响最小的最低位
0-255中颜色，RGB对应红绿蓝三种颜色

比如254和255都是白色，他们的差异很小很小，人眼几乎分辨不出

而最低位隐写就是利用这一微小差异，修改了像素灰度

而这种修改又是针对最低位的，造成了很微小的差别

比如灰度255和254,198和197，人眼几乎看不出。

