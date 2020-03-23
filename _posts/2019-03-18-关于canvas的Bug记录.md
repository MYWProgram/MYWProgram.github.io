---
layout: post
title: 关于canvas的Bug记录
subtitle: 导致canvas加载错误的因素
date: 2019-03-18
author: Myw
header-img: img/post-normal-bg.jpg
catalog: true
tags:
    - Bug
    - HTML
---

## canvas 画布小 bug

### 问题描述

使用`display: none;`属性制作的导航栏目，每个导航下面嵌套了需要 JS 加载的一些饼状图（canvas）；从而导致了切换 tab 之后加载错误。并且设置了 active 默认展示的 tab 是不会出现加载错误的，因为 active 对应的 tab 下没有`display: none;`。

### HTML 结构

```html
<div>
  <ul id="myTab" class="nav nav-tabs">
    <li class="active" id="tabFrist">
      <a href="#tab_1" data-toggle="tab">按年级统计</a>
    </li>
    <li>
      <a href="#tab_2" data-toggle="tab">所在地分布统计</a>
    </li>
    <li>
      <a href="#tab_3" data-toggle="tab">按学生属性统计</a>
    </li>
  </ul>
</div>
```

### 解决办法

```js
$("tabFrist")
  .siblings()
  .eq(0)
  .on("click", function() {
    // 选中 id 为 tabFrist 的导航栏，并且选中他的第一个兄弟节点，也就是上面的 tab_2 。
    if (!this.abc) {
      this.abc = this.abc || true;
      setTimeout(function() {
        // 此处为 canvas 的初始化加载函数。
        chart3Handler();
      }, 500);
    }
  });
$("tabFrist")
  .siblings()
  .eq(1)
  .on("click", function() {
    if (!this.abcd) {
      this.abcd = this.abcd || true;
      setTimeout(function() {
        // 对应的 canvas 所需的加载函数。
        chart3oneHandler();
        chart3twoHandler();
        chart3thrHandler();
        chart3fouHandler();
        chart3fivHandler();
        chart3sixHandler();
      }, 500);
    }
  });
```
