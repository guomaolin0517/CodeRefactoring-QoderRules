# Vue3代码规范（Rule v1.0.0）

## 版本核心信息（供Qoder Skill解析校验）

- 规则版本：1.0.0

- 适配Skill版本：vue3-code-check ≥ v1.0.0

- 生效时间：2026-03-17

- 变更类型：主版本（基于Vue3官方规范+前端研发落地要求，覆盖生产级约束）

- 技术栈适配：Vue3.2+、Vite4.0+、TypeScript4.5+、Pinia2.0+、Vue Router4.0+

- 违规等级定义：
        

    - 严重：直接导致生产故障（如内存泄漏、跨域异常、路由跳转失败），Skill强制阻断提交；

    - 高危：可能引发性能/安全问题（如XSS漏洞、未防抖节流、大文件未懒加载），Skill提示并记录；

    - 规范：格式/命名/代码风格问题（如组件命名不规范、缩进错误），Skill自动修复。

## 规则描述（description）

适配《Vue3官方风格指南》+ 前端产品研发落地要求，约束Vue3项目的组件命名、模板编写、脚本逻辑、样式规范、工程化配置、安全防护等维度，核心目标：

1. 规避生产环境常见故障（内存泄漏、XSS攻击、路由异常、性能瓶颈）；

2. 统一前端团队代码风格，降低多人协作、后期维护成本；

3. 支持Qoder Skill自动校验/修复，提升前端研发效率，减少人工评审工作量；

4. 符合前端代码评审标准，确保代码可复用、可扩展，适配产品长期迭代。

## 核心校验规则（含校验标准+修复方式+示例）

### 1. 命名规范（规范级，Skill可自动修复/提示）

#### 1.1 通用规则

校验标准：禁止使用拼音/拼音缩写/无意义命名（如shouye.vue/yhList/btn1）；命名需语义化，简洁明了，禁止冗余；

修复方式：Skill给出语义化命名建议，人工确认后修改；格式类错误（如大小写）自动修复。

#### 1.2 细分命名规则

|类型|命名格式|违规示例|正确示例|Skill修复能力|
|---|---|---|---|---|
|单文件组件（SFC）|大驼峰（PascalCase），后缀.vue（统一小写）|home.vue/user-list.vue|Home.vue/UserList.vue|自动修复（首字母大写+去连字符）|
|组件Props/Emits|小驼峰（camelCase）|user_id/UserName|userId/userName|自动修复（小驼峰格式）|
|变量/方法（setup内）|小驼峰（camelCase），禁止下划线|user_info/get_user|userInfo/getUser|自动修复（小驼峰格式）|
|常量（const）|全大写+下划线|maxPageSize/Max_Page|MAX_PAGE_SIZE|自动修复（全大写+加下划线）|
|Pinia仓库|use+大驼峰（如useUserStore），文件大驼峰|userStore.js/use_user_store.js|useUserStore.js|自动修复（添加use前缀+大驼峰）|
|路由路径|全小写+连字符（kebab-case）|/userInfo//User-List|/user-info//user-list|自动修复（全小写+连字符）|
### 2. 模板规范（规范级，Skill可自动修复/提示）

#### 2.1 模板结构（强制）

校验标准：SFC模板需遵循“template → script → style”顺序，template标签内只能有一个根节点；禁止模板内写复杂逻辑（如三目嵌套超过2层）；

修复方式：Skill自动调整标签顺序，提示拆分复杂逻辑，补充根节点；

正确示例：

```vue
<template>
  <div class="user-list">
    <!-- 模板内容 -->
  </div>
</template>

<script setup lang="ts">
// 脚本逻辑
</script>

<style scoped>
/* 样式 */
</style>
```

#### 2.2 指令使用（强制）

校验标准：
      1. v-bind缩写用“:”，v-on缩写用“@”，禁止混合使用完整写法和缩写；
      2. v-for必须搭配key，key值需唯一（禁止用index作为key）；
      3. v-if/v-else-if/v-else需连续编写，禁止插入其他标签；
      4. 禁止使用v-html（高危，需特殊审批）。
    

修复方式：Skill自动替换为缩写格式，提示补充v-for的key，调整v-if系列顺序；

正确示例：

```vue
<!-- 正确写法 -->
<div :class="isActive ? 'active' : ''" @click="handleClick"></div>
<ul>
  <li v-for="(item, id) in list" :key="id">{{ item.name }}</li>
</ul>
<div v-if="status === 1">启用</div>
<div v-else-if="status === 2">禁用</div>
<div v-else>未知</div>
```

#### 2.3 模板渲染（推荐）

校验标准：模板内文本渲染优先使用{{ }}，复杂计算逻辑需抽离到setup内的computed；禁止模板内调用异步方法；

修复方式：Skill提示将复杂逻辑抽离为computed，禁止模板内异步调用。

### 3. 脚本规范（严重/高危/规范级，分级处理）

#### 3.1 脚本风格（强制，规范级）

校验标准：优先使用<script setup lang="ts">语法，禁止使用Options API（除非兼容旧代码，需标注）；TypeScript必须开启严格模式（strict: true）；

修复方式：Skill提示替换Options API为Composition API，自动补充lang="ts"；

正确示例：

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const count = ref(0)
const doubleCount = computed(() => count.value * 2)
</script>
```

#### 3.2 响应式API使用（强制，高危级）

校验标准：
      1. 基本类型用ref，引用类型用reactive，禁止混用（如用ref包裹对象）；
      2. 解构reactive对象需用toRefs/toRef，禁止直接解构（会丢失响应式）；
      3. 禁止手动修改ref.value（setup外），禁止删除reactive对象的属性（会丢失响应式）；
    

修复方式：Skill自动替换ref/reactive使用错误，提示用toRefs解构，禁止违规修改；

正确示例：

```typescript
// 正确用法
const name = ref('张三')
const user = reactive({ age: 20 })
const { age } = toRefs(user)

// 禁止用法
const user = ref({ age: 20 }) // 引用类型用ref
const { age } = user // 直接解构reactive
```

#### 3.3 生命周期（强制，规范级）

校验标准：使用Vue3生命周期钩子（如onMounted、onUnmounted），禁止使用Vue2钩子（如mounted）；onUnmounted必须清理副作用（如定时器、事件监听、请求取消）；

修复方式：Skill自动替换Vue2钩子为Vue3钩子，提示补充副作用清理代码；

正确示例：

```typescript
import { onMounted, onUnmounted } from 'vue'

onMounted(() => {
  // 开启定时器
  const timer = setInterval(() => {
    console.log('定时器运行')
  }, 1000)
  
  // 清理副作用
  onUnmounted(() => {
    clearInterval(timer)
  })
})
```

#### 3.4 异步请求（强制，高危级）

校验标准：
      1. 异步请求需用axios，统一封装请求拦截器（添加token）、响应拦截器（统一错误处理）；
      2. 组件内请求需在onUnmounted中取消，避免内存泄漏；
      3. 禁止请求成功后直接修改Pinia仓库，需通过actions修改；
      4. 禁止在setup顶层直接发起请求（需在onMounted中）。
    

修复方式：Skill提示补充请求取消逻辑，提示通过Pinia actions修改仓库，调整请求发起位置；

正确示例：

```typescript
import { onMounted, onUnmounted } from 'vue'
import axios from 'axios'
import { useUserStore } from '@/stores/useUserStore'

const userStore = useUserStore()
const source = axios.CancelToken.source()

onMounted(async () => {
  try {
    const res = await axios.get('/api/user/list', {
      cancelToken: source.token
    })
    userStore.setUserList(res.data) // 通过actions修改仓库
  } catch (err) {
    if (axios.isCancel(err)) return
    console.error('请求失败：', err)
  }
})

onUnmounted(() => {
  source.cancel('组件卸载，取消请求') // 取消请求
})
```

### 4. 样式规范（规范级，Skill可自动修复/提示）

#### 4.1 样式隔离（强制）

校验标准：组件样式必须添加scoped属性，避免样式污染；全局样式统一放在src/styles目录下，禁止在组件内写全局样式（除非加:global()）；

修复方式：Skill自动为组件style标签添加scoped，提示将全局样式迁移到指定目录；

正确示例：

```vue
<style scoped>
.user-list {
  margin: 20px 0;
}
/* 全局样式需加:global() */
:global(.container) {
  width: 1200px;
  margin: 0 auto;
}
</style>
```

#### 4.2 样式命名（强制）

校验标准：样式类名使用BEM命名规范（block__element--modifier），禁止使用内联样式（除非动态样式，需标注）；

修复方式：Skill提示修改类名为BEM格式，提示删除不必要的内联样式；

正确示例：

```vue
<template>
  <div class="user-list">
    <div class="user-list__item">
      <button class="user-list__btn--primary">查看</button>
    </div>
  </div>
</template>

<style scoped>
.user-list { /* block */
  padding: 16px;
}
.user-list__item { /* element */
  margin-bottom: 12px;
}
.user-list__btn--primary { /* modifier */
  background: #409eff;
  color: #fff;
}
</style>
```

#### 4.3 样式优化（推荐）

校验标准：禁止使用!important（除非特殊场景，需标注）；优先使用flex/grid布局，禁止使用float布局；样式属性按“布局→盒模型→样式→其他”顺序编写；

修复方式：Skill提示删除不必要的!important，提示优化布局方式，调整属性顺序。

### 5. 工程化规范（严重/规范级，分级处理）

#### 5.1 目录结构（强制，规范级）

校验标准：遵循固定目录结构，禁止随意创建目录，文件放置位置统一：
      
src

├─ assets （静态资源：图片、字体、图标）
      
├─ components （公共组件，大驼峰命名）
      
├─ views （页面组件，大驼峰命名）
      
├─ router （路由配置）
      
├─ stores （Pinia仓库）
      
├─ api （接口请求封装）
      
├─ utils （工具函数）
      
├─ styles （全局样式）
      
└─ App.vue / main.ts （入口文件）
    

修复方式：Skill提示将文件移动到对应目录，提示修改目录/文件命名。

#### 5.2 路由配置（强制，严重级）

校验标准：
      1. 路由path与页面组件名对应，禁止路径与组件无关；
      2. 路由必须添加name（大驼峰，与组件名一致），禁止重名；
      3. 嵌套路由需配置children，父路由path结尾不加“/”；
      4. 页面路由需添加meta（title、requiresAuth等），权限路由需配置requiresAuth: true；
    

修复方式：Skill提示补充路由name、meta，调整path格式，阻断重名路由配置；

正确示例：

```typescript
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/views/Home.vue'
import UserList from '@/views/UserList.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home,
    meta: { title: '首页', requiresAuth: false }
  },
  {
    path: '/user-list',
    name: 'UserList',
    component: UserList,
    meta: { title: '用户列表', requiresAuth: true }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

#### 5.3 依赖管理（强制，高危级）

校验标准：
      1. 禁止安装无用依赖，禁止安装版本过低/过高的依赖（需符合项目统一版本）；
      2. 生产依赖（dependencies）与开发依赖（devDependencies）区分清晰，禁止混淆；
      3. 禁止使用CDN引入依赖（除非特殊审批），统一通过npm/yarn安装；
    

修复方式：Skill提示删除无用依赖，提示调整依赖版本，提示将依赖移动到正确的依赖项中。

### 6. 安全规范（严重/高危级，分级处理）

#### 6.1 XSS防护（强制，严重级）

校验标准：禁止使用v-html（除非特殊场景，需做XSS过滤）；用户输入内容必须做过滤处理；禁止直接将用户输入作为路由参数跳转；

修复方式：Skill阻断v-html使用（需审批），提示对用户输入做过滤，提示安全跳转路由；

正确示例：

```typescript
import { ref, computed } from 'vue'
import { router } from '@/router'

const userInput = ref('')
// 过滤用户输入（简单XSS过滤）
const filteredInput = computed(() => {
  return userInput.value.replace(/<|>|script/g, '')
})

// 安全跳转路由
const goToUser = (userId: string) => {
  // 校验userId格式，避免恶意输入
  if (/^\d+$/.test(userId)) {
    router.push({ name: 'UserDetail', params: { userId } })
  }
}
```

#### 6.2 跨域处理（强制，严重级）

校验标准：开发环境跨域通过Vite配置proxy解决，禁止在代码中直接写跨域地址；生产环境跨域通过后端接口代理，禁止前端配置跨域；

修复方式：Skill提示删除代码中跨域地址，提示配置Vite proxy；

正确示例（Vite配置）：

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080', // 后端接口地址
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
})
```

### 7. 其他核心规范（生产级）

#### 7.1 代码复用（强制，规范级）

校验标准：重复代码（超过3行）需抽离为工具函数/公共组件；禁止在多个组件中复制粘贴相同逻辑；

修复方式：Skill提示抽离重复代码为工具函数/公共组件。

#### 7.2 注释规范（强制，规范级）

校验标准：
      1. 组件注释：组件顶部需添加注释，说明组件功能、Props、Emits；
      2. 方法注释：公共方法、复杂逻辑方法需添加注释，说明功能、参数、返回值；
      3. 行注释：复杂逻辑行需添加注释，禁止无意义注释（如// 点击事件）；
    

修复方式：Skill自动补全注释模板，提示删除无意义注释；

正确示例：

```vue
<script setup lang="ts">
/**
 * 用户列表组件
 * @description 展示用户列表，支持搜索、分页、删除操作
 * @props {number} pageSize - 每页条数（必传，默认10）
 * @emits {Function} deleteUser - 删除用户时触发，参数为用户ID
 */
import { ref } from 'vue'

const props = defineProps({
  pageSize: {
    type: Number,
    default: 10,
    required: true
  }
})

const emit = defineEmits(['deleteUser'])

/**
 * 删除用户
 * @param {string} userId - 用户ID
 * @returns {void}
 */
const handleDelete = (userId: string) => {
  emit('deleteUser', userId)
}
</script>
```

#### 7.3 代码格式化（强制，规范级）

校验标准：缩进统一为2个空格，禁止使用Tab；每行代码长度不超过120字符；语句结尾必须加分号（TypeScript）；

修复方式：Skill自动调整缩进、换行，补充语句结尾分号。

## 版本变更记录（按倒序）

- v2.0.0：基于Vue3官方规范+产品研发落地要求重构，新增安全规范（XSS/跨域）、工程化规范、异步请求规范，适配TypeScript+Pinia，覆盖生产级约束；样式BEM命名、路由配置、依赖管理规则，优化响应式API使用规范；包含命名、模板、脚本基础规则，适配Vue3基础语法。

## 校验结果输出规范（供Qoder Skill参考）

1. 严重违规：列出问题+修复示例，Skill强制阻断代码提交（如XSS漏洞、跨域配置错误、路由重名）；

2. 高危违规：列出问题+自动修复代码，需人工确认（如未清理副作用、请求未取消、依赖混淆）；

3. 规范违规：Skill自动修复，生成修复报告（如命名错误、缩进错误、样式未隔离）；

4. 输出格式：按模块分类（命名/模板/脚本/样式），标注违规行数+修复方式。
> （注：文档部分内容可能由 AI 生成）