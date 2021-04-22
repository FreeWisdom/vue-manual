# 第7章 Vue-Router

## 1.1、`router-link`

* 路由跳转的标签；

## 1.2、`router-view`

* 负责展示当前路由对应的组件内容；

```vue
<template>
  <div id="nav">
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>
    <router-view/>
  </div>
</template>
```

## 1.3、路由懒加载

* 异步/懒 加载，首页刷新不需要下载该方式懒加载的路由，达到首屏快速响应目的；

```js
const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    // 异步/懒 加载，首页刷新不需要下载该方式懒加载的路由，达到首屏快速响应；
    component: () => import('../views/About.vue')
  }
]
```

# 2、VueX

* vuex 数据管理框架；
* vuex 创建了一个全局的仓库，用来存放全局的数据；

## 2.1、createStore 创建全局 store

> `store/index.js`

```js
import { createStore } from 'vuex'

// vuex 数据管理框架
// vuex 创建了一个全局的仓库，用来存放全局的数据
export default createStore({
  state: {
    name: 'zhz'
  },
  mutations: {
    // ❗️❗️❗️注意第一个参数❗️❗️❗️与 actions 中的参数不一样😯
    change (state, payload) {
      state.name = payload
    }
  },
  actions: {
    // ❗️❗️❗️注意第一个参数❗️❗️❗️与 mutations 中的参数不一样😯
    change (store, payload) {
      setTimeout(() => {
        store.commit('change', payload)
      }, 2000)
    }
  },
  modules: {
  }
})
```

## 2.2、`this.$store`取store数据

> `views/About.vue`

```vue
<template>
  <div class='about'>
    <h1>This is an about page</h1>
    <h2 @click="handleClick">{{myName}}</h2>
  </div>
</template>

<script>
export default {
  name: 'About',
  computed: {
    myName () {
      return this.$store.state.name
    }
  },
  methods: {
    handleClick () {
      this.$store.dispatch('change', 'canshu')
    }
  }
}
</script>
```

## 2.3、修改 store 数据

1. view 页面中通过`this.$store.dispatch('change')`派发名为 'change' 的 action ；
   * store 的 actions 接收到名为 'change' 的 action ，执行 change() 函数；
2. actions 的 change() 函数中通过`this.commit('change')`提交一个名为 'change' 的 commit ；
   * store 的 mutations 接收到名为 'change' 的 commit ，执行 change() 函数；
3. mutations 的 change() 函数中通过`this.state.name = 'Thales'`修改相应的数据；

## 2.4、可以不使用 action 吗？

1. **同步可跳，同步修改数据放 mutations 中：**若都是同步的修改 store 中的数据，在 view 页面可以跳过 action ，直接通过`this.$store.commit('change')`调用 mutations 的 change() 函数；

2. **异步不跳，异步修改数据放 actions 中：**vuex 的设计上 mutations 中只允许写同步代码，不允许写异步代码，故出现异步处理一般放在 actions 中处理；

3. 使用 store 数据修改有迹可循；

## 2.5、compositionAPI中使用vuex

* 通过**`import { useStore } from 'vuex'`**将 store 引入到 view 页面，然后使用 setup 代替老的写法；

> `views/Home.vue`

```vue
<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <h1 @click="handelClick">{{name}}</h1>
  </div>
</template>

<script>
import { toRefs } from 'vue'
import { useStore } from 'vuex'

export default {
  name: 'Home',
  setup () {
    const store = useStore()
    const { name } = toRefs(store.state)
    const handelClick = () => {
      store.dispatch('homeChange', 'THALES')
    }
    return { handelClick, name }
  }
}
</script>
```

> `store/index.js`

```js
import { createStore } from 'vuex'

export default createStore({
  state: {
    name: 'zhz'
  },
  mutations: {
    homeChange (state, payload) {
      state.name = payload
    }
  },
  actions: {
    homeChange (store, payload) {
      setTimeout(() => {
        store.commit('homeChange', payload)
      }, 3000)
    }
  },
  modules: {
  }
})
```

