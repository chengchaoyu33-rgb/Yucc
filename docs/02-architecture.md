# WMS-Lite 技术架构文档

## 一、架构概述

### 1.1 架构设计原则

| 原则 | 说明 |
|------|------|
| **简洁优先** | 避免过度设计，选择成熟稳定的技术方案 |
| **模块化** | 功能模块独立，低耦合高内聚 |
| **可扩展** | 支持水平扩展，支持数据库切换 |
| **易维护** | 代码结构清晰，文档完善 |
| **安全可靠** | 身份认证、权限控制、数据校验 |

### 1.2 整体架构图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              WMS-Lite 系统架构                               │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                                 客户端层                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  PC 浏览器   │  │  移动浏览器  │  │  PDA 设备   │  │  第三方系统  │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                  前端层                                      │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        Vue 3 + Element Plus                            │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         │  │
│  │  │  路由层  │ │  状态层  │ │  视图层  │ │  组件层  │ │  工具层  │         │  │
│  │  │ Router  │ │ Pinia   │ │ Views   │ │Component│ │ Utils   │         │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘         │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ HTTP / WebSocket
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                  网关层                                      │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                           Nginx (可选)                                 │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                    │  │
│  │  │  静态资源    │  │  反向代理    │  │  负载均衡    │                    │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                  API 层                                      │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        RESTful API + WebSocket                         │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                    │  │
│  │  │ 认证中间件   │  │ 日志中间件   │  │ 校验中间件   │                    │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                    │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                    │  │
│  │  │ 权限中间件   │  │ 错误中间件   │  │ 限流中间件   │                    │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                               业务逻辑层                                     │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                            Controllers                                 │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         │  │
│  │  │ AuthCtrl│ │ BaseCtrl│ │ StockCtrl│ │ InCtrl │ │ OutCtrl │         │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘         │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                        │                                    │
│                                        ▼                                    │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                             Services                                   │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         │  │
│  │  │ AuthSvc │ │ BaseSvc │ │ StockSvc │ │ InSvc  │ │ OutSvc  │         │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘         │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                     │  │
│  │  │ ReportSvc│ │ TaskSvc │ │ AlertSvc │ │ BarcodeSvc│                    │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘                     │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                数据访问层                                    │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                            Prisma ORM                                  │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                    │  │
│  │  │  Model 层   │  │  Query 层   │  │  Migration  │                    │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                  数据层                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   MySQL     │  │ PostgreSQL  │  │   SQLite    │  │    Redis    │        │
│  │   (默认)    │  │   (可选)    │  │   (可选)    │  │   (可选)    │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 二、前端架构

### 2.1 前端技术栈

```
┌─────────────────────────────────────────────────────────────────┐
│                        前端技术栈                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  核心框架                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Vue 3.4+                                                │   │
│  │  • Composition API (组合式 API)                          │   │
│  │  • <script setup> 语法糖                                 │   │
│  │  • 响应式系统                                            │   │
│  │  • Teleport / Suspense                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  UI 组件库                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Element Plus 2.5+                                       │   │
│  │  • 60+ 高质量组件                                        │   │
│  │  • 完整的 TypeScript 支持                                │   │
│  │  • 主题定制                                              │   │
│  │  • 国际化支持                                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  构建工具                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Vite 5.0+                                               │   │
│  │  • 极速冷启动                                            │   │
│  │  • 即时热更新                                            │   │
│  │  • 按需编译                                              │   │
│  │  • 优化的生产构建                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  状态管理                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Pinia 2.1+                                              │   │
│  │  • 直观的 API                                            │   │
│  │  • TypeScript 支持                                       │   │
│  │  • DevTools 集成                                         │   │
│  │  • 模块化设计                                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 前端目录结构

```
frontend/
├── public/                      # 静态资源
│   ├── favicon.ico
│   └── images/
│
├── src/
│   ├── api/                     # API 接口封装
│   │   ├── index.js             # API 入口
│   │   ├── request.js           # Axios 封装
│   │   ├── auth.js              # 认证相关接口
│   │   ├── product.js           # 商品相关接口
│   │   ├── warehouse.js         # 仓库相关接口
│   │   ├── stock.js             # 库存相关接口
│   │   ├── inbound.js           # 入库相关接口
│   │   ├── outbound.js          # 出库相关接口
│   │   └── system.js            # 系统相关接口
│   │
│   ├── components/              # 公共组件
│   │   ├── common/              # 通用组件
│   │   │   ├── Pagination.vue   # 分页组件
│   │   │   ├── SearchForm.vue   # 搜索表单
│   │   │   ├── TableSelect.vue  # 表格选择器
│   │   │   ├── ImageUpload.vue  # 图片上传
│   │   │   └── Dialog.vue       # 弹窗组件
│   │   │
│   │   ├── business/            # 业务组件
│   │   │   ├── ProductSelect.vue    # 商品选择器
│   │   │   ├── WarehouseSelect.vue  # 仓库选择器
│   │   │   ├── LocationSelect.vue   # 库位选择器
│   │   │   ├── SupplierSelect.vue   # 供应商选择器
│   │   │   └── CustomerSelect.vue   # 客户选择器
│   │   │
│   │   └── layout/              # 布局组件
│   │       ├── AppLayout.vue    # 应用布局
│   │       ├── Sidebar.vue      # 侧边栏
│   │       ├── Header.vue       # 顶部栏
│   │       ├── Breadcrumb.vue   # 面包屑
│   │       └── Tabs.vue         # 标签页
│   │
│   ├── views/                   # 页面视图
│   │   ├── dashboard/           # 首页看板
│   │   │   └── index.vue
│   │   │
│   │   ├── base/                # 基础数据管理
│   │   │   ├── product/         # 商品管理
│   │   │   │   ├── index.vue    # 列表页
│   │   │   │   ├── form.vue     # 表单页
│   │   │   │   └── detail.vue   # 详情页
│   │   │   ├── category/        # 分类管理
│   │   │   ├── warehouse/       # 仓库管理
│   │   │   ├── zone/            # 库区管理
│   │   │   ├── location/        # 库位管理
│   │   │   ├── supplier/        # 供应商管理
│   │   │   └── customer/        # 客户管理
│   │   │
│   │   ├── inbound/             # 入库管理
│   │   │   ├── order/           # 入库单
│   │   │   └── record/          # 入库记录
│   │   │
│   │   ├── outbound/            # 出库管理
│   │   │   ├── order/           # 出库单
│   │   │   └── record/          # 出库记录
│   │   │
│   │   ├── stock/               # 库存管理
│   │   │   ├── current/         # 实时库存
│   │   │   ├── record/          # 库存流水
│   │   │   ├── alert/           # 库存预警
│   │   │   └── freeze/          # 库存冻结
│   │   │
│   │   ├── task/                # 库内作业
│   │   │   ├── inventory/       # 库存盘点
│   │   │   └── move/            # 移库管理
│   │   │
│   │   ├── report/              # 报表分析
│   │   │   ├── stock/           # 库存报表
│   │   │   ├── inbound/         # 入库报表
│   │   │   └── outbound/        # 出库报表
│   │   │
│   │   ├── system/              # 系统管理
│   │   │   ├── user/            # 用户管理
│   │   │   ├── role/            # 角色管理
│   │   │   ├── permission/      # 权限管理
│   │   │   ├── log/             # 操作日志
│   │   │   └── config/          # 系统配置
│   │   │
│   │   └── error/               # 错误页面
│   │       ├── 404.vue
│   │       └── 500.vue
│   │
│   ├── store/                   # Pinia 状态管理
│   │   ├── index.js             # Store 入口
│   │   ├── user.js              # 用户状态
│   │   ├── app.js               # 应用状态
│   │   ├── permission.js        # 权限状态
│   │   └── tagsView.js          # 标签页状态
│   │
│   ├── router/                  # 路由配置
│   │   ├── index.js             # 路由入口
│   │   ├── routes.js            # 路由定义
│   │   └── guards.js            # 路由守卫
│   │
│   ├── utils/                   # 工具函数
│   │   ├── index.js             # 工具入口
│   │   ├── auth.js              # 认证工具
│   │   ├── storage.js           # 存储工具
│   │   ├── format.js            # 格式化工具
│   │   ├── validate.js          # 验证工具
│   │   └── permission.js        # 权限工具
│   │
│   ├── hooks/                   # 组合式函数
│   │   ├── useTable.js          # 表格 Hook
│   │   ├── useForm.js           # 表单 Hook
│   │   ├── useDict.js           # 字典 Hook
│   │   └── usePermission.js     # 权限 Hook
│   │
│   ├── styles/                  # 样式文件
│   │   ├── index.scss           # 样式入口
│   │   ├── variables.scss       # 变量定义
│   │   ├── mixins.scss          # 混入
│   │   ├── reset.scss           # 重置样式
│   │   └── element.scss         # Element 主题定制
│   │
│   ├── directives/              # 自定义指令
│   │   ├── permission.js        # 权限指令
│   │   └── loading.js           # 加载指令
│   │
│   ├── constants/               # 常量定义
│   │   ├── index.js             # 常量入口
│   │   ├── status.js            # 状态常量
│   │   └── options.js           # 选项常量
│   │
│   ├── App.vue                  # 根组件
│   └── main.js                  # 入口文件
│
├── index.html                   # HTML 模板
├── vite.config.js               # Vite 配置
├── .env.development             # 开发环境变量
├── .env.production              # 生产环境变量
└── package.json
```

### 2.3 前端核心模块

#### 2.3.1 路由设计

```javascript
// router/routes.js

const routes = [
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/login/index.vue'),
    meta: { title: '登录', requiresAuth: false }
  },
  {
    path: '/',
    component: () => import('@/components/layout/AppLayout.vue'),
    redirect: '/dashboard',
    children: [
      {
        path: 'dashboard',
        name: 'Dashboard',
        component: () => import('@/views/dashboard/index.vue'),
        meta: { title: '首页', icon: 'HomeFilled' }
      },
      // 基础数据
      {
        path: 'base',
        name: 'Base',
        meta: { title: '基础数据', icon: 'Grid' },
        children: [
          {
            path: 'product',
            name: 'Product',
            component: () => import('@/views/base/product/index.vue'),
            meta: { title: '商品管理' }
          },
          // ... 其他子路由
        ]
      },
      // 入库管理
      {
        path: 'inbound',
        name: 'Inbound',
        meta: { title: '入库管理', icon: 'Download' },
        children: [
          {
            path: 'order',
            name: 'InboundOrder',
            component: () => import('@/views/inbound/order/index.vue'),
            meta: { title: '入库单' }
          }
        ]
      },
      // ... 其他模块
    ]
  }
]
```

#### 2.3.2 状态管理设计

```javascript
// store/user.js
import { defineStore } from 'pinia'
import { login, logout, getUserInfo } from '@/api/auth'

export const useUserStore = defineStore('user', {
  state: () => ({
    token: '',
    userInfo: null,
    permissions: []
  }),

  getters: {
    isLoggedIn: (state) => !!state.token,
    username: (state) => state.userInfo?.username || ''
  },

  actions: {
    async login(credentials) {
      const { token } = await login(credentials)
      this.token = token
    },

    async getUserInfo() {
      const { user, permissions } = await getUserInfo()
      this.userInfo = user
      this.permissions = permissions
    },

    async logout() {
      await logout()
      this.$reset()
    }
  },

  persist: true
})
```

#### 2.3.3 API 封装设计

```javascript
// api/request.js
import axios from 'axios'
import { useUserStore } from '@/store/user'
import { ElMessage } from 'element-plus'

const request = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 30000
})

request.interceptors.request.use(
  (config) => {
    const userStore = useUserStore()
    if (userStore.token) {
      config.headers.Authorization = `Bearer ${userStore.token}`
    }
    return config
  },
  (error) => Promise.reject(error)
)

request.interceptors.response.use(
  (response) => {
    const { code, message, data } = response.data
    if (code === 0) {
      return data
    }
    ElMessage.error(message || '请求失败')
    return Promise.reject(new Error(message))
  },
  (error) => {
    if (error.response?.status === 401) {
      // Token 过期，跳转登录
    }
    ElMessage.error(error.message || '网络错误')
    return Promise.reject(error)
  }
)

export default request
```

---

## 三、后端架构

### 3.1 后端技术栈

```
┌─────────────────────────────────────────────────────────────────┐
│                        后端技术栈                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  运行环境                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Node.js 18+ LTS                                         │   │
│  │  • V8 引擎                                               │   │
│  │  • 异步 I/O                                              │   │
│  │  • 事件驱动                                              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Web 框架                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Express 4.18+                                           │   │
│  │  • 路由系统                                              │   │
│  │  • 中间件机制                                            │   │
│  │  • 丰富的生态系统                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ORM 框架                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Prisma 5.0+                                             │   │
│  │  • 类型安全                                              │   │
│  │  • 自动迁移                                              │   │
│  │  • 查询构建器                                            │   │
│  │  • 多数据库支持                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  身份认证                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  JWT (jsonwebtoken)                                      │   │
│  │  • 无状态认证                                            │   │
│  │  • 跨域支持                                              │   │
│  │  • 自定义载荷                                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 后端目录结构

```
backend/
├── src/
│   ├── controllers/             # 控制器层
│   │   ├── auth.controller.js   # 认证控制器
│   │   ├── product.controller.js
│   │   ├── category.controller.js
│   │   ├── warehouse.controller.js
│   │   ├── zone.controller.js
│   │   ├── location.controller.js
│   │   ├── supplier.controller.js
│   │   ├── customer.controller.js
│   │   ├── stock.controller.js
│   │   ├── inbound.controller.js
│   │   ├── outbound.controller.js
│   │   ├── inventory.controller.js
│   │   ├── report.controller.js
│   │   ├── user.controller.js
│   │   ├── role.controller.js
│   │   └── system.controller.js
│   │
│   ├── services/                # 服务层
│   │   ├── auth.service.js      # 认证服务
│   │   ├── product.service.js
│   │   ├── category.service.js
│   │   ├── warehouse.service.js
│   │   ├── zone.service.js
│   │   ├── location.service.js
│   │   ├── supplier.service.js
│   │   ├── customer.service.js
│   │   ├── stock.service.js
│   │   ├── inbound.service.js
│   │   ├── outbound.service.js
│   │   ├── inventory.service.js
│   │   ├── report.service.js
│   │   ├── user.service.js
│   │   ├── role.service.js
│   │   └── system.service.js
│   │
│   ├── routes/                  # 路由定义
│   │   ├── index.js             # 路由入口
│   │   ├── auth.routes.js
│   │   ├── product.routes.js
│   │   ├── category.routes.js
│   │   ├── warehouse.routes.js
│   │   ├── zone.routes.js
│   │   ├── location.routes.js
│   │   ├── supplier.routes.js
│   │   ├── customer.routes.js
│   │   ├── stock.routes.js
│   │   ├── inbound.routes.js
│   │   ├── outbound.routes.js
│   │   ├── inventory.routes.js
│   │   ├── report.routes.js
│   │   ├── user.routes.js
│   │   ├── role.routes.js
│   │   └── system.routes.js
│   │
│   ├── middlewares/             # 中间件
│   │   ├── auth.middleware.js   # 认证中间件
│   │   ├── permission.middleware.js # 权限中间件
│   │   ├── validate.middleware.js   # 校验中间件
│   │   ├── error.middleware.js      # 错误处理中间件
│   │   ├── logger.middleware.js     # 日志中间件
│   │   └── rateLimit.middleware.js  # 限流中间件
│   │
│   ├── validators/              # 参数校验
│   │   ├── auth.validator.js
│   │   ├── product.validator.js
│   │   ├── stock.validator.js
│   │   ├── inbound.validator.js
│   │   └── outbound.validator.js
│   │
│   ├── utils/                   # 工具函数
│   │   ├── index.js             # 工具入口
│   │   ├── response.js          # 响应格式化
│   │   ├── logger.js            # 日志工具
│   │   ├── jwt.js               # JWT 工具
│   │   ├── password.js          # 密码工具
│   │   ├── code.js              # 编码生成器
│   │   └── helpers.js           # 辅助函数
│   │
│   ├── config/                  # 配置文件
│   │   ├── index.js             # 配置入口
│   │   ├── database.js          # 数据库配置
│   │   ├── jwt.js               # JWT 配置
│   │   └── constants.js         # 常量定义
│   │
│   ├── app.js                   # Express 应用
│   └── server.js                # 服务入口
│
├── prisma/
│   ├── schema.prisma            # 数据库模型
│   └── seed.js                  # 初始数据
│
├── tests/                       # 测试文件
│   ├── unit/                    # 单元测试
│   ├── integration/             # 集成测试
│   └── setup.js                 # 测试配置
│
├── .env.example                 # 环境变量示例
├── package.json
└── jest.config.js               # Jest 配置
```

### 3.3 后端分层架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        后端分层架构                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Routes 路由层                          │   │
│  │  • 定义 API 端点                                          │   │
│  │  • 绑定中间件                                             │   │
│  │  • 路由分组                                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                 Middlewares 中间件层                      │   │
│  │  • 认证校验                                               │   │
│  │  • 权限校验                                               │   │
│  │  • 参数校验                                               │   │
│  │  • 日志记录                                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                 Controllers 控制器层                      │   │
│  │  • 接收请求                                               │   │
│  │  • 参数处理                                               │   │
│  │  • 调用服务                                               │   │
│  │  • 返回响应                                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   Services 服务层                         │   │
│  │  • 业务逻辑处理                                           │   │
│  │  • 事务管理                                               │   │
│  │  • 数据组装                                               │   │
│  │  • 跨模块调用                                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Models 模型层                          │   │
│  │  • 数据模型定义 (Prisma)                                  │   │
│  │  • 数据库操作                                             │   │
│  │  • 关联关系                                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   Database 数据库层                       │   │
│  │  • SQLite / MySQL / PostgreSQL                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.4 核心代码示例

#### 3.4.1 Express 应用配置

```javascript
// src/app.js
const express = require('express')
const cors = require('cors')
const helmet = require('helmet')
const compression = require('compression')

const routes = require('./routes')
const { errorHandler, notFoundHandler } = require('./middlewares/error.middleware')
const { requestLogger } = require('./middlewares/logger.middleware')

const app = express()

app.use(helmet())
app.use(cors())
app.use(compression())
app.use(express.json())
app.use(express.urlencoded({ extended: true }))
app.use(requestLogger)

app.use('/api', routes)

app.use(notFoundHandler)
app.use(errorHandler)

module.exports = app
```

#### 3.4.2 控制器示例

```javascript
// src/controllers/product.controller.js
const productService = require('../services/product.service')
const { success, error } = require('../utils/response')

class ProductController {
  async list(req, res, next) {
    try {
      const { page, pageSize, ...filters } = req.query
      const result = await productService.list(filters, { page, pageSize })
      res.json(success(result))
    } catch (err) {
      next(err)
    }
  }

  async getById(req, res, next) {
    try {
      const { id } = req.params
      const product = await productService.getById(id)
      if (!product) {
        return res.status(404).json(error('商品不存在'))
      }
      res.json(success(product))
    } catch (err) {
      next(err)
    }
  }

  async create(req, res, next) {
    try {
      const product = await productService.create(req.body)
      res.status(201).json(success(product, '创建成功'))
    } catch (err) {
      next(err)
    }
  }

  async update(req, res, next) {
    try {
      const { id } = req.params
      const product = await productService.update(id, req.body)
      res.json(success(product, '更新成功'))
    } catch (err) {
      next(err)
    }
  }

  async delete(req, res, next) {
    try {
      const { id } = req.params
      await productService.delete(id)
      res.json(success(null, '删除成功'))
    } catch (err) {
      next(err)
    }
  }
}

module.exports = new ProductController()
```

#### 3.4.3 服务层示例

```javascript
// src/services/product.service.js
const { PrismaClient } = require('@prisma/client')
const prisma = new PrismaClient()

class ProductService {
  async list(filters, pagination) {
    const { page = 1, pageSize = 20 } = pagination
    const skip = (page - 1) * pageSize

    const where = {}
    if (filters.keyword) {
      where.OR = [
        { code: { contains: filters.keyword } },
        { name: { contains: filters.keyword } }
      ]
    }
    if (filters.categoryId) {
      where.categoryId = filters.categoryId
    }
    if (filters.status !== undefined) {
      where.status = filters.status
    }

    const [list, total] = await Promise.all([
      prisma.product.findMany({
        where,
        skip,
        take: pageSize,
        include: { category: true },
        orderBy: { createdAt: 'desc' }
      }),
      prisma.product.count({ where })
    ])

    return { list, total, page, pageSize }
  }

  async getById(id) {
    return prisma.product.findUnique({
      where: { id },
      include: { category: true }
    })
  }

  async create(data) {
    return prisma.product.create({
      data,
      include: { category: true }
    })
  }

  async update(id, data) {
    return prisma.product.update({
      where: { id },
      data,
      include: { category: true }
    })
  }

  async delete(id) {
    return prisma.product.delete({
      where: { id }
    })
  }
}

module.exports = new ProductService()
```

---

## 四、数据库架构

### 4.1 数据库选型

| 数据库 | 适用场景 | 特点 |
|--------|----------|------|
| **MySQL** | 默认/生产环境 | 成熟稳定、生态丰富、性能优秀 |
| **PostgreSQL** | 高级需求 | 功能强大、扩展性好 |
| **SQLite** | 开发测试/小型部署 | 零配置、单文件、轻量级 |

### 4.2 Prisma 配置

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

// 数据库模型定义见 03-database.md
```

### 4.3 数据库连接配置

```javascript
// src/config/database.js
module.exports = {
  development: {
    url: 'mysql://root:password@localhost:3306/wms_lite_dev'
  },
  production: {
    url: process.env.DATABASE_URL
  }
}
```

---

## 五、安全架构

### 5.1 安全措施

```
┌─────────────────────────────────────────────────────────────────┐
│                        安全架构                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  身份认证                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • JWT Token 认证                                        │   │
│  │  • Token 过期机制                                        │   │
│  │  • 密码加密存储 (bcrypt)                                 │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  权限控制                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • RBAC 角色权限模型                                     │   │
│  │  • 接口级权限控制                                        │   │
│  │  • 数据级权限过滤                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  数据安全                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • 参数校验 (Joi / class-validator)                      │   │
│  │  • SQL 注入防护 (Prisma ORM)                             │   │
│  │  • XSS 防护                                              │   │
│  │  • CSRF 防护                                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  传输安全                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • HTTPS 加密传输                                        │   │
│  │  • CORS 跨域配置                                         │   │
│  │  • 安全响应头 (Helmet)                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  访问控制                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • 接口限流                                              │   │
│  │  • IP 黑名单                                             │   │
│  │  • 登录失败锁定                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 JWT 认证流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            JWT 认证流程                                      │
└─────────────────────────────────────────────────────────────────────────────┘

    客户端                        服务端                       数据库
       │                            │                           │
       │  1. 登录请求                │                           │
       │  (username, password)      │                           │
       │ ─────────────────────────▶ │                           │
       │                            │  2. 查询用户               │
       │                            │ ─────────────────────────▶ │
       │                            │                           │
       │                            │  3. 返回用户信息           │
       │                            │ ◀───────────────────────── │
       │                            │                           │
       │                            │  4. 验证密码               │
       │                            │  5. 生成 JWT Token         │
       │                            │                           │
       │  6. 返回 Token             │                           │
       │ ◀───────────────────────── │                           │
       │                            │                           │
       │  7. 携带 Token 访问 API     │                           │
       │  (Authorization: Bearer)   │                           │
       │ ─────────────────────────▶ │                           │
       │                            │  8. 验证 Token             │
       │                            │  9. 解析用户信息           │
       │                            │                           │
       │  10. 返回数据              │                           │
       │ ◀───────────────────────── │                           │
       │                            │                           │
```

---

## 六、性能优化

### 6.1 前端优化

| 优化项 | 说明 |
|--------|------|
| 路由懒加载 | 按需加载页面组件 |
| 组件按需引入 | Element Plus 按需加载 |
| 图片懒加载 | 使用 vue-lazyload |
| 虚拟滚动 | 大数据列表使用虚拟滚动 |
| Gzip 压缩 | 生产环境启用压缩 |
| CDN 加速 | 静态资源使用 CDN |

### 6.2 后端优化

| 优化项 | 说明 |
|--------|------|
| 数据库索引 | 常用查询字段建立索引 |
| 分页查询 | 大数据量使用分页 |
| 连接池 | 数据库连接池管理 |
| 缓存策略 | 热点数据使用缓存 |
| 异步处理 | 耗时操作异步处理 |

### 6.3 数据库优化

```sql
-- 常用索引示例
CREATE INDEX idx_product_code ON product(code);
CREATE INDEX idx_product_category ON product(category_id);
CREATE INDEX idx_stock_product ON stock(product_id);
CREATE INDEX idx_stock_location ON stock(location_id);
CREATE INDEX idx_inbound_order_status ON inbound_order(status);
CREATE INDEX idx_inbound_order_date ON inbound_order(created_at);
```

---

## 七、扩展性设计

### 7.1 水平扩展

```
┌─────────────────────────────────────────────────────────────────┐
│                        水平扩展架构                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                     ┌─────────────┐                             │
│                     │   Nginx     │                             │
│                     │  负载均衡   │                             │
│                     └─────────────┘                             │
│                           │                                     │
│              ┌────────────┼────────────┐                        │
│              │            │            │                        │
│              ▼            ▼            ▼                        │
│        ┌─────────┐  ┌─────────┐  ┌─────────┐                   │
│        │ Node 1  │  │ Node 2  │  │ Node 3  │                   │
│        └─────────┘  └─────────┘  └─────────┘                   │
│              │            │            │                        │
│              └────────────┼────────────┘                        │
│                           │                                     │
│                           ▼                                     │
│                     ┌─────────────┐                             │
│                     │   MySQL     │                             │
│                     │  主从复制   │                             │
│                     └─────────────┘                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 插件化设计

```javascript
// 插件接口设计
const pluginInterface = {
  name: 'plugin-name',
  version: '1.0.0',
  
  // 生命周期钩子
  hooks: {
    beforeInbound: async (order) => {},
    afterInbound: async (order) => {},
    beforeOutbound: async (order) => {},
    afterOutbound: async (order) => {}
  },
  
  // 扩展路由
  routes: (router) => {
    router.get('/plugin/custom', handler)
  },
  
  // 扩展中间件
  middlewares: []
}
```

---

## 八、监控与日志

### 8.1 日志系统

```javascript
// src/utils/logger.js
const winston = require('winston')

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
})

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }))
}

module.exports = logger
```

### 8.2 监控指标

| 指标类型 | 指标项 | 说明 |
|----------|--------|------|
| **系统指标** | CPU 使用率 | 服务器 CPU 占用 |
| | 内存使用率 | 服务器内存占用 |
| | 磁盘使用率 | 磁盘空间占用 |
| **应用指标** | 请求响应时间 | API 响应耗时 |
| | 请求成功率 | API 成功率 |
| | 并发连接数 | 当前连接数 |
| **业务指标** | 入库单数量 | 今日入库单 |
| | 出库单数量 | 今日出库单 |
| | 库存预警数 | 预警商品数 |
