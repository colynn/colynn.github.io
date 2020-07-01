---
layout: post
title: vue 使用中的汇总
subtitle:
tags: [vue, js]
comments: true
---

# Vue 生命周期

[![vue-lifecycle]({{ site.url }}/img/vue/lifecycle.png)]({{ site.url }}/img/vue/lifecycle.png)

# Vue 路由

## 1. Vue 路由跳转方式

### 1. router-link

```
1. 不带参数
 
<router-link :to="{name:'home'}"> 
<router-link :to="{path:'/home'}"> //name,path都行, 建议用name  
// 注意：router-link中链接如果是'/'开始就是从根路由开始，如果开始不带'/'，则从当前路由开始。
 
 
 
2.带参数
 
<router-link :to="{name:'home', params: {id:1}}">  
 
// params传参数 (类似post)
// 路由配置 path: "/home/:id" 或者 path: "/home:id" 
// 不配置path ,第一次可请求,刷新页面id会消失
// 配置path,刷新页面id会保留
 
// html 取参  $route.params.id
// script 取参  this.$route.params.id
 
 
<router-link :to="{name:'home', query: {id:1}}"> 
 
// query传参数 (类似get,url后面会显示参数)
// 路由可不配置
 
// html 取参  $route.query.id
// script 取参  this.$route.query.id


```

### 2. this.$router.push() (函数里面调用)

```
1.  不带参数
 
this.$router.push('/home')
this.$router.push({name:'home'})
this.$router.push({path:'/home'})
 
 
 
2. query传参 
 
this.$router.push({name:'home',query: {id:'1'}})
this.$router.push({path:'/home',query: {id:'1'}})
 
// html 取参  $route.query.id
// script 取参  this.$route.query.id
 
 
 
3. params传参
 
this.$router.push({name:'home',params: {id:'1'}})  // 只能用 name
 
// 路由配置 path: "/home/:id" 或者 path: "/home:id" ,
// 不配置path ,第一次可请求,刷新页面id会消失
// 配置path,刷新页面id会保留
 
// html 取参  $route.params.id
// script 取参  this.$route.params.id
 
 
 
4. query和params区别
query类似 get, 跳转之后页面 url后面会拼接参数,类似?id=1, 非重要性的可以这样传, 密码之类还是用params刷新页面id还在
 
params类似 post, 跳转之后页面 url后面不会拼接参数 , 但是刷新页面id 会消失

```

### 3.  this.$router.replace() (用法同上,push)
### 4.  this.$router.go(n) 

```
this.$router.go(n)
向前或者向后跳转n个页面，n可为正整数或负整数
```


* `this.$router.push` 跳转到指定url路径，并想history栈中添加一个记录，点击后退会返回到上一个页面

* `this.$router.replace`
跳转到指定url路径，但是history栈中不会有记录，点击返回会跳转到上上个页面 (就是直接替换了当前页面)

* `this.$router.go(n)`
向前或者向后跳转n个页面，n可为正整数或负整数


## 2. 动态路由的思考

### 前言
先说下为什么需要动态路由？比如我们想让不同的角色看到不同的菜单及页面，你均通过 `v-show`或是`if`语句，你觉得还够用吗？这个时候动态路由就显得特别重要。


### 如何解决
实现用户角色菜单及页面的控制，需要解决两个问题：
问题1. 根据用户拥有的权限id的集合决定用户能访问哪些前端路由；
问题2. 根据用户拥有的权限id的集合决定用户能看到哪些按钮或者模块。

对于问题1，我们在用户登录后首先获取相应的路由信息，然后通过`vue-router`的`addRouters` 添加至前端路由中即可，但是有个问题，由于用户的信息是存放在 vuex 中，刷新页面时就会导致404。
    * 当然你可以把权限存放至浏览器的 `sessionStorage` 中，每次刷新并不会清空用户权限，但是这样做的话，用户手动改一下 `sessionStorage`就能看到其他页面，不是很完美的方案，而且需要二次存储（vuex/ sessionStorage）;
    * vue 提供了一个`Navigation Guards`的功能，这样页面加载时我们可以先初始化一个空路由（或是默认均开放给用户的路由），紧接校验用户信息，没有的话就再次请求权限，然后通过`addRouters` 添加路由， 目前来看应该是相对完美的方案。

对于问题2，可以添加[自定义的vue命令来控制](https://vuejs.org/v2/guide/custom-directive.html)

```
// custom-directive
Vue.directive('permissionaction', {
    inserted: function(el, binding) {
        const { value } = binding
        const all_permission = '*:*:*'
        const permissions = store.getters && store.getters.permisaction

        if (value && value instanceof Array && value.length > 0) {
            const permissionFlag = value

            const hasPermissions = permissions.some(permission => {
                return all_permission === permission || permissionFlag.includes(permission)
            })

            if (!hasPermissions) {
                el.parentNode && el.parentNode.removeChild(el)
            }
            } else {
                throw new Error(`请设置操作权限标签值`)
            }
        }
    }
})

```

```
// component use
<button v-permissionaction=["permission_id"]>删除</button>
```

### 注意点
1. `vue-router` 提供了`addRouters`来进行添加路由，但是却没有移除路由的相关选项，当切换用户或是变更角色时，可以另外创建一个 router实例来替换之前的实例。 [详见这里-Feature request: replace routes dynamically](https://github.com/vuejs/vue-router/issues/1234#issuecomment-357941465)

2. Vue mathcing priority: the earlier a route is defined, the higher priority it gets, 所以如果你想匹配其他的页面至 /404, 请你一定将 `{ path: '*', redirect: '/404', hidden: true }` 定义至路由的__最后__。


# Vue State Management


# 附录

## 1.Scoped CSS

1. 含义

当 <style>标签有 `scoped`属性时，css 样式仅会应用在当前组件内。 父组件的样式不会影响子组件，然而子组件的根节点均会被父节点及子节点的`scoped`样式影响。这样是为了统一布局，父组件可以定义子组件的根元素。

2. 如何使用

你可以在一个组件内使用`scoped`和`non-scoped`样式：

```
<style>
/* global styles */
</style>

<style scoped>
/* local styles */
</style>

```

3. Deep Selectors

如果你想在一个包含`scoped`样式的selector 可以深度选择， 比如，影响子组件，你可以使用`>>>`选择符： 

```
<style scoped>
.a >>> .b { /* ... */ }
</style>
```

The above will be compiled into:

```
.a[data-v-f3f3eg9] .b { /* ... */ }
```

一些预解释器（像 Sass）, 也许不能正确地解析 `>>>`.在这种情况下，你可以使用 `/deep/` 或是`::v-deep` 选择符来替代。 

4. Dynamically Generated Content

DOM content created with v-html are not affected by scoped styles, but you can still style them using deep selectors.
通过`v-html`创建的 DOM内容，不被`scoped`样式影响， 但是你可以使用`deep selector`改变样式。

## 参考

1. [Scoped CSS](https://vue-loader.vuejs.org/guide/scoped-css.html#deep-selectors)