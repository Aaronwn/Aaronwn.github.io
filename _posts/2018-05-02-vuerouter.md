---
layout:     post
title:      vue router基础梳理
subtitle:   记录工作中常用vue router知识点
date:       2018-05-02
author:     Aaronwn
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - vue
---


## Vue-router简介

> 简单说 路由就是url和组件的一种映射关系，根据不同的url，加载不同的组件，进而渲染不同的页面效果。

router-link: 是路由入口，该组件用于帮助用户进行导航 ，用 to 属性指定目标地址；

router-view : 是路由的出口，路由匹配到的组件将渲染在这里,即渲染 router-link指向的目标地址。

## 如何创建vue-router
### 步骤
1. 创建组件
2. 配置路由
3. 生成路由实例
4. 将router挂载到根实例
### 完整demo
``` js
import Vue from 'vue'
import App from './App'
import VueRouter from 'vue-router'
// 1.引入组件
import Home from './components/home.vue'
import Detail from './components/detail.vue'
import Detail2 from './components/detail2.vue'
Vue.config.productionTip = false
Vue.use(VueRouter)
// 2.配置路由
const routes = [
  {
    path: '/',
    component: Home
  },
  {
    path: '/detail',
    component: Detail,
    children: [ //嵌套router
      {
        path: 'detail2', //此处不带斜线/
        component: Detail2
      }
    ]
  },
  {
    path: '/detail2',
    name: 'detail2',
    component: Detail2
  },
]
// 3.生成路由实例
const router = new VueRouter({
  routes
})
new Vue({
  el: '#app',
  // 4.挂载到根实例
  router,
  components: { App },
  template: '<App/>'
})
```
**demo效果**
![img](/img/in-post/vue-router-demo1.jpg)

### 编程式导航
![img](/img/in-post/vue-router-js.jpg)

### 命名路由
有时候，通过一个名称来标识一个路由显得更方便一些，特别是在链接一个路由，或者是执行一些跳转的时候。你可以在创建 Router 实例的时候，在 routes 配置中给某个路由设置名称。
```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```

### 路由重定向和别名

#### 重定向
```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})
```
#### 别名
``` js
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```
> 『重定向』的意思是，当用户访问 /a时，URL 将会被替换成 /b，然后匹配路由为 /b，那么『别名』又是什么呢？

/a 的别名是 /b，意味着，当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a，就像用户访问 /a 一样。
## 路由模式

mode 表示路由的配置模式:两种

hash 模式（默认）:使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。（在当前页面的 url 后面加上路径 ）
history 模式: 通过history 完成 URL 跳转。（history模式更美观一些，当然要做一些额外的工作）

```js
//创建路由实例 
const router = new VueRouter({
mode: 'history',//跳转页面
routes
})
```

### 路由的滚动 scrollBehavior 

>它可以让你切换路由时页面随意定位。

``` js
const router = new VueRouter({
  routes,
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到哪个的位置
     return {x:0,y:0} //对于所有路由导航，简单地让页面滚动到顶部
  }
})

```

### 路由懒加载

#### 懒加载定义
懒加载也叫延迟加载，即在需要的时候进行加载，随用随载。

#### 为什么需要懒加载

在单页应用中，如果没有应用懒加载，运用webpack打包后的文件将会异常的大，造成进入首页时，需要加载的内容过多，延时过长，不利于用户体验，而运用懒加载则可以将页面进行划分，需要的时候加载页面，可以有效的分担首页所承担的加载压力，减少首页加载用时

#### 为什么需要异步组件
单页应用的一个核心问题在于一进入页面你需要下载整个打包后的app.js文件并执行。使用异步组件我们可以实现在使用时才加载我们需要的组件脚本。例如我们在路由切换时，切换到对应路由再加载对应的组件脚本，可以提升我们的性能。

#### 如何与webpack配合实现懒加载
1、在webpack配置文件中的output路径配置chunkFilename属性
``` js
output: {
        path: resolve(__dirname, 'dist'),
        filename: options.dev ? '[name].js' : '[name].js?[chunkhash]',
        chunkFilename: 'chunk[id].js?[chunkhash]',
        publicPath: options.dev ? '/assets/' : publicPath
    },
```
chunkFilename路径将会作为组件懒加载的路径
2、配合webpack支持的异步加载方法
- resolve => require([URL], resolve), 支持性好
- () => system.import(URL) , webpack官网上已经声明将逐渐废除, 不推荐使用
- () => import(URL), webpack官网推荐使用, 属于es7范畴, 需要配合babel的syntax-dynamic-import插件使用, 具体使用方法如下：
``` bash
npm install --save-dev babel-core babel-loader babel-plugin-syntax-dynamic-import babel-preset-es2015

use: [{
        loader: 'babel-loader',
        options: {
          presets: [['es2015', {modules: false}]],
          plugins: ['syntax-dynamic-import']
        }
      }]
```
#### 具体实例中实现懒加载
1、路由中配置异步组件
```js
export default new Router({
    routes: [
        {
            mode: 'history',
            path: '/my',
            name: 'my',
            component:  resolve => require(['../page/my/my.vue'], resolve),//懒加载
        },
    ]
})
```
2、实例中配置异步组件
```js
const Foo = () => import('components/foo" './Foo.vue')  

```



