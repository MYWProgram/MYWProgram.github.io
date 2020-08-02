---
layout: post
title: Vue-Router介绍、问题记录
subtitle: 模式介绍、路由传参、路由监听
date: 2019-05-30
author: Myw
header-img: img/post-normal-bg.jpg
catalog: true
tags:
    - Vue
---

## 前言

路由是在单页面中代替多页面跳转的一种实现，Vue-Router 分为三种模式；下面就介绍一下三种模式，并记录我在开发中踩过的坑。

## 三种模式

### hash

早期的前端路由就是基于`location.hash`来实现的，这个 API 就是 URL 中 # 后面的值。

简单了解一下 hash 路由实现原理：

- URL 中的 hash 值只是客户端的一种状态，也就是说向服务端发送请求时不会包含 hash 值部分。
- hash 值的改变会在浏览器中增加记录，因此可以通过记录来进行前进后退操作。
- a 标签设定 href 属性会改变 hash 值，JS `location.hash`也有同样的效果。
- 使用`hashchange`可以监听 hash 值的改变。

了解了上面的实现原理，也就不难明白为何 Vue-Router 中使用 hash 模式的路由会在 URL 中带上 # 以及后面的一串值了。

### history

HTML5 提供了 history 这个 API 来实现 URL 的变化，在了解之前先看看下面两个 history 重要的 API：

- `history.pushState()`：在浏览器的浏览历史中新增一个浏览记录。
- `history.replaceState()`：在浏览器的浏览历史中修改当前页面的记录。

再看 history 路由的实现原理：

- 使用`history.pushState()`和`history.replaceState()`两个 API 来操作 URL 变化。
- 可以使用`popstate`来监听 URL 的变化。
- 不论是新增还是替换浏览记录，都不会触发`popstate`事件，所以需要手动操作页面跳转。

由于可以直接操作（新增、替换）浏览记录，不再像 hash 模式一样只能操作 hash 值，使得 URL 更加自定义化。

Vue-Router 中 history 模式正是采用了这一特性，URL 地址中就不会有 # 出现。

**但是在 Vue 开发中使用 history 路由模式还需要服务端配合；因为此时的 URL 每次请求将会把完整地址发送到服务端，服务端进而决定返回对应的渲染部分。**

### abstract

支持所有 JavaScript 运行环境（就包括浏览器和 Node.js 服务器端）；如果发现没有浏览器的 API ，路由会自动进入这个模式。

## 路由传参

在了解传参的方式之前，先来看看 Vue-Router 中几个属性：

- 有两个属性来控制跳转的路由落地点，分别是`path`和`name`。
- 有两个属性来携带每次路由跳转时候的参数，分别是`query`和`params`。

**`$router`是 Vue 的路由实例，而`$route`是对应的路由对象，路由跳转的方法在实例上，路由传递的参数在对象上。**

### params 传参

#### 普通路由配置

```js
// 路由配置。
{
  path: "/about",
  name: "About",
  component: About
}
// 路由跳转，传递参数。
this.$router.push({
  name: "About",
  params: {
    id: 1
  }
});
```

这种做法的好处是传递的参数不会拼接在 URL 中，看起来美观一点；但是缺点就是**刷新当前页面**之后，会导致**参数丢失**而获取不到的问题。

#### 动态路由配置

```js
// 在对应路由的 path 处，加上传递的参数。
{
  path: "/about/:id",
  name: "About",
  component: About
}
// 路由跳转，传递参数。
this.$router.push({
  name: "About",
  params: {
    id: 1
  }
});
```

这种方式和上面的相比，刷新页面不会丢失参数；但是 URL 后面会加上"/1(参数值)"这样的一串值。

#### path 定位落地点

```js
// 动态路由配置。
{
  path: "/about/:id",
  name: "About",
  component: About
}
// 使用 path 定位路由落地点，需要加上对应的实际参数。
this.$router.push({
  path: "/about/1"
});
```

这种方式和上面一种效果一致，但是写法不同。

**上面三种方式都可以用下面的方式获取参数：**

- HTML 中使用插值：`$route.psrams.id`。
- JS 中：`this.$route.params.id`。

### query 传参

```js
{
  path: "/about",
  name: "About",
  component: About
}
// 使用 query 携带参数。
this.$router.push({
  // 这里使用 name 或者 path 定位路由落地点都可以。
  name: "About",
  query: {
    id: 1
  }
});
```

这种方式同样不会有刷新页面参数丢失的问题，但是会在 URL 结尾跟上"?id=1(参数值)"这样一串值。

同样可以通过`$route.query.id`和`this.$route.query.id`来获取参数。

### 验证路由参数

有时候会使用路由传递多个参数，这时希望验证参数的格式，可以使用动态路由配置，在 path 中对应参数后加上正则表达式：

```js
{
  path: "/about/:id(\\d+)/:color(/^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$/)",
  name: "About",
  component: About
}
```

上面的方式加入了正则来对参数进行验证；就只能传递 id 为数字，color 为 16 进制颜色；否则将不能正确获取参数。

## 新窗口跳转

`this.$touter.push()`方法只能在浏览器当前 tab 页进行跳转，有时候需要打开新窗口并携带参数，可以使用下面这个 API:

```js
let routeData = this.$router.resolve({ path: '/xxx', query: {  id: id } });
window.open(routeData.href, '_blank');
```

## 路由监听

在开发中常常遇到根据路由的变化来实时地进行一些操作的需求，可以使用 watch 来监听路由对象完成这一实现。

### 使用 watch

举个例子，导航栏点击了对应的 item，刷新之后高亮的 item 不变：

```js
// 在 data 中存储了 currentitem 用于判断当前 item 的高亮样式。
// 有名为 Home、About 两个路由配置，使用两个路由做了导航栏。
export default {
  watch: {
    // 注意：这里监听的是路由对象。
    $route(to) {
      // 这里加一层安全判断，防止同组件多个导航的干扰。
      if (["Home", "About"].includes(to.name)) {
        this.currentItem = to.name;
      }
    }
  }
};
```

### 使用导航守卫

还是上面的例子，使用路由提供的钩子函数来解决：

```js
// 配置路由独享守卫。
{
  path: "/about/:id",
  name: "About",
  component: About,
  beforeEnter: (to, from, next) => {
    if (["Home", "About"].includes(to.name)) {
      window.sessionStorage.setItem("activeNavItem", to.name);
    }
    next();
  }
}
// 导航页面获取。
export default {
  data() {
    return {
      currentItem: window.sessionStorage.getItem("activeNavItem") || "Home"
    }
  }
}
```

当然还可以使用组件内的守卫：

```js
export default {
  beforeRouteEnter(to, from, next) {
    if (["Home", "About"].includes(to.name)) {
      window.sessionStorage.setItem("activeNavItem", to.name);
    }
    next();
  }
}
```

使用路由提供的钩子，需要注意以下两点：

1. 钩子函数内的三个参数不要忘记写。
2. 函数结尾需要调用`next()`，否则路由将会被拦截不再进行跳转。

## 后记

本篇文章会持续记录我对 Vue-Router 的学习，以及开发中遇到的相关问题。
