# WMS-Lite API 接口设计文档

## 一、接口概述

### 1.1 接口规范

| 规范项 | 说明 |
|--------|------|
| **协议** | HTTP/HTTPS |
| **数据格式** | JSON |
| **字符编码** | UTF-8 |
| **时间格式** | ISO 8601 (YYYY-MM-DDTHH:mm:ss.sssZ) |
| **版本管理** | URL路径版本 (/api/v1) |

### 1.2 接口地址

| 环境 | 地址 |
|------|------|
| 开发环境 | http://localhost:3001/api |
| 测试环境 | https://test-api.wms-lite.com/api |
| 生产环境 | https://api.wms-lite.com/api |

### 1.3 请求方式

| 方法 | 说明 | 使用场景 |
|------|------|----------|
| GET | 获取资源 | 查询操作 |
| POST | 创建资源 | 新增操作 |
| PUT | 更新资源 | 完整更新 |
| PATCH | 部分更新 | 部分字段更新 |
| DELETE | 删除资源 | 删除操作 |

### 1.4 统一响应格式

#### 成功响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    // 响应数据
  }
}
```

#### 分页响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [],
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

#### 错误响应

```json
{
  "code": 40001,
  "message": "参数错误",
  "data": null
}
```

### 1.5 错误码定义

| 错误码范围 | 说明 |
|------------|------|
| 0 | 成功 |
| 400xx | 参数错误 |
| 401xx | 认证错误 |
| 403xx | 权限错误 |
| 404xx | 资源不存在 |
| 500xx | 服务器错误 |

**详细错误码**:

| 错误码 | 说明 |
|--------|------|
| 40001 | 参数校验失败 |
| 40002 | 参数格式错误 |
| 40101 | 未登录 |
| 40102 | Token过期 |
| 40103 | Token无效 |
| 40301 | 无权限访问 |
| 40401 | 资源不存在 |
| 50001 | 服务器内部错误 |
| 50002 | 数据库错误 |

---

## 二、认证接口

### 2.1 用户登录

**接口地址**: `POST /api/auth/login`

**请求参数**:

```json
{
  "username": "admin",
  "password": "admin123"
}
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 用户名 |
| password | string | 是 | 密码 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 7200,
    "user": {
      "id": "xxx",
      "username": "admin",
      "nickname": "管理员",
      "avatar": null,
      "role": {
        "id": "xxx",
        "code": "admin",
        "name": "系统管理员"
      }
    }
  }
}
```

### 2.2 获取当前用户信息

**接口地址**: `GET /api/auth/info`

**请求头**:

```
Authorization: Bearer {token}
```

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "user": {
      "id": "xxx",
      "username": "admin",
      "nickname": "管理员",
      "email": "admin@example.com",
      "phone": "13800138000",
      "avatar": null
    },
    "role": {
      "id": "xxx",
      "code": "admin",
      "name": "系统管理员"
    },
    "permissions": [
      "product:view",
      "product:create",
      "product:update",
      "product:delete"
    ]
  }
}
```

### 2.3 用户登出

**接口地址**: `POST /api/auth/logout`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": null
}
```

### 2.4 修改密码

**接口地址**: `PUT /api/auth/password`

**请求参数**:

```json
{
  "oldPassword": "admin123",
  "newPassword": "newpass123"
}
```

---

## 三、基础数据接口

### 3.1 商品管理

#### 3.1.1 获取商品列表

**接口地址**: `GET /api/products`

**请求参数**:

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| page | int | 否 | 1 | 页码 |
| pageSize | int | 否 | 20 | 每页数量 |
| keyword | string | 否 | - | 关键词(编码/名称) |
| categoryId | string | 否 | - | 分类ID |
| status | int | 否 | - | 状态 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "xxx",
        "code": "P001",
        "barcode": "6901234567890",
        "name": "商品名称",
        "category": {
          "id": "xxx",
          "name": "分类名称"
        },
        "spec": "500g",
        "unit": "个",
        "brand": "品牌",
        "price": 99.00,
        "costPrice": 50.00,
        "minStock": 10,
        "maxStock": 100,
        "status": 1,
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ],
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

#### 3.1.2 获取商品详情

**接口地址**: `GET /api/products/:id`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": "xxx",
    "code": "P001",
    "barcode": "6901234567890",
    "name": "商品名称",
    "category": {
      "id": "xxx",
      "name": "分类名称"
    },
    "spec": "500g",
    "unit": "个",
    "brand": "品牌",
    "origin": "产地",
    "price": 99.00,
    "costPrice": 50.00,
    "minStock": 10,
    "maxStock": 100,
    "shelfLife": 365,
    "image": "/uploads/xxx.jpg",
    "description": "商品描述",
    "status": 1,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

#### 3.1.3 创建商品

**接口地址**: `POST /api/products`

**请求参数**:

```json
{
  "code": "P001",
  "barcode": "6901234567890",
  "name": "商品名称",
  "categoryId": "xxx",
  "spec": "500g",
  "unit": "个",
  "brand": "品牌",
  "origin": "产地",
  "price": 99.00,
  "costPrice": 50.00,
  "minStock": 10,
  "maxStock": 100,
  "shelfLife": 365,
  "image": "/uploads/xxx.jpg",
  "description": "商品描述"
}
```

#### 3.1.4 更新商品

**接口地址**: `PUT /api/products/:id`

**请求参数**: 同创建商品

#### 3.1.5 删除商品

**接口地址**: `DELETE /api/products/:id`

#### 3.1.6 批量删除商品

**接口地址**: `DELETE /api/products`

**请求参数**:

```json
{
  "ids": ["id1", "id2", "id3"]
}
```

#### 3.1.7 更新商品状态

**接口地址**: `PATCH /api/products/:id/status`

**请求参数**:

```json
{
  "status": 0
}
```

### 3.2 商品分类管理

#### 3.2.1 获取分类树

**接口地址**: `GET /api/categories/tree`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "id": "xxx",
      "code": "C001",
      "name": "一级分类",
      "parentId": null,
      "sort": 1,
      "children": [
        {
          "id": "xxx",
          "code": "C001-01",
          "name": "二级分类",
          "parentId": "xxx",
          "sort": 1,
          "children": []
        }
      ]
    }
  ]
}
```

#### 3.2.2 创建分类

**接口地址**: `POST /api/categories`

**请求参数**:

```json
{
  "code": "C001",
  "name": "分类名称",
  "parentId": null,
  "sort": 1
}
```

### 3.3 仓库管理

#### 3.3.1 获取仓库列表

**接口地址**: `GET /api/warehouses`

#### 3.3.2 创建仓库

**接口地址**: `POST /api/warehouses`

**请求参数**:

```json
{
  "code": "W001",
  "name": "主仓库",
  "address": "北京市朝阳区xxx",
  "contact": "张三",
  "phone": "13800138000",
  "area": 1000.00,
  "type": "normal"
}
```

### 3.4 库位管理

#### 3.4.1 获取库位列表

**接口地址**: `GET /api/locations`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| warehouseId | string | 否 | 仓库ID |
| zoneId | string | 否 | 库区ID |
| status | int | 否 | 状态 |

#### 3.4.2 获取库位树

**接口地址**: `GET /api/locations/tree`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "id": "xxx",
      "code": "W001",
      "name": "主仓库",
      "children": [
        {
          "id": "xxx",
          "code": "Z001",
          "name": "存储区",
          "children": [
            {
              "id": "xxx",
              "code": "A-01-01",
              "name": "A区1排1层"
            }
          ]
        }
      ]
    }
  ]
}
```

### 3.5 供应商管理

#### 3.5.1 获取供应商列表

**接口地址**: `GET /api/suppliers`

#### 3.5.2 创建供应商

**接口地址**: `POST /api/suppliers`

**请求参数**:

```json
{
  "code": "S001",
  "name": "供应商名称",
  "contact": "联系人",
  "phone": "13800138000",
  "email": "supplier@example.com",
  "address": "地址",
  "bankName": "开户银行",
  "bankAccount": "账号",
  "taxNumber": "税号"
}
```

### 3.6 客户管理

#### 3.6.1 获取客户列表

**接口地址**: `GET /api/customers`

#### 3.6.2 创建客户

**接口地址**: `POST /api/customers`

---

## 四、库存管理接口

### 4.1 实时库存

#### 4.1.1 获取库存列表

**接口地址**: `GET /api/stocks`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |
| keyword | string | 否 | 商品编码/名称 |
| warehouseId | string | 否 | 仓库ID |
| locationId | string | 否 | 库位ID |
| categoryId | string | 否 | 分类ID |
| hasStock | int | 否 | 是否有库存: 0无 1有 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "xxx",
        "product": {
          "id": "xxx",
          "code": "P001",
          "name": "商品名称",
          "spec": "500g",
          "unit": "个"
        },
        "location": {
          "id": "xxx",
          "code": "A-01-01",
          "name": "A区1排1层"
        },
        "batchNo": "B20240101",
        "productionDate": "2024-01-01",
        "expireDate": "2025-01-01",
        "quantity": 100,
        "frozenQty": 10,
        "availableQty": 90
      }
    ],
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

#### 4.1.2 获取商品库存汇总

**接口地址**: `GET /api/stocks/summary`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| productId | string | 是 | 商品ID |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "productId": "xxx",
    "productCode": "P001",
    "productName": "商品名称",
    "totalQty": 500,
    "totalFrozenQty": 50,
    "totalAvailableQty": 450,
    "locations": [
      {
        "locationId": "xxx",
        "locationCode": "A-01-01",
        "quantity": 100,
        "frozenQty": 10
      }
    ]
  }
}
```

### 4.2 库存流水

#### 4.2.1 获取库存流水

**接口地址**: `GET /api/stock-records`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |
| productId | string | 否 | 商品ID |
| locationId | string | 否 | 库位ID |
| type | string | 否 | 变动类型 |
| orderNo | string | 否 | 单据编号 |
| startDate | string | 否 | 开始日期 |
| endDate | string | 否 | 结束日期 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "xxx",
        "product": {
          "id": "xxx",
          "code": "P001",
          "name": "商品名称"
        },
        "location": {
          "id": "xxx",
          "code": "A-01-01"
        },
        "batchNo": "B20240101",
        "type": "inbound",
        "typeText": "入库",
        "changeQty": 100,
        "beforeQty": 0,
        "afterQty": 100,
        "orderType": "purchase",
        "orderNo": "RK20240101001",
        "operatorName": "张三",
        "createdAt": "2024-01-01T10:00:00.000Z"
      }
    ],
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

### 4.3 库存预警

#### 4.3.1 获取预警列表

**接口地址**: `GET /api/stock-alerts`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| type | string | 否 | 预警类型: low/high/expire |
| status | int | 否 | 处理状态 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "xxx",
        "product": {
          "id": "xxx",
          "code": "P001",
          "name": "商品名称"
        },
        "type": "low",
        "typeText": "低库存",
        "threshold": 10,
        "currentValue": 5,
        "status": 0,
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ],
    "total": 10,
    "page": 1,
    "pageSize": 20
  }
}
```

### 4.4 库存冻结

#### 4.4.1 冻结库存

**接口地址**: `POST /api/stocks/freeze`

**请求参数**:

```json
{
  "productId": "xxx",
  "locationId": "xxx",
  "batchNo": "B20240101",
  "quantity": 10,
  "reason": "质量问题",
  "orderId": null,
  "orderType": null
}
```

#### 4.4.2 解冻库存

**接口地址**: `POST /api/stocks/unfreeze`

**请求参数**:

```json
{
  "freezeId": "xxx"
}
```

---

## 五、入库管理接口

### 5.1 入库单管理

#### 5.1.1 获取入库单列表

**接口地址**: `GET /api/inbound-orders`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |
| orderNo | string | 否 | 单据编号 |
| type | string | 否 | 入库类型 |
| status | int | 否 | 状态 |
| supplierId | string | 否 | 供应商ID |
| warehouseId | string | 否 | 仓库ID |
| startDate | string | 否 | 开始日期 |
| endDate | string | 否 | 结束日期 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "xxx",
        "orderNo": "RK20240101001",
        "type": "purchase",
        "typeText": "采购入库",
        "supplier": {
          "id": "xxx",
          "code": "S001",
          "name": "供应商名称"
        },
        "warehouse": {
          "id": "xxx",
          "code": "W001",
          "name": "主仓库"
        },
        "status": 0,
        "statusText": "草稿",
        "totalQty": 100,
        "totalAmount": 5000.00,
        "expectDate": "2024-01-05",
        "actualDate": null,
        "createdBy": "张三",
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ],
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

#### 5.1.2 获取入库单详情

**接口地址**: `GET /api/inbound-orders/:id`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": "xxx",
    "orderNo": "RK20240101001",
    "type": "purchase",
    "typeText": "采购入库",
    "supplier": {
      "id": "xxx",
      "code": "S001",
      "name": "供应商名称"
    },
    "warehouse": {
      "id": "xxx",
      "code": "W001",
      "name": "主仓库"
    },
    "status": 0,
    "statusText": "草稿",
    "totalQty": 100,
    "totalAmount": 5000.00,
    "expectDate": "2024-01-05",
    "actualDate": null,
    "remark": "备注",
    "items": [
      {
        "id": "xxx",
        "product": {
          "id": "xxx",
          "code": "P001",
          "name": "商品名称",
          "spec": "500g",
          "unit": "个"
        },
        "location": {
          "id": "xxx",
          "code": "A-01-01",
          "name": "A区1排1层"
        },
        "expectQty": 50,
        "actualQty": 0,
        "batchNo": null,
        "productionDate": null,
        "expireDate": null,
        "price": 50.00,
        "amount": 2500.00
      }
    ],
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

#### 5.1.3 创建入库单

**接口地址**: `POST /api/inbound-orders`

**请求参数**:

```json
{
  "type": "purchase",
  "supplierId": "xxx",
  "warehouseId": "xxx",
  "expectDate": "2024-01-05",
  "remark": "备注",
  "items": [
    {
      "productId": "xxx",
      "locationId": "xxx",
      "expectQty": 50,
      "price": 50.00
    }
  ]
}
```

#### 5.1.4 更新入库单

**接口地址**: `PUT /api/inbound-orders/:id`

**请求参数**: 同创建入库单

#### 5.1.5 删除入库单

**接口地址**: `DELETE /api/inbound-orders/:id`

> 仅草稿状态可删除

#### 5.1.6 提交审核

**接口地址**: `POST /api/inbound-orders/:id/submit`

#### 5.1.7 审核入库单

**接口地址**: `POST /api/inbound-orders/:id/review`

**请求参数**:

```json
{
  "approved": true,
  "remark": "审核通过"
}
```

#### 5.1.8 确认入库

**接口地址**: `POST /api/inbound-orders/:id/complete`

**请求参数**:

```json
{
  "items": [
    {
      "id": "xxx",
      "actualQty": 48,
      "locationId": "xxx",
      "batchNo": "B20240101",
      "productionDate": "2024-01-01",
      "expireDate": "2025-01-01"
    }
  ]
}
```

#### 5.1.9 取消入库单

**接口地址**: `POST /api/inbound-orders/:id/cancel`

---

## 六、出库管理接口

### 6.1 出库单管理

#### 6.1.1 获取出库单列表

**接口地址**: `GET /api/outbound-orders`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |
| orderNo | string | 否 | 单据编号 |
| type | string | 否 | 出库类型 |
| status | int | 否 | 状态 |
| customerId | string | 否 | 客户ID |
| warehouseId | string | 否 | 仓库ID |
| startDate | string | 否 | 开始日期 |
| endDate | string | 否 | 结束日期 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "xxx",
        "orderNo": "CK20240101001",
        "type": "sale",
        "typeText": "销售出库",
        "customer": {
          "id": "xxx",
          "code": "C001",
          "name": "客户名称"
        },
        "warehouse": {
          "id": "xxx",
          "code": "W001",
          "name": "主仓库"
        },
        "status": 0,
        "statusText": "草稿",
        "totalQty": 50,
        "totalAmount": 2500.00,
        "expectDate": "2024-01-05",
        "actualDate": null,
        "createdBy": "张三",
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ],
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

#### 6.1.2 获取出库单详情

**接口地址**: `GET /api/outbound-orders/:id`

#### 6.1.3 创建出库单

**接口地址**: `POST /api/outbound-orders`

**请求参数**:

```json
{
  "type": "sale",
  "customerId": "xxx",
  "warehouseId": "xxx",
  "expectDate": "2024-01-05",
  "remark": "备注",
  "items": [
    {
      "productId": "xxx",
      "expectQty": 10,
      "price": 50.00
    }
  ]
}
```

#### 6.1.4 更新出库单

**接口地址**: `PUT /api/outbound-orders/:id`

#### 6.1.5 删除出库单

**接口地址**: `DELETE /api/outbound-orders/:id`

#### 6.1.6 提交审核

**接口地址**: `POST /api/outbound-orders/:id/submit`

#### 6.1.7 审核出库单

**接口地址**: `POST /api/outbound-orders/:id/review`

**请求参数**:

```json
{
  "approved": true,
  "remark": "审核通过"
}
```

#### 6.1.8 确认出库

**接口地址**: `POST /api/outbound-orders/:id/complete`

**请求参数**:

```json
{
  "items": [
    {
      "id": "xxx",
      "actualQty": 10,
      "locationId": "xxx",
      "batchNo": "B20240101"
    }
  ]
}
```

#### 6.1.9 取消出库单

**接口地址**: `POST /api/outbound-orders/:id/cancel`

---

## 七、库内作业接口

### 7.1 库存盘点

#### 7.1.1 获取盘点任务列表

**接口地址**: `GET /api/inventory-tasks`

#### 7.1.2 创建盘点任务

**接口地址**: `POST /api/inventory-tasks`

**请求参数**:

```json
{
  "warehouseId": "xxx",
  "type": "partial",
  "remark": "月度盘点",
  "items": [
    {
      "productId": "xxx",
      "locationId": "xxx",
      "batchNo": "B20240101"
    }
  ]
}
```

#### 7.1.3 开始盘点

**接口地址**: `POST /api/inventory-tasks/:id/start`

#### 7.1.4 盘点录入

**接口地址**: `POST /api/inventory-tasks/:id/count`

**请求参数**:

```json
{
  "itemId": "xxx",
  "actualQty": 98,
  "remark": "差异原因"
}
```

#### 7.1.5 完成盘点

**接口地址**: `POST /api/inventory-tasks/:id/complete`

#### 7.1.6 处理盘点差异

**接口地址**: `POST /api/inventory-tasks/:id/handle-diff`

**请求参数**:

```json
{
  "items": [
    {
      "itemId": "xxx",
      "action": "adjust",
      "remark": "调整原因"
    }
  ]
}
```

### 7.2 移库管理

#### 7.2.1 创建移库单

**接口地址**: `POST /api/move-orders`

**请求参数**:

```json
{
  "warehouseId": "xxx",
  "productId": "xxx",
  "fromLocationId": "xxx",
  "toLocationId": "xxx",
  "batchNo": "B20240101",
  "quantity": 50,
  "remark": "移库原因"
}
```

#### 7.2.2 完成移库

**接口地址**: `POST /api/move-orders/:id/complete`

---

## 八、报表分析接口

### 8.1 库存报表

#### 8.1.1 库存汇总报表

**接口地址**: `GET /api/reports/stock-summary`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| warehouseId | string | 否 | 仓库ID |
| categoryId | string | 否 | 分类ID |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "totalSku": 100,
    "totalQty": 10000,
    "totalAmount": 500000.00,
    "categorySummary": [
      {
        "categoryId": "xxx",
        "categoryName": "分类名称",
        "skuCount": 20,
        "totalQty": 2000,
        "totalAmount": 100000.00
      }
    ]
  }
}
```

#### 8.1.2 库存周转报表

**接口地址**: `GET /api/reports/stock-turnover`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| startDate | string | 是 | 开始日期 |
| endDate | string | 是 | 结束日期 |
| categoryId | string | 否 | 分类ID |

### 8.2 出入库报表

#### 8.2.1 入库统计报表

**接口地址**: `GET /api/reports/inbound-statistics`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| startDate | string | 是 | 开始日期 |
| endDate | string | 是 | 结束日期 |
| type | string | 否 | 入库类型 |
| supplierId | string | 否 | 供应商ID |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "totalOrders": 50,
    "totalQty": 5000,
    "totalAmount": 250000.00,
    "dailyStatistics": [
      {
        "date": "2024-01-01",
        "orderCount": 5,
        "totalQty": 500,
        "totalAmount": 25000.00
      }
    ],
    "typeStatistics": [
      {
        "type": "purchase",
        "typeText": "采购入库",
        "orderCount": 40,
        "totalQty": 4000,
        "totalAmount": 200000.00
      }
    ]
  }
}
```

#### 8.2.2 出库统计报表

**接口地址**: `GET /api/reports/outbound-statistics`

### 8.3 首页看板

#### 8.3.1 获取看板数据

**接口地址**: `GET /api/dashboard`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "stockOverview": {
      "totalSku": 100,
      "totalQty": 10000,
      "lowStockCount": 5,
      "expireSoonCount": 3
    },
    "todayInbound": {
      "orderCount": 10,
      "totalQty": 500,
      "totalAmount": 25000.00
    },
    "todayOutbound": {
      "orderCount": 15,
      "totalQty": 300,
      "totalAmount": 15000.00
    },
    "pendingOrders": {
      "inboundCount": 5,
      "outboundCount": 8,
      "inventoryCount": 2
    },
    "inboundTrend": [
      { "date": "2024-01-01", "qty": 500 },
      { "date": "2024-01-02", "qty": 600 }
    ],
    "outboundTrend": [
      { "date": "2024-01-01", "qty": 300 },
      { "date": "2024-01-02", "qty": 400 }
    ],
    "topProducts": [
      { "productCode": "P001", "productName": "商品A", "outQty": 1000 },
      { "productCode": "P002", "productName": "商品B", "outQty": 800 }
    ]
  }
}
```

---

## 九、系统管理接口

### 9.1 用户管理

#### 9.1.1 获取用户列表

**接口地址**: `GET /api/users`

#### 9.1.2 创建用户

**接口地址**: `POST /api/users`

**请求参数**:

```json
{
  "username": "operator",
  "password": "operator123",
  "nickname": "操作员",
  "email": "operator@example.com",
  "phone": "13800138001",
  "roleId": "xxx",
  "status": 1
}
```

#### 9.1.3 更新用户

**接口地址**: `PUT /api/users/:id`

#### 9.1.4 删除用户

**接口地址**: `DELETE /api/users/:id`

#### 9.1.5 重置密码

**接口地址**: `POST /api/users/:id/reset-password`

### 9.2 角色管理

#### 9.2.1 获取角色列表

**接口地址**: `GET /api/roles`

#### 9.2.2 创建角色

**接口地址**: `POST /api/roles`

**请求参数**:

```json
{
  "code": "operator",
  "name": "操作员",
  "description": "仓库操作人员",
  "permissionIds": ["xxx", "xxx"]
}
```

#### 9.2.3 更新角色权限

**接口地址**: `PUT /api/roles/:id/permissions`

**请求参数**:

```json
{
  "permissionIds": ["xxx", "xxx"]
}
```

### 9.3 权限管理

#### 9.3.1 获取权限树

**接口地址**: `GET /api/permissions/tree`

### 9.4 操作日志

#### 9.4.1 获取操作日志

**接口地址**: `GET /api/operation-logs`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |
| userId | string | 否 | 用户ID |
| module | string | 否 | 模块 |
| startDate | string | 否 | 开始日期 |
| endDate | string | 否 | 结束日期 |

### 9.5 系统配置

#### 9.5.1 获取系统配置

**接口地址**: `GET /api/system-configs`

#### 9.5.2 更新系统配置

**接口地址**: `PUT /api/system-configs/:key`

**请求参数**:

```json
{
  "value": "新配置值"
}
```

---

## 十、打印管理接口

### 10.1 打印模板管理

#### 10.1.1 获取模板列表

**接口地址**: `GET /api/print/templates`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| type | string | 否 | 模板类型: label, document, barcode |
| category | string | 否 | 业务分类 |
| status | int | 否 | 状态 |
| keyword | string | 否 | 关键词搜索 |
| page | int | 否 | 页码，默认1 |
| pageSize | int | 否 | 每页数量，默认20 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "uuid",
        "code": "TPL001",
        "name": "商品标签模板",
        "type": "label",
        "category": "product",
        "width": 50,
        "height": 30,
        "isDefault": 1,
        "status": 1,
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ],
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

---

#### 10.1.2 获取模板详情

**接口地址**: `GET /api/print/templates/:id`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": "uuid",
    "code": "TPL001",
    "name": "商品标签模板",
    "type": "label",
    "category": "product",
    "description": "用于打印商品标签",
    "width": 50,
    "height": 30,
    "unit": "mm",
    "orientation": "portrait",
    "margin": "2,2,2,2",
    "background": null,
    "isDefault": 1,
    "status": 1,
    "elements": [
      {
        "id": "uuid",
        "type": "text",
        "name": "商品名称",
        "x": 5,
        "y": 5,
        "width": 40,
        "height": 8,
        "rotation": 0,
        "zIndex": 1,
        "dataSource": "productName",
        "defaultValue": null,
        "format": null,
        "styles": {
          "fontFamily": "SimHei",
          "fontSize": 10,
          "fontWeight": "bold",
          "color": "#000000",
          "textAlign": "left"
        }
      },
      {
        "id": "uuid",
        "type": "barcode",
        "name": "商品条码",
        "x": 5,
        "y": 15,
        "width": 40,
        "height": 12,
        "rotation": 0,
        "zIndex": 2,
        "dataSource": "barcode",
        "styles": {
          "barcodeType": "CODE128",
          "displayValue": true,
          "textAlign": "center",
          "fontSize": 8,
          "height": 30
        }
      }
    ],
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

---

#### 10.1.3 创建模板

**接口地址**: `POST /api/print/templates`

**请求参数**:

```json
{
  "code": "TPL001",
  "name": "商品标签模板",
  "type": "label",
  "category": "product",
  "description": "用于打印商品标签",
  "width": 50,
  "height": 30,
  "unit": "mm",
  "orientation": "portrait",
  "margin": "2,2,2,2",
  "elements": [
    {
      "type": "text",
      "name": "商品名称",
      "x": 5,
      "y": 5,
      "width": 40,
      "height": 8,
      "dataSource": "productName",
      "styles": {
        "fontFamily": "SimHei",
        "fontSize": 10,
        "fontWeight": "bold"
      }
    }
  ]
}
```

**响应示例**:

```json
{
  "code": 0,
  "message": "创建成功",
  "data": {
    "id": "uuid"
  }
}
```

---

#### 10.1.4 更新模板

**接口地址**: `PUT /api/print/templates/:id`

**请求参数**: 同创建模板

---

#### 10.1.5 删除模板

**接口地址**: `DELETE /api/print/templates/:id`

**响应示例**:

```json
{
  "code": 0,
  "message": "删除成功"
}
```

---

#### 10.1.6 复制模板

**接口地址**: `POST /api/print/templates/:id/copy`

**响应示例**:

```json
{
  "code": 0,
  "message": "复制成功",
  "data": {
    "id": "new-uuid",
    "code": "TPL001_copy"
  }
}
```

---

#### 10.1.7 设为默认模板

**接口地址**: `PUT /api/print/templates/:id/default`

**响应示例**:

```json
{
  "code": 0,
  "message": "设置成功"
}
```

---

### 10.2 数据源接口

#### 10.2.1 获取可用数据源

**接口地址**: `GET /api/print/datasources`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| category | string | 是 | 业务分类: product, inbound, outbound, stock, location |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "field": "productCode",
      "label": "商品编码",
      "type": "string"
    },
    {
      "field": "productName",
      "label": "商品名称",
      "type": "string"
    },
    {
      "field": "barcode",
      "label": "条形码",
      "type": "string"
    },
    {
      "field": "spec",
      "label": "规格",
      "type": "string"
    },
    {
      "field": "unit",
      "label": "单位",
      "type": "string"
    },
    {
      "field": "price",
      "label": "价格",
      "type": "number",
      "format": "currency"
    },
    {
      "field": "quantity",
      "label": "数量",
      "type": "number"
    },
    {
      "field": "batchNo",
      "label": "批次号",
      "type": "string"
    },
    {
      "field": "locationCode",
      "label": "库位编码",
      "type": "string"
    },
    {
      "field": "warehouseName",
      "label": "仓库名称",
      "type": "string"
    }
  ]
}
```

---

### 10.3 打印执行接口

#### 10.3.1 打印预览

**接口地址**: `POST /api/print/preview`

**请求参数**:

```json
{
  "templateId": "uuid",
  "data": [
    {
      "productCode": "P001",
      "productName": "商品名称",
      "barcode": "1234567890",
      "quantity": 100
    }
  ]
}
```

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "previewUrl": "/api/print/preview/uuid",
    "pageCount": 1
  }
}
```

---

#### 10.3.2 执行打印

**接口地址**: `POST /api/print/execute`

**请求参数**:

```json
{
  "templateId": "uuid",
  "businessType": "product",
  "businessId": "uuid",
  "businessNo": "P001",
  "data": [
    {
      "productCode": "P001",
      "productName": "商品名称",
      "barcode": "1234567890",
      "quantity": 100
    }
  ],
  "copies": 1,
  "printerName": "ZDesigner ZT410"
}
```

**响应示例**:

```json
{
  "code": 0,
  "message": "打印任务已发送",
  "data": {
    "recordId": "uuid",
    "status": "success"
  }
}
```

---

#### 10.3.3 批量打印

**接口地址**: `POST /api/print/batch`

**请求参数**:

```json
{
  "templateId": "uuid",
  "businessType": "inbound",
  "items": [
    {
      "businessId": "uuid-1",
      "businessNo": "RK20240101001",
      "data": { "productCode": "P001", "productName": "商品1" }
    },
    {
      "businessId": "uuid-2",
      "businessNo": "RK20240101002",
      "data": { "productCode": "P002", "productName": "商品2" }
    }
  ],
  "copies": 1
}
```

**响应示例**:

```json
{
  "code": 0,
  "message": "批量打印任务已创建",
  "data": {
    "taskId": "uuid",
    "totalCount": 2
  }
}
```

---

### 10.4 打印记录接口

#### 10.4.1 获取打印记录

**接口地址**: `GET /api/print/records`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| templateId | string | 否 | 模板ID |
| businessType | string | 否 | 业务类型 |
| businessNo | string | 否 | 业务单号 |
| startDate | string | 否 | 开始日期 |
| endDate | string | 否 | 结束日期 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "uuid",
        "templateName": "商品标签模板",
        "businessType": "product",
        "businessNo": "P001",
        "copies": 1,
        "printerName": "ZDesigner ZT410",
        "operatorName": "管理员",
        "printedAt": "2024-01-01T10:00:00Z"
      }
    ],
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### 10.5 条码规则接口

#### 10.5.1 获取条码规则列表

**接口地址**: `GET /api/print/barcode-rules`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "uuid",
        "code": "RULE001",
        "name": "商品条码规则",
        "type": "CODE128",
        "prefix": "P",
        "sequence": 1001,
        "padding": 6,
        "suffix": null
      }
    ]
  }
}
```

---

#### 10.5.2 生成条码

**接口地址**: `POST /api/print/barcode-rules/:id/generate`

**请求参数**:

```json
{
  "count": 10
}
```

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "barcodes": [
      "P001001",
      "P001002",
      "P001003"
    ],
    "nextSequence": 1011
  }
}
```

---

## 十一、公共接口

### 11.1 文件上传

**接口地址**: `POST /api/upload`

**请求方式**: multipart/form-data

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| file | file | 是 | 文件 |

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "url": "/uploads/2024/01/xxx.jpg",
    "filename": "xxx.jpg",
    "size": 102400
  }
}
```

### 11.2 数据导出

**接口地址**: `GET /api/export/:type`

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| type | string | 是 | 导出类型: stock/inbound/outbound |
| ... | ... | ... | 其他筛选参数同列表接口 |

**响应**: Excel 文件下载

### 11.3 数据导入

**接口地址**: `POST /api/import/:type`

**请求方式**: multipart/form-data

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| type | string | 是 | 导入类型: product |
| file | file | 是 | Excel 文件 |

### 11.4 获取字典数据

**接口地址**: `GET /api/dict/:type`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": [
    { "value": "purchase", "label": "采购入库" },
    { "value": "return", "label": "退货入库" }
  ]
}
```

### 11.5 生成单据编号

**接口地址**: `GET /api/gen-order-no/:type`

**响应示例**:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "orderNo": "RK20240101001"
  }
}
```

---

## 十二、WebSocket 接口

### 12.1 连接地址

```
ws://localhost:3001/ws
```

### 12.2 认证

连接时通过 query 参数传递 token:

```
ws://localhost:3001/ws?token=xxx
```

### 12.3 消息格式

```json
{
  "event": "stock_alert",
  "data": {
    "type": "low",
    "productId": "xxx",
    "productName": "商品名称",
    "currentQty": 5,
    "threshold": 10
  }
}
```

### 12.4 事件类型

| 事件 | 说明 |
|------|------|
| stock_alert | 库存预警 |
| inbound_complete | 入库完成 |
| outbound_complete | 出库完成 |
| inventory_diff | 盘点差异 |

---

## 十三、接口调用示例

### 13.1 cURL 示例

```bash
# 登录
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123"}'

# 获取商品列表
curl -X GET "http://localhost:3001/api/products?page=1&pageSize=20" \
  -H "Authorization: Bearer {token}"

# 创建入库单
curl -X POST http://localhost:3001/api/inbound-orders \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "purchase",
    "supplierId": "xxx",
    "warehouseId": "xxx",
    "items": [{"productId": "xxx", "expectQty": 10}]
  }'
```

### 13.2 JavaScript 示例

```javascript
import axios from 'axios'

const api = axios.create({
  baseURL: 'http://localhost:3001/api',
  timeout: 30000
})

api.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// 登录
const login = async (username, password) => {
  const { data } = await api.post('/auth/login', { username, password })
  return data
}

// 获取商品列表
const getProducts = async (params) => {
  const { data } = await api.get('/products', { params })
  return data
}

// 创建入库单
const createInboundOrder = async (orderData) => {
  const { data } = await api.post('/inbound-orders', orderData)
  return data
}
```
