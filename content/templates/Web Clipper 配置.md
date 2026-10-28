---
type: guide
status: active
created: 2026-10-28
kind: ever
last_reviewed: 2026-10-28
---
# 浏览器剪藏（Obsidian Web Clipper）配置

给 [[知识库使用指南]] 配套的浏览器端配置。装一次，之后看到任何网页/公众号都是两次点击进库。

## 1. 安装

- Chrome / Edge：应用商店搜 **Obsidian Web Clipper**（作者 Obsidian.md）安装；
- Safari / iOS：App Store 搜 **Obsidian Web Clipper**；
- 装好后点扩展图标 → 设置 → 把 Vault 选成 `my-obsidian`。

## 2. 新建模板「知识库剪藏」

设置 → Templates → New template，然后逐项粘：

**Name**
```
知识库剪藏
```

**Note name**（文件名）
```
{{date|date:"YYYYMMDD"}} {{title|safe_name}}
```

**Note location**（保存目录）
```
-1. 捕获
```

**Properties**（属性，一行一个）
```
type
capture
status
inbox
source
{{url}}
author
{{author}}
created
{{date|date:"YYYY-MM-DD"}}
kind
tech
review_after
{{date|date:"YYYY-MM-DD"|add_days:90}}
last_reviewed
{{date|date:"YYYY-MM-DD"}}
```

> `add_days` 不是官方内置函数。官方支持 `{{date|date:"YYYY-MM-DD"}}`，
> 如果 `review_after` 填不上，就先留空，写完这篇在 Obsidian 里跑一次
> `python scripts/kb_review.py --apply` 会自动补上。

**Note content**（正文模板）
```
# {{title}}

> 来源：{{url}}
> 作者：{{author}} · 抓取于 {{date|date:"YYYY-MM-DD HH:mm"}}
> 我为什么存它：

## 原文要点

{{content}}

## 我的判断

- 可信度：一手经验 / 官方文档 / 二手转述
- 适用边界：
- 三个月后还成立吗：

## 连接

- 主题导航：[[]]
- 相关笔记：[[]]
```

## 3. 用

- 网页上按扩展快捷键（默认 `Alt+Shift+O`）→ 选「知识库剪藏」→ 保存；
- 微信公众号文章在电脑浏览器打开后同样可剪；手机微信里「用浏览器打开」再剪；
- 剪完的内容在 `-1. 捕获`，用 [[知识关联体检]] 的「收件箱积压」盯着别让它堆。
