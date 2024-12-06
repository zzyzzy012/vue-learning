# Vue-directive

自定义指令主要是为了重用涉及普通元素的底层 DOM 访问的逻辑。

一个自定义指令由一个包含类似组件生命周期钩子的对象来定义。钩子函数会接收到指令所绑定元素作为其参数。

```vue
<script setup>
// 在模板中启用 v-focus
const vFocus = {
  mounted: (el) => el.focus()
}
</script>

<template>
  <input v-focus />
</template>
```

没有`setup`函数，定义自定义指令，通过`directives`，局部注册自定义指令。

```javascript
export default {
  setup() {
    /*...*/
  },
  directives: {
    // 在模板中启用 v-focus
    focus: {
      /* ... */
    }
  }
}
```

定义全局注册指令，通过`app.directive('指令名称', {}对象 | () => {}函数简写)`

```javascript
const app = createApp({})

// 使 v-focus 在所有组件中都可用
app.directive('focus', {
  /* ... */
})
```

例子

```vue
<script setup>
import { ref } from 'vue'

let n = ref(100)
const bigNum = (element, binding ) => {
  element.innerHTML = binding.value * 10
}

</script>

<template>
	<span v-big-num="n"><span>
</template>
```

## 指令钩子

- `el`：指令绑定到的元素。这可以用于直接操作 DOM。
- `binding`：一个对象，包含以下属性。
  - `value`：传递给指令的值。例如在 `v-my-directive="1 + 1"` 中，值是 `2`。
  - `oldValue`：之前的值，仅在 `beforeUpdate` 和 `updated` 中可用。无论值是否更改，它都可用。
  - `arg`：传递给指令的参数 (如果有的话)。例如在 `v-my-directive:foo` 中，参数是 `"foo"`。

```javascript
const myDirective = {
  // 在绑定元素的 attribute 前
  // 或事件监听器应用前调用
  created(el, binding, vnode) {
    // 下面会介绍各个参数的细节
  },
  // 在元素被插入到 DOM 前调用
  beforeMount(el, binding, vnode) {},
  // 在绑定元素的父组件
  // 及他自己的所有子节点都挂载完成后调用
  mounted(el, binding, vnode) {},
  // 绑定元素的父组件更新前调用
  beforeUpdate(el, binding, vnode, prevVnode) {},
  // 在绑定元素的父组件
  // 及他自己的所有子节点都更新后调用
  updated(el, binding, vnode, prevVnode) {},
  // 绑定元素的父组件卸载前调用
  beforeUnmount(el, binding, vnode) {},
  // 绑定元素的父组件卸载后调用
  unmounted(el, binding, vnode) {}
}
```

函数简写形式：仅仅需要在 `mounted` 和 `updated` 上实现相同的行为，除此之外并不需要其他钩子。

```javascript
// 指令会在模板重新渲染时再次调用
const xxx = (element, binding) => {
 // element 真实的dom元素
 // binding绑定，可以获取value
}
```

# Vue-Plugins

插件 (Plugins) 是一种能为 Vue 添加全局功能的工具代码。

定义插件，导出`myPlugin`。

```js
export const myPlugin = {
  install(app, options) {
    // 配置此应用
  }
}
```

在main.js中使用myPlugin插件。

```js
import { myPlugin } from '@/.../myPlugin.js'

app.use(myPlugin, {
  /* 可选的选项 */
})
```

