---
title: "ezFilter"
date: 2026-10-18
draft: false
---

看题看题，不想写归档啥的了，从八点半做到十点四十八

趁热打铁，我也是真的累了

<img width="389" height="430" alt="image" src="https://github.com/user-attachments/assets/3018fc56-978e-497a-b363-dc5014dc6eb7" />

<img width="309" height="413" alt="image" src="https://github.com/user-attachments/assets/d685d130-6933-46ba-93dd-ad4ca0f36596" />


大概翻译一下就是有1024个文件然后真正的flag只在其中一个里面

然后还会概率触发challenge

challenge的规则就是从十个单词里面抽五个进行SHA-256编码

最后输出challenge要求的taget

这个是要求写脚本的

差不多就是这样
<img width="407" height="383" alt="image" src="https://github.com/user-attachments/assets/29039565-5eee-4f97-99df-fb6381e32fbc" />

好吧进入正题，这个题我首先想到的是二分法，试了一下就发现了一个大问题，
题目会返回三个状态hot warm cold，这对二分法显得很没用，而且二分法每次缩小的范围越来越小，可能会使用很多次

<img width="245" height="151" alt="image" src="https://github.com/user-attachments/assets/26493bc6-2daa-4575-80d5-8c9c54d050c3" />

然后就通过搜索引擎和ai接触到了三分搜索这个东西

https://www.google.com.hk/goto?url=CAESZAHrOzAVH1MS2Q3S0aMp0zYTOjRmy1LciD0cSCO9YM5t3FS9YcXpczdhi2NkFF2r_IUiz8pxBUUF8uGo9RYg0ejHipSYHoido6uWmTTtfNZzj1JXrtxe5Farf_O0Y7o8q2WkdH4

太阴了卧槽

<img width="593" height="299" alt="image" src="https://github.com/user-attachments/assets/55331912-cb4a-44c1-8511-734f403a0e84" />

666运气差的时候第一步就触发challenge了

还是跑脚本，再-H加个请求头，请求头里加你选的五个单词就行了

这里有个坑，也是我个人的习惯不太好
<img width="575" height="187" alt="image" src="https://github.com/user-attachments/assets/032b5c82-e362-44d6-94f4-5e346cd1c6b7" />

没有保存会话的cookie,导致403

还是要习惯加入-c和-b啊

大概就是http://8000-643b04ee-83a9-4431-b947-9eb20440024f.challenge.ctfplus.cn/probe?lo=0&hi=1023
进行一个全范围的搜索
会返回hot，按题目公式

length = 1024
mid = 0 + 1024 // 2 = 512
half = (1024 - 1) // 4 = 255
middle = [512 - 255, 512 + 255] = [257, 767]

所以flag的范围在【257,767】之间

接下来就要用到三分搜索了、

要分成三组总length是767-257+1=511

分成三组，理想是511/3=170.33

所以分成171,170,170三组

首先选择的查询长度是171+170=341
也就是区间【257，597】

为啥？当时看到的时候我也很懵逼分成三组的话不应该一组一组查吗？

后来又深入了解了三分搜索这个东西

关键是：查询区间中，hot 和 warm 都在查询区间内部。

因此查询区间不能只容纳 171 个 hot 候选，而应该容纳因此查询区间不能只容纳 171 个 hot 候选，而应该容纳：
hot + warm
= 171 + 170
= 341

这样的话length = 597 - 257 + 1
       = 341

mid = 257 + 341 // 2
    = 427

half = (341 - 1) // 4
     = 85

middle = [427 - 85, 427 + 85]
       = [342,512]

hot:
342..512
共 171 个

warm:
257..341 和 513..597
共 85 + 85 = 170 个

cold:
598..767
共 170 个
三组都是170！！！
话不多说，我先尝尝味

<img width="560" height="206" alt="image" src="https://github.com/user-attachments/assets/bca8762e-89c3-44e9-bfdd-356232a0a227" />

返回hot继续三分
length = 597 - 257 + 1 = 341
mid    = 257 + 341 // 2 = 427
half   = (341 - 1) // 4 = 85
middle = [427 - 85, 427 + 85]
       = [342,512]
一共 171 个候选，分三分56,57,58

下一步请求
<img width="577" height="160" alt="image" src="https://github.com/user-attachments/assets/0741cfae-c8a7-4154-8f91-4c35994293b6" />

返回hot，一下重复操作省略大概就是返回hot直接三分，返回cold就把剩下的一组拿出来，返回warm就来个二分

最后锁定390,391,392三个
<img width="572" height="161" alt="image" src="https://github.com/user-attachments/assets/6f60314a-124a-4395-a97f-817dfc9671f1" />

390,391是cold所以答案就是392

最后一步触发challenge
<img width="565" height="265" alt="image" src="https://github.com/user-attachments/assets/54dc08fe-ee2e-48ce-86c2-ede8c46f505c" />

<img width="575" height="146" alt="image" src="https://github.com/user-attachments/assets/5233de51-7403-494f-9365-e149dff0e3c1" />

跑脚本，拿flag

总结：web手，除了web啥都会，昨天听摩斯电码，今天研究算法



