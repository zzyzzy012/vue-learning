# Vue内置组件

## keepAlive

<KeepAlive>在多个组件间动态切换时缓存被移除的组件实例。

默认情况下，一个组件实例在被替换掉后会被销毁。这会导致它丢失其中所有已变化的状态——当这个组件再一次被显示时，会创建一个只带有初始状态的新实例。

想要组件能在被“切走”的时候保留它们的状态，可以用 `<KeepAlive>` 内置组件将这些动态组件包装起来。

```vue
<!-- 非活跃的组件将会被缓存！ -->
<KeepAlive>
  <component :is="activeComponent" />
</KeepAlive>
```

`<KeepAlive>` 默认会缓存内部的所有组件实例，但我们可以通过 `include` 和 `exclude` prop 来定制该行为。这两个 prop 的值都可以是一个以英文逗号分隔的字符串、一个正则表达式，或是包含这两种类型的一个数组：

```vue
<!-- 以英文逗号分隔的字符串 -->
<KeepAlive include="a,b">
  <component :is="view" />
</KeepAlive>

<!-- 正则表达式 (需使用 `v-bind`) -->
<KeepAlive :include="/a|b/">
  <component :is="view" />
</KeepAlive>

<!-- 数组 (需使用 `v-bind`) -->
<KeepAlive :include="['a', 'b']">
  <component :is="view" />
</KeepAlive>
```

它会根据组件的 `name`选项进行匹配，所以组件如果想要条件性地被 `KeepAlive` 缓存，就必须显式声明一个 `name` 选项。

```javascript
<script setup>
defineOptions({
  name: 'MyComponent' // 显式声明组件的 name
})
// 相当于传统的 export default
export default {
  name: 'ComponentA', // 显式声明 name 选项
};
</script>
```

一个持续存在的组件可以通过 `onActivated(() => {})`和 `onDeactivated(() => {})`注册相应的两个状态的生命周期钩子。

- `onActivated` 在组件挂载时也会调用，并且 `onDeactivated` 在组件卸载时也会调用。
- 这两个钩子不仅适用于 `<KeepAlive>` 缓存的根组件，也适用于缓存树中的后代组件。

## Teleport

`<Teleport>` 接收一个 `to` prop 来指定传送的目标。`to` 的值可以是一个 CSS 选择器字符串，也可以是一个 DOM 元素对象。可以多次使用<Teleport>，内容将按顺序添加到目标元素上。

```vue
<script setup>
// 禁用
let isMobile = true
<Teleport :disabled="isMobile">
  <div v-if="open" class="modal">
    <p>Hello from the modal!</p>
    <button @click="open = false">Close</button>
  </div>
</Teleport>
</script>
```

场景：一个组件模板的一部分在逻辑上从属于该组件，但从整个应用视图的角度来看，它在 DOM 中应该被渲染在整个 Vue 应用外部的其他地方。比如，弹窗一般都以整个body为准，而不是其父元素。希望触发弹窗的按钮和弹窗本身是在同一个组件中，因为它们都与组件的开关状态有关。但这意味着该弹窗将与按钮一起渲染在应用 DOM 结构里很深的地方。这会导致该弹窗的 CSS 布局代码很难写。

<MyModal>

```vue
<script setup>
import { ref } from 'vue'

const open = ref(false)
</script>

<template>
  <button @click="open = true">Open Modal</button>

  <div v-if="open" class="modal">
    <p>Hello from the modal!</p>
    <button @click="open = false">Close</button>
  </div>
</template>

<style scoped>
.modal {
  position: fixed;
  z-index: 999;
  top: 20%;
  left: 50%;
  width: 300px;
  margin-left: -150px;
}
</style>
```

- `position: fixed` 能够相对于浏览器窗口放置有一个条件，那就是不能有任何祖先元素设置了 `transform`、`perspective` 或者 `filter` 样式属性。
- 这个模态框的 `z-index` 受限于它的容器元素。如果有其他元素与 `<div class="outer">` 重叠并有更高的 `z-index`，则它会覆盖住我们的模态框。

```vue
<button @click="open = true">Open Modal</button>

<Teleport to="body">
  <div v-if="open" class="modal">
    <p>Hello from the modal!</p>
    <button @click="open = false">Close</button>
  </div>
</Teleport>
```

## suspence

`<Suspense>` 是一项实验性功能。在组件树中协调对异步依赖的处理。它让我们可以在组件树上层等待下层的多个嵌套异步依赖项解析完成，并可以在等待时渲染一个加载状态。

简单来说：子组件里面有异步任务，并且还需要用到返回的结果，需要在父组件中用<suspence>，里面包裹<template v-slot="default || fallback">，需要使用插槽，再在里面包裹子组件。

app.vue

```vue
<script setup>
import Sentence from '@/components/Sentence.vue'
</script>

<template>
  <div class="app">
    <Suspense>
      <template #default>
        <Sentence />
      </template>
    </Suspense>
  </div>
</template>
```

Sentence.vue

```vue
<script setup name="Sentence">
import axios from 'axios'
import { reactive } from 'vue'
let randomList = reactive([])
const getRandomList = async () => {
  const {data: {code, content}} = await axios.get('https://api.uomg.com/api/rand.qinghua?format=json')
  // console.log(data)
  console.log(code,content)
  randomList.push({id: code, content})
}
</script>

<template>
  <div class="sentence">
    <button @click="getRandomList">获取句子</button>
    <ul>
      <li v-for="item in randomList" :key="item.id">{{ item.content }}</li>
    </ul>
  </div>
</template>
```

