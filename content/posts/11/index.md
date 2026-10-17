---
title: "区块链入门"
date: 2026-10-17
draft: false
---
<img width="385" height="426" alt="image" src="https://github.com/user-attachments/assets/063f3d40-2b54-4cd0-a26e-f0dec6002298" />


如题区块链入门，夹带点私货

<img width="1438" height="1426" alt="278f3c58109d79a64dc6d5fa6a5c56ab" src="https://github.com/user-attachments/assets/5ca0a47e-f9ae-4f1e-8012-037f610f3218" />

<img width="1438" height="790" alt="2bfd5061c478fbbd6226eb68f9e48bf3" src="https://github.com/user-attachments/assets/e2f473ea-a5b1-43cf-b0ac-91cd7edd8c3c" />

假期想象0.8utsd配上3000倍杠杆直接撬动地球，结果是爆仓速度堪比光速

咳咳，回到题上，毕竟是入门题，以入门学习为主

<img width="200" height="380" alt="image" src="https://github.com/user-attachments/assets/5e1902ef-b563-40ac-9aa5-ae31f1297ae1" />

思路也是非常的简单，源码都给你了

<img width="533" height="224" alt="image" src="https://github.com/user-attachments/assets/36606de2-c22f-4453-b97c-e741a1b53173" />

这是那个链，给它当个大账本吧，然后这个题唯一要做的就是把这个账本里内置的小程序里的开关打开

也就是对暗号源码里有写

<img width="327" height="47" alt="image" src="https://github.com/user-attachments/assets/94778a27-2054-4eed-bfa7-019e35772df5" />
看见password就应激这一块

emmmm累了

大概就是
看懂一份最简单的 Solidity 合约，从源码定位唯一的状态修改点。
会用 eth_call 做零风险预演 ，先模拟，再上链。
节点不能帮我签名，必须自己: keccak → ABI 编码 → RLP → ECDSA 签名 → raw tx

keccak256 + RLP + 一次 HTTP POST。这就是上链了吧
