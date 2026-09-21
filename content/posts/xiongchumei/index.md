---
title: "ssrf题目复现"
date: 2026-09-19
draft: false
---
题目长这样
<img width="1000" height="627" alt="屏幕截图 2026-09-17 205016" src="https://github.com/user-attachments/assets/0babc3ca-466e-46fa-8388-d16f98d6f8d7" />
正常跟他玩都是陷阱蜜罐
<img width="1271" height="764" alt="屏幕截图 2026-09-17 205354" src="https://github.com/user-attachments/assets/a4e77bcb-08a9-497b-a590-8811721adbac" />

<img width="1265" height="760" alt="屏幕截图 2026-09-17 205303" src="https://github.com/user-attachments/assets/c7a230a7-b3b8-4103-9a0f-3c0960dfa902" />
咋确定是ssrf呢？

<img width="388" height="99" alt="屏幕截图 2026-09-17 205621" src="https://github.com/user-attachments/assets/36ffb22c-f3dd-4cc0-a1f2-143a0dfe25e8" />

可能是因为这里写的有吧（审题这一块）
咳咳那实战呢啥也不知道
<img width="647" height="731" alt="屏幕截图 2026-09-17 205951" src="https://github.com/user-attachments/assets/04820ce7-d49b-4326-a0d0-00b93623248d" />
先curl一下，发现有POST 和url，url 这个参数名就是 SSRF 的典型特征，服务器要拿他发请求
但现实是还是没思路
渗透测试很值钱的一招，故意畸形输入报错

<img width="795" height="101" alt="image" src="https://github.com/user-attachments/assets/4d7a187c-4a3a-49b3-9390-63327e5e80dd" />

<img width="1230" height="719" alt="image" src="https://github.com/user-attachments/assets/fe8fdddc-0f18-42ce-abff-9aae63712f44" />

这次是真拿到关键了，gopher

Gopher 是 1991 年的老协议（RFC 1436），比 HTTP 还早，后来被 Web 淘汰了。它的设计极简：

客户端连上 TCP 端口  →  把 selector 原样写进 socket  →  读回响应

"原样写进 socket"这六个字，就是它的全部威力。

gopher://<主机>:<端口>/<item-type><selector>
                            ↑ 一个字符，表示条目类型
题上说在54321端口，先打一下   
<img width="1257" height="719" alt="屏幕截图 2026-09-17 211049" src="https://github.com/user-attachments/assets/550014a6-7eb7-43f3-b3ef-d4defc1948e5" />
<img width="1277" height="737" alt="屏幕截图 2026-09-17 211103" src="https://github.com/user-attachments/assets/25106464-425a-4280-9a6d-ec222e831837" />
OK啊拿到secret-honey的路径了

<img width="964" height="527" alt="屏幕截图 2026-09-17 211731" src="https://github.com/user-attachments/assets/5343b189-034c-45d1-8d24-68429b296345" />
<img width="779" height="484" alt="屏幕截图 2026-09-17 211737" src="https://github.com/user-attachments/assets/2f1fec55-008a-4a71-8830-1d8447d979e5" />
404了？
       这里又学习了gopher 的 item-type，后端会无条件吃掉路径的第一个字符
   <img width="905" height="554" alt="image" src="https://github.com/user-attachments/assets/4b120015-fc86-4272-b1d7-b2733527a98b" />
   加了一个-拿到flag


   总结一下

   gopher 协议（RFC 1436）规定，URL 里 端口/ 后面的第一个字符是 item-type（资源类型）：

item-type	含义
0	文本文件
1	目录
7	搜索服务器
_	查询（最常用于 SSRF）

SSRF（Server-Side Request Forgery，服务端请求伪造）：服务器端接收了用户可控的输入，并把它当成"要去请求的目标地址"，替你去发起请求。

① 回显型（有回显） 服务器把请求到的内容返回给你。你能直接读到内网服务的数据。这题就是回显型，是最舒服的情况。

② 盲 SSRF（无回显） 服务器只告诉你"成功/失败"，或者什么都不说。这种要靠：

时间盲打：目标端口开着→响应快，关着→连接被拒/卡住
错误信息：像这题的 Connection refused
带外（OOB）：让服务器去访问你自己的 DNSLog / VPS，从你这边看有没有回连


利用方式	说明
端口扫描	探测内网开放端口（这题前几步就是这么干的）
读本地文件	file:///etc/passwd（如果 file 协议没禁）
打内网 Web	访问内网后台、云元数据 169.254.169.254
打未授权服务	Redis / MySQL / FastCGI / Memcached（靠 gopher）
绕过访问控制	内网服务信任来源 IP，以为是"自己人"

防御
协议白名单：只允许 http / https，干掉 gopher、dict、file、ftp
目标 IP 校验：解析域名后检查 IP，禁止回环 127.0.0.0/8、私网 10/8、172.16/12、192.168/16、链路本地 169.254/16、::1
禁止跟随重定向（或者每一跳都重新校验）
防 DNS 重绑定：先解析 → 校验 IP → 再用同一个 IP 去连接
网络层隔离：出站流量强制走代理，内网服务别和应用放同一个网络命名空间
