# Vue Note

### 项目基本架构

```
vue-demo/
├── src/
│   ├── components/
│   │   ├── BaseButton.vue      # 基础按钮组件
│   │   └── UserCard.vue        # 用户卡片组件
│   ├── views/
│   │   ├── HomeView.vue        # 首页（用户列表）
│   │   └── UserDetail.vue      # 用户详情页
│   ├── router/
│   │   └── index.js            # 路由配置
│   ├── stores/
│   │   └── user.js             # 用户状态管理
│   ├── api/
│   │   └── user.js             # 用户API接口
│   ├── utils/
│   │   └── request.js          # 请求工具
│   ├── App.vue                 # 根组件
│   └── main.js                 # 入口文件
├── index.html
└── package.json
```

### Vue 组件

```vue
<template>
  <div>内容</div>
</template>

<script setup>  //setup 语法糖模式 (vue3)
// 逻辑代码
</script>

<style>
/* 样式（可选） */
</style>
```

- 在 Vue 的 template 中，你可以使用**所有标准 HTML 标签**：
  - 布局类：`<div>`、`<span>`、`<p>`、`<section>`
  - 表单类：`<input>`、`<button>`、`<form>`
  - 媒体类：`<img>`、`<video>`

Vue 只是在这些 HTML 标签基础上增加了**指令**（以 `v-` 开头或 `@`、`:` 开头的特殊属性）。//v-model,v-on....

### Dom

在 Vue 中，**DOM（文档对象模型）** 是浏览器用来表示和操作网页的接口。Vue 的核心就是**高效地管理和更新 DOM**。

### Diff 算法解决什么问题？

#### 问题背景

如果每次数据变化都**销毁所有 DOM 再重新创建**，性能极差（DOM 操作很昂贵）。

#### Diff 算法的作用

**Diff（差异）算法**是 Vue 用来**比较新旧虚拟 DOM，找出最小变更集，只更新必要的真实 DOM**的策略。

#### 实现(最长递增子序列)

**贪心+动态规划**实现

### ref 和 reactive   **两种并列的响应式实现方案**

| 特性           | `ref()`                        | `reactive()`                         |
| :------------- | :----------------------------- | :----------------------------------- |
| **数据类型**   | 任何类型（值或对象）           | **只有对象**（Object/Array/Map/Set） |
| **访问方式**   | `.value`                       | 直接属性访问                         |
| **解构后**     | 保持响应式                     | **丢失响应式**                       |
| **"封装"概念** | 把值包装在 `{ value: ... }` 里 | 直接代理整个对象                     |

### toRefs → 把对象"拆分"成多个响应式字段的工具（反向操作）

作用 : **在解构 `reactive` 对象时保持响应式**，而不仅仅是同步更新视图。

实现: 再解构`reactive`**对象**时,将其中对象的每个属性转换成`ref`,并与原对象保持**引用**关系

```vue
<script setup>
import { reactive, toRefs } from 'vue'

// 模拟从 API 或 Pinia store 获取的数据
const user = reactive({
  name: '张三',
  age: 25,
  email: 'zhang@example.com'
})

// ✅ 使用 toRefs 解构 - 保持双向绑定
const { name, age } = toRefs(user)

// 修改 name 会同时更新：
// 1. 当前组件的视图
// 2. 原 user 对象中的 name 属性
const handleChange = () => {
  console.log('新名字：', name.value)  // 通过 .value 访问
  console.log('原对象也变了：', user.name)  // 同样更新了！
}

const reset = () => {
  name.value = '张三'
  age.value = 25
}
</script>
```

`toRef` **转换单个属性**`const fooRef = toRef(state , 'foo')`

### computed

`computed`（计算属性）是 Vue 中用于**基于现有数据派生出新数据**的特性，它具有**缓存机制**，只有依赖的数据变化时才会重新计算。

```vue
<script setup>
const fullName = computed({
  // 读取时
  get() {
    return firstName.value + '-' + lastName.value
  },
  // 赋值时（如 v-model 绑定）
  set(newValue) {
    // newValue 是 "李-四"
    const [first, last] = newValue.split('-')
    firstName.value = first
    lastName.value = last
  }
})

// 修改 fullName 会触发 setter，进而修改 firstName 和 lastName
fullName.value = '李-四'
</script>
```

| 特性       | Computed               | Watch                    |
| :--------- | :--------------------- | :----------------------- |
| **目的**   | 根据数据**计算出新值** | 数据变化时**执行副作用** |
| **返回值** | 有，可直接用于模板     | 无，只是监听回调         |
| **缓存**   | 有缓存                 | 无缓存                   |
| **适用**   | 简单的数据转换         | 异步操作、复杂逻辑       |

### watchEffect 高级侦听器

| 特性             | `watch`                              | `watchEffect`          |
| :--------------- | :----------------------------------- | :--------------------- |
| **依赖声明**     | 显式指定（手动传入）                 | 自动追踪（执行时收集） |
| **初始化执行**   | ❌ 默认不执行（需 `immediate: true`） | ✅ **立即执行**         |
| **获取旧值**     | ✅ `newVal`, `oldVal`                 | ❌ 无旧值概念           |
| **Cleanup 机制** | ❌ 需手动处理                         | ✅ 内置副作用清理       |
| **源码可读性**   | 一眼看出监听什么                     | 需检查函数体才知道依赖 |

```vue
watch([count, age], ([newCount, newAge], [oldCount, oldAge]) => {
  console.log(newCount, newAge)
}, { immediate: true })
```

```vue
watchEffect(() => {
  // 只要函数体里用了 count 或 age，变更时自动触发
  console.log(count.value)  
  console.log(age.value)
  // 如果加上 someOtherRef.value，也会自动加入监听，无需修改配置
})
```

内置副作用清理是 `watchEffect` 最大优势。当**依赖频繁变化导致上一次的副作用还在进行中**（如 API 请求未返回），需要取消上次的操作。

需要**精准控制**或**比较新旧值**时使用`watch`

### 生命周期

Vue 3 生命周期钩子（Lifecycle Hooks）是组件从**创建→挂载→更新→卸载**过程中自动执行的回调函数，用于在特定时机执行逻辑。

#### 创建阶段（Setup）

```vue
<script setup>
import { ref } from 'vue'

// setup 本身相当于 beforeCreate + created
// 此时组件实例已创建，但 DOM 未生成
const count = ref(0)
console.log('setup 执行：组件初始化')

// 注意：此时不能访问 DOM（template 还未渲染）
</script>
```

#### 挂载阶段（Mounting）

```vue
<script setup>
import { ref, onBeforeMount, onMounted } from 'vue'

const elRef = ref(null)

// DOM 即将挂载（极少使用）
onBeforeMount(() => {
  console.log('DOM 即将插入页面')
})

// DOM 已挂载（最常用）
onMounted(() => {
  console.log('DOM 已渲染完成')
  console.log(elRef.value)  // 可以获取到 DOM 元素
  
  // 常用于：API 请求、DOM 操作、图表初始化
  fetchData()
  initChart()
})
</script>
```

#### 更新阶段（Updating）

```vue
<script setup>
import { ref, onBeforeUpdate, onUpdated } from 'vue'

const count = ref(0)

// 数据变化，DOM 即将更新
onBeforeUpdate(() => {
  console.log('数据变了，准备更新 DOM')
})

// DOM 已更新完成
onUpdated(() => {
  console.log('DOM 已更新')
  // 注意：避免在此处修改状态，可能导致无限循环
})
</script>
```

#### 卸载阶段（Unmounting）

```vue
<script setup>
import { onBeforeUnmount, onUnmounted } from 'vue'

let timer = null

onMounted(() => {
  timer = setInterval(() => {
    console.log('定时器运行中')
  }, 1000)
})

// 组件即将卸载（清理工作的最后时机）
onBeforeUnmount(() => {
  console.log('组件即将被销毁')
})

// 组件已卸载（必须在此清理副作用）
onUnmounted(() => {
  console.log('组件已销毁')
  // 必须清理：定时器、事件监听、WebSocket、外部资源
  clearInterval(timer)
  window.removeEventListener('resize', handleResize)
})
</script>
```

| 组合式 API (Vue 3) | 执行时机             |
| :----------------- | :------------------- |
| `setup()`          | 组件初始化           |
| `onBeforeMount`    | DOM 挂载前           |
| `onMounted`        | DOM 挂载后           |
| `onBeforeUpdate`   | 数据更新，DOM 更新前 |
| `onUpdated`        | 数据更新，DOM 更新后 |
| `onBeforeUnmount`  | 组件卸载前           |
| `onUnmounted`      | 组件卸载后           |
| `onActivated`      | KeepAlive 组件激活   |
| `onDeactivated`    | KeepAlive 组件停用   |
| `onErrorCaptured`  | 捕获子孙组件错误     |

**生命周期流程**

```
创建阶段
  ↓
setup() [初始化响应式数据]
  ↓
onBeforeMount [DOM 即将生成]
  ↓
onMounted ✅ [DOM 已生成，可访问 DOM/API 请求]
  ↓
更新阶段（数据变化时触发）
  ↓
onBeforeUpdate [即将重新渲染]
  ↓
onUpdated [渲染完成]
  ↓
卸载阶段
  ↓
onBeforeUnmount [即将销毁]
  ↓
onUnmounted ✅ [已销毁，清理副作用]
```

### BEM架构和layout布局

**BEM**

block + element + modifier

**BEM** **命名模式**

BEM 命名约定的模式是：

```javascript
.block {}

.block__element {}

.block--modifier {}
```

- 每一个块(block)名应该有一个命名空间（前缀）
  - `block` 代表了更高级别的抽象或组件。
  - `block__element` 代表 .block 的后代，用于形成一个完整的 .block 的整体。
  - `block--modifier` 代表 .block 的不同状态或不同版本。 使用两个连字符和下划线而不是一个，是为了让你自己的块可以用单个连字符来界定。如：

```javascript
.sub-block__element {}

.sub-block--modifier {}
```

**BEM 命名法的好处**

BEM的关键是，可以获得更多的描述和更加清晰的结构，从其名字可以知道某个标记的含义。于是，通过查看 HTML 代码中的 class 属性，就能知道元素之间的关联。

常规的命名法示例：

```javascript
<div class="article">
    <div class="body">
        <button class="button-primary"></button>
        <button class="button-success"></button>
    </div>
</div>
```

- 这种写法从 DOM 结构和类命名上可以了解每个元素的意义，但无法明确其真实的层级关系。在 css 定义时，也必须依靠层级选择器来限定约束作用域，以避免跨组件的样式污染。

使用了 BEM 命名方法的示例：

```javascript
<div class="article">
    <div class="article__body">
        <div class="tag"></div>
        <button class="article__button--primary"></button>
        <button class="article__button--success"></button>
    </div>
</div>
```

- 通过 BEM 命名方式，模块层级关系简单清晰，而且 css 书写上也不必作过多的层级选择。

#### **如何使用 BEM 命名法**

**什么时候应该用 BEM 格式**

- 使用 BEM 的诀窍是，你要知道什么时候哪些东西是应该写成 BEM 格式的。
- 并不是每个地方都应该使用 BEM 命名方式。当需要明确关联性的模块关系时，应当使用 BEM 格式。
- 比如只是一条公共的单独的样式，就没有使用 BEM 格式的意义：

```javascript
.hide {
    display: none !important;
}
```

**在 CSS 预处理器中使用 BEM 格式**

- BEM 的一个槽点是，命名方式长而难看，书写不雅。相比 BEM 格式带来的便利来说，我们应客观看待。
- 而且，一般来说会使用通过 LESS/SASS 等预处理器语言来编写 CSS，利用其语言特性书写起来要简单很多。

> 以 LESS 为例：

```css
.article {
    max-width: 1200px;
    &__body {
        padding: 20px;
    }
    &__button {
        padding: 5px 8px;
        &--primary {background: blue;}
        &--success {background: green;}
    }
}
```

#### Layout

在layout中分布着不同模块的文件夹<img src="C:\Users\Tu\AppData\Roaming\Typora\typora-user-images\image-20260127145438011.png" alt="image-20260127145438011" style="zoom:33%;" />

在根目录下的`index.vue`下集成所有模块,每个模块在各自`index.vue`中定义自己的格式

### 父子组件传参

**父组件给子组件传参**

父组件只需要传数据和监听事件

```js
<Child :title="msg" @update="handler" /> 			//在父组件上实现
```

子组件通过`defineProps`收取参数 

```vue
<template>				
    <div class="menu">
        菜单区域 {{ title }}
        <div>{{ data }}</div>
    </div>
</template>
 
<script setup lang="ts">
defineProps<{				//在子组件上实现
    title:string,
    data:number[]
}>()
</script>
```

withdefaults 默认参数 : 如果未传入对应参数则使用默认值

```vue
type Props = {
    title?: string,
    data?: number[]
}
withDefaults(defineProps<Props>(), {     //泛型自变量模式
    title: "张三",
    data: () => [1, 2, 3]  
})
```

**子组件给父组件传参**

子组件通过`defineEmits`派发一个事件

```vue
<template>
    <div class="menu">
        <button @click="clickTap">派发给父组件</button>
    </div>
</template>


<script setup lang="ts">
import { reactive } from 'vue'
const list = reactive<number[]>([4, 5, 6])


const emit = defineEmits(['on-click'])

//如果用了ts可以这样两种方式
// const emit = defineEmits<{
//     (e: "on-click", name: string): void
// }>()
const clickTap = () => {
    emit('on-click', list)
}

</script>
```

| 宏            | 在哪里写   | 作用                   |
| :------------ | :--------- | :--------------------- |
| `defineProps` | **子组件** | 声明"我能接收什么数据" |
| `defineEmits` | **子组件** | 声明"我会发出什么事件" |
| `v-bind`      | **父组件** | 传递数据给子组件       |
| `v-on`        | **父组件** | 监听子组件事件         |

### 全局组件

在App.vue里使用`app.component('命名' , 引入的组件名称)`

**批量引入**

使用for循环实现

### 递归组件

通过`interface Tree`接口实现

```vue
interface Tree{		//在Tree这个子组件中实现结构逻辑,调用<Tree><Tree/>实现递归,在主页面调用实现页面
	name : string,				//<Tree><Tree/> 中调用v-if 判断子树是否是叶子节点
	checked : boolean
	children ?: Tree[]
}
```

自定义命名: 

- 自己在子组件中定义`script`
- 使用第三方插件实现 `defineoptions`

### 动态组件

**`<component :is>` 语法**

通过 `:is` 属性动态切换组件，组件实例会被缓存或销毁。

```vue
<template>
  <div>
    <!-- 动态组件容器 -->
    <component 
      :is="currentComponent" 
      :key="componentId"
      title="传入的标题"
      @confirm="handleConfirm"
    />
    
    <!-- 切换按钮 -->
    <button @click="switchTo('ComponentA')">显示A</button>
    <button @click="switchTo('ComponentB')">显示B</button>
  </div>
</template>

<script setup>
import { ref, shallowRef } from 'vue'
import ComponentA from './ComponentA.vue'
import ComponentB from './ComponentB.vue'

// 当前组件（可以是组件对象或组件名）
const currentComponent = shallowRef(ComponentA)

// 切换函数
const switchTo = (name) => {
  if (name === 'ComponentA') {
    currentComponent.value = ComponentA
  } else {
    currentComponent.value = ComponentB
  }
}
</script>
```

**`v-if/v-else` 条件渲染**

通过条件判断控制组件的渲染/销毁

```vue
<template>
  <div>
    <!-- 条件渲染 -->
    <ComponentA 
      v-if="showA" 
      title="A的标题"
      @confirm="handleConfirm"
    />
    <ComponentB 
      v-else 
      title="B的标题"
      @confirm="handleConfirm"
    />
    
    <!-- 切换按钮 -->
    <button @click="showA = !showA">切换组件</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import ComponentA from './ComponentA.vue'
import ComponentB from './ComponentB.vue'

const showA = ref(true)
</script>
```

### 插槽  `<slot>`

`v-slot` 具名插槽

**多个插槽位置**：子组件有多个 `<slot>`，父组件通过 `v-slot` 指定内容放到哪里。

```vue
<template>
  <div class="layout">
    <header>
      <slot name="header">			//子组件自己定义标识符
        <!-- 默认头部 -->
        <h1>默认标题</h1>
      </slot>
    </header>
    
    <main>
      <slot></slot>  <!-- 默认插槽 -->
    </main>
    
    <footer>
      <slot name="footer"></slot>
    </footer>
    
    <!-- 具名插槽可以带数据 -->
    <slot name="actions" :close="closeModal"></slot>
  </div>
</template>

<script setup>
const closeModal = () => {
  console.log('关闭弹窗')
}
</script>
```

```vue
<template>
  <Layout>
    <!-- 具名插槽 v-slot:名称 -->
    <template v-slot:header>		//把数据插入定义好的标识符header
      <h1>自定义标题</h1>
      <p>副标题</p>
    </template>
    
    <!-- 缩写语法：#名称 -->
    <template #footer>
      <p>版权信息 © 2024</p>
    </template>
    
    <!-- 作用域插槽：接收子组件数据 -->
    <template #actions="{ close }">
      <button @click="close">关闭</button>
    </template>
  </Layout>
</template>
```

并且子组件可以通过`v-for`实现在父组件中输出遍历后的数据

```js
<div v-for "items : data">
	<slot : data = "item"></slot>
</div>
```

### 异步组件

Vue 异步组件是**按需加载**的组件，只有在需要渲染时才会从服务器加载对应的代码，有效减少首屏加载体积

`Suspense` 是 Vue 3 内置组件，专门处理异步组件的加载状态，更声明式。

```vue
<template>
  <button @click="showModal = true">打开弹窗</button>

  <!-- Suspense 包裹异步组件 -->
  <Suspense v-if="showModal">
    <!-- 异步组件加载成功后的内容 -->
    <template #default>
      <AsyncModal @close="showModal = false" />
    </template>
    
    <!-- 加载中状态 -->
    <template #fallback>
      <div class="loading-modal">
        <SpinningLoader />
        <p>正在加载弹窗...</p>
      </div>
    </template>
  </Suspense>
</template>

<script setup>
import { ref, defineAsyncComponent } from 'vue'

const showModal = ref(false)

// 定义异步组件
const AsyncModal = defineAsyncComponent(() => 
  import('./Modal.vue')
)
</script>
```

### `Teleport`

vue自带的组件,用于将**子组件的 DOM 结构**渲染到**组件模板之外的指定位置**,常用于模态框、提示框等需要脱离父级样式层叠的 UI 元素(可用于视图的位置变更)

```vue
<teleport to="目标选择器">
  <!-- 这里的 DOM 会被移动到目标位置 -->
</teleport>
```

```vue
<Teleport : disabled = "false"  to = "body" > //disabled为true的时候,忽略to的值
<><>
</Teleport>
```

### `Keep-alive` 

vue自带组件,用于页面**缓存**,可以通过`include`定义需要缓存的页面

`max = 10` 允许10个缓存页面,如果有更多的页面需要缓存,使用lru算法替换

### `translation` 动画组件

### 组合式API

组合式API实现代码复用的核心是 **"可组合函数"（Composables）** ——将可复用逻辑抽离成独立的函数，在多个组件中调用。

```js
// hooks/useCounter.js
export function useCounter(initialValue = 0) { 
  const count = ref(initialValue);  // 一个 ref
  
  function increment() {
    count.value++;
  }
  
  function decrement() {
    count.value--;
  }
  
  // 返回一个对象，包装多种数据类型
  return {
    count,        // ref
    increment,    // 函数
    decrement     // 函数
  };
}
```

在组件中怎么使用 : 返回的数据类型,无论是函数还是数据,都能够在组件中被单独调用

```vue
<script setup>
import { useCounter } from '@/hooks/useCounter';

// 调用可组合函数，获取所有返回值
const { count, increment, decrement } = useCounter(10);
</script>

<template>
  <p>{{ count }}</p>
  <button @click="increment">+</button>
</template>
```

### Proxy 配置跨域

在配置文件中,将需要跨的域封装为"/api"或自定义格式,在组件中使用"/api/xxx"即可跨域访问
