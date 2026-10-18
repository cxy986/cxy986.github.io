---
title: "伪加密"
date: 2026-10-18
draft: false
---


老早做的我记得我写了啊，当时还研究了半天来着

算了再写一遍吧

跟那个快做完了结果没保存的中国人能飞有关系

这个题是中国人能飞的part1

现在学精了啊，看见压缩包知道先看十六进制了
而不是跟个愣头青一样爆破一个压根不存在的密码

<img width="313" height="269" alt="屏幕截图 2026-10-18 185852" src="https://github.com/user-attachments/assets/0704d1f0-2daf-470d-80d6-5052030899bb" />
<img width="313" height="290" alt="屏幕截图 2026-10-18 185841" src="https://github.com/user-attachments/assets/f5c06c17-f42d-4513-8fa2-2561bccf3362" />

知识点

ZIP文件由三部分组成：
1. 数据区（Data Area）：存放压缩的实际文件数据。
2. 目录区（Central Directory）：记录文件信息的目录。
3. 目录结束标志（End of Central Directory, EOCD）。
在ZIP的全局方式位标记（General Purpose Bit Flag）中：
• 奇数（如 09 00 或 01 00）：表示该文件已加密。
• 偶数（如 00 00）：表示该文件未加密。
伪加密的做法是：文件真正的数据并没有加密，但却把目录区或文件头的标志位改成了奇数（比如从 00 00 改为 09 00 或 01 00），导致解压时弹窗提示输入密码

反正就一句话，重点看pk+6的地方看见09 08自动应激改成08 08就行了

这个题改了四处

<img width="338" height="295" alt="屏幕截图 2026-10-18 190459" src="https://github.com/user-attachments/assets/1d284dfa-5146-4deb-b01c-75feea5c72df" />

<img width="345" height="269" alt="屏幕截图 2026-10-18 190508" src="https://github.com/user-attachments/assets/03e557d1-658c-4a60-b4cb-d40c2369cb69" />

修改处标红

完了就能打开了

<img width="257" height="147" alt="image" src="https://github.com/user-attachments/assets/daf8cb2e-dea4-4045-af25-0662a857e3e7" />

里面还有一个真加密的文件

<img width="272" height="137" alt="image" src="https://github.com/user-attachments/assets/a0b7572d-caa6-4c6c-89f9-b6fe6bdbf3e2" />

应该就要用第一个获得的qctf2026????
爆破了吧

构造payload

zip2john 出 hash

cd E:\john-1.9.0-jumbo-1-win64\run
.\zip2john.exe E:\study\zipflag\out\final.zip > E:\study\zipflag\final_zip2john.hash

<img width="575" height="59" alt="image" src="https://github.com/user-attachments/assets/fdefb1ce-8727-4957-841a-b101d827de3a" />

hashcat掩码爆破
cd E:\hashcat\hashcat-7.1.2
.\hashcat.exe -m 17210 -a 3 `
  E:\study\zipflag\final_clean.hash `
  "qctf2026?1?1?1?1" -1 "?l?d_" `
  --potfile-path E:\study\zipflag\final.potfile -w 3

payload解析

-m 17210	PKZIP (Uncompressed)。压缩条目用 17200；多文件混合用 17230／17225

-1 "?l?d_"	自定义字符集 1：?l=a-z(26) + ?d=0-9(10) + _(1) = 37 个

"qctf2026?1?1?1?1"	12 位掩码，后 4 位从字符集 1 里取，共 37⁴ = 1,874,161

-w 3	workload profile，桌面端最高档
--potfile-path	结果写哪；不写就是 hashcat 目录下的 hashcat.potfile

<img width="571" height="264" alt="image" src="https://github.com/user-attachments/assets/c65ca249-7f66-4eed-8fa5-f29a8c0d4217" />

看结果

<img width="142" height="41" alt="image" src="https://github.com/user-attachments/assets/7c4a5703-101a-4bf4-8ab8-ce6d2268df8d" />

密码是qctf2026g0od

拿flag

<img width="121" height="50" alt="image" src="https://github.com/user-attachments/assets/b12473da-efe3-4008-ab7f-fffbbdcca20f" />


