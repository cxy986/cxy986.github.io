---
title: "misc修复图片"
date: 2026-10-08
draft: false
---




# 题目名称 被揉坏的芙芙

- **赛事**：2026 ？ CTF
- **分类**：Web 
- **分值**：200


## 一、信息收集
<img width="551" height="83" alt="image" src="https://github.com/user-attachments/assets/827131e5-e30e-4763-a2e8-b4dcfc019f9d" />
题上说芙芙图片被挤坏了，用imhex工具看看

<img width="948" height="527" alt="image" src="https://github.com/user-attachments/assets/9718d953-2f8c-410c-8b2e-d11f3bac8a1f" />

能看到IHDR中宽是

<img width="146" height="35" alt="image" src="https://github.com/user-attachments/assets/88715c93-b6b1-45f1-8271-713dd4ac7453" />

宽就剩144了，被挤成面条了，也是锁定问题，后面的CRC也不对，全是00 00 00 00也需要改

那怎么确定原本的宽呢？

解压后总字节数 = 高 × (1 + 宽 × 每像素字节数)
每像素字节数 = 通道数 × 位深/8   （颜色类型6=RGBA → 4通道，位深8 → 4 字节）

已知高是1524，再结合题干中给出的神秘字符（很可能就是正确的CRC）

<img width="207" height="37" alt="image" src="https://github.com/user-attachments/assets/7e4f44ee-c426-41e4-866b-620baf6dbe5b" />


## 二、漏洞分析

修复被恶意改坏的图片

## 三、利用过程

利用正确的crc逆推出正确的宽为2560

修改IHDR中宽的十六进制为00 00 0A 00

<img width="961" height="121" alt="image" src="https://github.com/user-attachments/assets/9967301a-91e6-444c-898d-320d6d4c2b9c" />

哈希验证一下

<img width="543" height="35" alt="image" src="https://github.com/user-attachments/assets/f88c4092-df98-467e-b2a0-d5dd326f6a6f" />

结果与题干中的神秘字符完全一样！

保存，另存，得到flag

<img width="839" height="496" alt="image" src="https://github.com/user-attachments/assets/addec572-b947-4dce-913e-9c954b3bd5ed" />



## 四、Payload/exp
python

import zlib, struct
d = open('芙.png','rb').read()
off, idat = 8, b''
while off < len(d):
    n = struct.unpack('>I', d[off:off+4])[0]
    if d[off+4:off+8] == b'IDAT': idat += d[off+8:off+8+n]   # 只取 body
    off += 12 + n
raw = zlib.decompress(idat)
print('压缩:', len(idat), ' 解压:', len(raw), ' 每行:', len(raw)/1524, ' 宽:', (len(raw)/1524-1)/4)

## 五、Flag

crc32 is important

## 六、总结 / 踩坑

就是修复图片，修改文件头，你要说踩坑吧...还真有，这是我做的第一个misc的题，为啥，我是芙厨

byd出题人没说是哪个芙，我还以为是芙宁娜呢，兴冲冲的做题还以为有芙芙美图看

结果是芙莉莲啊。。。莫名有点失望。。。
