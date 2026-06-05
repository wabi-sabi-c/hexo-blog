---
title: HTML a 标签
tags: [My Tools]
categories: [Tools]
permalink: /Tools/HTML a 标签/

---


## 点击链接新开一个标签页，原标签页仍在

<!--more-->

HTML a 标签


```html

<a href="地址" target="_blank" rel="noopener noreferrer"> 名称</a>

```

### 各属性说明


1. **`href="地址"`**
必填，填写跳转链接（网址/锚点/文件路径），点击文字跳转至对应地址。

2. **`target="_blank"`**
点击链接**在新标签页打开页面**；
默认不写是 `_self`：当前页面直接跳转。

3. **`rel="noopener noreferrer"`（安全属性，配合`_blank`标配）**
- `noopener`：禁用新页面通过 `window.opener` 获取原页面对象，**防止安全漏洞、钓鱼窃取页面权限**；
- `noreferrer`：跳转时不向目标网站发送来源页referer信息，隐藏来源地址。

> 规范：只要使用 `target="_blank"`，**推荐固定带上 rel="noopener noreferrer"**。

### 示例


```html

<a href="https://www.baidu.com" target="_blank" rel="noopener noreferrer">打开百度</a>

```
点击「打开百度」→ 新标签页打开百度首页。