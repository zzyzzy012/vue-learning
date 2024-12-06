# Vue响应式

## ref和reactive

`reactive()`数据整个直接修改，页面不会更新，而`ref()`可以直接修改。

```vue
<script setup>
import { ref, reactive } from 'vue'
  
let car = reactive({brand: '宝马', price: '1000万'})
const changeCar = () => {
  // car = {brand: '奔驰', price: '800万'}  // 页面不会更新
  // car = reactive({brand: '奔驰', price: '800万'})  // 页面不会更新
 
  console.log(car)
  Object.assign(car, {brand: '奔驰', price: '800万'})  // 会更新
}

let house = ref({type: '别墅', price: '888万'})
const changeHouse = () => {
  house.value = {type: '公寓', price: '777万'}  // 页面会更新
  console.log(house.value)
}
</script>

<template>
<div>宝马车: {{ car.brand }}，价格是 {{ car.price }}</div>
<button @click="changeCar">修改汽车信息</button>

<div>房子:{{ house.type }}, 价格是{{ house.price }}</div>
<button @click="changeHouse">修改房子信息</button>
</template>
```

`toRefs()`把`reactive()`所定义的响应式对象转换成`ref()`定义的响应式对象，场景：解构赋值`reactive()`对象的属性，直接解构，数据不是响应式。

`toRef(reactive响应式对象, 属性)`基于响应式对象上的一个属性，创建一个对应的`ref()`。

```vue
<script setup>
import { ref, reactive, toRefs, toRef } from 'vue'
let games = reactive({id: 'vsdfgdt01', name: '王者荣耀'})
let {id, name} = toRefs(games)
const changeGames = () => {
  id.value +='*'
  name.value +='~'
  console.log(id.value, games.id);
  console.log(name.value, games.name);
}
// 如果是toRef()
let id = toRef(games, 'id')
let name = toRef(games, 'name')
</script>

<template>
	<div>游戏id是：{{ id }} 游戏名字是： {{ name }}</div>
	<button @click="changeGames">修改游戏id和名字</button>
</template>

```

## shallowRef和shallowReactive

只进行浅层次解析

- `shallowRef`:只改变第一层数据或整个数据。比如`person.value`可以改变，`person.value.name`等就无法改变。
- `shallowReactive`:只改变第一层对象或整个对象。比如`toy.brand`可以改变，而`toy.feature.type`等就无法改变。

场景：只需要整体改变而不关注内部深层属性的单一变化，可以使用。常常用于对大型数据结构的性能优化或是与外部的状态管理系统集成。

```vue
<script setup>
import { shallowRef, shallowReactive } from 'vue'
let count = shallowRef(0)
const person = shallowRef({
  name: '张三',
  age: '18岁'
})

const toy = shallowReactive({
  brand: '迪士尼',
  feature: {
    type: '小熊',
    price: '1000元'
  }
})

function changeCount() {
  count.value++
}
function changeName() {
  person.value.name = '李四'
}
function changeAge() {
  person.value.age = '20岁'
}
function changePerson() {
  person.value = {
    name: '李四',
    age: '20岁'
  }
}

function changeBrand() {
  toy.brand = 'jellycat'
}
function changeType() {
  toy.feature.type = '小兔子'
}
function changePrice() {
  toy.feature.price = '500元'
}
function changeToy() {
  Object.assign(toy, {
    brand: 'jellycat',
    feature: {
      type: '小兔子',
      price: '500元'
  }
  })
}
</script>

<template>
  <div class="app">
    <div>总数：{{ count }}</div>
    <div>这个人的名字是：{{ person.name }}这个人的年龄是{{ person.age }}</div>
    <button @click="changeCount">change count</button>
    <button @click="changeName">change name</button>
    <button @click="changeAge">change age</button>
    <button @click="changePerson">change person</button>
    <div>玩具品牌是{{ toy.brand }}，类型是{{ toy.feature.type }}，价格是{{ toy.feature.price }}</div>
    <button @click="changeBrand">change brand</button>
    <button @click="changeType">change type</button>
    <button @click="changePrice">change price</button>
    <button @click="changeToy">change toy</button>
  </div>
</template>
```

## readonly和shallowreadonly

只能传入响应式数据，也就是由`ref()`和`reactive()`定义的数据，不能直接传入数字、字符串等。

```vue
<script setup>
import { ref,reactive, readonly } from 'vue'
let count = ref(0)
let count2 = readonly(count)
const person = reactive({
  name: '张三',
  age: '18岁',
  wealth: {
    money: '100万元',
    assets: ['房子', '车子', '手机']
  }
})
let person2 = readonly(person)
</script>

<template>
  <div class="app">
    <div>总数：{{ count }}</div>
    <div>总数2：{{ count2 }}</div>
    <button @click="count++">change count</button>
    <button @click="count2++">change count2</button>
    <div>person数据，名字是：{{ person.name }}，年龄是{{ person.age }}，财产有{{ person.wealth.money }}，还有{{ person.wealth.assets.join('、') }}</div>
    <div>person2数据，名字是：{{ person2.name }}，年龄是{{ person2.age }}，财产有{{ person2.wealth.money }}，还有{{ person2.wealth.assets.join('、') }}</div>
    <button @click="person.name = '李四'">change person name</button>
    <button @click="person2.name = '李四'">change person2 name</button>
  </div>
</template>
```

`shallowReadonly()`只有根层级的属性变为了只读，第二层及后面层数据可以修改。

```javascript
<script setup>
import { ref,reactive, shallowReadonly } from 'vue'
let count = ref(0)
let count2 = shallowReadonly(count)
const person = reactive({
  name: '张三',
  age: '18岁',
  wealth: {
    money: '100万元',
    assets: ['房子', '车子', '手机']
  }
})
let person2 = shallowReadonly(person)
</script>

<template>
  <div class="app">
    <div>总数：{{ count }}</div>
		// 第一层不能修改
    <div>总数2：{{ count2 }}</div>
    <button @click="count++">change count</button>
    <button @click="count2++">change count2</button>
    <div>person数据，名字是：{{ person.name }}，年龄是{{ person.age }}，财产有{{ person.wealth.money }}，还有{{ person.wealth.assets.join('、') }}</div>
    <div>person2数据，名字是：{{ person2.name }}，年龄是{{ person2.age }}，财产有{{ person2.wealth.money }}，还有{{ person2.wealth.assets.join('、') }}</div>
    <button @click="person.wealth.money = '200万元'">change person money</button>
		// 第一层不能修改
    <button @click="person2.name = '李四'">change person2 name</button>
		// 第二层及后面层可以修改
    <button @click="person2.wealth.money = '300万元'">change person2 money</button>
  </div>
</template>
```

## toRaw和markRaw

`toRaw()` 可以返回由 `reactive()`、`readonly()`、`shallowReactive()`或者 `shallowReadonly()`]创建的代理对应的原始对象。`toRaw()`返回的对象不是响应式的，不会引起视图的更新。

用于临时读取而不引起代理访问/跟踪开销，或是写入而不触发更改的特殊方法。

```javascript
<script setup>
import { reactive, toRaw } from 'vue'
const person = reactive({ name: '张三', age: 18})
let person2 = toRaw(person)
console.log(`响应式数据：`, person)  // Proxy(Object) {name: '张三', age: 18}
console.log(`原始数据：`, person2) // {name: '张三', age: 18}
</script>

<template>
  <div class="app">
    <div>person数据，名字是：{{ person.name }}，年龄是{{ person.age }}</div>
    <div>person2数据，名字是：{{ person2.name }}，年龄是{{ person2.age }}</div>
		// 可以修改年龄，可以引起视图更新。
    <button @click="person.age++; console.log(person)">change person age</button>
		// 可以修改年龄，不会引起视图更新。
    <button @click="person2.age++; console.log(person2)">change person2 name</button>
  </div>
</template>
```

`markRaw()`将一个对象标记为不可被转为代理。返回该对象本身。也就是用`reactive()`等转换为响应式数据不奏效。

```javascript
<script setup>
import { reactive, markRaw } from 'vue'
const person =markRaw({ name: '张三', age: 18})
let person2 = reactive(person)
console.log(person)  // 原始数据{ name: '张三', age: 18}
console.log(person2) // 即使转换reactive()也不奏效，仍是原始数据{ name: '张三', age: 18}
</script>

<template>
  <div class="app">
    <div>person数据，名字是：{{ person.name }}，年龄是{{ person.age }}</div>
    <div>person2数据，名字是：{{ person2.name }}，年龄是{{ person2.age }}</div>
		// 控制台年龄可以改变，但是页面不改变
    <button @click="person.age++; console.log(person)">change person age</button>
		// 控制台年龄可以改变，但是页面不改变
    <button @click="person2.age++; console.log(person2)">change person2 name</button>
  </div>
</template>
```

## customRef

`customRef()` 预期接收一个工厂函数作为参数，这个工厂函数接受 `track` 和 `trigger` 两个函数作为参数，并返回一个带有 `get` 和 `set` 方法的对象。

`track()`依赖追踪。当调用`get()`时，Vue会记录当前属性的访问情况，并将依赖它的组件或计算属性记录下来。

`trigger()`负责触发更新。当调用`set()`时，属性的值发生改变，Vue会通知所有依赖这个属性的组件或计算属性，重新执行或重新渲染，以确保视图和数据同步。

```javascript
let initvalue = ''
let msg = customRef((track, trigger) => {
  	// 返回的是一个对象，里面包含get()和set()
    return {
      get() {
        track()
        return initValue  // 定义好初始值
      },
      set(value) {
        initValue = value
        trigger()
      }
    }
  }
```

一般定义为hooks

/utils/useMsgDelay.ts

```javascript
import { customRef } from 'vue'
// 一定义，修改内容，页面立马更新
// const msg = ref('你好')

// 希望延迟1s页面才更新
export default function(initValue:string, delay:number) {
  // 防抖处理
  let timer = null
  let msg = customRef((track, trigger) => {
    return {
      get() {
        track()
        console.log('get')
        return initValue
      },
      set(value) {
        // 一输入就清空，确保最后一次输入才有效，延迟更新
        clearTimeout(timer)
        timer = setTimeout(() => {
          initValue = value
          console.log('set', value)
          trigger()
        }, delay)
      }
    }
  })
  return { msg }
}
```



```vue
<script setup>
import useMsgDelay from '@/utils/useMsgDelay'

let { msg } = useMsgDelay('Hello', 2000)
</script>

<template>
  <div class="app">
    {{ msg }}
    <input type="text" placeholder="请输入内容" v-model="msg">
  </div>
</template>
```

