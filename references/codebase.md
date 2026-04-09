# 代码范式与技术栈

## 默认技术栈

```
Vue 3 (Composition API + <script setup>) + Vite 5
状态管理：Pinia（Setup Store 写法）
路由：Vue Router 4（懒加载 + meta 守卫）
HTTP：Axios（统一拦截器 + 错误分类）
工具库：@vueuse/core
主力 UI：Element Plus（全局注册）
辅助 UI：Ant Design Vue（ConfigProvider 主题 + 按需引入）
图表：ECharts 5（按需引入，tree-shaking）
Markdown：markdown-it + KaTeX + highlight.js + Mermaid
安全：DOMPurify（显式白名单）
导出：html2canvas
```

## 代码模板

### Pinia Store（Setup Store）

```js
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

// localStorage 安全读写（放在 store 外面）
const safeGetItem = key => {
  try { return JSON.parse(localStorage.getItem(key)) }
  catch { return null }
}
const safeSetItem = (key, value) => {
  try { localStorage.setItem(key, JSON.stringify(value)) }
  catch { /* 静默失败 */ }
}
const safeRemoveItem = key => {
  try { localStorage.removeItem(key) }
  catch { /* 静默失败 */ }
}

export const useXxxStore = defineStore('xxx', () => {
  const data = ref(safeGetItem('xxx') || defaultValue)
  const isEmpty = computed(() => !data.value)

  function setData(newData) {
    data.value = newData
    safeSetItem('xxx', newData)
  }
  function clear() {
    data.value = null
    safeRemoveItem('xxx')
  }

  return { data, isEmpty, setData, clear }
})
```

### Axios 请求封装

```js
const request = axios.create({
  baseURL: '', timeout: 100000,
  headers: { 'Content-Type': 'application/json' }
})

// 请求拦截：挂 token + 过滤空参数
request.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  if (config.params) {
    Object.keys(config.params).forEach(key => {
      const val = config.params[key]
      if (val === '' || val === null || val === undefined) delete config.params[key]
    })
  }
  config.metadata = { startTime: Date.now() }
  return config
})

// 响应拦截：401 防抖 + 动态导入防循环依赖
let isHandling401 = false
request.interceptors.response.use(
  response => {
    const res = response.data
    if (res.code !== undefined && res.code !== 200)
      return Promise.reject(new Error(res.message || '请求失败'))
    return res
  },
  error => {
    if (error.response?.status === 401 && !isHandling401) {
      isHandling401 = true
      import('../store/user').then(({ useUserStore }) => {
        useUserStore().logout()
        isHandling401 = false
      })
    }
    return Promise.reject(error)
  }
)
```

### API 模块

```js
import request from '@/utils/request'

/** 获取列表 */
export const getList = params =>
  request({ url: '/api/xxx/list', method: 'get', params })

/** 新增 */
export const create = data =>
  request({ url: '/api/xxx', method: 'post', data })

/** 更新 */
export const update = (id, data) =>
  request({ url: `/api/xxx/${id}`, method: 'put', data })

/** 删除 */
export const remove = id =>
  request({ url: `/api/xxx/${id}`, method: 'delete' })
```

规范：箭头函数 + 隐式 return、JSDoc 中文注释、barrel export 汇总。

### Composable（单例 vs 实例）

```js
// 单例：模块级 ref = 全局共享
const count = ref(0)
export function useXxx() {
  return { count: computed(() => count.value), increment: () => count.value++ }
}

// 实例：函数内 ref = 每次调用独立
export const useYyy = () => {
  const value = ref(0)
  return { value }
}
```

要点：服务端不可用时静默降级到 localStorage。

### 数据获取（并行 + 部分失败容忍）

```js
const fetchAll = async () => {
  loading.value = true
  try {
    const [res1, res2, res3] = await Promise.allSettled([api1(), api2(), api3()])
    if (res1.status === 'fulfilled') data1.value = res1.value.data
    if (res2.status === 'fulfilled') data2.value = res2.value.data
  } finally {
    loading.value = false
    nextTick(() => initCharts())
  }
}
onMounted(() => fetchAll())
```

### 表格 + 分页

```js
const tableData = ref([])
const loading = ref(false)
const pagination = reactive({ current: 1, size: 10, total: 0 })

const fetchData = async () => {
  loading.value = true
  try {
    const res = await getList({ ...searchForm, page: pagination.current, size: pagination.size })
    tableData.value = res.data.records
    pagination.total = res.data.total
  } finally { loading.value = false }
}
const handleSearch = () => { pagination.current = 1; fetchData() }
```

### 表单提交

```js
const handleSubmit = async () => {
  const valid = await formRef.value.validate().catch(() => false)
  if (!valid) return
  submitting.value = true
  try {
    await createApi(form)
    ElMessage.success('创建成功')
  } finally { submitting.value = false }
}
```

### 删除确认

```js
const handleDelete = async row => {
  try {
    await ElMessageBox.confirm('确定要删除吗？', '提示', { type: 'warning' })
    await removeApi(row.id)
    ElMessage.success('删除成功')
    fetchData()
  } catch (error) {
    if (error !== 'cancel') ElMessage.error('删除失败')
  }
}
```

### ECharts（主题感知）

```js
import echarts from '@/utils/echarts'

let chartInstance = null
const getCSSVar = (name, fallback) =>
  getComputedStyle(document.documentElement).getPropertyValue(name).trim() || fallback

const initChart = () => {
  if (!chartRef.value) return
  chartInstance?.dispose()
  chartInstance = echarts.init(chartRef.value)
  chartInstance.setOption({ /* 用 getCSSVar 读主题色 */ })
}

// MutationObserver 监听 data-theme 变化自动重绘
const observer = new MutationObserver(() => nextTick(initChart))
onMounted(() => {
  window.addEventListener('resize', () => chartInstance?.resize())
  observer.observe(document.documentElement, { attributes: true, attributeFilter: ['data-theme'] })
})
onBeforeUnmount(() => { observer.disconnect(); chartInstance?.dispose() })
```

### 路由守卫

```js
router.beforeEach((to, from, next) => {
  const userStore = useUserStore()
  if (to.meta.requiresAuth && !userStore.isLoggedIn)
    return next({ path: '/login', query: { redirect: to.fullPath } })
  if (to.meta.requiresAdmin && !userStore.isAdmin) return next('/home')
  next()
})
router.afterEach(to => { if (to.meta.title) document.title = to.meta.title })
```

## UI 范式

### 组件层级

```
原子组件（StatCard, TagChip, EmptyState）
  → 功能组件（QuestionList, FilterBar, DataList）
    → 页面组件（xxxManage.vue, xxxDetail.vue）
      → 布局组件（MainLayout.vue, AdminLayout.vue）
```

### 毛玻璃面板

```css
.glass-panel {
  background: var(--glass-bg, rgba(255, 255, 255, 0.7));
  backdrop-filter: var(--glass-blur, blur(12px));
  border: var(--glass-border, 1px solid rgba(255, 255, 255, 0.5));
  border-radius: var(--radius-md, 16px);
}
```

### 动画标配

```css
.fade-in { animation: fadeIn 0.5s cubic-bezier(0.2, 0.8, 0.2, 1) forwards; }
.delay-100 { animation-delay: 100ms; }
.delay-200 { animation-delay: 200ms; }
.hover-lift { transition: transform 0.2s, box-shadow 0.2s; }
.hover-lift:hover { transform: translateY(-2px); box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
```

### 加载 / 空状态

```
表格：v-loading 指令
按钮：:loading 属性
无数据：<EmptyState description="暂无数据" />
新用户无数据 → 不显示空图表，显示引导卡片
```

## Vite 配置要点

```js
// 关键配置
resolve: { alias: { '@': path.resolve(__dirname, 'src') } }
// 手动分包
manualChunks: {
  'echarts': ['echarts/core', 'echarts/charts', 'echarts/components', 'echarts/renderers'],
  'element-plus': ['element-plus'],
  'ant-design-vue': ['ant-design-vue']
}
// 生产环境去 console
esbuild: mode === 'production' ? { drop: ['console', 'debugger'] } : {}
```
