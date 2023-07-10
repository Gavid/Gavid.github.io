@[TOC](vuex)

# VUEX 是什么？

> Vuex 是一个专为 Vue .js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

## vuex 框架图

![](https://img-blog.csdnimg.cn/20190709162649746.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)

## 什么情况下使用 VUEX ？

Vuex 可以帮助我们管理共享状态，并附带了更多的概念和框架。这需要对短期和长期效益进行权衡。

> 1. _简单的应用_ 是不需要使用 vuex 的，或者最多使用简单的 **store模式** 就足够了。
> 2. _中大型的应用_ , 则需要考虑用 vuex 更好的在组件外部管理状态。

# VUEX 基本概念

1. 每一个 Vuex 应用的核心就是 *store(仓库)* ，其中包含应用大部分的 *state(状态)*

   > 1. vuex 的状态存储是**响应式**的
   >
   > 2. 不能直接改变 store 中的状态
   >
   >    > 改变状态的唯一途径：**commit(提交) mutation**

2. 由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在 **计算属性** 中返回即可。触发变化也仅仅是在组件的 **methods 中提交 mutation**。

# VUEX 核心概念

## State 

### 单一状态树

1. Vuex 使用*单一状态树*，用一个对象就包含了全部的应用层级状态。
2. 每个应用将仅仅包含一个store 实例。
3. 单状态树 和 模块化 并不冲突。

### 在Vue 组件中获得 Vuex 状态

1. 最简单的方法就是在*计算属性* 中返回某个状态。但是，这种模式导致组件依赖全局状态单例。

   ```js
     computed: {
       count () {
         return store.state.count
       }
     }
   ```

2. Vuex 通过 `store` 选项，提供了一种机制将状态从根组件**注入**到每一个子组件中（需调用 `Vue.use(Vuex)`）：

   > 1. 创建 Vue 时，注册 `store` 选项。
   >
   > ```js
   > const app = new Vue({
   > el: '#app',
   > // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
   > store,
   > components: { Counter },
   > template: `
   >  <div class="app">
   >    <counter></counter>
   >  </div>
   > `
   > })
   > ```
   >
   > 2. 该 store 实例会注入到根组件下的所有子组件中，且子组件能通过 `this.$store` 访问到。
   >
   > ```js
   > const Counter = {
   > template: `<div>{{ count }}</div>`,
   > computed: {
   >  count () {
   >    return this.$store.state.count
   >  }
   > }
   > }
   > ```

### mapState 辅助函数

1. 当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，我们可以使用 `mapState` 辅助函数帮助我们生成计算属性，让你少按几次键：

   ```js
   // 在单独构建的版本中辅助函数为 Vuex.mapState
   import { mapState } from &#39;vuex&#39;
   
   export default {
     // ...
     computed: mapState({
       // 箭头函数可使代码更简练
       count: state =&gt; state.count,
   
       // 传字符串参数 &#39;count&#39; 等同于 `state =&gt; state.count`
       countAlias: &#39;count&#39;,
   
       // 为了能够使用 `this` 获取局部状态，必须使用常规函数
       countPlusLocalState (state) {
         return state.count + this.localCount
       }
     })
   } 
   ```

**Note:** 当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 `mapState` 传一个字符串数组

```js
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

### 对象展开运算符

1. `mapState` 函数返回的是一个对象。使用*对象展开运算符*，可以将它与局部计算属性混合使用。

   ```js
   computed: {
     localComputed () { /* ... */ },
     // 使用对象展开运算符将此对象混入到外部对象中
     ...mapState({
       // ...
     })
   }
   ```

### 组件仍然保持局部状态

> 使用 Vuex 并不意味着你需要将**所有的**状态放入 Vuex。虽然将所有的状态放到 Vuex 会使状态变化更显式和易调试，但也会使代码变得冗长和不直观。如果有些状态严格属于单个组件，最好还是作为组件的局部状态。你应该根据你的应用开发需要进行权衡和确定。

## Getter

> Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。
>
> ```js
> const store = new Vuex.Store({
> state: {
>  todos: [
>    { id: 1, text: '...', done: true },
>    { id: 2, text: '...', done: false }
>  ]
> },
> getters: {
>  doneTodos: state => {
>    return state.todos.filter(todo => todo.done)
>  }
> }
> })
> ```

### 通过属性访问

1. Getter 会暴露为 `store.getters` 对象，你可以以属性的形式访问这些值：

   ```js
   store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
   ```

2. Getter 也可以接受其他 getter 作为第二个参数：

   ```js
   getters: {
     // ...
     doneTodosCount: (state, getters) => {
       return getters.doneTodos.length
     }
   }
   
   ```

   ```js
   store.getters.doneTodosCount // -> 1
   
   ```

3. 我们可以很容易地在任何组件中使用它：

   ```js
   computed: {
     doneTodosCount () {
       return this.$store.getters.doneTodosCount
     }
   }
   
   ```

**Note:** *getter 在通过属性访问时是作为 Vue 的响应式系统的一部分缓存其中的。*

### 通过方法访问

1. 可以通过让 getter 返回一个函数，来实现给 getter 传参。

   ```js
   getters: {
     // ...
     getTodoById: (state) => (id) => {
       return state.todos.find(todo => todo.id === id)
     }
   }
   
   ```

   ```js
   store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
   
   ```

**Note:** *getter 在通过方法访问时，每次都会去进行调用，而不会缓存结果*

### mapGetters 辅助函数

`mapGetters` 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性：

```js
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}

```

## Mutation

> **更改** Vuex 的 store 中的状态的唯一方法是**提交 mutation**。

> Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 **事件类型 (type)** 和 一个 **回调函数 (handler)**。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数:
>
> ```js
> const store = new Vuex.Store({
> state: {
>  count: 1
> },
> mutations: {
>  increment (state) {
>    // 变更状态
>    state.count++
>  }
> }
> })
> 
> ```

> 不能直接调用一个 mutation handler，要唤醒一个 mutation handler，你需要以相应的 type 调用 **store.commit** 方法：
>
> ```js
> store.commit('increment')
> 
> ```

### 提交载荷（Payload）

1. 可以向 `store.commit` 传入额外的参数，即 mutation 的 **载荷（payload）**,载荷可以是一个对象，方便传多个值：

   ```js
   mutations: {
     increment (state, n) {
       state.count += n
     }
   }
   
   ```

   ```js
   store.commit('increment', 10)
   
   ```

### 对象风格的提交方式

提交 mutation 的另一种方式是直接使用包含 `type` 属性的对象：

```js
store.commit({
  type: 'increment',
  amount: 10
})

```

### Mutation 需遵守 Vue 的响应规则

> - 最好提前在你的store 中初始化好所有的所需属性
> - 当需要在对象上添加新属性时：
>   - 使用 `Vue.set(obj, 'newProp', 123)`  **（推荐）**
>   - 以新对象替换老对象。

### 使用常量替代Mutation 事件类型（可选）

> 使用常量替代 mutation 事件类型可以使 linter 之类的工具发挥作用，同时把这些常量放在单独的文件中可以让你的代码合作者对整个 app 包含的 mutation 一目了然：
>
> ```js
> // mutation-types.js
> export const SOME_MUTATION = 'SOME_MUTATION'
> 
> ```
>
> ```js
> // store.js
> import Vuex from 'vuex'
> import { SOME_MUTATION } from './mutation-types'
> 
> const store = new Vuex.Store({
> state: { ... },
> mutations: {
>  // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
>  [SOME_MUTATION] (state) {
>    // mutate state
>  }
> }
> })
> 
> ```

### Mutation 必须是同步函数

### 在组件中提交 Mutation

> - 可以在组件中使用 `this.$store.commit('xxx')` 提交 mutation
> - 使用 `mapMutations` 辅助函数将组件中的 methods 映射为 `store.commit` 调用（需要在根节点注入 `store`）
>
> ```js
> import { mapMutations } from 'vuex'
> 
> export default {
>  // ...
>  methods: {
>    ...mapMutations([
>      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
> 
>      // `mapMutations` 也支持载荷：
>      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
>    ]),
>    ...mapMutations({
>      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
>    })
>  }
> }
> 
> ```

## Action

1. Action 与 Mutation 的区别：

   > - Action 提交的是 mutation，而不是直接变更状态。
   > - Action 可以包含任意异步操作。

2. Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 `context.commit` 提交一个 mutation，或者通过 `context.state` 和 `context.getters` 来获取 state 和 getters。

   ```js
   const store = new Vuex.Store({
     state: {
       count: 0
     },
     mutations: {
       increment (state) {
         state.count++
       }
     },
     actions: {
       increment (context) {
         context.commit('increment')
       }
     }
   })
   
   ```

3. 当需要多次调用 `commit` 的时候，会经常用到 ES2015 的 [参数解构](https://github.com/lukehoban/es6features#destructuring) 来简化代码：

   ```js
   actions: {
     increment ({ commit }) {
       commit('increment')
     }
   }
   
   ```

### 分发Action

> Action 通过 `store.dispatch` 方法触发：
>
> ```js
> store.dispatch('increment')
> 
> ```

1. 可以在 action 内部执行**异步**操作:

   ```js
   actions: {
     incrementAsync ({ commit }) {
       setTimeout(() => {
         commit('increment')
       }, 1000)
     }
   }
   
   ```

2. Actions 支持同样的载荷方式和对象方式进行分发：

   ```js
   // 以载荷形式分发
   store.dispatch('incrementAsync', {
     amount: 10
   })
   
   // 以对象形式分发
   store.dispatch({
     type: 'incrementAsync',
     amount: 10
   })
   
   ```

### 在组件中分发 Action

1. 使用 `this.$store.dispatch('xxx')` 分发 action **(推荐)**

2. 使用 `mapActions` 辅助函数将组件的 methods 映射为 `store.dispatch` 调用（需要先在根节点注入 `store`）：

   ```js
   import { mapActions } from 'vuex'
   
   export default {
     // ...
     methods: {
       ...mapActions([
         'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
   
         // `mapActions` 也支持载荷：
         'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
       ]),
       ...mapActions({
         add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
       })
     }
   }
   
   ```

### 组合Action 

- Action 通常是异步的，那么如何知道 action 什么时候结束呢？更重要的是，我们如何才能组合多个 action，以处理更加复杂的异步流程？

  1. 明白 `store.dispatch` 可以处理被触发的 action 的处理函数返回的 Promise，并且 `store.dispatch` 仍旧返回 Promise：

     ```js
     actions: {
       actionA ({ commit }) {
         return new Promise((resolve, reject) => {
           setTimeout(() => {
             commit('someMutation')
             resolve()
           }, 1000)
         })
       }
     }
     
     ```

  2. 可以在另外一个 action 中也可以使用：

     ```js
     actions: {
       // ...
       actionB ({ dispatch, commit }) {
         return dispatch('actionA').then(() => {
           commit('someOtherMutation')
         })
       }
     }
     
     ```

  3. 利用 [async / await](https://tc39.github.io/ecmascript-asyncawait/)，我们可以如下组合 action：

     ```js
     // 假设 getData() 和 getOtherData() 返回的是 Promise
     
     actions: {
       async actionA ({ commit }) {
         commit('gotData', await getData())
       },
       async actionB ({ dispatch, commit }) {
         await dispatch('actionA') // 等待 actionA 完成
         commit('gotOtherData', await getOtherData())
       }
     }
     
     ```

     > **Note:** 一个 `store.dispatch` 在不同模块中可以触发多个 action 函数。在这种情况下，只有当所有触发函数完成后，返回的 Promise 才会执行。

## Module

> 由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。为了解决以上问题，Vuex 允许我们将 store 分割成**模块（module）**。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：
>
> ```js
> const moduleA = {
> state: { ... },
> mutations: { ... },
> actions: { ... },
> getters: { ... }
> }
> 
> const moduleB = {
> state: { ... },
> mutations: { ... },
> actions: { ... }
> }
> 
> const store = new Vuex.Store({
> modules: {
>  a: moduleA,
>  b: moduleB
> }
> })
> 
> store.state.a // -> moduleA 的状态
> store.state.b // -> moduleB 的状态
> 
> ```

### 模块的局部状态

1. 对于模块内部的 mutation 和 getter，接收的第一个参数是**模块的局部状态对象**。

   ```js
   const moduleA = {
     state: { count: 0 },
     mutations: {
       increment (state) {
         // 这里的 `state` 对象是模块的局部状态
         state.count++
       }
     },
   
     getters: {
       doubleCount (state) {
         return state.count * 2
       }
     }
   }
   
   ```

2. 对于模块内部的 action，局部状态通过 `context.state` 暴露出来，根节点状态则为 `context.rootState`：

   ```js
   const moduleA = {
     // ...
     actions: {
       incrementIfOddOnRootSum ({ state, commit, rootState }) {
         if ((state.count + rootState.count) % 2 === 1) {
           commit('increment')
         }
       }
     }
   }
   
   ```

3. 对于模块内部的 getter，根节点状态会作为第三个参数暴露出来：

   ```js
   const moduleA = {
     // ...
     getters: {
       sumWithRootCount (state, getters, rootState) {
         return state.count + rootState.count
       }
     }
   }
   
   ```

### 命名空间

> 默认情况下，模块内部的 action、mutation 和 getter 是注册在**全局命名空间**的——这样使得多个模块能够对同一 mutation 或 action 作出响应。
>
> 如果希望你的模块具有更高的封装度和复用性，你可以通过添加 `namespaced: true` 的方式使其成为带命名空间的模块。当模块被注册后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名。例如：
>
> ```js
> const store = new Vuex.Store({
> modules: {
>  account: {
>    namespaced: true,
> 
>    // 模块内容（module assets）
>    state: { ... }, // 模块内的状态已经是嵌套的了，使用 `namespaced` 属性不会对其产生影响
>    getters: {
>      isAdmin () { ... } // -> getters['account/isAdmin']
>    },
>    actions: {
>      login () { ... } // -> dispatch('account/login')
>    },
>    mutations: {
>      login () { ... } // -> commit('account/login')
>    },
> 
>    // 嵌套模块
>    modules: {
>      // 继承父模块的命名空间
>      myPage: {
>        state: { ... },
>        getters: {
>          profile () { ... } // -> getters['account/profile']
>        }
>      },
> 
>      // 进一步嵌套命名空间
>      posts: {
>        namespaced: true,
> 
>        state: { ... },
>        getters: {
>          popular () { ... } // -> getters['account/posts/popular']
>        }
>      }
>    }
>  }
> }
> })
> 
> ```

> **Note:** 启用了命名空间的 getter 和 action 会收到局部化的 `getter`，`dispatch` 和 `commit`。

#### 在带命名空间的模块内访问全局内容（Global Assets）

- 如果你希望使用全局 state 和 getter，`rootState` 和 `rootGetter` 会作为第三和第四参数传入 getter，也会通过 `context` 对象的属性传入 action。

- 若需要在全局命名空间内分发 action 或提交 mutation，将 `{ root: true }` 作为第三参数传给 `dispatch` 或 `commit` 即可。

  ```js
  modules: {
    foo: {
      namespaced: true,
  
      getters: {
        // 在这个模块的 getter 中，`getters` 被局部化了
        // 你可以使用 getter 的第四个参数来调用 `rootGetters`
        someGetter (state, getters, rootState, rootGetters) {
          getters.someOtherGetter // -> 'foo/someOtherGetter'
          rootGetters.someOtherGetter // -> 'someOtherGetter'
        },
        someOtherGetter: state => { ... }
      },
  
      actions: {
        // 在这个模块中， dispatch 和 commit 也被局部化了
        // 他们可以接受 `root` 属性以访问根 dispatch 或 commit
        someAction ({ dispatch, commit, getters, rootGetters }) {
          getters.someGetter // -> 'foo/someGetter'
          rootGetters.someGetter // -> 'someGetter'
  
          dispatch('someOtherAction') // -> 'foo/someOtherAction'
          dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'
  
          commit('someMutation') // -> 'foo/someMutation'
          commit('someMutation', null, { root: true }) // -> 'someMutation'
        },
        someOtherAction (ctx, payload) { ... }
      }
    }
  }
  
  ```

#### 在带命名空间的模块注册全局 action

- 若需要在带命名空间的模块注册全局 action，你可添加 `root: true`，并将这个 action 的定义放在函数 `handler` 中。例如：

  ```js
  {
    actions: {
      someOtherAction ({dispatch}) {
        dispatch('someAction')
      }
    },
    modules: {
      foo: {
        namespaced: true,
  
        actions: {
          someAction: {
            root: true,
            handler (namespacedContext, payload) { ... } // -> 'someAction'
          }
        }
      }
    }
  }
  
  ```

#### 带命名空间的绑定函数

- 当使用 `mapState`, `mapGetters`, `mapActions` 和 `mapMutations` 这些函数来绑定带命名空间的模块时，写起来可能比较繁琐：

  ```js
  computed: {
    ...mapState({
      a: state => state.some.nested.module.a,
      b: state => state.some.nested.module.b
    })
  },
  methods: {
    ...mapActions([
      'some/nested/module/foo', // -> this['some/nested/module/foo']()
      'some/nested/module/bar' // -> this['some/nested/module/bar']()
    ])
  }
  
  ```

- 对于这种情况，你可以将模块的空间名称字符串作为第一个参数传递给上述函数，这样所有绑定都会自动将该模块作为上下文。于是上面的例子可以简化为：

  ```js
  computed: {
    ...mapState('some/nested/module', {
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    ...mapActions('some/nested/module', [
      'foo', // -> this.foo()
      'bar' // -> this.bar()
    ])
  }
  
  ```

- 而且，你可以通过使用 `createNamespacedHelpers` 创建基于某个命名空间辅助函数。它返回一个对象，对象里有新的绑定在给定命名空间值上的组件绑定辅助函数：

  ```js
  import { createNamespacedHelpers } from 'vuex'
  
  const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')
  
  export default {
    computed: {
      // 在 `some/nested/module` 中查找
      ...mapState({
        a: state => state.a,
        b: state => state.b
      })
    },
    methods: {
      // 在 `some/nested/module` 中查找
      ...mapActions([
        'foo',
        'bar'
      ])
    }
  }
  
  ```

#### 给插件开发者的注意事项

- 如果你开发的[插件（Plugin）](https://vuex.vuejs.org/zh/guide/plugins.html)提供了模块并允许用户将其添加到 Vuex store，可能需要考虑模块的空间名称问题。对于这种情况，你可以通过插件的参数对象来允许用户指定空间名称：

  ```js
  // 通过插件的参数对象得到空间名称
  // 然后返回 Vuex 插件函数
  export function createPlugin (options = {}) {
    return function (store) {
      // 把空间名字添加到插件模块的类型（type）中去
      const namespace = options.namespace || ''
      store.dispatch(namespace + 'pluginAction')
    }
  }
  
  ```

### 模块动态注册

- 在 store 创建**之后**，你可以使用 `store.registerModule` 方法注册模块：

  ```js
  // 注册模块 `myModule`
  store.registerModule('myModule', {
    // ...
  })
  // 注册嵌套模块 `nested/myModule`
  store.registerModule(['nested', 'myModule'], {
    // ...
  })
  
  ```

- 模块动态注册功能使得其他 Vue 插件可以通过在 store 中附加新模块的方式来使用 Vuex 管理状态。

- 也可以使用 `store.unregisterModule(moduleName)` 来动态卸载模块。注意，你不能使用此方法卸载静态模块（即创建 store 时声明的模块）。

#### 保留 state

- 在注册一个新 module 时，你很有可能想保留过去的 state，例如从一个服务端渲染的应用保留 state。你可以通过 `preserveState` 选项将其归档：`store.registerModule('a', module, { preserveState: true })`。
- 当你设置 `preserveState: true` 时，该模块会被注册，action、mutation 和 getter 会被添加到 store 中，但是 state 不会。这里假设 store 的 state 已经包含了这个 module 的 state 并且你不希望将其覆写。

### 模块重用

有时我们可能需要创建一个模块的多个实例，例如：

- 创建多个 store，他们公用同一个模块 (例如当 `runInNewContext` 选项是 `false` 或 `'once'` 时，为了[在服务端渲染中避免有状态的单例](https://ssr.vuejs.org/en/structure.html#avoid-stateful-singletons))
- 在一个 store 中多次注册同一个模块

实际上这和 Vue 组件内的 `data` 是同样的问题。因此解决办法也是相同的——使用一个函数来声明模块状态（仅 2.3.0+ 支持）：

```js
const MyReusableModule = {
  state () {
    return {
      foo: 'bar'
    }
  },
  // mutation, action 和 getter 等等...
}

```

# 项目结构

```bash
├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块

```

# 插件

Vuex 的 store 接受 `plugins` 选项，这个选项暴露出每次 mutation 的钩子。Vuex 插件就是一个函数，它接收 store 作为唯一参数：

```js
const myPlugin = store => {
  // 当 store 初始化后调用
  store.subscribe((mutation, state) => {
    // 每次 mutation 之后调用
    // mutation 的格式为 { type, payload }
  })
}

```

然后像这样使用：

```js
const store = new Vuex.Store({
  // ...
  plugins: [myPlugin]
})

```

### 在插件内提交 Mutation（掌握）

- **在插件中不允许直接修改状态——类似于组件，只能通过提交 mutation 来触发变化。**
- 通过提交 mutation，插件可以用来同步数据源到 store。例如，同步 websocket 数据源到 store（下面是个大概例子，实际上 `createPlugin` 方法可以有更多选项来完成复杂任务）：

```js
export default function createWebSocketPlugin (socket) {
  return store => {
    socket.on('data', data => {
      store.commit('receiveData', data)
    })
    store.subscribe(mutation => {
      if (mutation.type === 'UPDATE_DATA') {
        socket.emit('update', mutation.payload)
      }
    })
  }
}

```

```js
const plugin = createWebSocketPlugin(socket)

const store = new Vuex.Store({
  state,
  mutations,
  plugins: [plugin]
})

```

### 生成 State 快照（了解）

有时候插件需要获得状态的“快照”，比较改变的前后状态。想要实现这项功能，你需要对状态对象进行深拷贝：

```js
const myPluginWithSnapshot = store => {
  let prevState = _.cloneDeep(store.state)
  store.subscribe((mutation, state) => {
    let nextState = _.cloneDeep(state)

    // 比较 prevState 和 nextState...

    // 保存状态，用于下一次 mutation
    prevState = nextState
  })
}

```

**生成状态快照的插件应该只在开发阶段使用**，使用 webpack 或 Browserify，让构建工具帮我们处理：

```js
const store = new Vuex.Store({
  // ...
  plugins: process.env.NODE_ENV !== 'production'
    ? [myPluginWithSnapshot]
    : []
})

```

**上面插件会默认启用。**在发布阶段，你需要使用 webpack 的 [DefinePlugin](https://webpack.js.org/plugins/define-plugin/) 或者是 Browserify 的 [envify](https://github.com/hughsk/envify) 使 `process.env.NODE_ENV !== 'production'` 为 `false`。

### 内置 Logger 插件（了解）

Vuex 自带一个日志插件用于一般的调试:

```js
import createLogger from 'vuex/dist/logger'

const store = new Vuex.Store({
  plugins: [createLogger()]
})

```

`createLogger` 函数有几个配置项：

```js
const logger = createLogger({
  collapsed: false, // 自动展开记录的 mutation
  filter (mutation, stateBefore, stateAfter) {
    // 若 mutation 需要被记录，就让它返回 true 即可
    // 顺便，`mutation` 是个 { type, payload } 对象
    return mutation.type !== "aBlacklistedMutation"
  },
  transformer (state) {
    // 在开始记录之前转换状态
    // 例如，只返回指定的子树
    return state.subTree
  },
  mutationTransformer (mutation) {
    // mutation 按照 { type, payload } 格式记录
    // 我们可以按任意方式格式化
    return mutation.type
  },
  logger: console, // 自定义 console 实现，默认为 `console`
})

```

- 日志插件还可以直接通过 `<script>` 标签引入，它会提供全局方法 `createVuexLogger`。
- **要注意，logger 插件会生成状态快照，所以仅在开发环境使用。**

# 严格模式

开启严格模式，仅需在创建 store 的时候传入 `strict: true`：

```js
const store = new Vuex.Store({
  // ...
  strict: true
})

```

在严格模式下，无论何时发生了状态变更且不是由 mutation 函数引起的，将会抛出错误。这能保证所有的状态变更都能被调试工具跟踪到。

- **不要在发布环境下启用严格模式**！严格模式会深度监测状态树来检测不合规的状态变更——请确保在发布环境下关闭严格模式，以避免性能损失

类似于插件，我们可以让构建工具来处理这种情况：

```js
const store = new Vuex.Store({
  // ...
  strict: process.env.NODE_ENV !== 'production'
})

```

# 表单处理

- 当在严格模式中使用 Vuex 时，在属于 Vuex 的 state 上使用 `v-model` 会比较棘手：

```html
<input v-model="obj.message">

```

假设这里的 `obj` 是在计算属性中返回的一个属于 Vuex store 的对象，在用户输入时，`v-model` 会试图直接修改 `obj.message`。在严格模式中，由于这个修改不是在 mutation 函数中执行的, 这里会抛出一个错误。

### 解决方案：

1. **用“Vuex 的思维”去解决这个问题的方法是：给 `<input>` 中绑定 value，然后侦听 `input` 或者 `change`事件，在事件回调中调用 action:（了解）**

```html
<input :value="message" @input="updateMessage">

```

```js
// ...
computed: {
  ...mapState({
    message: state => state.obj.message
  })
},
methods: {
  updateMessage (e) {
    this.$store.commit('updateMessage', e.target.value)
  }
}

```

```js
// ...
mutations: {
  updateMessage (state, message) {
    state.obj.message = message
  }
}

```

2. **双向绑定的计算属性（掌握）**

必须承认，这样做比简单地使用“`v-model` + 局部状态”要啰嗦得多，并且也损失了一些 `v-model` 中很有用的特性。另一个方法是使用带有 setter 的双向绑定计算属性：

```html
<input v-model="message">

```

```js
// ...
computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}

```


