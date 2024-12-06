# Vue组件

## 组件注册

```javascript
// 全局注册
// 1.导入组件
import MyComponent from '@/components/MyComponent.vue'
// 2.全局注册
app.component('MyComponent', MyComponent)
// 支持链式调用
app
  .component('ComponentA', ComponentA)
  .component('ComponentB', ComponentB)
  .component('ComponentC', ComponentC)
```

如果在@/components/PageContainer.vue注册的组件，可以不用引入，直接使用。

```vue
<script setup>
// 局部注册
// 1.导入注册，直接使用
import ComponentA from './ComponentA.vue'
</script>

<template>
	<!-- 2.使用 -->
  <ComponentA />
</template>
```

## Props

在父组件中使用子组件，通过`props`传递数据给子组件。

```vue
<script setup>
import { ref } from 'vue'
import ChildComponent from '@/components/ChildComponent.vue'

const title = ref('hello vue3')

</script>

<template>
  <div>
    <ChildComponent :title="title"></ChildComponent>
  </div>
</template>
```

`props`命名格式：

如果一个 `prop` 的名字很长，应使用 camelCase 形式，可以直接在模板的表达式中使用，也可以避免在作为属性 key 名时必须加上引号。在<template>模板内为了和 HTML attribute 对齐，通常会将其写为 kebab-case 形式。

```vue
defineProps({
  greetingMessage: String
})

<span>{{ greetingMessage }}</span>
<MyComponent greeting-message="hello" />
```

静态/动态props：

```vue
// 静态
<BlogPost title="My journey with Vue" />
// 动态v-bind简写:
<BlogPost :title="post.title" />
```

在子组件中通过 `defineProps()`接收父组件传递的值，告诉子组件会接收哪些外部数据。可以是数组类型`defineProps(['title'])`或者对象类型。

```vue
<script setup>
const props = defineProps({
  title: {
    type: String,
    required: true
  }
})

</script>

<template>
  <div>
    {{ props.title }}
  </div>
</template>
```

props校验

```javascript
const props = defineProps({
  propA: {
    type: String | [Array, Number],  // 类型，一种或多种类型
    required: true, // 是否必传
    default: ,// 默认值
    // 如果是对象或数组需要工厂函数返回
    default() {
    	return 
  	},
  	validator(value, props) { // 在 3.4+ 中完整的 props 作为第二个参数传入
   	 return
  	}
  }
})
```

```javascript
defineProps({
  // 基础类型检查
  propA: Number,
  // 多种可能的类型
  propB: [String, Number],
  // 必传，且为 String 类型
  propC: {
    type: String,
    required: true
  },
  // Number 类型的默认值
  propE: {
    type: Number,
    default: 100
  },
  // 对象类型的默认值
  propF: {
    type: Object,
    // 对象或数组的默认值
    // 必须从一个工厂函数返回。
    // 该函数接收组件所接收到的原始 prop 作为参数。
    default(rawProps) {
      return { message: 'hello' }
    }
  },
  // 自定义类型校验函数
  // 在 3.4+ 中完整的 props 作为第二个参数传入
  propG: {
    validator(value, props) {
      // The value must match one of these strings
      return ['success', 'warning', 'danger'].includes(value)
    }
  }
})
```

##  事件

**子组件使用 `defineEmits`** 来定义和发射事件

- 在子组件中，使用 `defineEmits` 来声明可以触发的事件。
- 通过`emits('事件名', 参数)`将该事件发射数据到父组件。
- 子组件的数据通过`defineEmits`来传递给父组件。

```javascript
<script setup>
const props = defineProps({
  title: {
    type: String,
    required: true
  },
  count: {
    type: Number,
    required: true
  }
})

const emits = defineEmits(['increment'])
const emitIncrement = (n) => {
  emits('increment', n)
}
</script>

<template>
  <div>
    {{ props.title }}
    <button @click="emitIncrement(5)">click me increment count: {{ count }}</button>
  </div>
</template>
```

父组件通过 `@事件名` 来监听子组件发射的事件。

当子组件发射事件时，父组件会接收到并执行相应的回调函数。

在父组件中可以使用 kebab-case 形式来监听，在模板中也推荐使用 kebab-case 形式来编写监听器。

```javascript
<script setup>
import { ref } from 'vue'
import ChildComponent from '@/components/ChildComponent.vue'

const title = ref('hello vue3')
const count = ref(0)

const onIncrement = (n) => {
  // count.value++
  count.value += 5
}
</script>

<template>
  <div>
    <ChildComponent :title="title" :count="count" @increment="onIncrement"></ChildComponent>
  </div>
</template>
```

## mitt

mitt官网：https://github.com/developit/mitt

使用

创建utils/emitter.js

```javascript
import mitt from 'mitt'

const emiiter = mitt()

export default emiiter
```

在main.js中导入

```javascript
import emiter from '@/utils/emitter'
app.use(emiter)
```

基本语法

```javascript
import mitt from 'mitt'

const emitter = mitt()

// listen to an event
emitter.on('foo', e => console.log('foo', e) )

// listen to all events
emitter.on('*', (type, e) => console.log(type, e) )

// fire an event
emitter.emit('foo', { a: 'b' })

// clearing all events
emitter.all.clear()

// working with handler references:
function onFoo() {}
emitter.on('foo', onFoo)   // listen
emitter.off('foo', onFoo)  // unlisten
```

## V-model

`v-model`在父组件的子组件上绑定，实现数据的双向绑定。

`defineModel()`在子组件上定义，可以修改父组件中传递过来的数据。

`defineModel()` 返回的值是一个 ref。它可以像其他 ref 一样被访问以及修改，不过它能起到在父组件和当前变量之间的双向绑定的作用：

- 它的 `.value` 和父组件的 `v-model` 的值同步；
- 当它被子组件变更了，会触发父组件绑定的值一起更新。

`defineModel` 是一个便利宏。编译器将其展开为以下内容：

- 一个名为 `modelValue` 的 prop，本地 ref 的值与其同步；
- 一个名为 `update:modelValue` 的事件，当本地 ref 的值发生变更时触发。

```vue
<!-- Child.vue -->
<script setup>
const model = defineModel()

function update() {
  model.value++
}
</script>

<template>
  <div>Parent bound v-model is: {{ model }}</div>
  <button @click="update">Increment</button>
</template>
```

父组件用 `v-model` 绑定一个值。

```vue
<!-- Parent.vue -->
<Child v-model="countModel" />
```

### 底层机制

vue2中，`v-model`是`:value`和`@input`的简写。

vue3中，`v-model`是`:modelValue=username`和`@update:modelValue="$event => (username = $event)"`的简写。如果是`input`元素，可以简写`:value="username"`和` @input="$event => (username = $event)"`。

在组件中通过`v-model`简写，需要用`defineProps`接收数据和`defineEmits`发送事件。

```vue
<!-- Parent.vue -->
<!-- 1.原生HTML元素 -->
<!-- <input v-model="username" />  // 简写 -->
<input
  :value="username"
  @input="username = $event.target.value"
/>
<!-- 2.用在组件上 -->
<!-- <Child v-model="username" />  // 简写 -->
<Child :modelValue="username" @update:modelValue="$event => (username = $event)" />

<!-- Child.vue -->
<script setup>
const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])
</script>

<template>
<!-- 因为是input元素，可以用:value和@input简写，其他元素需要写:value=modelValue和@update:modelValue -->
  <input
    :value="modelValue"
    @input="emit('update:modelValue', $event.target.value)"
  />
</template>
```

### 实战案例

ChannelSelect.vue，在子组件中通过`defineModel()`绑定父组件传递的数据，`v-model`也可以直接绑定。

```vue
<script setup>
import { useArticleStore } from '@/stores/index'
import { onMounted } from 'vue'

const articleStore = useArticleStore()
onMounted(() => articleStore.getChannelList())

const channelList = articleStore.channelList

// defineProps({
//   modelValue: {
//     type: [Number, String]
//   }
// })
// defineEmits(['update:modelValue'])

const model = defineModel({
  name: 'modelValue',
  type: [Number, String],
})
</script>

<template>
      <!-- :modelValue="modelValue"
    @update:modelValue="$emit('update:modelValue', $event)" -->
  <div class="channel-select" style="width: 100%;">
    <el-select
    placeholder="请选择"
    style="width: 100%;"
    v-model="model"
    >
      <!-- label为显示的文字，value为选中的值，一般是id -->
      <el-option v-for="channel in channelList" :key="channel.id" :label="channel.cate_name" :value="channel.id"></el-option>
    </el-select>
  </div>
</template>
```

在父组件中通过`v-model`绑定。

```vue
<!-- vue2中，v-model是:value和@input的简写 -->
<!-- vue3中，v-model是:modelValue=""和@update:modelValue的简写 -->
<!-- 运用到子组件的V-model绑定数据 -->
<Channel-select v-model="params.cate_id"></Channel-select>
```

## 透传 Attributes

**透传 attribute**指的是传递给一个组件，却没有被该组件声明为 props 或 emits 的 attribute 或者 `v-on` 事件监听器。最常见的例子就是 `class`、`style` 和 `id`。

**透传 attribute**是指由父组件传入，且没有被子组件声明为 props 或是组件自定义事件的 attributes 和事件处理函数。

如果一个子组件的根元素已经有了 `class` 或 `style` attribute，它会和从父组件上继承的值合并。同样的规则也适用于 `v-on` 事件监听器。

默认情况下，若是单一根节点组件，`$attrs` 中的所有属性都是直接自动继承自组件的根元素。而多根节点组件则不会如此，同时也可以通过配置`ingeritAttrs`选项来显式地关闭该行为。

`$attrs`:一个包含了组件所有透传 attributes 的对象。

父组件的数据传递给子组件

```vue
<!-- parent.vue -->
<script setup>
import { ref } from 'vue'
import ChildComponent from '@/components/ChildComponent'  
  
const a = ref('a')
const b = ref('b')
const c = ref('c')

</script>

<template>
  <div>
    <p>父组件数据展示：a:{{ a }}, b:{{ b }}, c: {{ c }}</p>
    <!-- v-bind="{}"可以更对象格式 -->
    <!-- 表示:a="a" :b="b" :c="c" -->
    <ChildComponent v-bind="{a: a, b: b, c: c}"></ChildComponent>
  </div>
</template>
```

如果子组件没有用`defineProps()`接收，则都在`$attrs`上，通过`v-bind="$attrs"`传递给孙组件。

```vue
<script setup>
import GrandChildComponent from './GrandChildComponent.vue'
</script>

<template>
  <div>
    <GrandChildComponent v-bind="$attrs"></GrandChildComponent>
  </div>
</template>
```

在孙组件上通过`defineProps()`接收子组件的`$attrs`。

```vue
<script setup>
defineProps(['a', 'b', 'c'])
</script>

<template>
  <div>
    <span>{{ a }}</span>
    <span>{{ b }}</span>
    <span>{{ c }}</span>
  </div>
</template>
```

反之，子组件数据传递给父组件，在父组件中定义方法接收，`:`将方法传递给子组件，子组件不需要改

```vue
<!-- parent.vue -->
<script setup>
import { ref } from 'vue'
import ChildComponent from '@/components/ChildComponent'  
  
const a = ref('a')
const b = ref('b')
const c = ref('c')

const receivedData = ref('')
const receiveGrandData = (value) => {
  receivedData.value = value
}
</script>

<template>
  <div>
    <p>父组件数据展示：a:{{ a }}, b:{{ b }}, c: {{ c }}</p>
    <!-- v-bind="{}"可以更对象格式 -->
    <!-- 表示:a="a" :b="b" :c="c" -->
    <ChildComponent v-bind="{a: a, b: b, c: c, receiveGrandData: receiveGrandData}"></ChildComponent>
  </div>
</template>
```

孙组件通过`defineProps()`接收方法

```vue
<script setup>
import { ref } from 'vue'

defineProps(['a', 'b', 'c', 'receiveGrandData'])

const d = ref('d')

</script>

<template>
  <div>
    <span>{{ a }}</span>
    <span>{{ b }}</span>
    <span>{{ c }}</span>
    <span>孙组件数据： {{ d }}</span>
    <button @click="receiveGrandData(d)">click me send d to parent.vue</button>
  </div>
</template>
```

## 依赖注入

`provide` 和 `inject` :一个父组件相对于其所有的后代组件，会作为**依赖提供者**。任何后代的组件树，无论层级有多深，都可以**注入**由父组件提供给整条链路的依赖。

![](D:\学习\program\前端学习\Vue\Vue-notes\images\provide-inject.C0gAIfVn.png)



`provide()`为组件后代提供数据。`provide()` 接受两个参数：**第一个参数被称为<u>注入名</u>**，可以是一个字符串或是一个 `Symbol`，**第二个参数是要提供的值**，值可以是任意类型，包括响应式的状态。

```javascript
<!-- parent.vue -->
<script setup>
import { ref, provide } from 'vue'
import { countSymbol } from './injectionSymbols'

// 提供静态值
provide('path', '/project/')

// 提供响应式的值
const count = ref(0)
provide('count', count)

// 提供时将 Symbol 作为 key
provide(countSymbol, count)
</script>
```

应用层`app.provide()`:除了在一个组件中提供依赖，还可以在整个应用层面提供依赖：在应用级别提供的数据在该应用内的所有组件中都可以注入。

```javascript
import { createApp } from 'vue'

const app = createApp({})

app.provide(/* 注入名 */ 'message', /* 值 */ 'hello!')
```

`inject()`注入一个由祖先组件或整个应用 (通过 `app.provide()`) 提供的值。

- 第一个参数是注入的 key。
- 第二个参数是可选的，即在没有匹配到 key 时使用的默认值。
- 第二个参数也可以是一个工厂函数，用来返回某些创建起来比较复杂的值。在这种情况下，你必须将 `true` 作为第三个参数传入，表明这个函数将作为工厂函数使用，而非值本身。

```javascript
<script setup>
import { inject } from 'vue'
import { countSymbol } from './injectionSymbols'

// 注入不含默认值的静态值
const path = inject('path')

// 注入响应式的值
const count = inject('count')

// 通过 Symbol 类型的 key 注入
const count2 = inject(countSymbol)

// 注入一个值，若为空则使用提供的默认值
const bar = inject('path', '/default-path')

// 注入一个值，若为空则使用提供的函数类型的默认值
const fn = inject('function', () => {})

// 注入一个值，若为空则使用提供的工厂函数
const baz = inject('factory', () => new ExpensiveObject(), true)
</script>
```

如果子传父，则在父组件中定义函数，`provide('函数名', 函数)`传递给子组件。

```vue
<!-- parent.vue -->
<script setup>
import { ref, provide } from 'vue'

const parentMoney = ref('100万元')
provide('parentMoney', parentMoney)

const getChildData = (value) => {
  parentMoney.value = value
}
provide('getChildData', getChildData)
</script>

<template>
  <div>
    <div>父亲的钱：{{ parentMoney }}</div>
  </div>
</template>
```

子组件`inject('接收')`

```javascript
<script setup>
import { ref, inject } from 'vue'
const childMoney = ref('10000万元')
const receivedData = inject('parentMoney')
const getChildData = inject('getChildData')
</script>

<template>
  <div>
    <div>孩子接收父组件的钱：{{ receivedData }}</div>
    <button @click="getChildData(childMoney)">getChildData</button>
  </div>
</template>
```

## `$refs`和`$parent`

`$refs`:一个包含 DOM 元素和组件实例的对象。

先在模板`<template>`中绑定方法，传入参数`$refs`获取，js定义方法，传入参数`refs`。

```vue
<!-- parent.vue -->
<script setup>
import { ref } from 'vue'
import ChildComponent from '@/components/ChildComponent.vue'
import ChildComponent2 from '@/components/ChildComponent2.vue'

const childRef1 = ref()
const childRef2 = ref()

const getChildRef = (refs) => {
  for ( let key in refs) {
    console.log(refs[key])
  }
}
</script>

<template>
  <div>
    <ChildComponent ref="childRef1"></ChildComponent>
    <ChildComponent2 ref="childRef2"></ChildComponent2>
    <div @click="getChildRef($refs)">ddd 拿childRef</div>
  </div>
</template>
```

`$parent`:当前组件可能存在的**父组件实例**，如果当前组件是顶层组件，则为 `null`。

要先在模板`<template>`中绑定方法，传入参数`$parent`获取，js定义方法，传入参数`parent`。

```vue
<!-- child.vue -->
<script setup>
const getParent = (parent) => {
  console.log(parent)
}
</script>

<template>
  <div>
    <div @click="getParent($parent)">ddd获取parentRef</div>
  </div>
</template>
```

`$refs`和`$parent`都要在`onmounted()`DOM元素挂载后才能获取。

## 插槽

`<slot>` 元素是一个**插槽出口** (slot outlet)，标示了父元素提供的**插槽内容** (slot content) 将在哪里被渲染。

`<slot>`用在子元素中，因为不确定内容，但确定结构，在父元素中通过`<template>`，在里面写内容。如果是具名插槽`<slot name="header">`，则在`<template v-slot:header>`或`<template #header>`中写内容。

![](D:\学习\program\前端学习\Vue\Vue-notes\images\slots.CKcE8XYd.png)

插槽内容可以访问到父组件的数据作用域，因为插槽内容本身是在父组件模板中定义的。

**默认内容**:在子组件的`<slot></slot>`定义默认内容即可。

`具名插槽`：带 `name` 的插槽。没有提供 `name` 的 `<slot>` 出口会隐式地命名为“default”。

要为具名插槽传入内容，使用一个含 `v-slot` 指令的 `<template>` 元素，并将目标插槽的名字传给该指令，可以简写为`#`，`#default`就是默认插槽，可以省略不写。

```vue
<BaseLayout>
  // <template v-slot:header> 可以简写为 <template #header>
  <template v-slot:header>
    <!-- header 插槽的内容放这里 -->
  </template>
</BaseLayout>
```

![](D:\学习\program\前端学习\Vue\Vue-notes\images\named-slots.CCIb9Mo_.png)

插槽内容**无法访问**子组件的数据。Vue 模板中的表达式只能访问其定义时所处的作用域，这和 JavaScript 的词法作用域规则是一致的。换言之：

> 父组件模板中的表达式只能访问父组件的作用域；子组件模板中的表达式只能访问子组件的作用域。

可以像对组件传递`props`那样，向一个插槽的出口上传递 attributes。

```vue
<!-- Game.vue 的模板 -->
<script setup>
import { ref, reactive } from 'vue'

const games = reactive([
   {id:'fdgsbfg01', name: '王者荣耀'} ,
   {id:'fdgsbfg02', name: '英雄联盟'} ,
   {id:'fdgsbfg03', name: '明日方舟'} ,
   {id:'fdgsbfg04', name: '第五人格'} ,
	])
const title = ref('我爱玩的游戏')
</script>

<template>
  <div class="game">
    <slot :game="games" :title="title">游戏</slot>
  </div>
</template>
```

当需要接收插槽`props`时，默认插槽和具名插槽的使用方式有一些小区别。

默认插槽接受`props`，通过子组件标签上的 `v-slot="slotProps"` 指令，直接接收到了一个插槽`props`对象:`slotProps`是一个对象，包括子组件向父组件传递的所有props数据。

```vue
<Game v-slot="slotProps">
  <h2>{{ slotProps.title }}</h2>
  <ul>
    <li v-for="game in slotProps.games" :key="game.id">{{ game.name}}</li>
  </ul>
</Game>
<!-- 对象解构 -->
<Game v-slot="{ games, title }">
  <h2>{{ title }}</h2>
  <ul>
    <li v-for="game in games" :key="game.id">{{ game.name}}</li>
  </ul>
</Game>
```

具名作用域插槽：向具名插槽传入`props`,父组件的`<template>`通过`v-slot:name="slotProps"`或者`#name="slotProps"`接收`props`。

```vue
<!-- Game.vue 的模板 -->
<template>
  <div class="game">
    <slot name = "game" :game="games" :title="title">游戏</slot>
  </div>
</template>
```



```vue
<Game>
  <template v-solt:game="gameProps">
    {{ gameProps }}
  </template>
</Game>
```

