# element-plus组件库常用组件

## 基础组件

### layout布局

`<el-row>`和`<el-col>`

`<el-row>`的`:gutter="20"`控制列之间的间隔

`<el-col>`分为24栏，通过`:span="6"`控制分为几栏。`:offset="6"`控制列偏移。

```vue
  <el-row :gutter="20">
    <el-col :span="6"><div class="grid-content ep-bg-purple" /></el-col>
    <el-col :span="6" :offset="6">
      <div class="grid-content ep-bg-purple" />
    </el-col>
  </el-row>
```

### container布局容器

`<el-container>`：外层容器。 当子元素中包含 `<el-header>` 或 `<el-footer>` 时，全部子元素会垂直上下排列， 否则会水平左右排列。

`<el-header>`：顶栏容器。

`<el-aside>`：侧边栏容器。

`<el-main>`：主要区域容器。

`<el-footer>`：底栏容器。

```vue
<template>
  <div class="common-layout">
    <el-container>
      <el-header>Header</el-header>
      <el-container>
        <el-aside width="200px">Aside</el-aside>
        <el-container>
          <el-main>Main</el-main>
          <el-footer>Footer</el-footer>
        </el-container>
      </el-container>
    </el-container>
  </div>
</template>
```

### Button按钮

`<el-button>`的`size="'large' | 'default' | 'small'"`表示尺寸，`type="'primary' | 'success' | 'warning' | 'danger' | 'info' "`表示类型，`plain`是否为朴素按钮，`round`是否为圆角按钮，`circle`是否为圆形按钮，`disabled`是否禁用。`:icon=""`显示图标。

```vue
<el-button>Default</el-button>
<el-button type="primary">Primary</el-button>
<el-button type="success">Success</el-button>
```

### Link链接

`<el-link>`文字链接，`type="'primary' | 'success' | 'warning' | 'danger' | 'info' | 'default'"`，underline是否下划线，`disabled`是否禁用，`href`原生`href`属性，`target="'_blank' | '_parent' | '_self' | '_top'"`同原生`target `属性。

```vue
<el-link href="https://element-plus.org" target="_blank">default</el-link>
<el-link type="primary">primary</el-link>
```

## Form表单组件

### form表单

`<el-form>`基本用法

`ref="formRef"`获取整个表单，`:model="formModel"`绑定表单数据，`:rules="formRules"`绑定表单规则，进行规则校验。

`:inline="true"`将表单变为行内表单，简写`inline`。

`el-form-item`中`prop=""`绑定`formRules`里面的规则。`label`标签名称，`label-width="80px"`标签宽度，单位一般为`px`。

`el-form`和`el-form-item`还有`size="large | default | small"`。

`el-input`中的`v-model="formModel."`进行数据的双向绑定。`:disabled=true`一般绑定在此。

**表单校验**可以直接`{}`，不一定非要响应式。

- `pattern: `正则表达式。
- `type: ""`
  - "string": 校验字符串类型。"number": 校验数字类型。"boolean": 校验布尔类型。"array": 校验数组类型。"object": 校验对象类型。"date": 校验日期类型。"integer": 校验整数类型。"email": 校验邮箱格式。"url": 校验URL格式。"phone": 校验手机号码格式（某些情况下，可能是自定义校验）。"idcard": 校验身份证号格式。"pattern": 自定义正则表达式校验。


- `trigger: 'blur | change'`
- 当每条规则校验完成，一般要对整个表单进行校验，通过`ref()`获取整个表单，`validate()`整个表单校验，`formRef.value.validate()`。

```js
const formRules = {
  name: [
    { required: true, message: 'Please input Activity name', trigger: 'blur' },
    { min: 3, max: 5, message: 'Length should be 3 to 5', trigger: 'blur' },
  ],
  nickname: [
    { required: true, message: '请输入用户昵称', trigger: 'blur' },
    { pattern: /^.{2,10}$/, message: '昵称长度在 2 到 10 个字符', trigger: 'blur' }
  ],
  email: [
    { required: true, message: '请输入用户邮箱', trigger: 'blur' },
    // {type: 'email', message: '请输入正确的邮箱格式', trigger: ['blur', 'change'] }
  ]
}
```

**自定义校验：**`{ validator: checkOldPwd, trigger: 'blur' }`。

`rule` 是验证规则对象，`value`获取输入的数据，`callback()`不管成功还是失败都要执行。

```js
const checkOldPwd = (rule, value, callback) => {
  if (value === '123456' ) {
    callback()
  } else {
    callback(new Error('原密码错误'))
  }
}
```

示例

```vue
<script setup>
import PageContainer from '@/components/PageContainer.vue'
import { ref } from 'vue'
import { userUpdatePasswordAPI } from '@/apis/user'
import { ElMessage } from 'element-plus';

const formRef = ref(null)
const defaultFormModel = ref({
  old_pwd: '',
  new_pwd: '',
  re_pwd: ''
})
const formModel = ref({})
formModel.value = {...defaultFormModel.value}

const checkOldPwd = (rule, value, callback) => {
  if (value === '123456' ) {
    callback()
  } else {
    callback(new Error('原密码错误'))
  }
}
const checkNewPwd = (rule, value, callback) => {
  if (value === formModel.value.old_pwd) {
    callback(new Error('新密码不能与原密码相同'))
  }
    callback()
}
const checkRePwd = (rule, value, callback) => {
  console.log(rule, value, callback)
  if (value!== formModel.value.new_pwd) {
    callback(new Error('两次密码输入不一致'))
  }
  callback()
}
const formRules = {
  old_pwd: [
    { required: true, message: '请输入原密码', trigger: 'blur' },
    { validator: checkOldPwd, trigger: 'blur' }
  ],
  new_pwd: [
    { required: true, message: '请输入新密码', trigger: 'blur' },
    {validator: checkNewPwd, trigger: 'blur'}
  ],
  re_pwd: [
    { required: true, message: '请确认新密码', trigger: 'blur' },
    { validator: checkRePwd, trigger: 'blur' }
  ]
}

const onsubmit = async () => {
  await formRef.value.validator()
  const res = await userUpdatePasswordAPI(formModel.value)
  console.log(res)
  ElMessage.success('密码修改成功')
}
const onReset = () => {
  formModel.value = {...defaultFormModel.value}
}
</script>

<template>
  <el-form
    ref="formRef"
    :model="formModel"
    :rules="formRules"
    label-width="80px"
    :style="{width: '50%'}">
    <el-form-item label="原密码" prop="old_pwd">
      <el-input v-model="formModel.old_pwd" placeholder="请输入原密码"></el-input>
    </el-form-item>
    <el-form-item label="新密码" prop="new_pwd">
      <el-input v-model="formModel.new_pwd" placeholder="请输入新密码"></el-input>
    </el-form-item>
    <el-form-item label="确认密码" prop="re_pwd">
      <el-input v-model="formModel.re_pwd" placeholder="请再次确认新密码"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="onsubmit">修改密码</el-button>
      <el-button @click="onReset">重置</el-button>
    </el-form-item>
    </el-form>
</template>
```

### Input 输入框

`<el-input>`的`type="'text' | 'textarea' | 'password' | 'button' | 'checkbox' | 'file' | 'number' | 'radio' | ..."`等同原生`input`类型。`disabled`是否禁用，`size="'large' | 'default' | 'small'"`输入框尺寸，只在 `type` 不为`textarea`时有效。

| 事件名 | 说明                                                        | 类型       |
| ------ | ----------------------------------------------------------- | ---------- |
| blur   | 当选择器的输入框失去焦点时触发                              | `Function` |
| focus  | 当选择器的输入框获得焦点时触发                              | `Function` |
| change | 仅当 modelValue 改变时，当输入框失去焦点或用户按Enter时触发 | `Function` |
| input  | 在 Input 值改变时触发                                       | `Function` |
| clear  | 在点击由 `clearable` 属性生成的清空按钮时触发               | `Function` |

```vue
<el-input v-model="input" style="width: 240px" placeholder="Please input" />
```

### Input Number 数字输入框

`<el-input-number />`仅允许输入标准的数字值，可定义范围。`v-model="num"`绑定数据，`:min="1" :max="10"`最大值和最小值限定。

`@change="handleChange"`绑定值被改变时触发，`@blur="onBlur"`失去焦点时触发，`@focus="onFocus"`获得焦点时触发。

```vue
<template>
  <el-input-number v-model="num" :min="1" :max="10" @change="handleChange" />
</template>

<script lang="ts" setup>
import { ref } from 'vue'

const num = ref(1)
const handleChange = (value: number) => {
  console.log(value)
}
</script>
```

### Checkbox 多选框

`<el-checkbox>`的`v-model=""`绑定值，双向绑定，如果不用语法糖，`:model-value=""`，`@change="onChange"`当绑定值变化时触发的事件。`label=""`表示展示的文字。`size="'large' | 'default' | 'small'"`，`disabled`是否禁用，`border`是否有边框，**`checked`是否勾选。**

```vue
<el-checkbox v-model="checked1" label="Option 1" size="large" />
<el-checkbox v-model="checked2" label="Option 2" size="large" />
```

### Select选择器

`<el-select>`基本用法

`v-model` 的值为当前被选中的 `el-option` 的`value`属性值。

`size="small | default |large"`输入框尺寸。

`disabled`表示整个选择器不可用，也可用在`<el-option>`上，表示当前项禁用。完整写法`:disabled="true"`。

`el-opiton`的`:label=""`表示选项标签。如果是静态值，`:label`和`:value`可以不用加`:`。

```vue
    <el-select
      v-model="value"
      placeholder="Select"
      size="large"
      style="width: 240px"
    >
      <el-option
        v-for="item in options"
        :key="item.value"
        :label="item.label"
        :value="item.value"
      />
    </el-select>
```

### Upload上传

`<el-upload>`基本使用

`:show-file-list="false"`是否显示已上传文件列表。

`:auto-upload="false"`一般开启这个属性，自己手动上传。

`:on-change="selectImg"`文件状态改变时的钩子，添加文件、上传成功和上传失败时都会被调用。

```vue
  <el-upload
    class="avatar-uploader"
    :show-file-list="false"
    :on-success="handleAvatarSuccess"
    :on-change="selectImg"
  >
    <img v-if="imageUrl" :src="imageUrl" class="avatar" />
    <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
  </el-upload>
```

## Data数据展示

### card卡片

`<el-card>`卡片包含标题，内容以及操作区域。

`el-card`组件由 `header` `body` 和 `footer`组成。 `header` 和 `footer`是可选的，其内容取决于一个具名的`slot`，必须要要`<template #header>`和`<template #footer>`，而内容不需要有`<slot>`。

```vue
<template>
  <el-card style="max-width: 480px">
    <template #header>
      <div class="card-header">
        <span>Card name</span>
      </div>
    </template>
    <p v-for="o in 4" :key="o" class="text item">{{ 'List item ' + o }}</p>
    <template #footer>Footer content</template>
  </el-card>
</template>
```

### carousel走马灯

`<el-carousel>`在有限空间内，循环播放同一类型的图片、文字等内容。

`el-carousel` 和 `el-carousel-item`

```vue
<el-carousel height="500px">
  <el-carousel-item v-for="item in bannerList" :key="item.id">
    <img :src="item.imgUrl" alt="">
  </el-carousel-item>
</el-carousel>
```

### table表格

`<el-table>`基本用法

 `:data="tableData"` 绑定表格数据，

 `el-table-column` 中用 `prop=""` 对应表格数据中的键名即可填入数据， `label=""` 定义表格的列名。 `width` 属性来定义列宽。`type="index"`可以显示序号。

**自定义列**通过 `slot` 默认插槽`<template #default>`可以获取到`row, column, $index, store`（table 内部的状态管理）的数据，可以进行解构。

```vue
<script setup>
import { Edit, Delete } from '@element-plus/icons-vue'
import { ref } from 'vue'
import ChannelSelect from './components/ChannelSelect.vue'

const formRef = ref()
const formModel = ref({})

const articleList = ref([
  {
      "Id": 5961,
      "title": "新的文章啊",
      "pub_date": "2022-07-10 14:53:52.604",
      "state": "已发布",
      "cate_name": "体育"
    },
    {
      "Id": 5962,
      "title": "新的文章啊",
      "pub_date": "2022-07-10 14:54:30.904",
      "state": '草稿',
      "cate_name": "体育"
    }
])

const onEdit = (row) => {
  console.log(row)
}
const onDelete = (row) => {
  console.log(row)
}
</script>

<template>
  <!-- :data="articleList"表格数据绑定 -->
  <el-table :data="articleList" style="width: 100%">
    <!-- prop属性绑定数据列，label属性为表头每列标题 -->
    <el-table-column prop="title" label="文章标题"></el-table-column>
    <el-table-column prop="cate_name" label="分类"></el-table-column>
    <el-table-column prop="pub_date" label="发表时间"></el-table-column>
    <el-table-column prop="state" label="状态"></el-table-column>
    <el-table-column label="操作">
      <!-- #default为插槽，row为当前行数据 -->
      <template #default="{ row }">
        <el-button circle plain :icon="Edit" type="primary" @click="onEdit(row)"></el-button>
        <el-button circle plain :icon="Delete" type="danger" @click="onDelete(row)"></el-button>
  </template>
  </el-table-column>
  </el-table>
</template>
```

### pagination分页

`<el-pagination>`基本使用

`page-size / v-model:page-size=""`每页显示条目个数。

`current-page / v-model:current-page=""`当前页数。

`:page-sizes="[100, 200, 300, 400]"`选择每页显示多少条条目。

`layout="total, sizes, prev, pager, next, jumper"`布局。

`:total="400"`总条目数。

`size`分页大小，`:background="true | false"`是否有背景颜色，`:disabled=""`是否禁用。

`@size-change="handleSizeChange"` `page-size` 改变时触发。

`@current-change="handleCurrentChange"` `current-page` 改变时触发。

```vue
<script lang="ts" setup>
import { ref } from 'vue'
import type { ComponentSize } from 'element-plus'
const currentPage4 = ref(4)
const pageSize4 = ref(100)
const size = ref<ComponentSize>('default')
const background = ref(false)
const disabled = ref(false)

const handleSizeChange = (val: number) => {
  console.log(`${val} items per page`)
}
const handleCurrentChange = (val: number) => {
  console.log(`current page: ${val}`)
}
</script>
<template>
  <el-pagination
        v-model:current-page="currentPage4"
        v-model:page-size="pageSize4"
        :page-sizes="[100, 200, 300, 400]"
        layout="total, sizes, prev, pager, next, jumper"
        :total="400"
        :size="size"
        :disabled="disabled"
        :background="background"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
</template></template>
```

### Infinite Scroll无限滚动

在要实现滚动加载的列表上添加`v-infinite-scroll="load"`，并赋值相应的加载方法，可实现滚动到底部时自动执行加载方法。`:infinite-scroll-disabled="disabled"`表示是否要禁用无限滚动。

```vue
<script setup>
// 商品列表无限加载
const disabled = ref(false)
const load = async () => {
  // 加载到底部，page+1
  reqData.value.page++
  const { data: {result} } = await getSubGoodsAPI(reqData.value)
  // 用.concat()可以，用.push()页面报错了，用...运算符解决
  // subGoodsList.value = [...subGoodsList.value, ...result.items]
  subGoodsList.value = subGoodsList.value.concat(result.items)
  console.log('subGoodsList', subGoodsList.value)
  // 加载完毕，取消事件
  if (result.items.length === 0) {
    disabled.value = true
  }
}
</script>

<template>
  <!-- 商品列表无限加载，disabled为true时，加载完毕 -->
  <div class="body" v-infinite-scroll="load" :infinite-scroll-disabled="disabled">
    <!-- 商品列表-->
    <goods-item v-for="goods in subGoodsList" :key="goods.id" :goods="goods"></goods-item>
    </div>
</template>
```

### Empty空状态

`<el-empty>`空状态时的占位提示。 `image=""` 通过设置`image`传入图片 URL。`:image-size="200"`属性来控制图片大小，`description=""`描述信息。

`<template #default>`作为底部内容的内容。

```vue
<el-empty
    image="https://shadow.elemecdn.com/app/element/hamburger.9cf7b091-55e9-11e9-a976-7f4d0b07eef6.png"
    :image-size="200"
    description="description"
>
  <el-button type="primary">Button</el-button>
</el-empty>
```

## Navigation导航

### breadcrumb面包屑

`<el-breadcrumb>`显示当前页面的路径，快速返回之前的任意页面。

`<el-breadcrumb>`的`separator=""`作为分隔符，默认为`/`。`:separator-icon="ArrowRight"`也可以用图标作为分隔符，`separator` 将会失效。

`<el-breadcrumb-item>`的`:to=""`表示路由跳转目标，同 vue-router 的`to `属性。

```vue
<template>
  <el-breadcrumb :separator-icon="ArrowRight">
    <el-breadcrumb-item :to="{ path: '/' }">homepage</el-breadcrumb-item>
    <el-breadcrumb-item>promotion management</el-breadcrumb-item>
    <el-breadcrumb-item>promotion list</el-breadcrumb-item>
    <el-breadcrumb-item>promotion detail</el-breadcrumb-item>
  </el-breadcrumb>
</template>

<script lang="ts" setup>
import { ArrowRight } from '@element-plus/icons-vue'
</script>
```

### dropdown下拉菜单

`<el-dropdown>`下拉菜单，`@command="onCommand"`下拉项被点击时触发，参数是从下拉菜单`<el-dropdown-item>`绑定的`command`值。可以调整尺寸`size="'large' | 'default' | 'small'"`，设置禁用`disabled`。

`slot`插槽`<template #dropdown>`下拉列表，通常是 `<el-dropdown-menu>` 组件。

`<el-dropdown-item>`的`command`绑定值。可以设置`disabled`，`divided`是否显示分隔符，`:icon=""`自定义图标。

```vue
<template>
  <!-- command可以获取每个下拉菜单的command值 -->
	<el-dropdown placement="bottom-end" @command="onCommand">
    <span class="el-dropdown__box">
      <el-avatar :src="userStore.user.user_pic || avatar" />
      <el-icon><CaretBottom /></el-icon>
    </span>
    <template #dropdown>
      <!-- 下拉菜单，一个是跳转路由，一个是退出登录 -->
      <el-dropdown-menu>
        <el-dropdown-item command="profile" :icon="User">基本资料</el-dropdown-item>
        <el-dropdown-item command="avatar" :icon="Crop">更换头像</el-dropdown-item>
        <el-dropdown-item command="password" :icon="EditPen">重置密码</el-dropdown-item>
        <el-dropdown-item command="logout" :icon="SwitchButton">退出登录</el-dropdown-item>
      </el-dropdown-menu>
    </template>
	</el-dropdown>
</template>

<script setup>
const onCommand = async (key) => {
  if (key === 'logout') {
    // 退出登录
    // 清空token和用户信息
    // await 表示成功才执行下面代码
    await ElMessageBox.confirm('确认要退出登录吗？', '温馨提示', {
      type: 'warning',
      confirmButtonText: '确定',
      cancelButtonText: '取消',
    })
    userStore.setUserToken('')
    userStore.setUser({})
    router.push('/login')
  } else {
    // 跳转路由
    router.push(`/user/${key}`)
  }
}
</script>
```

### menu菜单

`<el-menu>`的`router`，以`<el-menu-item>`的`index`属性作为`path`进行路由跳转，使用 `:default-active="$router.path"` 来设置加载时的激活项。`active-text-color="" ` 激活项的菜单文字颜色，`background-color=""`菜单背景颜色，`text-color="#fff"`菜单文字颜色。

`<el-sub-menu>`，`index=""`唯一标识。`disabled`禁用。`<tempalte #title>`自定义标题内容。

`<el-menu-item>`，`index=""`唯一标识。`disabled`禁用。

`el-menu-item-group`组件可以实现菜单进行分组，分组名可以通过`title`属性直接设定，也可以通过具名`slot`来设定`<template #title>`。

```vue
<template>
	<el-menu
    active-text-color="#ffd04b"
    background-color="#232323"
    :default-active="$route.path"
    text-color="#fff"
    router
           >
    <el-menu-item index="/article/channel">
      <el-icon><Management /></el-icon>
      <span>文章分类</span>
    </el-menu-item>
    <el-menu-item index="/article/manage">
      <el-icon><Promotion /></el-icon>
      <span>文章管理</span>
    </el-menu-item>
    <el-sub-menu index="/user">
      <template #title>
        <el-icon><UserFilled /></el-icon>
        <span>个人中心</span>
      </template>
      <el-menu-item index="/user/profile">
        <el-icon><User /></el-icon>
        <span>基本资料</span>
      </el-menu-item>
      <el-menu-item index="/user/avatar">
        <el-icon><Crop /></el-icon>
        <span>更换头像</span>
      </el-menu-item>
      <el-menu-item index="/user/password">
        <el-icon><EditPen /></el-icon>
        <span>重置密码</span>
      </el-menu-item>
    </el-sub-menu>
  </el-menu>
</template>
```

### tabs标签页

`<el-tabs>`分隔内容上有关联但属于不同类别的数据集合。

`<el-tabs>`的`v-model=""`绑定值，选中选项卡的`name`，默认值是第一个`tab`的`name`。事件`@tab-click="onTabClick"`tab 被选中时触发，`@tab-change="onTabChange"`在`activeName` 改变时触发。

`<el-tab-pane>`的`label=""`选项卡标题，`name=""`与选项卡`v-model=""`绑定值`value`对应的标识符，表示选项卡别名。`disabled`是否禁用。

```vue
<script>
const tabChange = () => {
  // 重新调用接口，页面要重新加载，赋值为1
  reqData.value.page = 1
  console.log('activeName', reqData.value.sortField);
  getSubGoodsList()
}
</script>
<template>
	<!-- name和后端接口的sortField对应，v-model绑定reqData.sortField -->
  <!-- tab-change事件，activeName改变时触发 -->
  <!-- 比如点击publishTime，reqData.sortField的值就会变为publishTime，触发tabChange事件 -->
  <el-tabs v-model="reqData.sortField" @tab-change="tabChange">
    <el-tab-pane label="最新商品" name="publishTime"></el-tab-pane>
    <el-tab-pane label="最高人气" name="orderNum"></el-tab-pane>
    <el-tab-pane label="评论最多" name="evaluateNum"></el-tab-pane>
  </el-tabs>
</template>
```

## Feedback反馈组件

### Dialog 对话框

`<el-dialog>`的 `model-value / v-model="dialogVisible"` 表示是否显示`Dialog`。 `title=""` 定义标题，`center`表示是否居中显示。

`Dialog`分为两个部分：`body` 和 `footer`，`footer` 需要具名为 `footer` 的 `slot`也就是`<template #footer>`。

```vue
<script setup>
const isShow = ref(true)
const activeAddress = ref({})
// 记录当前选中项的信息
const switchAddress = (item) => {
  activeAddress.value = item
}

const confirm = () => {
  curAddress.value = activeAddress.value
  isShow.value = false
}
</script>

<template>
  <!-- 切换地址 -->
   <el-dialog v-model="isShow" title="切换收货地址" width="30%" center>
    <div class="addressWrapper">
      <!-- 选中的项才添加active激活类名 -->
      <div class="text item" v-for="item in checkInfo.userAddresses" @click="switchAddress(item)" :class="{active: item.id === activeAddress.id}"  :key="item.id">
        <ul>
        <li><span>收<i />货<i />人：</span>{{ item.receiver }} </li>
        <li><span>联系方式：</span>{{ item.contact }}</li>
        <li><span>收货地址：</span>{{ item.fullLocation + item.address }}</li>
        </ul>
      </div>
    </div>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="isShow = false">取消</el-button>
        <el-button type="primary" @click="confirm">确定</el-button>
      </span>
    </template>
  </el-dialog>
</template>
```

### Drawer抽屉

`v-model`控制是否显示抽屉，`true | faulse`。

`title=""`显示抽屉标题。

`direction="'rtl' | 'ltr' | 'ttb' | 'btt'"`控制`<el-drawer>`打开方向。

`size="50%"`控制抽屉组件大小，直接数字`xx`，默认解析为以像素为单位。

```vue
<script lang="ts" setup>
import { reactive, ref } from 'vue'
import { ElDrawer, ElMessageBox } from 'element-plus'

const formLabelWidth = '80px'
let timer

const table = ref(false)
const loading = ref(false)

const gridData = [
  {
    date: '2016-05-02',
    name: 'Peter Parker',
    address: 'Queens, New York City',
  },
  {
    date: '2016-05-04',
    name: 'Peter Parker',
    address: 'Queens, New York City',
  },
  {
    date: '2016-05-01',
    name: 'Peter Parker',
    address: 'Queens, New York City',
  },
  {
    date: '2016-05-03',
    name: 'Peter Parker',
    address: 'Queens, New York City',
  },
]
</script>

<el-drawer
    v-model="table"
    title="I have a nested table inside!"
    direction="rtl"
    size="50%"
  >
    <el-table :data="gridData">
      <el-table-column property="date" label="Date" width="150" />
      <el-table-column property="name" label="Name" width="200" />
      <el-table-column property="address" label="Address" />
    </el-table>
  </el-drawer>
```

### Message消息提示框

`<ElMessage>`的`type`属性可以定义不同的状态，默认为`info`。`type: 'success' | 'warning' | 'info' | 'error'`，可以显示「成功、警告、消息、错误」类的操作反馈。`message`消息文字。

```vue
ElMessage({
    message: 'Congrats, this is a success message.',
    type: 'success',
  })
// 简写
ElMessage.success('Congrats, this is a success message.')
```

### MessageBox消息弹出框

`<ElMessageBox>`，第一个参数是`message内容`，第二个参数是`title标题`，第三个参数对象。

`type: 'success' | 'info' | 'warning' | 'error'`。

```vue
  ElMessageBox.confirm(
    'proxy will permanently delete the file. Continue?',
    'Warning',
    {
      confirmButtonText: 'OK',
      cancelButtonText: 'Cancel',
      type: 'warning',
    }
  )
```

### Popconfirm气泡确认框

`<el-popconfirm>`点击某个元素弹出一个简单的气泡确认框。

`title=""`标题，`confirm-button-text=""`确认按钮文字，`cancel-button-text=""`取消按钮文字，`@confirm="onConfirm"`表示点击确认按钮时触发。

```vue
<script setup>
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/useUserStore.js'

const router = useRouter()
const userStore = useUserStore()

const onConfirm = () => {
  try {
    userStore.clearUserInfo()
    router.push('/login')
  } catch (error) {
    console.error("退出登录时出错:", error)
  }
}
</script>

<template>            
  <el-popconfirm title="确认退出吗?" confirm-button-text="确认" cancel-button-text="取消" @confirm="onConfirm">
    <template #reference>
      <a href="javascript:;">退出登录</a>
    </template>
  </el-popconfirm>
</template>
```

### Loading加载

提供了两种调用 Loading 的方法：指令和服务。 

自定义指令 `v-loading="loading"`，只需要绑定 `boolean` 值即可。 

> 默认状况下，Loading 遮罩会插入到绑定元素的子节点。 通过添加 `body` 修饰符，可以使遮罩插入至 Dom 中的 body 上。

```vue
<template>
  <el-table v-loading="loading" :data="tableData" style="width: 100%">
    <el-table-column prop="date" label="Date" width="180" />
    <el-table-column prop="name" label="Name" width="180" />
    <el-table-column prop="address" label="Address" />
  </el-table>
</template>
```