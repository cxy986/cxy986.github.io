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
