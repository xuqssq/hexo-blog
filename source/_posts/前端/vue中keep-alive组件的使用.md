---
title: vue中keep-alive组件的使用
toc:
toc_number:
tags:
  - Vue
abbrlink: 6962
date: 2020-06-30 11:43:08
categories: 前端
description:
katex:
aside:
aplayer:
highlight_shrink:
---

## 前言

&emsp; 在开发中经常有从列表跳到详情页，然后返回详情页的时候需要缓存列表页的状态（比如滚动位置信息），这个时候就需要保存状态，要缓存状态；vue 里提供了 keep-alive 组件用来缓存状态。 &emsp; 可以用以下几种方案解决问题；

## 一、利用 meta 标签

直接上代码, 1、首先在路由中的 meta 标签中记录 keepAlive 的属性为 true

```js
    path: '/classify',
    name: 'classify',
    component: () => import('@/views/classify/classify.vue'),
    meta: {
      title: '雷石淘券券',
      keepAlive: true
    }
  },
```

2、在创建 router 实例的时候加上 scrollBehavior 方法

```js
export default new Router({
  mode: "history",
  base: process.env.BASE_URL,
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition;
    } else {
      return {
        x: 0,
        y: 0
      };
    }
  }
});
```

3/在需要缓存的 router-view 组件上包裹 keep-alive 组件

```vue
<keep-alive>
   <router-view v-if='$route.meta.keepAlive'></router-view>
</keep-alive>
<router-view v-if="!$route.meta.keepAlive"></router-view>
```

4、由于有些情况下需要缓存有些情况下不需要缓存，可以在缓存的组件里的路由钩子函数中做判断

```js
beforeRouteLeave (to, from, next) {
    this.loading = true
    if (to.path === '/goods_detail') {
      from.meta.keepAlive = true
    } else {
      from.meta.keepAlive = false
     // this.$destroy()
    }
    next()
  },
```

&emsp;支持可以支持组件的缓存，但是这种方法有 bug，首先第一次打开页面的时候并不缓存，即第一次从列表页跳到详情页，再回来并没有缓存，后面在进入详情页才会被缓存 &emsp;并且只会缓存第一次进入的状态，不会重新请求数据，如果当页面 A 选中一个分类跳到 B 页面，再从 B 列表页面跳往详情页，此时会缓存这个状态，并且以后再从 A 页面的其他分类跳到 B 页面都不会重新被缓存，以至于每次从详情页返回 B 页面都会跳第一次缓存的状态；当你的项目只有一种状态需要缓存，可以考虑使用这种方法

## 二、 使用 include、exclude 属性和 beforeRouteEnter 钩子函数

&emsp;首先介绍一下 include 和 exclude vue 文档([https://cn.vuejs.org/v2/api/](https://cn.vuejs.org/v2/api/#keep-alive)) 是在 vue2.0 以后新增的属性 include 是需要缓存的组件； exclude 是除了某些组件都缓存； include 和 exclude 属性允许组件有条件地缓存。二者都可以用逗号分隔字符串、正则表达式或一个数组来表示：

```vue
<!-- 逗号分隔字符串 -->
<keep-alive include="a,b">
  <component :is="view"></component>
</keep-alive>

<!-- 正则表达式 (使用 `v-bind`) -->
<keep-alive :include="/a|b/">
  <component :is="view"></component>
</keep-alive>

<!-- 数组 (使用 `v-bind`) -->
<keep-alive :include="['a', 'b']">
  <component :is="view"></component>
</keep-alive>
```

&emsp;匹配首先检查组件自身的 name 选项，如果 name 选项不可用，则匹配它的局部注册名称 (父组件 components 选项的键值)。匿名组件不能被匹配。

&emsp;max 只在 2.5.0 新增，最多可以缓存多少组件实例。一旦这个数字达到了，在新实例被创建之前，已缓存组件中最久没有被访问的实例会被销毁掉。

```vue
<keep-alive :max="10">
  <component :is="view"></component>
</keep-alive>
```

##### activated 与 deactivated。

&emsp;简单介绍一下在被 keep-alive 包含的组件/路由中，会多出两个生命周期的钩子:activated 与 deactivated。文档：在 2.2.0 及其更高版本中，activated 和 deactivated 将会在 树内的所有嵌套组件中触发。 activated 在组件第一次渲染时会被调用，之后在每次缓存组件被激活时调用。 **activated 调用时机：** 第一次进入缓存路由/组件，在 mounted 后面，beforeRouteEnter 守卫传给 next 的回调函数之前调用：

beforeMount=> 如果你是从别的路由/组件进来(组件销毁 destroyed/或离开缓存 deactivated)=>mounted=> activated 进入缓存组件 => 执行 beforeRouteEnter 回调

因为组件被缓存了，再次进入缓存路由/组件时，不会触发这些钩子：// beforeCreate created beforeMount mounted 都不会触发。

deactivated：组件被停用(离开路由)时调用使用了 keep-alive 就不会调用 beforeDestroy(组件销毁前钩子)和 destroyed(组件销毁)，因为组件没被销毁，被缓存起来了。这个钩子可以看作 beforeDestroy 的替代，如果你缓存了组件，要在组件销毁的的时候做一些事情，你可以放在这个钩子里。如果你离开了路由，会依次触发：

组件内的离开当前路由钩子 beforeRouteLeave => 路由前置守卫 beforeEach =>全局后置钩子 afterEach => deactivated 离开缓存组件 => activated 进入缓存组件(如果你进入的也是缓存路由

**项目中缓存使用方法：** 1、在创建的 router 对象上加 scrollBehavior 方法，同上； 2、将需要缓存的组件加在 include 属性里

```vue
<keep-alive :include="['home', 'classify', 'search']">
      <router-view></router-view>
</keep-alive>
```

3、在 beforeRouteEnter 的 next 回掉函数里，对返回 A 页面不需要缓存的的情况初始化，即将本来需要写在 created 里的东西写在这里；注意一定要将所有的需要初始化的数据要写一遍，不然会有 bug;所以不太推荐

```js
beforeRouteEnter (to, from, next) {
    next(vm => {
      // 通过 `vm` 访问组件实例
      if (from.path !== '/goods_detail') { // 一定是从A进到B页面才刷新
        vm.titleText = vm.$route.query.name
        vm.categoryUpper = vm.$route.query.categoryUpper
        vm.goods = []
        vm.page = 1
        vm.catsIndex = 0
        vm.is_search = false
        vm.getCats2()// 是本来写在created里面的各种
      }
    })
  }
```

## 三、使用 include、exclude 属性和 beforeRouteLeave 钩子函数和 vuex (推荐使用)

&emsp;第三种方法和第二种相似，不同的地方在于，将需要缓存的组件保存到全局变量，可以在路由的钩子函数里灵活的控制哪些组件需要缓存，那些不需要缓存；跟第二种方法相比，不需要每次再重新初始化数据，但是需要在 vuex 中保存数据；上代码 1、在创建的 router 对象上加 scrollBehavior 方法，同上； 2、将需要缓存的组件加在 include 属性里

```vue
<keep-alive :include="catch_components">
      <router-view></router-view>
</keep-alive>
```

3、在 store 里加入需要缓存的的组件的变量名，和相应的方法；

```js
export default new Vuex.Store({
  state: {
    catch_components: []
  },
  mutations: {
    GET_CATCHE_COMPONENTS(state, data) {
      state.catch_components = data;
    }
  }
});
```

3、在 beforeRouteLeave 钩子函数里控制需要缓存的组件

```js
beforeRouteLeave (to, from, next) { //要在离开该组件的时候控制需要缓存的组件，否则将出现第一次不缓存的情况
    this.busy = true
    if (to.path === '/goods_detail') { // 去往详情页的时候需要缓存组件，其他情况下不需要缓存
      this.$store.commit('GET_CATCHE_COMPONENTS', ['home'])
    } else {
      this.$store.commit('GET_CATCHE_COMPONENTS', [])
    }
    next()
  },
```

&emsp;以上是在 vue 项目里使用 keep-alive 的情况，网上有一些配合 this.$destroy()方法使用的，但我在使用过程中验证了，并不好用；如果有多个状态，并且有选择的缓存，那么第三个方法是最好的选择；如果你不想用 vuex 使用第二种方法也可以，但需要注意初始化数据。

&emsp;另外，在我们的项目中遇到路由相同但参数不同的情况组件被复用，不更新的问题，vue 官方给出了 [响应路由参数变化](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#响应路由参数的变化)

```js
watch: {
    '$route' (to, from) {
      document.title = this.$route.query.name
      this.getDefault() //根据参数数据响应
    }

  },
```
