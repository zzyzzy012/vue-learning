# Vue-Router-basic

## 单页面应用

客户端路由的作用是在单页应用 (SPA) 中将浏览器的 URL 和用户看到的内容绑定起来。当用户在应用中浏览不同页面时，URL 会随之更新，但页面不需要从服务器重新加载。

![](D:\学习\program\前端学习\Vue\Vue-notes\images\spaVSmpa.png)

- 单页应用类网站：系统类网站 / 内部网站 / 文档类网站 / 移动端站点
- 多页应用类网站：公司官网 / 电商类网站

## 创建路由器实例

create-vue脚手架工具，自动配置好`router`，路由器实例是通过调用 `createRouter()` 函数创建的。

配合路由的组件放在`@/views/xxx.vue`文件夹中。

`component:`可以是页面开头`import xxx from '@/views/xxx.vue'`，或者箭头函数`() => {}`返回`@/views/xxx.vue`下面的路由组件。

`src/router/index.js`中创建路由器实例，并且默认导出。

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import news from '@/views/news.vue'
import movie from '@/views/movie.vue'
import comic from '@/views/comic.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
  	{path: '/news', component:() => import('@/views/news.vue')},
    {path: '/movie', component:() => import('@/views/movie.vue')},
    { path: '/comic', component: () => import('@/views/comic.vue') },
  ]
})

export default router
```

## 注册路由器插件

`main.js`文件导入`router`，使用`.use()`将其注册为插件。和大多数的 Vue 插件一样，`use()` 需要在 `mount()` 之前调用。

插件作用：

1. 全局注册`RouterView` 和 `RouterLink` 组件。
2. 添加全局 `$router` 和 `$route` 属性。
3. 启用 `useRouter()` 和 `useRoute()` 组合式函数。
4. 触发路由器解析初始路由。

```javascript
// main.js导入router
import { createApp } from 'vue'
import { createPinia } from 'pinia'

import App from './App.vue'
import router from './router'

const app = createApp(App)

app.use(createPinia())
app.use(router)

app.mount('#app')
```

## 访问路由器和当前路由

`router` 作为路由器实例提及，即由 `createRouter()` 返回的对象。在应用中，访问该对象的方式取决于上下文。

- 在组合式 API 中，它可以通过调用 `useRouter()` 来访问。
- 在选项式 API 中，它可以通过 `this.$router` 来访问。

当前路由会以 `route` 被提及。基于不同 API 风格的组件，它可以通过 `useRoute()` 或 `this.$route` 来访问。

```vue
<script setup>
import { useRoute, useRouter } from 'vue-router'

const router = useRouter()
const route = useRoute()
</script>
```

## RouterView

组件 `RouterView` 和 `RouterLink` 都是全局注册的，因此它们不需要在组件模板中导入。

但也可以通过局部导入它们: `import { RouterView, RouterLink } from 'vue-router'`。

在模板中，组件的名字可以是 PascalCase 风格或 kebab-case 风格的。Vue 的模板编译器支持两种格式，因此 `<RouterView>` 和 `<router-view>` 通常是等效的。

如果使用 DOM 内模板，组件名字必须使用 kebab-case 风格且不支持自闭合标签。不能直接写 `<RouterView />`，而需要使用 `<router-view></router-view>`。

```vue
<template>
  <div class="app">
    <div class="navigator">
      <RouterLink to="/news" class="news">news</RouterLink>
      <RouterLink to="/movie" class="movie">movie</RouterLink>
      <RouterLink to="/comic" class="comic">comic</RouterLink>
    </div>
    <div class="main-content">
      <RouterView></RouterView>
    </div>
  </div>
</template>
```

## RouterLink

`<RouterLink>`组件替换`<a>`标签，`<RouterLink>`在浏览器中就是`<a>`。

- 能跳转，`<a>`标签的`href`属性`href="#/"`，RouterLink和`to`属性`to=""`。
- 能高亮，添加类名`class="router-link-active router-link-exact-active`。
  - `.router-link-active`模糊匹配，比如`to="/my"`，能匹配`"/my","/my/a","/my/b"...`。
  - `.router-link-exact-active`精确匹配，比如`to="/my"`，仅能匹配`"/my"`。
  - 可以自定义高亮的这两个类名，在创建路由器实例时（`@/router/index.js`文件中），配置`linkActiveClass`和`linkExactActiveClass`。

局部改变类名

```vue
<RouterLink
  activeClass="border-indigo-500"
  exactActiveClass="border-indigo-700"
  ...
>
```

全局改变类名

```js
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [],
  activeClass: 'active',
  linkExactActiveClass: 'exact-active',
})
```

- `<RouterLink to="/news" class="news">news</RouterLink>`替换<a href="#/news">news</a>。

`to`的两种写法：

1. 字符串格式
2. 对象格式

```vue
<RouterLink to="/comic" class="comic">comic</RouterLink>
<RouterLink :to="{path:'/comic'}" class="comic">comic</RouterLink>
```

## 命名路由

`src/router/index.js`配置命名路由`name:''`。

```javascript
{name: 'manhua', path: '/comic', component:() => import('@/views/news.vue')}
```

`<RouterLink :to="">`可以通过name路由命名跳转或者path路由路径跳转。

```javascript
<RouterLink :to="{name:'manhua'}" class="comic">comic</RouterLink>
```

所有路由的命名**都必须是唯一的**。如果为多条路由添加相同的命名，路由器只会保留最后那一条。

使用 `name` 优点：

- 没有硬编码的 URL。
- `params` 的自动编码/解码。
- 防止你在 URL 中出现打字错误。
- 绕过路径排序，例如展示一个匹配相同路径但排序较低的路由。

## 命名视图

可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果`router-view`没有设置名字，那么默认为 default。

```javascript
<router-view class="view left-sidebar" name="LeftSidebar" />
<router-view class="view main-content" />
<router-view class="view right-sidebar" name="RightSidebar" />
```

`src/router/index.js`，对于同个路由，多个视图就需要多个组件，`components: {}`，与 `<router-view>` 上的 `name` 属性匹配。

```javascript
const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: '/',
      components: {
        default: Home,
        // LeftSidebar: LeftSidebar 的缩写
        LeftSidebar,
        // 它们与 `<router-view>` 上的 `name` 属性匹配
        RightSidebar,
      },
    },
  ],
})
```

## 路由的匹配语法

默认情况下，所有路由是不区分大小写的，并且能匹配带有或不带有尾部斜线的路由。例如，路由 `/users` 将匹配 `/users`、`/users/`、甚至 `/Users/`。这种行为可以通过 `strict` 和 `sensitive` 选项来修改，它们既可以应用在整个全局路由上，又可以应用于当前路由上：

`?` 修饰符(0 个或 1 个)将一个参数标记为可选。

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes: [
    // 将匹配 /users/posva 而非：
    // - /users/posva/ 当 strict: true
    // - /Users/posva 当 sensitive: true
    { path: '/users/:id', sensitive: true },
    // 将匹配 /users, /Users, 以及 /users/42 而非 /users/ 或 /users/42/
    { path: '/users/:id?' },
  ],
  strict: true, // applies to all routes
})
```

## 嵌套路由

`src/pages`文件目录或者`src/views`文件目录下创建路由组件。

`children` 配置另一个路由数组，就像 `routes` 本身一样，`path: ""`先不用`/`，有下一级才`/`。

```javascript
children: [{name: '', path: '', component: ''}, {name: '', path: '', component: ''}]
```

**以 / 开头的嵌套路径将被视为根路径，利用组件嵌套，而不必使用嵌套的 URL。**

```javascript
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      {
        // 当 /user/:id/profile 匹配成功
        // UserProfile 将被渲染到 User 的 <router-view> 内部
        path: 'profile',
        component: UserProfile,
      },
      {
        // 当 /user/:id/posts 匹配成功
        // UserPosts 将被渲染到 User 的 <router-view> 内部
        path: 'posts',
        component: UserPosts,
      },
    ],
  },
]
```

嵌套路由的`path: ""`为空，可以允许子路由渲染到父路由的路由出口中。

```javascript
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      // 当 /user/:id 匹配成功
      // UserHome 将被渲染到 User 的 <router-view> 内部
      { path: '', component: UserHome },
    ],
  },
]
```

## 不同的历史记录模式

用 `createWebHistory()` 创建 HTML5 模式，推荐。URL 会看起来漂亮。

```javascript
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [],
})
```

hash 模式是用 `createWebHashHistory()` 创建的，URL前加`#`，**不利于 SEO**，适合于后台后端开发。

```javascript
import { createRouter, createWebHashHistory } from 'vue-router'

const router = createRouter({
  history: createWebHashHistory(),
  routes: [],
})
```

## 路由传参

### 查询参数传参query传参

1. 字符串拼接

```javascript
:to="`/news/detail/?id=${news.id}&title=${news.title}&content=${news.content}`"
```

2. 对象模式

```javascript
:to="{
      path:'/news/detail',
      query: {
        id: news.id,
        title: news.title,
        content: news.content
      }
```

查询传参获取参数

- 在<script>中通过`route.query.属性名`获取，还可以解构赋值`let { query } = toRefs(route)`。

```javascript
<script setup>
import { useRoute } from 'vue-router'
import { toRefs } from 'vue'

let route = useRoute()
console.log(route)

// 如果 let { query: { id, title, content } } = toRefs(route)不能生效
let { query } = toRefs(route)
</script>
```

- 在<template>模板中通过`$route.query.属性名`，比如`$route.query.id`，`$route.query.title`。

`news.vue`

```vue
<script setup>
import { reactive} from 'vue'

const newsList = reactive([
  {
    id: 'dfgdfbgffgb001',
    title: '号外号外，金子涨价！',
    content: '涨了45元，要598元了'
  }
])
</script>

<template>
  <div class="news">
    <h1>今日新闻</h1>
    <!-- :to="`/news/detail/?id=${news.id}&title=${news.title}&content=${news.content}`" -->
    <router-link 
    v-for="news in newsList" :key="news.id" 
    :to="{
      path:'/news/detail',
      query: {
        id: news.id,
        title: news.title,
        content: news.content
      }
    }"
    >
    {{ news.title }}
  </router-link>
    <RouterView></RouterView>
  </div>
</template>
```

`detail.vue`，获取上一级路由传递的参数。

```vue
<script setup>
import { useRoute } from 'vue-router'
import { toRefs } from 'vue'

let route = useRoute()
console.log(route)

// 如果 let { query: { id, title, content } } = toRefs(route)不能生效
let { query } = toRefs(route)

</script>

<template>
  <div class="detail">
    <h1>新闻详情</h1>
    <h2>标题：{{ query.title }}</h2>
    <p>{{ query.content }}</p>
  </div>
</template>
```

### 动态路由传参params传参

需要在`src/router/index.js`文件中改造路由，占位`/:key`，可以`name`命名 ，`?`表示该属性可有可无`/:key?`。

```javascript
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {name: 'xinwen', path: '/news', component:() => import('@/views/news.vue'),
  		children: [{
        name: 'xwxiangqing',
       	path:'detail/:id/:title/:content?',
        component:() => import('@/views/detail.vue')
      }
    ]},
    {path: '/movie', component:() => import('@/views/movie.vue')},
    { path: '/comic', component: () => import('@/views/comic.vue') },
  ]
})

export default router
```

params传参2种写法，① 字符串 ② 对象式。

```javascript
:to="`/news/detail/${news.id}/${news.title}/${news.content}`"
:to="{
      name: 'xwxiangqing',
      params: {
        id: news.id,
        title: news.title,
        content: news.content
      }
```

动态路由传参获取参数

- 在<script>中通过`route.params.属性名`获取，还可以解构赋值`let { params } = toRefs(route)`。

```javascript
<script setup>
import { useRoute } from 'vue-router'
import { toRefs } from 'vue'

let route = useRoute()
console.log(route)

let { params } = toRefs(route)
</script>
```

- 在<template>模板中通过`$route.params.属性名`，比如`$route.params.title`，`$route.params.content`。

`news.vue`

```javascript
<script setup>
import { reactive} from 'vue'

const newsList = reactive([
  {
    id: 'dfgdfbgffgb001',
    title: '号外号外，金子涨价！',
    content: '涨了45元，要598元了'
  }
])
</script>

<template>
  <div class="news">
    <h1>今日新闻</h1>
    <router-link 
    v-for="news in newsList" :key="news.id" 
    :to="{
      name: 'xwxiangqing',
      params: {
        id: news.id,
        title: news.title,
        content: news.content
      }
    }"
    >
    {{ news.title }}
  </router-link>
    <RouterView></RouterView>
  </div>
</template>
```

`detail.vue`

```vue
<script setup>
import { useRoute } from 'vue-router'
import { toRefs } from 'vue'

let route = useRoute()
console.log(route)

let { params } = toRefs(route)
</script>

<template>
  <div class="detail">
    <h1>新闻详情</h1>
    <h2>标题：{{ params.title }}</h2>
    <p>{{ params.content }}</p>
  </div>
</template>
```

## 路由组件传参props

在`src/router/index.js`配置，有三种写法：

1. `props = true`,和`params`参数搭配
2. 函数式写法：`props(route) { return route.query }`，如果用`query`配置，但是`params`也可以
3. 对象写法：`props: { id: 'xxx', title: 'dffdg' }`

```javascript
// /router/index.js，配置route，导出router
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {name: 'xinwen', path: '/news', component:() => import('@/views/news.vue'), 
     children: [{
      name: 'xwxiangqing',
      path:'detail/:id/:title/:content?',
      component:() => import('@/views/detail.vue'),
     	props: true
      }
    ]},
    {path: '/movie', component:() => import('@/views/movie.vue')},
    { path: '/comic', component: () => import('@/views/comic.vue') },
  ]
})

export default router
```

在`detail.vue`通过`defineProps(['id', 'title', 'content'])`接收

```vue
<script setup>
import { useRoute } from 'vue-router'

let route = useRoute()
console.log(route)

defineProps(['id', 'title', 'content'])
</script>
<template>
  <div class="detail">
    <h1>新闻详情</h1>
    <h2>标题：{{ title }}</h2>
    <p>{{ content }}</p>
  </div>
</template>
```

对于有命名视图的路由，你必须为每个命名视图定义 `props` 配置：

```javascript
const routes = [
  {
    path: '/user/:id',
    components: { default: User, sidebar: Sidebar },
    props: { default: true, sidebar: false }
  }
]
```

## 编程式导航

除了使用 `<router-link>` 创建 a 标签来定义导航链接，我们还可以借助 router 的实例方法，通过编写代码来实现。

`router.push()` :向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，会回到之前的 URL。

属性 `to` 与 `router.push` 接受的对象种类相同，所以两者的规则完全相同。

```javascript
// 声明式
<router-link :to="...">
//编程式
router.push(...)

// 字符串路径
router.push('/users/eduardo')

// 带有路径的对象
router.push({ path: '/users/eduardo' })

// 命名的路由，并加上参数，让路由建立 url
router.push({ name: 'user', params: { username: 'eduardo' } })

// 带查询参数，结果是 /register?plan=private
router.push({ path: '/register', query: { plan: 'private' } })

// 带 hash，结果是 /about#team
router.push({ path: '/about', hash: '#team' })
```

`router.replace()`，类似于 `router.push`，唯一不同的是，它在导航时不会向 history 添加新记录，取代了当前的条目。

```javascript
// 声明式
<RouterLink replace :to="{path:'/comic'}" class="comic">comic</RouterLink>
//编程式
router.replace(...)
```

`window.history.go(n)`:在历史堆栈中前进或后退多少步。

```javascript
// 向前移动一条记录，与 router.forward() 相同
router.go(1)

// 返回一条记录，与 router.back() 相同
router.go(-1)

// 前进 3 条记录
router.go(3)

// 如果没有那么多记录，静默失败
router.go(-100)
router.go(100)
```

`detail.vue`

```vue
<script setup>
import { reactive} from 'vue'
import { useRouter } from 'vue-router'

const newsList = reactive([])

const router = useRouter()
const onClick = (news) => {
  // to怎么写，push()/replace()方法里面就可以怎么写
  router.push({
    name: 'xwxiangqing',
    params: {
      id: news.id,
      title: news.title,
      content: news.content
    }
  })
}
</script>

<template>
  <div class="news">
    <h1>今日新闻</h1>
    <li v-for="news in newsList" :key="news.id" >
      <button @click="onClick(news)">click</button>
      <router-link
        :to="{
          name: 'xwxiangqing',
          params: {
            id: news.id,
            title: news.title,
            content: news.content
          }
      }"
      >
        {{ news.title }}
      </router-link>
    </li>
    <RouterView></RouterView>
  </div>
</template>
```

## 重定向

```javascript
const routes = [{ path: '/', redirect: '/home' }]
```

# Vue-Router-advanced

## RouterOptions

`history,routes,linkActiveClass,linkExactActiveClass,scrollBehavior,sensitive,strict`

### scrollBehavior

scrollBehavior当在页面之间导航时控制滚动的功能。可以返回一个 Promise 来延迟滚动。

希望在切换路由时，页面自动滚动到特定的位置，比如顶部，或记住上次访问的位置。

```javascript
function scrollBehavior(to, from, savedPosition) {
  // to要导航到的路由地址
  // from要离开的路由地址
  // savedPosition 要保存的页面位置，如果不存在则是 null
}
```

`scrollBehavior` 函数需要返回一个对象，这个对象指定滚动到的位置。以是以下几种情况：

1. `{ left: 0, top: 0 }`：页面会滚动到左上角，适用于每次进入新页面都滚动到顶部。
2. `savedPosition`：如果有保存的位置（例如点击浏览器返回按钮时），会滚动到之前的位置。
3. `{ el: '#elementID' }`：滚动到指定的元素。这里的 `#elementID` 是页面中元素的 ID。

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes: [],
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      // 使用浏览器的前进/后退功能时，返回到上一次的位置
      return savedPosition;
    } else if (to.hash) {
      // 如果路由中有 hash，例如 #section1，则滚动到该元素
      return { el: to.hash };  // 在Vue3中，使用 `el` 属性代替 `selector`
    } else {
      // 否则滚动到顶部
      return { left: 0, top: 0 };  // 使用 `left` 和 `top` 替代Vue2 `x` 和 `y`
    }
  }
});
```

## onBeforeRouteUpdate

`onBeforeRouteUpdate`路由更新前的钩子，是一个路由守卫，适用于组件已经挂载，但路由参数发生变化时的场景。例如，在同一个组件中展示不同的文章详情，根据文章 ID 变化去获取不同的内容。

**使用场景**：在 `Vue 3` 中，当组件依赖于路由参数时，比如 `id`，路由参数变化不会重新创建组件，而是更新参数。此时需要用 `onBeforeRouteUpdate` 来监听路由参数变化，执行一些更新操作。

```javascript
import { onBeforeRouteUpdate } from 'vue-router';
onBeforeRouteUpdate((to, from, next) => {
  // `to` 是目标路由，表示即将进入的路由。
  // `from` 是来源路由，表示当前的路由。
  // `next` 一个函数，必须调用它来继续导航，否则路由会卡住。
  // 执行某些更新操作
  next(); // 确保调用 next() 来继续导航
	})
```

假设我们有一个博客文章详情组件，根据 URL 中的文章 ID 显示内容。我们使用 `onBeforeRouteUpdate` 监听 ID 的变化，以便在不重建组件的情况下重新加载文章。

```vue
<script setup>
import { ref, onMounted } from 'vue';
import { useRoute, onBeforeRouteUpdate } from 'vue-router';

const route = useRoute();
const article = ref({});

const fetchArticle = async (id) => {
  // 模拟 API 请求获取文章详情
  article.value = await fetch(`/api/articles/${id}`).then(res => res.json());
};

// 页面首次加载时获取文章数据
onMounted(() => {
  fetchArticle(route.params.id);
});

// 当路由参数变化时重新获取文章数据
onBeforeRouteUpdate((to, from, next) => {
  fetchArticle(to.params.id);  // 使用新参数加载内容
  next();  // 确保调用 next() 来继续导航
});
</script>

<template>
  <div>
    <h1>{{ article.title }}</h1>
    <p>{{ article.content }}</p>
  </div>
</template>
```

