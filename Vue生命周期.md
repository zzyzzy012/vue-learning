# Vue生命周期

## Vue3生命周期

每个 Vue 组件实例在创建时都需要经历一系列的初始化步骤，比如设置好数据侦听，编译模板，挂载实例到 DOM，以及在数据改变时更新 DOM。在此过程中，它也会运行被称为生命周期钩子的函数，让开发者有机会在特定阶段运行自己的代码。

![](D:\学习\program\前端学习\Vue\Vue-notes\images\lifecycle_zh-CN.W0MNXI0C.png)

## Vue3生命周期钩子

`setup()`。

`onBeforeMount():`在组件被挂载之前被调用。组件已经完成了其响应式状态的设置，但还没有创建 DOM 节点。它即将首次执行 DOM 渲染过程。

`onMounted` :可以用来在组件完成初始渲染并创建 DOM 节点后运行代码。

`onBeforeUpdate()`:在组件即将因为响应式状态变更而更新其 DOM 树之前调用。

`onUpdated()`:在组件因为响应式状态变更而更新其 DOM 树之后调用。用来在 Vue 更新 DOM 之前访问 DOM 状态。

`onBeforeUnmount()`:在组件实例被卸载之前调用。

`onUnmounted()`注册一个回调函数，在组件实例被卸载之后调用。

- 其所有子组件都已经被卸载。
- 所有相关的响应式作用 (渲染作用以及 `setup()` 时创建的计算属性和侦听器) 都已经停止。


- 可以在这个钩子中手动清理一些副作用，例如计时器、DOM 事件监听器或者与服务器的连接。

```vue
<script setup>
import { ref, onBeforeMount, onMounted, onBeforeUpdate, onUpdated, onBeforeUnmount, onUnmounted } from 'vue'

const movie = ref('怦然心动')
onBeforeMount(() => {
  console.log('before mount')
})
onMounted(() => {
  console.log('mounted')
})
onBeforeUpdate(() => {
  console.log('before update')
  console.log(movie.value)
})
onUpdated(() => {
  console.log('updated')
})
onBeforeUnmount(() => {
  console.log('before unmount')
})
onUnmounted(() => {
  console.log('unmounted')
})
</script>

<template>
  <div>
    <p>我推荐的电影是：{{ movie }}</p>
    <button @click="movie = '肖申克的救赎'">点我换电影</button>
  </div>
</template>
```



## Vue2生命周期

1. **创建阶段**：初始化数据和事件监听。
2. **挂载阶段**：将组件挂载到 DOM 上。
3. **更新阶段**：当数据更新时重新渲染 DOM。
4. **销毁阶段**：清理数据和事件。

![](D:\学习\program\前端学习\Vue\Vue-notes\images\lifecycle.png)

## Vue2生命周期钩子

1. `beforeCreated() {}`

   - **执行时机**：实例刚开始初始化，还未开始创建 `data`、`computed` 和 `methods` 等属性。
   - **场景**：很少使用，因为此时没有 `data` 等属性可以访问。
2. `created() {}`

   - **执行时机**：实例已创建完成，`data`、`methods` 等已初始化，但还未挂载到 DOM。

   - **场景**：适合在组件创建后**初次获取数据**，如通过 `axios` 发送 API 请求获取数据。

3. `beforeMount() {}`
   - **执行时机**：在挂载到 DOM 之前，模板已经编译好，但还未插入真实 DOM。

   - **场景**：此时一般不需要操作 DOM，因为它尚未挂载。

4. `mounted() {}`（最常用之一）
   - **执行时机**：组件挂载完成，DOM 已经生成，可以进行 DOM 操作。

   - **场景**：适合进行**DOM 操作**，如第三方库的初始化，或对 DOM 进行修改等。

5. `beforeUpdate() {}`
   - **执行时机**：当数据发生变化，但还未重新渲染 DOM。

   - **场景**：可以在数据更新但未重新渲染前做一些逻辑处理。

6. `updated() {}`
   - **执行时机**：数据更新并且 DOM 重新渲染完成。

   - **场景**：适合在 DOM 更新后执行某些操作，比如根据新数据更新页面或状态。

7. `beforeDestroy() {}`
   - **执行时机**：组件实例即将销毁，还未清理事件监听器和子组件。

   - **场景**：适合在组件销毁前清理计时器、事件监听等。常用于手动销毁 `setInterval`、`eventListener` 等防止内存泄漏。

8. `destroyed() {}`
   - **执行时机**：组件实例已销毁，数据绑定和事件监听已清理干净。
   - **场景**：很少使用，但如果需要确认组件已完全销毁，可以使用此钩子。

```javascript
beforeCreate() {
  console.log('beforeCreate: 数据和事件未初始化');
}
created() {
  console.log('created: 组件已创建，可以访问 data 和 methods');
  this.fetchData();  // 初次获取数据
}
beforeMount() {
  console.log('beforeMount: 即将挂载到 DOM');
}
mounted() {
  console.log('mounted: DOM 已经挂载，可以操作 DOM');
  this.initializeCarousel();  // 初始化轮播图插件等
}
beforeUpdate() {
  console.log('beforeUpdate: 数据变化，但 DOM 还未更新');
}
updated() {
  console.log('updated: DOM 已重新渲染');
}
beforeDestroy() {
  console.log('beforeDestroy: 即将销毁组件，清理事件等');
  clearInterval(this.timer);
}
destroyed() {
  console.log('destroyed: 组件已销毁');
}
```

