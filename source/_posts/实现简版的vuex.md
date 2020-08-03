---
title: 实现简版的vuex
id: '20200718005'
tags:
  - vue
date: 2020-07-18 10:31:10
tag: vue
category: web
---

一、使用vuex实现一个简单的示例

#### 1、安装vuex
```
npm install vuex --save
```
#### 2、创建store文件夹，在文件夹下新建index.js文件
```
import Vue from "vue";
import Vuex from "vuex";
Vue.use(Vuex)
export default new Vuex.Store({
    state: {
        count: 0
    },
    getters: {
        doubleCount(state) {
            return state.count \* 2
        }
    },
    mutations: {
        add(state) {
            state.count++
        }
    },
    actions: {
    //这里用了解构赋值 其实是
    //   add(context){
    //       setTimeout(() => {
    //          context.commit("add")
    //        }, 1000);}

        add({commit}) {
            setTimeout(() => {
                commit("add")
            }, 1000);
        }
    }
})
```
#### 3、在根组件上添加实例
``
import store from "./store";
new Vue({
  store,
  render: h => h(App),
}).$mount('#app')
```
#### 4、在组件中使用
```
<p @click="$store.commit('add')">count:{{ $store.state.count }}</p>
<p @click="$store.dispatch('add')">async count:{{ $store.state.count }}</p>
```
二、实现简版的KVuex

#### 1、新建kstore文件夹，并将store下的js复制到当前文件夹下，修改如下代码:
```
//重我们需要实现的kvuex引入Vuex
import Vuex from './kvuex'
```
#### 2、修改main.js
```
//引入kstore下的index中export的store
import store from './kstore'
new Vue({
  store,
  render: h => h(App)
}).$mount('#app')
```
#### 3、store类的声明，install的实现
```
// 保存构造函数引用，避免import
let Vue;

class Store {
  constructor(options) {
    //将state变成响应式
    this.state = new Vue({
      data: options.state
    })
  }
}

function install(\_Vue) {
  Vue = \_Vue;
    // 全局混入， 将$store放到Vue.prototype上，
    // 因为main.js是这样引用的
    // new Vue({
    //     store,
    //     render: h => h(App),
    // }).$mount('#app')
  Vue.mixin({
    beforeCreate() {
        //只有根组件我们才传了store实例
      if (this.$options.store) {
        Vue.prototype.$store = this.$options.store
      }
    }
  })

}

// Vuex
export default {
  Store,
  install
}
```
#### 4、实现commit：根据用户传入的type获取并执行mutation
```
class Store {
  constructor(options) {
    // this.$options = options;
    //获取全部的mutation
    this.\_mutations = options.mutations;
    //将state变成响应式
    this.state = new Vue({
      data: options.state
    })

    // 绑定commit、dispatch的上下文问store实例
    this.commit = this.commit.bind(this)   
  }

  // eg:@click="$store.commit('add')"
  // type: mutation的类型
  // payload：载荷，是参数 对应mutations里面，如minus(state,n){state.count-n}

  commit(type, payload) {
  //此时this\_mutations\['add'\]就是add(state) { state.counter++}
    const entry = this.\_mutations\[type\]
    //如果entry存在，执行
    if (entry) {
      entry(this.state, payload)
    }
  }

}
```
#### 5、实现dispatch：根据用户传入的type获取并执行actions
```
class Store {
  constructor(options) {
    // this.$options = options;
    //获取全部的mutation
    this.\_mutations = options.mutations;
    //获取全部的actions
    this.\_actions = options.actions;
    //将state变成响应式
    this.state = new Vue({
      data: options.state
    })

    // 绑定commit上下文，否则action中调用时可能出问题
    this.commit = this.commit.bind(this) 
     // 绑定actions上下文，
    this.dispatch = this.dispatch.bind(this)  
  }

  // eg:@click="$store.commit('add')"
  // type: mutation的类型
  // payload：载荷，是参数 对应mutations里面，如minus(state,n){state.count-n}

  commit(type, payload) {
  //此时this\_mutations\['add'\]就是add(state) { state.counter++}
    const entry = this.\_mutations\[type\]
    //如果entry存在，执行
    if (entry) {
      entry(this.state, payload)
    }
  }

  //@click="$store.dispatch('add')"
  //add({commit}) {
  //         setTimeout(() => {
  //             commit("add")
  //         }, 1000);
  //     }
  //actions和mutations的区别是 actions能处理异步，而mutations只能处理同步
dispatch(type, payload) {
    const entry = this.\_actions\[type\]
    if (entry) {
    //这里与commit不同，需要传入this
      entry(this, payload)
    }
  }

}
```