# ezFilter 题解（完整复现版）

## 1. 题目信息

- 题目：`ezFilter`
- 类型：Misc / 动态环境
- 目标地址：

  ```text
  http://8000-c5b5b449-456b-4243-9398-0dd661eef248.challenge.ctfplus.cn/
  ```

- 最终 flag：

  ```text
  QCTF{da15233c-1292-41bb-9ebc-f8fb0d23308e}
  ```

这道题的关键不在于猜 flag，而在于正确理解 `/probe` 返回的三种状态，并用这些状态逐步排除不可能的文件编号。题目还加了一个动态 challenge，用来限制请求频率；challenge 本身可以根据服务器返回的规则本地穷举求解。

---

## 2. 首页真实信息

直接访问根路径 `/`，页面返回了如下帮助信息：

```text
ez-filter -- Filter the flag

一个文件目录里有 1024 个文本文件，编号 0 到 1023。
每个文件形如 QCTF{<guid>}，但只有一个是真 flag。

接口
----
GET /
    本帮助。

GET /probe?lo=<int>&hi=<int>
    询问闭区间 [lo, hi]。回答三种状态之一：

      hot    flag 的编号落在区间中部
      warm   flag 的编号在区间内，但不在中部
      cold   flag 的编号不在区间内

    可以通过以下公式，计算出中部对应的区间：

        length = hi - lo + 1
        mid    = lo + length // 2
        half   = (length - 1) // 4
        middle = [mid - half, mid + half]

GET /entry?index=<int>
    取回某个编号的文本内容。index 必须在 0 到 1023 之间。
```

这里有两个容易误解的点：

1. `/probe` 不是普通的“询问某一半是否包含 flag”。它有 `hot`、`warm`、`cold` 三种结果。
2. 全范围返回 `hot`，也不能说明 flag 是某一个固定位置。它只说明 flag 在这个查询区间计算出来的 `middle` 中。

题目提示“至少，它能告诉你大概在哪里”，对应的就是利用这三个状态做区间过滤。

---

## 3. 先验证接口行为

### 3.1 全范围探测

根据首页，合法编号范围是 `0..1023`，因此先访问：

```text
/probe?lo=0&hi=1023
```

真实响应：

```json
{
  "result": "hot",
  "lo": 0,
  "hi": 1023,
  "length": 1024
}
```

计算过程如下：

```text
length = 1023 - 0 + 1 = 1024
mid    = 0 + 1024 // 2 = 512
half   = (1024 - 1) // 4 = 255
middle = [512 - 255, 512 + 255] = [257, 767]
```

所以这次 `hot` 的含义是：flag 编号在 `257..767` 之间。为了后续实现统一，程序可以把初始候选集合设成 `0..1023`，然后把每次服务器返回的状态转换成候选集合。

### 3.2 越界请求不能使用

我实际测试了：

```text
/probe?lo=-256&hi=767
```

服务器返回：

```json
{
  "error": "out_of_range",
  "message": "区间必须落在 0..1023 之间。",
  "slots": 1024
}
```

因此不能通过构造负数区间或超过 1023 的区间来改变过滤逻辑，所有探测都必须使用合法区间。

### 3.3 读取 entry 时出现 challenge

访问：

```text
/entry?index=0
```

服务器返回 HTTP 401，并给出类似下面的 JSON：

```json
{
  "error": "challenge_required",
  "message": "额度用完了。先解 challenge：把答案放进 X-EZ-Auth 请求头，然后重发同一个请求。",
  "auth_header": "X-EZ-Auth",
  "challenge": {
    "id": "2bc6c0cd25d009f3",
    "algorithm": "sha256",
    "separator": "-",
    "pick": 5,
    "word_count": 16,
    "words": [
      "frooskrouck",
      "froxchees",
      "zhiexbuen",
      "snaizuex",
      "krudsoom",
      "chautgluep",
      "cuecktrual",
      "preapkys",
      "pleerbleest",
      "smoilflyst",
      "wuastheel",
      "troupchiash",
      "koinbruas",
      "zuamyysclock",
      "zuesthiezye",
      "chagkuck"
    ],
    "target": "bb7eb7955d5c952acc3d5100236bcb57cc17c80e418184dcd24dd5dd9a36e4a2"
  }
}
```

服务器要求：从 `words` 中选出恰好 5 个不同单词，使用 `-` 拼接，顺序也参与计算，使得拼接字符串的摘要等于 `target`。

例如候选形式是：

```text
wordA-wordB-wordC-wordD-wordE
```

注意这里是“排列”而不是“组合”：

- 选中哪 5 个单词很重要；
- 这 5 个单词的顺序也很重要；
- 每个单词最多使用一次。

当 `word_count=16`、`pick=5` 时，最多需要测试：

```text
P(16, 5) = 16 × 15 × 14 × 13 × 12 = 524160
```

这对本地 Python 穷举是可行的，而且不需要向服务器发送这些猜测。

另一个实际观察到的细节是：`algorithm` 并不固定。我在一次运行中见到过 `sha256`，也见到过 `md5`。因此不能把算法写死成 SHA-256，应该读取服务器返回的算法名称，再调用 Python 的 `hashlib.new(algorithm, data)`。

---

## 4. `/probe` 三态如何转成候选集合

对于一次请求 `[lo, hi]`，先按照题目公式计算 `middle`。

假设当前还可能是集合 `C`，那么：

```text
hot  = C ∩ middle
warm = C ∩ ([lo, hi] - middle)
cold = C - [lo, hi]
```

收到服务器结果后，只保留对应集合：

- 返回 `hot`：flag 在本次查询区间的中部；
- 返回 `warm`：flag 在本次查询区间内，但在中部之外；
- 返回 `cold`：flag 不在本次查询区间内。

这样每次查询都会删除一部分不可能的编号。

### 4.1 为什么不能把 warm 当成“区间左半边”

`warm` 不是固定的左侧，也不是固定的右侧，而是：

```text
整个 [lo, hi] - middle
```

如果查询区间没有覆盖整个候选范围，warm 可能包含区间中部左侧的编号；如果区间和候选集合的位置关系不同，保留下来的形状也会不同。因此实现时应直接按照公式计算集合，不应凭感觉写成普通二分。

---

## 5. 实际定位过程

下面是强化版脚本最终成功运行时的真实探测记录。`理论分组`是脚本严格按照题面公式计算出的三组大小，服务器返回的 `result` 决定实际保留哪一组。

| 步骤 | 查询区间 | 返回结果 | 保留后的候选数 | 结论 |
|---:|---:|---|---:|---|
| 0 | `[0, 1023]` | `hot` | 511 | 全范围的 middle 是 257..767 |
| 1 | `[257, 597]` | `cold` | 170 | 排除 257..597，保留 598..767 |
| 2 | `[598, 710]` | `warm` | 56 | 在查询区间内，但不在其 middle |
| 3 | `[579, 691]` | `warm` | 18 | 继续按三态规则过滤 |
| 4 | `[573, 685]` | `warm` | 6 | 候选缩小到 6 个 |
| 5 | `[571, 683]` | `cold` | 2 | 排除 571..683 |
| 6 | `[684, 684]` | `cold` | 1 | 排除 684，唯一候选为 685 |
| 确认 | `[685, 685]` | `hot` | 1 | 单点确认通过 |

除全范围基线 `[0,1023]` 外，后续自动选择的查询区间都采用奇数长度。这样做不是猜测服务端实现，而是让自动选择器只使用题面公式在边界上更直观、便于人工复核的区间；实际每次响应的 `lo`、`hi`、`length` 也都由脚本校验。

### 第 0 步：查询 `[0,1023]`

```text
length = 1023 - 0 + 1 = 1024
mid    = 0 + 1024 // 2 = 512
half   = (1024 - 1) // 4 = 255
middle = [257, 767]
```

服务器返回 `hot`，因此初始候选直接变成：

```text
C = {257, 258, ..., 767}
```

这次基线结果会被真正使用，不是只打印后丢弃。

### 第 1 步：查询 `[257,597]`

```text
length = 341
mid    = 257 + 341 // 2 = 427
half   = (341 - 1) // 4 = 85
middle = [342,512]
```

因为当前候选在 257..767 中，三类候选大小为：

```text
hot  = 342..512，共 171 个
warm = 257..341、513..597，共 170 个
cold = 598..767，共 170 个
```

服务器返回 `cold`，所以只保留：

```text
C = {598, 599, ..., 767}
```

### 第 2 到第 4 步

脚本用前缀和精确计算每个合法区间的三组候选数量，选择最大组尽可能小的区间。真实路径是：

```text
[598,710] -> warm -> 56 个
[579,691] -> warm -> 18 个
[573,685] -> warm -> 6 个
```

注意第二个区间的左端小于当前候选最小值 598，这是合法的：查询区间只要满足 `0 <= lo <= hi <= 1023` 即可。区间外的候选自然属于 `cold`；服务器返回 `warm` 时，脚本只保留区间内且不在 middle 的那部分。

### 第 5 步：查询 `[571,683]`

服务器返回 `cold`。由于当前候选只有 6 个，结果排除掉区间内的候选后，只剩 2 个编号。

### 第 6 步和单点确认

查询 `[684,684]` 返回 `cold`，因此 684 被排除，只剩：

```text
index = 685
```

随后再查询单点 `[685,685]`：

```text
length = 1
mid    = 685 + 1 // 2 = 685
half   = 0
middle = [685,685]
```

服务器返回 `hot`。长度为 1 时 middle 只能是该点，因此这是不依赖区间边界解释的最终确认。

---

## 6. challenge 的正确处理方式

脚本在收到 HTTP 401 后做以下事情：

1. 读取 JSON 中的 `algorithm`、`separator`、`pick`、`words` 和 `target`；
2. 使用 `itertools.permutations(words, pick)` 枚举所有不重复排列；
3. 用 separator 拼接当前排列；
4. 使用动态算法计算摘要；
5. 摘要等于 target 时，把拼接结果放入 `X-EZ-Auth`；
6. 重发刚才完全相同的请求。

伪代码如下：

```python
for selected in itertools.permutations(words, pick):
    candidate = separator.join(selected)
    digest = hashlib.new(algorithm, candidate.encode("utf-8")).hexdigest()
    if digest == target:
        auth = candidate
        break
```

### 为什么必须保存 Cookie

题目是动态环境，challenge 往往和当前 HTTP 会话绑定。服务器返回 401 时，可能同时设置了 Cookie 或在会话中记录了当前待解 challenge。

如果每次请求都用一个全新的 HTTP 客户端，虽然本地算出了正确字符串，服务器也可能返回：

```json
{
  "error": "no_pending_challenge",
  "message": "当前没有待解的 challenge。"
}
```

因此脚本使用：

```python
cookie_jar = http.cookiejar.CookieJar()
opener = build_opener(HTTPCookieProcessor(cookie_jar))
```

后续所有请求都通过同一个 `opener` 发送，从而保持 Cookie 会话。

实际运行中还观察到少量 `403 no_pending_challenge`，随后同一请求重试又成功。这说明 challenge 状态可能在服务端被消费或更新。脚本遇到这个特定错误时会清空旧的 `X-EZ-Auth`，退避后重发原请求；如果服务器重新给出 401，就自动求解新 challenge。

---

## 7. 最终读取 flag

唯一候选确定为 685 后，访问：

```text
/entry?index=685
```

真实成功响应不是 JSON，而是纯文本：

```text
QCTF{da15233c-1292-41bb-9ebc-f8fb0d23308e}
```

因此读取程序不能强制所有成功响应都用 `json.loads()`；对于 `/entry` 的纯文本响应，需要保留原始字符串。

---

## 8. 可复现脚本

脚本文件：[`ezfilter_solve.py`](./ezfilter_solve.py)

只使用 Python 标准库，包含：

- HTTP 请求和超时；
- Cookie 会话；
- 401 challenge 自动求解；
- `md5`、`sha256` 等动态摘要算法；
- `/probe` 三态候选过滤；
- `/entry` 纯文本返回处理；
- 重试、退避、控制请求频率；
- 控制台和日志文件输出。

运行命令：

```text
python ezfilter_solve.py http://8000-c5b5b449-456b-4243-9398-0dd661eef248.challenge.ctfplus.cn/ --log ezfilter_run_hardened.log
```

成功结束时会输出类似：

```text
===== 结果 =====
index = 685
entry = {"raw": "QCTF{da15233c-1292-41bb-9ebc-f8fb0d23308e}"}
history = [{"step": 0, "lo": 0, "hi": 1023, "result": "hot", "remaining": 511}, ...]
requests = 13
```

这里的 `requests` 是实际 HTTP 发送次数，包含 challenge 失败后的重试；challenge 的本地排列计算不向服务器发送请求。完整 HTTP 日志保存在：[`ezfilter_run_hardened.log`](./ezfilter_run_hardened.log)。

---

## 9. 知识点总结

### 9.1 三态信息可以当作多叉过滤器

`hot`、`warm`、`cold` 每次提供的不是精确位置，而是对当前候选集合的分类。只要准确实现分类公式，反复请求就可以把 1024 个候选压缩到 1 个。

### 9.2 题目给出的数学定义就是协议

`mid` 和 `half` 的计算使用整数除法，边界还是闭区间。实现时必须严格按题面写：

```python
length = hi - lo + 1
mid = lo + length // 2
half = (length - 1) // 4
middle = range(mid - half, mid + half + 1)
```

最后的 `+1` 很重要，因为 Python 的 `range` 右端不包含。

### 9.3 Challenge 是本地哈希前像搜索

服务器给出摘要目标和有限单词表，要求寻找一个满足摘要的输入。这不是破解哈希算法，而是在有限候选空间中穷举。因为候选空间只有 `16P5=524160`，标准库 Python 足够完成。

### 9.4 排列和组合不同

如果只枚举 `combinations(words, 5)`，会漏掉绝大多数顺序。题目明确说“任意顺序”，而摘要输入字符串的顺序会改变，所以应使用 `permutations`。

### 9.5 HTTP 状态码和响应格式都要实际观察

本题至少出现了：

- `200`：成功返回 JSON 或 entry 文本；
- `401`：需要 challenge；
- `403`：challenge 答案状态不匹配或已没有待解 challenge。

并且 `/probe` 成功时返回 JSON，而 `/entry` 成功时直接返回文本。不能只根据接口名称猜测格式。

---

## 10. Flag

```text
QCTF{da15233c-1292-41bb-9ebc-f8fb0d23308e}
```

