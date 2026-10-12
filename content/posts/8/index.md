---
title: "目录扫描"
date: 2026-10-12
draft: false
---

# 信息收集
<img width="604" height="665" alt="image" src="https://github.com/user-attachments/assets/05b3efb6-0d84-4a0a-9446-32bcfede217b" />

简单web题，题干关键信息，扫目录

打开容器，看页面关键信息，有个输入框和俩按钮一个请求批准，一个检查

再看前端js

两个接口都是 POST。

两个请求都发送 textarea 中的原始字符串 body()

Checkout 使用 X-Approval 请求头传审批值。

前端没有把文本框内容解析成 JS 对象再 JSON.stringify；因此重复 JSON 键会原样送到服务端。

# 利用

既然题上明确给出扫目录，那就开扫吧

没有目录扫描工具，手操吧，常见的几个

/
 /src
 /source
 /app.py
 /server.py
 /main.py
 /debug
 /backup
 /robots.txt

 这次在/src的时候发现的重要源码

 明显的出现了两次flag

 <img width="1279" height="442" alt="image" src="https://github.com/user-attachments/assets/cdce353e-9e60-45b2-9bd4-cedba7ede25e" />

 可惜都不是真正的flag还要再看看代码

 重要发现：

 def audit_pairs(pairs):
    result = {}
    for key, value in pairs:
        if key not in result:     # ← 重复键保留【第一个】
            result[key] = value
    return result


def audit_decode(raw):
    value = json.loads(raw.decode("utf-8"), object_pairs_hook=audit_pairs)    # 别用默认的，用我自定义的
    if not isinstance(value, dict):
        raise ValueError("top-level JSON must be an object")
    return value


def checkout_decode(raw):
    value = json.loads(raw.decode("utf-8"))    # ← 原生解析，重复键保留【最后一个】
    if not isinstance(value, dict):
        raise ValueError("top-level JSON must be an object")
    return value

这就是关键漏洞，“两个人看同一文章看出了两个意思”

这段代码定义了三个函数

audit_decode —— 用来审批和盖章的
checkout_decode —— 用来真正下单的
audit_pairs —— 自己做的过滤器函数（看到 role = customer → 查小本子：没有 role → 抄下来。小本子现在有 role = customer
看到 role = admin → 查小本子：已经有 role 了 → 跳过，不覆盖）

checkout_decode是下单函数，而他又会把后写的把先写的覆盖掉

这是 Python 的既有规则
默认 json.loads：重复的键，后面的覆盖前面的（保留最后一个）
用了 object_pairs_hook：怎么处理完全由你自己写的函数决定。这道题写成了"先来的算数"（保留第一个）

所以在盖章的时候第一个值赢了，但如果有第二个值的话会覆盖掉第一个值，也就是买商品的时候会按第二个值的身份给你发

也就是先用规则 A（第一个值赢）读出审批单，拿去验证你的签名对不对
再用规则 B（最后一个值赢）重新读同一段文字，拿这个结果去下单

最后又有关键
order = checkout_decode(raw)                                    #  规则 B 读出来的订单
if order.get("product") == "flag" and order.get("role") == "admin":
    return jsonify({"product": "flag", "value": FLAG})          #  flag 只在这里返回

所以身份是"admin",商品是“flag”

# 构造palyload

{"account":"guest","product":"sticker","role":"customer","quantity":1,"role":"admin","product":"flag"}

#  获取flag

<img width="743" height="423" alt="image" src="https://github.com/user-attachments/assets/13f23971-ceca-4cca-b3b1-665e7a692466" />


总结：本题的核心不是猜 flag 或大量扫描目录，而是从题面识别“审批 → 结算”流程，查看前端确认 body 可控，再读取 /src 发现审计解析保留重复键的第一个值、结算解析保留最后一个值。把合法值写在前、目标值写在后，取得合法审批并用同一 body 结算，即可复现新容器的 flag 分支。

 
