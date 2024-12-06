# Vue插件

编写插件，使用`install(app) {}`函数

```javascript
const myPlugin = {
  install(app, options) {
    // 配置此应用
  }
}
```

在main.js中导入插件，并运用`app.use()`使用

```javascript
import { createApp } from 'vue'

const app = createApp({})

app.use(myPlugin, {
  /* 可选的选项 */
})
```

