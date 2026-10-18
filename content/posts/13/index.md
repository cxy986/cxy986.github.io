---
title: "中国人能飞，摩斯密码"
date: 2026-10-18
draft: false
---

绷不住啊，谁懂博客快写完的时候下楼拿了个外卖、

结果上来发现没保存的博客没了

就这样吧

不想写了，这题就是拼flag

伪加密获得part1，摩斯密码获得part2，音频音频元数据ID3v2.3 标签里有part3

就这吧，累了，毁灭吧


ID3v2.3 的核心文件结构

与记录在文件末尾且有 128 字节大小限制的 ID3v1 不同，ID3v2.3 记录在 MP3 文件的首部，采用变长架构，理论上可以存放无限大小的数据（如高清专辑图片）。它由以下三部分组成：
1. 标签头（Tag Header，固定 10 字节）
	• 前 3 字节固定为字符串 "ID3"。
	• 紧接着 2 字节为版本号（v2.3 对应 $03 00）。
	• 包含标志位（Flags）和整个标签的大小（Size）。
2. 标签帧（Frames，数量不限）
	• 每一个标签帧代表一项具体信息（如歌名、歌手）。
	• 每个帧由一个 4 字节的帧 ID（如 TIT2 代表标题）和实际内容组成。
3. 填充区（Padding，可选）
	• 在文件头留出的一段空白区域，方便用户以后修改标签时直接写入，而无需重新重写整个音频文件。

🏷️ 常见 ID3v2.3 标签帧 ID 对照表

ID3v2.3 使用 4 个大写字母作为标识符（Frame ID）来区分不同的元数据：

帧 ID	代表的含义	作用
TIT2	Title/Songname	歌曲标题（歌名）
TPE1	Lead performer	主要艺术家 / 歌手
TALB	Album	专辑名称
TRCK	Track number	音轨编号（如：01/12）
TYER	Year	录制年份（注：v2.4 中被改为 TDRC）
TCON	Content type	音乐流派（Genre）
APIC	Attached picture	专辑封面图片（可内嵌 JPEG/PNG）
COMM	Comments	评论 / 备注信息
USLT	Unsynchronized lyric	内嵌歌词
