# WMS-Lite 开发规范文档

## 一、代码规范

### 1.1 命名规范

#### 1.1.1 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| Vue 组件 | PascalCase | `ProductList.vue`, `UserForm.vue` |
| JavaScript 文件 | camelCase | `product.service.js`, `auth.middleware.js` |
| 样式文件 | kebab-case | `product-list.scss`, `theme-dark.scss` |
| 配置文件 | kebab-case | `vite.config.js`, `.env.development` |

#### 1.1.2 变量命名

```javascript
// 变量 - camelCase
const productList = []
const totalCount = 0

// 常量 - UPPER_SNAKE_CASE
const MAX_PAGE_SIZE = 100
const DEFAULT_TIMEOUT = 30000

// 函数 - camelCase，动词开头
function getProductList() {}
function createUser() {}
function calculateTotal() {}

// 类 - PascalCase
class ProductService {}
class StockManager {}

// 私有属性/方法 - _前缀
class Example {
  _privateField = null
  _privateMethod() {}
}

// 布尔值 - is/has/can 前缀
const isVisible = true
const hasPermission = false
const canEdit = true
```

#### 1.1.3 数据库命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 表名 | snake_case (小写) | `product`, `inbound_order` |
| 字段名 | camelCase | `productId`, `createdAt` |
| 索引名 | idx_表名_字段 | `idx_product_code` |
| 外键名 | fk_表名_字段 | `fk_order_product` |

### 1.2 注释规范

#### 1.2.1 文件头注释

```javascript
/**
 * @file 商品服务
 * @description 提供商品的增删改查等业务逻辑
 * @author Your Name
 * @date 2024-01-01
 */
```

#### 1.2.2 函数注释

```javascript
/**
 * 获取商品列表
 * @param {Object} filters - 筛选条件
 * @param {string} filters.keyword - 关键词
 * @param {string} filters.categoryId - 分类ID
 * @param {Object} pagination - 分页参数
 * @param {number} pagination.page - 页码
 * @param {number} pagination.pageSize - 每页数量
 * @returns {Promise<{list: Array, total: number}>} 商品列表和总数
 * @throws {Error} 当参数无效时抛出错误
 * @example
 * const result = await productService.list({ keyword: '苹果' }, { page: 1, pageSize: 20 })
 */
async function list(filters, pagination) {
  // ...
}
```

#### 1.2.3 行内注释

```javascript
// 单行注释：说明复杂逻辑
const result = data.filter(item => item.status === 1)

// TODO: 待实现的功能
// TODO(username): 待实现的功能，指定负责人

// FIXME: 需要修复的问题
// FIXME: 这里有个边界情况没处理

// HACK: 临时解决方案
// HACK: 由于API限制，这里做了特殊处理
```

### 1.3 代码格式化

#### 1.3.1 ESLint 配置

```javascript
// .eslintrc.js
module.exports = {
  root: true,
  env: {
    node: true,
    es2021: true
  },
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-recommended',
    '@vue/prettier'
  ],
  rules: {
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'prefer-const': 'error',
    'no-var': 'error',
    'eqeqeq': ['error', 'always'],
    'curly': ['error', 'multi-line'],
    'vue/multi-word-component-names': 'off',
    'vue/no-v-html': 'warn'
  }
}
```

#### 1.3.2 Prettier 配置

```javascript
// .prettierrc.js
module.exports = {
  semi: false,
  singleQuote: true,
  tabWidth: 2,
  trailingComma: 'none',
  printWidth: 100,
  bracketSpacing: true,
  arrowParens: 'avoid',
  endOfLine: 'lf',
  vueIndentScriptAndStyle: false
}
```

#### 1.3.3 EditorConfig

```ini
# .editorconfig
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false
```

---

## 二、前端开发规范

### 2.1 Vue 组件规范

#### 2.1.1 组件结构

```vue
<template>
  <div class="product-list">
    <!-- 模板内容 -->
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useStore } from '@/store'

// Props
const props = defineProps({
  productId: {
    type: String,
    required: true
  }
})

// Emits
const emit = defineEmits(['update', 'delete'])

// 响应式数据
const loading = ref(false)
const productList = ref([])

// 计算属性
const totalCount = computed(() => productList.value.length)

// 方法
function handleEdit(product) {
  emit('update', product)
}

// 生命周期
onMounted(() => {
  fetchProductList()
})
</script>

<style lang="scss" scoped>
.product-list {
  // 样式
}
</style>
```

#### 2.1.2 组件命名

```javascript
// 组件文件名：PascalCase
// ProductList.vue
// UserForm.vue

// 注册组件
import ProductList from './ProductList.vue'

export default {
  components: {
    ProductList
  }
}
```

#### 2.1.3 Props 定义

```javascript
// 推荐：详细定义
defineProps({
  productId: {
    type: String,
    required: true
  },
  pageSize: {
    type: Number,
    default: 20,
    validator: (value) => value > 0
  },
  status: {
    type: String,
    default: 'active',
    validator: (value) => ['active', 'inactive'].includes(value)
  }
})

// 不推荐：简单定义
defineProps(['productId', 'pageSize'])
```

### 2.2 TypeScript 规范

#### 2.2.1 类型定义

```typescript
// types/product.ts

// 接口定义
export interface Product {
  id: string
  code: string
  name: string
  categoryId?: string
  category?: Category
  price: number
  status: ProductStatus
  createdAt: string
  updatedAt: string
}

// 枚举定义
export enum ProductStatus {
  Disabled = 0,
  Enabled = 1
}

// 类型别名
export type ProductForm = Omit<Product, 'id' | 'createdAt' | 'updatedAt'>

// 泛型接口
export interface PaginatedResponse<T> {
  list: T[]
  total: number
  page: number
  pageSize: number
}
```

#### 2.2.2 API 类型

```typescript
// api/product.ts
import type { Product, ProductForm, PaginatedResponse } from '@/types'

export function getProductList(params: ListParams): Promise<PaginatedResponse<Product>> {
  return request.get('/products', { params })
}

export function createProduct(data: ProductForm): Promise<Product> {
  return request.post('/products', data)
}

export function updateProduct(id: string, data: Partial<ProductForm>): Promise<Product> {
  return request.put(`/products/${id}`, data)
}
```

### 2.3 样式规范

#### 2.3.1 CSS 类名规范

```scss
// BEM 命名规范
// Block__Element--Modifier

.product-list {
  // Block
  &__header {
    // Element
    display: flex;
    justify-content: space-between;
  }

  &__title {
    font-size: 18px;
    font-weight: bold;
  }

  &__item {
    padding: 12px;
    border-bottom: 1px solid #eee;

    &--active {
      // Modifier
      background-color: #f5f5f5;
    }

    &--disabled {
      opacity: 0.5;
      pointer-events: none;
    }
  }
}
```

#### 2.3.2 样式组织

```scss
// styles/variables.scss
$primary-color: #409eff;
$success-color: #67c23a;
$warning-color: #e6a23c;
$danger-color: #f56c6c;

$font-size-base: 14px;
$font-size-small: 12px;
$font-size-large: 16px;

$border-radius: 4px;
$spacing-unit: 8px;

// styles/mixins.scss
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin text-ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

// 组件中使用
<style lang="scss" scoped>
@import '@/styles/variables.scss';
@import '@/styles/mixins.scss';

.product-card {
  padding: $spacing-unit * 2;
  border-radius: $border-radius;
  
  &__name {
    @include text-ellipsis;
    font-size: $font-size-base;
  }
}
</style>
```

---

## 三、后端开发规范

### 3.1 项目结构规范

```
backend/
├── src/
│   ├── controllers/         # 控制器 - 处理请求响应
│   ├── services/            # 服务层 - 业务逻辑
│   ├── repositories/        # 数据访问层 - 数据库操作
│   ├── models/              # 数据模型 - Prisma 生成
│   ├── routes/              # 路由定义
│   ├── middlewares/         # 中间件
│   ├── validators/          # 参数校验
│   ├── utils/               # 工具函数
│   ├── config/              # 配置文件
│   ├── constants/           # 常量定义
│   └── types/               # 类型定义
├── tests/                   # 测试文件
├── prisma/                  # Prisma 配置
└── docs/                    # API 文档
```

### 3.2 控制器规范

```javascript
// controllers/product.controller.js
const productService = require('../services/product.service')
const { success, error } = require('../utils/response')

class ProductController {
  /**
   * 获取商品列表
   */
  async list(req, res, next) {
    try {
      const { page, pageSize, ...filters } = req.query
      const result = await productService.list(filters, { page, pageSize })
      res.json(success(result))
    } catch (err) {
      next(err)
    }
  }

  /**
   * 获取商品详情
   */
  async getById(req, res, next) {
    try {
      const { id } = req.params
      const product = await productService.getById(id)
      
      if (!product) {
        return res.status(404).json(error('商品不存在', 40401))
      }
      
      res.json(success(product))
    } catch (err) {
      next(err)
    }
  }

  /**
   * 创建商品
   */
  async create(req, res, next) {
    try {
      const product = await productService.create(req.body)
      res.status(201).json(success(product, '创建成功'))
    } catch (err) {
      next(err)
    }
  }

  /**
   * 更新商品
   */
  async update(req, res, next) {
    try {
      const { id } = req.params
      const product = await productService.update(id, req.body)
      res.json(success(product, '更新成功'))
    } catch (err) {
      next(err)
    }
  }

  /**
   * 删除商品
   */
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

### 3.3 服务层规范

```javascript
// services/product.service.js
const { PrismaClient } = require('@prisma/client')
const prisma = new PrismaClient()
const AppError = require('../utils/error')

class ProductService {
  /**
   * 获取商品列表
   */
  async list(filters, pagination) {
    const { page = 1, pageSize = 20 } = pagination
    const skip = (page - 1) * pageSize

    const where = this._buildWhereClause(filters)

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

  /**
   * 获取商品详情
   */
  async getById(id) {
    return prisma.product.findUnique({
      where: { id },
      include: { category: true }
    })
  }

  /**
   * 创建商品
   */
  async create(data) {
    // 检查编码是否已存在
    const existing = await prisma.product.findUnique({
      where: { code: data.code }
    })
    
    if (existing) {
      throw new AppError('商品编码已存在', 40001)
    }

    return prisma.product.create({
      data,
      include: { category: true }
    })
  }

  /**
   * 更新商品
   */
  async update(id, data) {
    // 检查商品是否存在
    const product = await this.getById(id)
    if (!product) {
      throw new AppError('商品不存在', 40401)
    }

    // 如果修改了编码，检查新编码是否已存在
    if (data.code && data.code !== product.code) {
      const existing = await prisma.product.findUnique({
        where: { code: data.code }
      })
      if (existing) {
        throw new AppError('商品编码已存在', 40001)
      }
    }

    return prisma.product.update({
      where: { id },
      data,
      include: { category: true }
    })
  }

  /**
   * 删除商品
   */
  async delete(id) {
    // 检查商品是否存在
    const product = await this.getById(id)
    if (!product) {
      throw new AppError('商品不存在', 40401)
    }

    // 检查是否有关联库存
    const stock = await prisma.stock.findFirst({
      where: { productId: id }
    })
    if (stock) {
      throw new AppError('商品存在库存，无法删除', 40002)
    }

    return prisma.product.delete({
      where: { id }
    })
  }

  /**
   * 构建查询条件
   * @private
   */
  _buildWhereClause(filters) {
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

    return where
  }
}

module.exports = new ProductService()
```

### 3.4 路由规范

```javascript
// routes/product.routes.js
const express = require('express')
const router = express.Router()
const productController = require('../controllers/product.controller')
const { authenticate, authorize } = require('../middlewares/auth.middleware')
const { validate } = require('../middlewares/validate.middleware')
const { productSchema } = require('../validators/product.validator')

// 所有路由需要认证
router.use(authenticate)

// 商品列表
router.get(
  '/',
  authorize('product:view'),
  productController.list
)

// 商品详情
router.get(
  '/:id',
  authorize('product:view'),
  productController.getById
)

// 创建商品
router.post(
  '/',
  authorize('product:create'),
  validate(productSchema.create),
  productController.create
)

// 更新商品
router.put(
  '/:id',
  authorize('product:update'),
  validate(productSchema.update),
  productController.update
)

// 删除商品
router.delete(
  '/:id',
  authorize('product:delete'),
  productController.delete
)

module.exports = router
```

### 3.5 中间件规范

```javascript
// middlewares/error.middleware.js
const logger = require('../utils/logger')
const AppError = require('../utils/error')

function errorHandler(err, req, res, next) {
  // 记录错误日志
  logger.error({
    message: err.message,
    stack: err.stack,
    url: req.originalUrl,
    method: req.method,
    body: req.body,
    query: req.query,
    params: req.params,
    user: req.user?.id
  })

  // 业务错误
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      code: err.code,
      message: err.message,
      data: null
    })
  }

  // 参数校验错误
  if (err.name === 'ValidationError') {
    return res.status(400).json({
      code: 40001,
      message: err.message,
      data: null
    })
  }

  // JWT 错误
  if (err.name === 'JsonWebTokenError') {
    return res.status(401).json({
      code: 40103,
      message: 'Token无效',
      data: null
    })
  }

  if (err.name === 'TokenExpiredError') {
    return res.status(401).json({
      code: 40102,
      message: 'Token已过期',
      data: null
    })
  }

  // 未知错误
  res.status(500).json({
    code: 50001,
    message: process.env.NODE_ENV === 'production' ? '服务器错误' : err.message,
    data: null
  })
}

function notFoundHandler(req, res) {
  res.status(404).json({
    code: 40400,
    message: '接口不存在',
    data: null
  })
}

module.exports = { errorHandler, notFoundHandler }
```

### 3.6 参数校验规范

```javascript
// validators/product.validator.js
const Joi = require('joi')

const createSchema = Joi.object({
  code: Joi.string().max(50).required().messages({
    'string.empty': '商品编码不能为空',
    'string.max': '商品编码不能超过50个字符',
    'any.required': '商品编码是必填项'
  }),
  barcode: Joi.string().max(50).allow(null, '').messages({
    'string.max': '商品条码不能超过50个字符'
  }),
  name: Joi.string().max(100).required().messages({
    'string.empty': '商品名称不能为空',
    'string.max': '商品名称不能超过100个字符',
    'any.required': '商品名称是必填项'
  }),
  categoryId: Joi.string().allow(null),
  spec: Joi.string().max(100).allow(null, ''),
  unit: Joi.string().max(20).allow(null, ''),
  brand: Joi.string().max(50).allow(null, ''),
  origin: Joi.string().max(50).allow(null, ''),
  price: Joi.number().min(0).allow(null),
  costPrice: Joi.number().min(0).allow(null),
  minStock: Joi.number().integer().min(0).allow(null),
  maxStock: Joi.number().integer().min(0).allow(null),
  shelfLife: Joi.number().integer().min(0).allow(null),
  image: Joi.string().allow(null, ''),
  description: Joi.string().allow(null, ''),
  status: Joi.number().valid(0, 1).default(1)
})

const updateSchema = createSchema.fork(
  ['code', 'name'],
  (schema) => schema.optional()
)

module.exports = {
  create: createSchema,
  update: updateSchema
}
```

---

## 四、Git 规范

### 4.1 分支管理

```
main           # 主分支，生产环境代码
  │
  ├── develop  # 开发分支
  │     │
  │     ├── feature/xxx    # 功能分支
  │     ├── feature/yyy
  │     │
  │     ├── bugfix/xxx     # Bug修复分支
  │     │
  │     └── refactor/xxx   # 重构分支
  │
  └── release/v1.0.0  # 发布分支
        │
        └── hotfix/xxx    # 紧急修复分支
```

### 4.2 分支命名规范

| 分支类型 | 命名格式 | 示例 |
|----------|----------|------|
| 功能分支 | feature/功能描述 | feature/product-management |
| Bug修复 | bugfix/问题描述 | bugfix/login-error |
| 重构 | refactor/重构描述 | refactor/api-structure |
| 发布 | release/版本号 | release/v1.0.0 |
| 热修复 | hotfix/问题描述 | hotfix/security-patch |

### 4.3 Commit 规范

#### 4.3.1 Commit 格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### 4.3.2 Type 类型

| Type | 说明 | 示例 |
|------|------|------|
| feat | 新功能 | feat(product): 添加商品导入功能 |
| fix | Bug修复 | fix(auth): 修复登录Token过期问题 |
| docs | 文档更新 | docs(api): 更新API文档 |
| style | 代码格式 | style: 格式化代码 |
| refactor | 重构 | refactor(stock): 重构库存计算逻辑 |
| perf | 性能优化 | perf(list): 优化列表查询性能 |
| test | 测试 | test(product): 添加商品服务单元测试 |
| chore | 构建/工具 | chore: 更新依赖版本 |
| revert | 回滚 | revert: 回滚上一次提交 |

#### 4.3.3 Scope 范围

```
- auth      认证相关
- product   商品相关
- stock     库存相关
- inbound   入库相关
- outbound  出库相关
- user      用户相关
- system    系统相关
```

#### 4.3.4 Commit 示例

```bash
# 新功能
feat(product): 添加商品批量导入功能

- 支持 Excel 文件导入
- 支持数据校验
- 支持错误信息导出

Closes #123

# Bug修复
fix(inbound): 修复入库单审核后库存未更新问题

修复了审核通过后库存数量未正确更新的Bug

Fixes #456

# 重构
refactor(stock): 重构库存服务层

- 抽取公共方法
- 优化查询性能
- 添加事务支持
```

### 4.4 Commitlint 配置

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'chore', 'revert']
    ],
    'subject-case': [0],
    'subject-max-length': [2, 'always', 100]
  }
}
```

---

## 五、测试规范

### 5.1 单元测试

#### 5.1.1 测试文件命名

```
tests/
├── unit/
│   ├── services/
│   │   ├── product.service.test.js
│   │   └── stock.service.test.js
│   └── utils/
│       └── helpers.test.js
└── integration/
    └── api/
        └── product.test.js
```

#### 5.1.2 测试用例编写

```javascript
// tests/unit/services/product.service.test.js
const productService = require('../../../src/services/product.service')
const { PrismaClient } = require('@prisma/client')

// Mock Prisma
jest.mock('@prisma/client')
const prisma = new PrismaClient()

describe('ProductService', () => {
  beforeEach(() => {
    jest.clearAllMocks()
  })

  describe('list', () => {
    it('应该返回商品列表和总数', async () => {
      // Arrange
      const mockProducts = [
        { id: '1', code: 'P001', name: '商品1' },
        { id: '2', code: 'P002', name: '商品2' }
      ]
      prisma.product.findMany.mockResolvedValue(mockProducts)
      prisma.product.count.mockResolvedValue(2)

      // Act
      const result = await productService.list({}, { page: 1, pageSize: 20 })

      // Assert
      expect(result.list).toHaveLength(2)
      expect(result.total).toBe(2)
      expect(result.page).toBe(1)
      expect(result.pageSize).toBe(20)
    })

    it('应该正确处理筛选条件', async () => {
      // Arrange
      prisma.product.findMany.mockResolvedValue([])
      prisma.product.count.mockResolvedValue(0)

      // Act
      await productService.list({ keyword: '苹果', status: 1 }, { page: 1, pageSize: 20 })

      // Assert
      expect(prisma.product.findMany).toHaveBeenCalledWith(
        expect.objectContaining({
          where: expect.objectContaining({
            OR: expect.any(Array),
            status: 1
          })
        })
      )
    })
  })

  describe('create', () => {
    it('应该成功创建商品', async () => {
      // Arrange
      const productData = { code: 'P001', name: '新商品' }
      prisma.product.findUnique.mockResolvedValue(null)
      prisma.product.create.mockResolvedValue({ id: '1', ...productData })

      // Act
      const result = await productService.create(productData)

      // Assert
      expect(result.code).toBe('P001')
      expect(result.name).toBe('新商品')
    })

    it('编码重复时应该抛出错误', async () => {
      // Arrange
      const productData = { code: 'P001', name: '新商品' }
      prisma.product.findUnique.mockResolvedValue({ id: '1', code: 'P001' })

      // Act & Assert
      await expect(productService.create(productData)).rejects.toThrow('商品编码已存在')
    })
  })
})
```

### 5.2 集成测试

```javascript
// tests/integration/api/product.test.js
const request = require('supertest')
const app = require('../../../src/app')
const { PrismaClient } = require('@prisma/client')

const prisma = new PrismaClient()

describe('Product API', () => {
  let token

  beforeAll(async () => {
    // 登录获取 Token
    const res = await request(app)
      .post('/api/auth/login')
      .send({ username: 'admin', password: 'admin123' })
    token = res.body.data.token
  })

  afterAll(async () => {
    await prisma.$disconnect()
  })

  describe('GET /api/products', () => {
    it('应该返回商品列表', async () => {
      const res = await request(app)
        .get('/api/products')
        .set('Authorization', `Bearer ${token}`)

      expect(res.status).toBe(200)
      expect(res.body.code).toBe(0)
      expect(res.body.data).toHaveProperty('list')
      expect(res.body.data).toHaveProperty('total')
    })

    it('未登录应该返回401', async () => {
      const res = await request(app).get('/api/products')

      expect(res.status).toBe(401)
    })
  })

  describe('POST /api/products', () => {
    it('应该成功创建商品', async () => {
      const res = await request(app)
        .post('/api/products')
        .set('Authorization', `Bearer ${token}`)
        .send({
          code: 'TEST001',
          name: '测试商品'
        })

      expect(res.status).toBe(201)
      expect(res.body.data.code).toBe('TEST001')
    })

    it('缺少必填字段应该返回400', async () => {
      const res = await request(app)
        .post('/api/products')
        .set('Authorization', `Bearer ${token}`)
        .send({})

      expect(res.status).toBe(400)
    })
  })
})
```

### 5.3 测试覆盖率

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  testMatch: ['**/*.test.js'],
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70
    }
  }
}
```

---

## 六、安全规范

### 6.1 密码安全

```javascript
const bcrypt = require('bcrypt')
const SALT_ROUNDS = 10

// 密码加密
async function hashPassword(password) {
  return bcrypt.hash(password, SALT_ROUNDS)
}

// 密码验证
async function verifyPassword(password, hash) {
  return bcrypt.compare(password, hash)
}
```

### 6.2 SQL 注入防护

```javascript
// 使用 Prisma ORM 自动防护 SQL 注入
// 正确做法
const user = await prisma.user.findFirst({
  where: { username: userInput }
})

// 错误做法（不要直接拼接 SQL）
const query = `SELECT * FROM user WHERE username = '${userInput}'`
```

### 6.3 XSS 防护

```javascript
// 使用 DOMPurify 清理用户输入
const DOMPurify = require('isomorphic-dompurify')

function sanitizeInput(input) {
  return DOMPurify.sanitize(input)
}
```

### 6.4 敏感信息处理

```javascript
// 不要在日志中记录敏感信息
logger.info({
  message: '用户登录',
  username: user.username,
  // 不要记录密码
  // password: user.password  ❌
})

// 响应中不要返回敏感字段
const user = await prisma.user.findUnique({
  where: { id },
  select: {
    id: true,
    username: true,
    nickname: true,
    // 不要返回 password 字段
  }
})
```

---

## 七、文档规范

### 7.1 README 结构

```markdown
# 项目名称

## 简介
项目简介和主要功能

## 快速开始

### 环境要求
- Node.js 18+
- npm 9+

### 安装
\`\`\`bash
npm install
\`\`\`

### 运行
\`\`\`bash
npm run dev
\`\`\`

## 项目结构
目录结构说明

## 技术栈
使用的技术和框架

## API 文档
API 文档链接

## 贡献指南
如何参与项目开发

## 许可证
开源协议
```

### 7.2 CHANGELOG 格式

```markdown
# Changelog

## [1.1.0] - 2024-01-15

### Added
- 添加商品批量导入功能
- 添加库存预警通知

### Changed
- 优化列表查询性能

### Fixed
- 修复入库单审核后库存未更新问题

### Deprecated
- 旧的 API 接口将在下个版本移除

## [1.0.0] - 2024-01-01

### Added
- 初始版本发布
```
