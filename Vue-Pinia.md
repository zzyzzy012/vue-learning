# Vue-Pinia

## 定义store

定义的store命名格式推荐是：

- 最好使用 store 的名字
- 同时以 `use` 开头且以 `Store` 结尾。 (比如 `useUserStore`，`useCartStore`，`useProductStore`)

`defineStore()`的第一个参数要求是一个**独一无二的**名字，也被用作 *id* ，是必须传入的， Pinia 将用它来连接 store 和 devtools。

`defineStore()` 的第二个参数可接受两类值：Setup 函数或 Option 对象。

```javascript
// 1.导入
import { defineStore } from 'pinia'

// 2.定义
export const useAlertsStore = defineStore('alerts', {
  // 其他配置...
})
```

### Option Store

与 Vue 的选项式 API 类似，可以传入一个带有 `state`、`actions` 与 `getters` 属性的 Option 对象。

`state` 是 store 的数据 (`data`)，`getters` 是 store 的计算属性 (`computed`)，而 `actions` 则是方法 (`methods`)。

```javascript
export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0, name: 'Eduardo' }),
  getters: {
    doubleCount: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++
    },
  },
})
```

### Setup Store

与 Vue 组合式 API 的 `setup` 函数相似，我们可以传入一个函数，该函数定义了一些响应式属性和方法，并且返回一个带有我们想暴露出去的属性和方法的对象。

- `ref()` 就是 `state` 属性
- `computed()` 就是 `getters`
- `function()` 就是 `actions`

```javascript
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const doubleCount = computed(() => count.value * 2)
  function increment() {
    count.value++
  }

  return { count, doubleCount, increment }
})
```

## 使用store

```javascript
<script setup>
import { useCounterStore } from '@/stores/counter'
// 可以在组件中的任意位置访问 `store` 变量 ✨
const store = useCounterStore()
</script>
```

## 从 Store 解构

`store` 是一个用 `reactive` 包装的对象，这意味着不需要在 getters 后面写 `.value`。就像 `setup` 中的 `props` 一样，**我们不能对它进行解构**。

为了从 store 中提取属性时保持其响应性，使用 `storeToRefs()`。它将为每一个响应式属性创建引用。

如果只使用 store 的状态而不调用任何 action 时，它也会非常有用。可以直接从 store 中解构 action，因为它们也被绑定到 store 上。

`storeToRefs()`只关注store里面的数据而不关注方法，如果是`toRefs()`则不管数据和方法都统一包装成`ref`数据。

```vue
<script setup>
import { storeToRefs } from 'pinia'

const { sum, name, age } = storeToRefs(countStore)
</script>
```

## State

### 访问数据

```javascript
countStore.sum
countStore.$state.sum
```

### 修改数据

```javascript
import { useCountStore } from '@/store/Count.vue'

const countStore = useCountStore()

// 1.直接修改
countStore.sum = 100
// 2.使用$patch，批量修改
countStore.$patch({
  sum: 200,
  name: '李四',
  age: '30岁'
})

// 如果修改数组或者对象，可以传入一个函数进行修改
store.$patch((state) => {
  state.items.push({ name: 'shoes', quantity: 1 })
  state.hasChanged = true
})
```



```javascript
// 3.在Count.js的actions里面修改
import { defineStore } from 'pinia'
export const useCountStore = defineStore('count', {
  actions: {
    increment(value) {
      // this指向当前的Store实例
      this.sum += value
    }
  },
  state() {
    return {
      sum: 100,
      name: '张三',
      age: '18岁'
    }
  }
})

// 在Count.vue组件里调用
countStore.increment(n.value)
```

重置数据

选项式API:通过调用 store 的$reset()`方法将 state 重置为初始值。

```javascript
const store = useStore()

store.$reset()
```

在setup式store中，需要自己定义`$reset()`

```javascript
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)

  function $reset() {
    count.value = 0
  }

  return { count, $reset }
})
```

### 订阅数据

`$subscript((mutate, state) => {}) `，`mutate`原来的数据，`state`变化的数据。在数据发生改变时调用。

## Getter

Getter 完全等同于 store 的 state 的计算值。可以通过 `defineStore()` 中的 `getters` 属性来定义它们。**推荐**使用箭头函数，并且它将接收 `state` 作为第一个参数。大多数时候，getter 仅依赖 state。不过，有时它们也可能会使用其他 getter。因此，即使在使用常规函数定义 getter 时，也可以通过 `this` 访问到**整个 store 实例**。

```javascript
// count.vue组件
// getters对数据进行修改
// 第一个参数是state
getters: {
  bigSum(state) {
    return state.sum * 10
  },
    UpperName() {
      return this.name.toUpperCase()
    }
}
```

## Action

Action 相当于组件中的method。它们可以通过 `defineStore()` 中的 `actions` 属性来定义。action 也可通过 `this` 访问**整个 store 实例**。

```javascript
export const useCounterStore = defineStore('main', {
  state: () => ({
    count: 0,
  }),
  actions: {
    increment() {
      this.count++
    },
    randomizeCounter() {
      this.count = Math.round(100 * Math.random())
    },
  },
})
```

Action 可以像函数或者通常意义上的方法一样被调用。

```vue
<script setup>
const store = useCounterStore()
// 将 action 作为 store 的方法进行调用
store.randomizeCounter()
</script>
<template>
  <!-- 即使在模板中也可以 -->
  <button @click="store.randomizeCounter()">Randomize</button>
</template>
```

