# Vue-basic

## 模板语法

文本插值：`{{}}`，`<span>Message: {{ msg }}</span>`

`v-html`:会识别标签，`<span v-html="rawHtml"></span></p>`

`v-text`:不会识别标签

`v-bind`:简写`:`，响应式绑定attribute，`<button :disabled="isButtonDisabled">Button</button>`

在 Vue 模板内，JavaScript 表达式可以被使用在如下场景上：

- 在文本插值中 (双大括号)
- 在任何 Vue 指令 (以 `v-` 开头的特殊 attribute) attribute 的值中



## 响应式基础

**`ref()`**: 可用于基本类型（如 `number`、`string`、`boolean` 等）的响应式数据，或者复杂数据类型。`ref()` 会将这些基本类型包装在一个对象中，并使用 `.value` 来访问和更新数据。

**`reactive()`**: 用于对象、数组等复杂数据类型。`reactive()` 不需要 `.value`，它会将整个对象转化为响应式对象。表单数据多用`reactive()`包装。

在模板引用时不需要`.value`，可以直接引用。

```vue
<script setup>
import { ref, reactive } from 'vue'

const count = ref(0)
const obj = reactive({ count: 1 })
const increment = () => {
  count.value++
  obj.count++
}
</script>
<template>
  <div>{{ count }}</div>
  <div>{{ obj.count }}</div>
	<button @click="increment">click me increment count</button>
</template>
```



## 计算属性

计算属性：根据其他响应式数据的变化而动态计算得出，**计算属性默认是只读的**，提供`getter`和`setter`方法。计算属性缓存 vs 方法：**计算属性值会基于其响应式依赖被缓存**。

```javascript
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  // getter
  get() {
    return firstName.value + ' ' + lastName.value
  },
  // setter
  set(newValue) {
    // 注意：我们这里使用的是解构赋值语法
    [firstName.value, lastName.value] = newValue.split(' ')
  }
})
fullName.value = 'Jane Smith'
</script>
```



## 侦听器

```javascript
// 第一个参数要监视的数据
// 第二个参数监视的回调
// 第三个参数配置对象
watch(要侦听的对象, () => {}, {
	// 可选
  deep: true, // 耗费性能
  immediate: true, // 获取初始数据，再在相关状态更改时重新请求数据
  once: true
})
```

监视四种类型数据

1. `ref()`定义的数据，可以监视基本数据类型或者对象类的复杂数据类型。
   - `ref()`如果是对象等复杂数据类型，监视的是地址。
   - 只改变对象属性值，newVal和oldVal是同一个值，若想监视，配置`deep: true`。
   - 改变整个对象，newVal, oldVal才不同。

```vue
<script setup>
import { ref, watch } from 'vue'

let sum = ref(100)

const stopWatch = watch(sum, (newVal, oldVal) => {
  console.log(sum, newVal, oldVal)
  // 停止监听
  if (newVal > 1000) {
    stopWatch()
  }
})

</script>

<template>
  <div>
    <p>sum值:{{ sum }}</p>
    <button @click="sum += 100">add sum 100</button>
  </div>
</template>
```

```vue
<script setup>
import { ref, watch } from 'vue'

let user = ref({
  name: '张三',
  age: 20
})

watch(user, (newVal, oldVal) => {
  // ref()如果是对象等复杂数据类型，监视的是地址
  // 只改变对象属性值，newVal和oldVal是同一个值
  // 改变整个对象，newVal, oldVal才不同
  console.log(user, newVal, oldVal)
})

</script>

<template>
  <div>
    <p>user姓名:{{ user.name }}, user年龄： {{ user.age }}</p>
    <button @click="user.name += '~'">change name</button>
    <button @click="user.age += 1">change age</button>
    <button @click="user = {name: '李四', age: 33}">change user</button>
  </div>
</template>
```

2. `reactive()`定义的对象类型数据，默认是开启深度监视的，且无法关闭。

- 监视整个响应式数据，newVal 和oldVal同一个值。
- 监视对象类型数据的某个属性
  1. 如果是基本类型，需要写成`get()`返回一个值，`() => `，newVal和oldVal同一个值。
  2. 如果是对象复杂类型
     - 如果直接写，能监视具体属性值变化，但不能监视整体变化，newVal 和 oldVal相同。
     - 如果是`() => `形式，能监视整体变化，但不能监视具体属性值变化。newVal 和 oldVal 不同。
     - **推荐函数式！如果希望能监视细枝末节，加`deep: true`。**

```vue
<script setup>
import { reactive, watch } from 'vue'

let user = reactive({
  name: '张三',
  age: 20
})

// 如果直接写，能监视具体属性值变化，但不能监视整体变化，newVal 和 oldVal相同。
// 如果是 () => 形式，能监视整体变化，但不能监视具体属性值变化。newVal 和 oldVal 不同。
watch(() =>user.toy, (newVal, oldVal) => {
  console.log(user, newVal, oldVal)
})

</script>

<template>
  <div>
    <p>user姓名:{{ user.name }}, user年龄： {{ user.age }}</p>
    <button @click="user.name += '~'">change name</button>
    <button @click="user.age += 1">change age</button>
    <!-- 要完全修改user,需要使用Object.assign()方法，重新分配里面的属性和值 -->
    <button @click="user = Object.assign(user, {name: '李四', age: 33})">change user</button>
  </div>
</template>
```

```vue
<script setup>
import { reactive, watch } from 'vue'

let user = reactive({
  name: '张三',
  age: 20,
  toy: {
    doll1: '小熊猫',
    doll2: '小兔子'
  }
})

watch(() => user.toy, (newVal, oldVal) => {
  console.log(user, newVal, oldVal)
})

</script>

<template>
  <div>
    <p>姓名:{{ user.name }}, 年龄： {{ user.age }}</p>
    <p>玩具1:{{ user.toy.doll1 }}, 玩具2： {{ user.toy.doll2 }}</p>
    <button @click="user.name += '~'">change name</button>
    <button @click="user.age += 1">change age</button>
    <button @click="user.toy.doll1 = '小狗'">change doll1</button>
    <button @click="user.toy.doll1 = '小猫'">change doll2</button>
    <button @click="user.toy = {doll1: '🐕', doll2: '🐱'}">change user</button>
  </div>
</template>
```

3. 监视数组`[]`

```vue
<script setup>
import { ref, watch } from 'vue'

let temp = ref(20)
let height = ref(40)

watch([temp, height], (val) => {
  let [newTemp, newHeight] = val
  console.log(newTemp, newHeight)
})
</script>

<template>
  <div>
    <p>当前水温是:{{ temp }}℃, 当前水位是：{{ height }}cm。</p>
    <button @click="temp += 10">change temp</button>
    <button @click="height += 10">change height</button>
  </div>
</template>
```



```javascript
const obj = reactive({ count: 0 })

// 不能直接侦听响应式对象的属性值
// 错误，因为 watch() 得到的参数是一个 number，数据不是响应式不知道数据动态变化的状态
watch(obj.count, (count) => {
  console.log(`Count is: ${count}`)
})

// 正确示范，getter函数，每次数据更新动态获取新值
watch(
  () => obj.count,
  (count) => {
    console.log(`Count is: ${count}`)
  }
)
```

`watchEffect()`自动跟踪回调的响应式依赖，回调会立即执行，不需要指定 `immediate: true`。

立即运行一个函数，同时响应式地追踪其依赖，并在依赖更改时重新执行。

有多个依赖项的侦听器来说，使用 `watchEffect()` 可以消除手动维护依赖列表的负担。

如果你需要侦听一个嵌套数据结构中的几个属性，`watchEffect()` 可能会比深度侦听器更有效，因为它将只跟踪回调中被使用到的属性，而不是递归地跟踪所有的属性。

```javascript
<script setup>
import { ref, watchEffect } from 'vue'

let temp = ref(20)
let height = ref(40)

// watch惰性，指明监视谁才会监视
// watch([temp, height], (val) => {
//   let [newTemp, newHeight] = val
//   console.log(newTemp, newHeight)
// })
watchEffect(() => {
  if (temp.value > 50 || height.value > 60) {
    console.log('给服务器发请求')
  }
})
</script>

<template>
  <div>
    <p>当前水温是:{{ temp }}℃, 当前水位是：{{ height }}cm。</p>
    <button @click="temp += 10">change temp</button>
    <button @click="height += 10">change height</button>
  </div>
</template>
```



## 表单输入绑定

`v-model` 可以用于各种不同类型的输入，`<textarea>`、`<select>` 元素。它会根据所使用的元素自动使用对应的 DOM 属性和事件组合：

- 文本类型的 `<input>` 和 `<textarea>` 元素会绑定 `value` property 并侦听 `input` 事件；
- `<input type="checkbox">` 和 `<input type="radio">` 会绑定 `checked` property 并侦听 `change` 事件；
- `<select>` 会绑定 `value` property 并侦听 `change` 事件。

修饰符：

- `v-model.lazy` 修饰符来改为在每次 `change` 事件后更新数据
- `v-model.number`
- `v-model.trim`

```javascript
<input
  :value="text"
  @input="event => text = event.target.value">
<input v-model="text">
```



```vue
<script setup>
import { ref } from 'vue'

const message = ref('hello Vue3')
const mutipleMessage = ref('hello my baby')

const checked = ref(true)
const checkedNames = ref([])

const picked = ref('One')

const selected = ref('')
</script>

<template>
  <!-- 单行文本 -->
  <input type="text" v-model="message" placeholder="edit me">
  <p>show message:{{ message  }}</p>
  <!-- 多行文本 -->
  <textarea v-model="mutipleMessage" placeholder="edit me[textarea]"></textarea>
  <p>show mutipleMessage:{{ mutipleMessage }}</p>
  
  
  <!-- 复选框 -->
  <input type="checkbox" id="checkbox" v-model="checked">
  <label for="checkbox">checkbox</label>
  <span>checked state: {{ checked }}</span>
  <div>Checked names: {{ checkedNames }}</div>

  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames" />
  <label for="jack">Jack</label>

  <input type="checkbox" id="john" value="John" v-model="checkedNames" />
  <label for="john">John</label>

  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames" />
  <label for="mike">Mike</label>
  
  
  <!-- 单选框 -->
  <div>Picked: {{ picked }}</div>

  <input type="radio" id="one" value="One" v-model="picked" />
  <label for="one">One</label>

  <input type="radio" id="two" value="Two" v-model="picked" />
  <label for="two">Two</label>

  <!-- 选择器 -->
  <div>Selected: {{ selected }}</div>
  <select v-model="selected">
    <option disabled value="">Please select one</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
</template>
```



## class 与 Style 绑定

`:class`

1. 通过对象动态地应用类名，可以根据条件为元素添加或移除特定的类。
2. 通过数组可以同时应用多个类，数组中的每个值代表一个类名，数组中的布尔表达式可以控制多个类的动态添加。

`:style` 通过对象动态地设置元素的内联样式。对象的键是 CSS 属性，值是对应的样式值。也支持 kebab-cased 形式的 CSS 属性 key。

```vue
<script setup>
  const isActive = ref(true)
	const hasError = ref(false)
	const activeColor = ref('red')
	const fontSize = ref(30)
</script>

<template>
	<div
    class="static"
    :class="{ active: isActive, 'text-danger': hasError }"
	></div>
	<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
	<div :style="{ 'font-size': fontSize + 'px' }"></div>
</template>
```



## 条件渲染

`v-if`和`v-else`和`v-else-if`，连在一起使用。

`v-if`按条件渲染，因为它确保了在切换时，条件区块内的事件监听器和子组件都会被销毁与重建。

`v-if`优先级高于`v-for`。

```vue
<script setup>
import { ref, reactive } from 'vue'

const weatherOptions = reactive(['sunny', 'rainy', 'cloudy', 'unknown'])
const weather = ref('sunny')

const changeWeather = () => {
  const randomIndex = Math.floor(Math.random() * weatherOptions.length)
  weather.value = weatherOptions[randomIndex]
}
</script>

<template>
  <div>
    <div v-if="weather === 'sunny'">
      <p>今天是晴天，记得带上太阳镜！😎</p>
    </div>
    <div v-else-if="weather === 'rainy'">
      <p>今天下雨，记得带伞哦！🌧️</p>
    </div>
    <div v-else-if="weather === 'cloudy'">
      <p>今天多云，气温适中。⛅</p>
    </div>
    <div v-else>
      <p>天气状况不明，请稍后再试。🤔</p>
    </div>
    <button @click="changeWeather">切换天气</button>
  </div>
</template>
```



`v-show`

1.  会在 DOM 渲染中保留该元素；`v-show` 仅切换了该元素上名为 `display` 的 CSS 属性。
2. `v-show` 不支持在 `<template>` 元素上使用，也不能和 `v-else` 搭配使用。
3. 如果需要频繁切换，则使用 `v-show` 较好；如果在运行时绑定条件很少改变，则 `v-if` 会更合适。




## 列表渲染

`v-for`: `v-for="(item, index) in items" :key=item.id`，`index`参数可选

```vue
<script setup>
import { reactive } from 'vue'

const fruits = reactive(['apple', 'banna', 'orange', 'pear', 'grape'])
</script>

<template>
  <div>
    <ul>
      <li v-for="item in fruits" :key="item">{{ item }}</li>
    </ul>
    <!-- n从1开始，而不是从0开始 -->
    <span v-for="n in 10">{{ n }}</span>
  </div>
</template>
```



## 事件处理

`v-on`简写为`@`，监听 DOM 事件，`v-on:click="handler"` 或 `@click="handler"`。

```vue
<script setup>
import { reactive } from 'vue'

const user = reactive({
  age: 18
})
const addAge = () => {
  user.age++
}
</script>

<template>
  <div>
    <span>{{ user.age }}</span>
    <button @click="addAge">add user age</button>
  </div>
</template>
```

**事件修饰符**:`.stop`,`.prevent`,`.self`,`.capture`,`.once`,`.passive`

**按键修饰符**:`.enter`,`.tab`,`.delete` (捕获“Delete”和“Backspace”两个按键),`.esc`,`.space`,`.up`,`.down`,`.left`,`.right`

**系统案件修饰符**:`.ctrl`,`.alt`,`.shift`,`.meta`

**鼠标按键修饰符**:`.left`,`.right`,`.middle`



## 模板引用

获取DOM元素，在DOM元素上绑定`ref`属性，通过`useTemplateRef()`获取。

```vue
<script setup>
import { useTemplateRef, onMounted } from 'vue'

// 第一个参数必须与模板中的 ref 值匹配
const input = useTemplateRef('my-input')

onMounted(() => {
  input.value.focus()
})
</script>

<template>
  <input ref="my-input" />
</template>
```

对于`v-for`，`ref`获取的是数组

```vue
<script setup>
import { ref, useTemplateRef, onMounted } from 'vue'

const list = ref([
  'apple',
  'banana',
  'orange'
])

const itemRefs = useTemplateRef('items')

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="items">
      {{ item }}
    </li>
  </ul>
</template>
```

组件上的`ref`

```vue
<script setup>
import { useTemplateRef, onMounted } from 'vue'
import Child from './Child.vue'

const childRef = useTemplateRef('child')

onMounted(() => {
  // childRef.value 将持有 <Child /> 的实例
})
</script>

<template>
  <Child ref="child" />
</template>
```

使用了 `<script setup>` 的组件是**默认私有**的：一个父组件无法访问到一个使用了 `<script setup>` 的子组件中的任何东西，除非子组件在其中通过 `defineExpose` 宏显式暴露：

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

// 像 defineExpose 这样的编译器宏不需要导入
defineExpose({
  a,
  b
})
</script>
```

