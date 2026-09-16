---
title: "第一周学习任务"
date: 2026-09-16
draft: false
---

## 不知道这算啥

科学上网就不说了

孩子们谷歌账号又没了，换手机号的时候忘了把谷歌绑的手机号也换了，现在被关家外面了
<img width="1250" height="719" alt="4" src="https://github.com/user-attachments/assets/e589f3f0-57f2-47b0-906c-cc57f611e17f" />

## agent使用
然后登场的就是我的大肥鱼和GPT了
<img width="957" height="407" alt="7" src="https://github.com/user-attachments/assets/63ad3fb9-1a2b-41d0-9319-0469e9962da0" />
<img width="1280" height="742" alt="1" src="https://github.com/user-attachments/assets/2cd4b6ea-3172-4d83-9edd-20f5f09a56d0" />
dsh还接的中转站，已经开始用来学习和做一些任务了，比如说我用dsh写脚本自动识别姓名，帮我完成团支书的任务
<img width="1280" height="722" alt="屏幕截图 2026-09-16 190359" src="https://github.com/user-attachments/assets/c6adda69-cfdb-4f50-a04d-b1cf994ca62f" />
还有进行一些学习啥的就不赘述了

## web基础
http和https我自认为已经很熟悉了
咳咳这是我自己对从浏览器按下enter到渲染出页面的理解
输入url>DNS解析成ip地址>网关发送>浏览器服务器tcp三次握手>tls握手>浏览器发送https请求>服务器处理响应>过去个html>浏览器解析渲染页面
（纯个人理解，写之前没查任何资料，欢迎指正补充）
burp的话我已经实操过了
<img width="1280" height="737" alt="5" src="https://github.com/user-attachments/assets/31bc0bc1-3f45-495f-ab61-01b3ec975ba5" />
<img width="1278" height="800" alt="屏幕截图 2026-09-16 191154" src="https://github.com/user-attachments/assets/d487ff11-ba6c-4028-acd9-b019b945876e" />
大概就是捉青蛙这个题要求收集青蛙图鉴，我用proxy拦截分析请求，发现他青蛙图鉴数量跟cookie里的1,2,3等数字有直接关联，于是尝试修改cookie，然后了解到了签名这个东西
然后用hashcat爆破签名密钥，伪造cookie再用repeater发送，最后收集全图鉴拿到flag
前端三件套
<img width="826" height="706" alt="2" src="https://github.com/user-attachments/assets/310f286d-1c3a-4f1b-8d7f-0c50de7b1bb5" />
<img width="813" height="703" alt="3" src="https://github.com/user-attachments/assets/c13ebddc-a306-4f25-8f68-1f9f6515f8d4" />
那个简单的静态网站是我自己搭的，又借助agent搭了个flask动态网站，目前已上线（不是豆包豆包帮我搭个动态网站，目前能看懂60%）
欢迎访问https://websecurity-q98a.onrender.com成为人类
浏览器开发者工具的话也用过
<img width="1139" height="662" alt="6" src="https://github.com/user-attachments/assets/df1f212a-9411-4df6-9c07-b34eb4762aaf" />
大概就是看源码，看请求，改用户身份（从user改成admin），看cookie，控制台这几个常用


以后会继续学习的，看能不能把moectf的web方向全过了（绝对不是因为想看群主女装）

