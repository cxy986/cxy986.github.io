# 题目名称

- **赛事**：2026 ? CTF
- **分类**：Misc
- **分值**：200


## 一、信息收集
进入页面身份是guest

<img width="1279" height="744" alt="image" src="https://github.com/user-attachments/assets/ef17d59e-3a24-43da-be95-c1868eecf644" />

目标很清晰，拿到internal的内部权限

## 二、漏洞分析

垂直越权

## 三、利用过程

之前在moectf做过类似的题，我直接打开了浏览器开发者工具

<img width="422" height="283" alt="image" src="https://github.com/user-attachments/assets/5fb9f990-5cbf-44ba-9a82-5e86688bcd57" />

发现应用程序里的role居然是可以编辑的，进一步印证了我的猜测

尝试直接修改值为internal，失败

之后查看源码

<img width="778" height="217" alt="image" src="https://github.com/user-attachments/assets/cfe6a01f-63d5-4ac9-bf8e-c66bbe117c4c" />

源码信息泄露，在注释得到了关键信息

个性化预览通过查询参数指定账号：/?name=<账号>
  · 内测演示账号：lain

搜索框输入

<img width="1031" height="125" alt="image" src="https://github.com/user-attachments/assets/95b14ca4-dd73-4272-8173-8c03c59eaa99" />

<img width="2559" height="1408" alt="image" src="https://github.com/user-attachments/assets/6805bc03-c6e9-408f-9022-b2c023338284" />

提示403权限不足，并给出关键提示所需权限为knight，修改应用程序中key-cookie的值为knight

<img width="734" height="214" alt="image" src="https://github.com/user-attachments/assets/1b215fdf-4e3b-493d-8a25-959c361d7a9b" />

f5刷新

  <img width="2551" height="1315" alt="image" src="https://github.com/user-attachments/assets/eba5e56e-029e-4f01-ac09-77864887e44a" />

  再次的到关键提示，客户端不受支持，必须要用期望客户端

  <img width="422" height="192" alt="image" src="https://github.com/user-attachments/assets/c1002e09-523b-4529-b66f-1a8ebcc4e9be" />

  在更多网络条件中关掉默认代理，并填写期望代理

  刷新得到flag

  <img width="1280" height="584" alt="image" src="https://github.com/user-attachments/assets/d1eb114e-f3ab-42af-95bc-e983ba40e319" />




## 四、Payload

纯浏览器打法没啥payload

## 五、Flag

动态flag

## 六、总结 / 踩坑

QA 注释泄露入口与测试账号，权限等级存于客户端 Cookie，客户端类型通过 User-Agent 判定
