---
layout: post
title: 关于inline-block元素的间距
subtitle: 怎样消除行内块元素的间距
date: 2019-03-18
author: Myw
header-img: img/post-normal-bg.jpg
catalog: true
tags:
    - Bug
    - HTML
    - CSS
---

## 现象

`inline-block` 元素标签（无论本身就是行内块元素或者通过 CSS 设置的行内块元素）换行或者空格分隔情况下会有间距。

## 原因

英文中每个单词都会有分隔符，所以自动会给行内块元素加上一个间隙；项目打包上线之后这些间隙会消除，因为打包的代码会进行压缩。

> 代码如下：

```html
<div class="space">
  <a href="#">XXX</a>
  <a href="#">XXX</a>
  <a href="#">XXX</a>
</div>
```

> 效果如下图：

![inline-block元素间隙](https://tva1.sinaimg.cn/large/00831rSTgy1gczgd7npfcj309i060weg.jpg)

## 修改 HTML 结构

```html
<!-- 1 -->
<div><a href="#">XXX</a><a href="#">XXX</a><a href="#">XXX</a></div>

<!-- 2 -->
<div>
  <a href="#">XXX</a
  ><!--
  --><a href="#">XXX</a
  ><!--
  --><a href="#">XXX</a>
</div>

<!-- 3 -->
<div>
  <a href="#">XXX</a
  ><a href="#">XXX</a
  ><a href="#">XXX</a>
</div>
```

### 使用 margin

给`inline-block`子元素`margin`属性设置一个负值，例如当文字大小为 12px 时，对应为 -3px。这种方法不太可靠，因为不同字体大小设置不一，并且项目上线之后会有元素重叠的现象。

```css
.space a {
  display: inline-block;
  margin-right: -3px;
}
```

### 设置字体大小

设置父元素`font-size: 0px;`，这个方法可以解决大多数浏览器，也是移动端最好的解决方式；同时子元素也可以重新设置字体大小，不会影响。

```css
.space {
  font-size: 0;
  -webkit-text-size-adjust: none; /* Chrome 浏览器中加上消除字体大小限制，新版本的 Chrome 浏览器已经取消了字体大小的限制。 */
}
.space a {
  font-size: 12px;
}
```

### 设置字符间距

设置父元素`letter-spacing: -3px;`，但是这样会让需要消除间隙的`inline-block`元素字体变小，其英文文本的子元素出现重叠问题，所以子元素需要添加`letter-spacing: 0px;`。

当然，`letter-spacing`换成`word-spacing`也可以解决。

> 效果如下图：

![字体变小、文本重叠](https://tva1.sinaimg.cn/large/00831rSTgy1gczgbi9ehdj30cs07u748.jpg)

```html
<div class="space">
  <a href="#">XXX</a>
  <a href="#">XXX</a>
  <a href="#">XXX</a>
  <span>This is a text</span>
</div>
```

```css
.space {
  letter-spacing: -3px;
}
.space a {
  letter-spacing: 0px;
}
.space span {
  letter-spacing: 0px;
}
```

> 修改后效果如下：

![字体正常、文本间距正常](https://tva1.sinaimg.cn/large/00831rSTgy1gczgd7xfuzj30ed088jrf.jpg)
