---
title: "战败CG"
date: 2026-10-21
draft: false
---
# 总结
一个啥子捡了个稀罕玩意，没仔细看说明就把他锁保险箱里了

结果代价是保险箱密码和钥匙也一起被锁保险箱里了

# 起因和过程

本来就想装个崩铁mod的，结果发现咸鱼商家给的游戏库真的太全了

<img width="853" height="512" alt="image" src="https://github.com/user-attachments/assets/d2b38dfc-f0c7-48ac-8004-4b96f691ddf0" />

居然还有想玩但是没钱买的老头环和黑猴

于是就下了

顶着5MB/s速度下了一整天，下午上完课兴致冲冲的解压文件并打开了启动器

结果启动器叽里咕噜说了一大堆英文看不懂给ai了

<img width="377" height="119" alt="image" src="https://github.com/user-attachments/assets/18277be1-847e-4dd1-84ca-3cde5b7dabc6" />

<img width="293" height="77" alt="image" src="https://github.com/user-attachments/assets/ff8cb8d9-913f-4c9a-a06d-a5868ed68851" />

卧槽我刚看见他说bitlocker没开...

ai的话不能全信我算是体会到了

结果给自己关外面了<img width="1080" height="1440" alt="be822165b9014206343ea9bfce0ae7d7" src="https://github.com/user-attachments/assets/53bce440-409c-45c7-a8f9-5dc1712ddbf3" />

<img width="1706" height="1279" alt="85ef0b5b0ae56c4ace4d197f435876c6" src="https://github.com/user-attachments/assets/38f2e8bb-2ed9-4795-afdd-ab89a1760e47" />

微软密码忘了，邮箱被封了

<img width="65" height="67" alt="image" src="https://github.com/user-attachments/assets/3152c0f4-0110-4538-a1a0-33cb0e208d92" />

都准备去电脑店重装系统了，学长给我救回来了

不敢想去重装完，我要顶着5mb/s的速度下总量快500g的原崩绝鸣有多绝望

解决完的那天晚上我也去研究了一下为啥这黑猴这么麻烦

# 复盘：

黑猴作为新游戏用的新技术是不一样嘿

那些比较老的游戏基本上都是传统 D 加密破解，找到校验代码，打补丁、换 exe

但这个不一样

这个是 Hypervisor 绕过（虚拟机绕过 D 加密）

1）Hypervisor 把整个 Windows 装进一台虚拟机里跑
2）Hypervisor 伪造一套真实物理机的硬件特征，D 加密就被骗过去了
3）游戏文件一个字节都没改，所以防篡改校验全过
4）代价是它要独占 CPU 的虚拟化扩展（VT-x），还要加载一个没有数字签名的内核驱动

所以我关了：
Windows 自己的 Hypervisor（抢 VT-x，两个不能共存）
VBS / 内存完整性 (HVCI)（会拦住未签名驱动）
驱动签名强制 (DSE)回应学长（你玩盗版游戏你改这作甚）

# 反思
1）全信ai的话没有自己检查自己的bitlocker到底开没开
2）面对来历不明的启动器他要啥我给他什么把BIOS/UEFI都改了
3）QQ封了没及时检查和换绑每个绑那个邮箱的各个账号，直到某天用不了才后悔
4)密码习惯不好，有多个不同密码还不记到备忘录，好记性不如烂笔头这一块

反正...卧槽这免费的老头环不错啊

<img width="1706" height="1279" alt="b4c4d61dfa50f4d138e210e9eabdddcd" src="https://github.com/user-attachments/assets/27db7d79-70a2-449d-a369-86bd1ef8d971" />
<img width="1440" height="1080" alt="c6e855b0c912af64046b8edf4a127bf8" src="https://github.com/user-attachments/assets/b635880f-5131-4f46-9d30-4d9a674699f8" />

除了不能联机啥都能干，已经值了，问就是下次还敢，只不过会更小心，操作之前先把风险和后果都了解一遍，评估后再决定
