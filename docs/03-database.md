# WMS-Lite 数据库设计文档

## 一、数据库概述

### 1.1 数据库选型

| 数据库 | 适用场景 | 特点 |
|--------|----------|------|
| **MySQL** | 默认/生产环境 | 成熟稳定、生态丰富、性能优秀 |
| **PostgreSQL** | 高级需求 | 功能强大、扩展性好 |
| **SQLite** | 开发测试/小型部署 | 零配置、单文件、轻量级 |

### 1.2 设计原则

| 原则 | 说明 |
|------|------|
| **规范化** | 遵循第三范式，减少数据冗余 |
| **适度反范式** | 高频查询场景适当冗余，提升性能 |
| **统一命名** | 表名小写下划线，字段名驼峰式 |
| **软删除** | 重要数据使用软删除，保留历史记录 |
| **审计字段** | 每张表包含创建时间、更新时间等审计字段 |

### 1.3 ER 图总览

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           数据库 ER 图                                       │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌─────────────┐
                              │    User     │
                              │   用户表    │
                              └─────────────┘
                                     │
                                     │ N:M
                                     ▼
                              ┌─────────────┐
                              │    Role     │
                              │   角色表    │
                              └─────────────┘
                                     │
                                     │ N:M
                                     ▼
                              ┌─────────────┐
                              │ Permission  │
                              │   权限表    │
                              └─────────────┘

┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│  Warehouse  │       │    Zone     │       │  Location   │
│   仓库表    │──1:N──│   库区表    │──1:N──│   库位表    │
└─────────────┘       └─────────────┘       └─────────────┘
                                                   │
                                                   │ 1:N
                                                   ▼
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│  Category   │       │   Product   │       │    Stock    │
│   分类表    │──1:N──│   商品表    │──1:N──│   库存表    │
└─────────────┘       └─────────────┘       └─────────────┘
                            │                      │
                            │                      │ 1:N
                            │                      ▼
                            │               ┌─────────────┐
                            │               │StockRecord  │
                            │               │ 库存流水表  │
                            │               └─────────────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
       ┌───────────┐ ┌───────────┐ ┌───────────┐
       │InboundItem│ │OutboundItem│ │InventoryItem│
       │入库明细   │ │出库明细   │ │盘点明细    │
       └───────────┘ └───────────┘ └───────────┘
              │             │
              │ N:1         │ N:1
              ▼             ▼
       ┌───────────┐ ┌───────────┐
       │ Inbound   │ │ Outbound  │
       │ 入库单    │ │ 出库单    │
       └───────────┘ └───────────┘
              │             │
              │ N:1         │ N:1
              ▼             ▼
       ┌───────────┐ ┌───────────┐
       │ Supplier  │ │ Customer  │
       │ 供应商    │ │  客户     │
       └───────────┘ └───────────┘
```

---

## 二、Prisma Schema 定义

### 2.1 完整 Schema 文件

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

// ==================== 系统管理模块 ====================

model User {
  userId       String   @id @default(uuid()) @map("user_id") @db.VarChar(36)
  username     String   @unique @db.VarChar(50)
  password     String   @db.VarChar(255)
  nickname     String?  @db.VarChar(50)
  email        String?  @db.VarChar(100)
  phone        String?  @db.VarChar(20)
  avatar       String?  @db.VarChar(255)
  status       Int      @default(1)      // 0: 禁用, 1: 启用
  roleCode     String?  @db.VarChar(50)
  role         Role?    @relation(fields: [roleCode], references: [code])
  createdAt    DateTime @default(now()) @map("created_at")
  updatedAt    DateTime @updatedAt @map("updated_at")
  
  @@index([roleCode])
  @@map("user")
}

model Role {
  roleCode      String   @id @default(uuid()) @map("role_code") @db.VarChar(50)
  code          String   @unique @db.VarChar(50)
  name          String   @db.VarChar(50)
  description   String?  @db.VarChar(255)
  status        Int      @default(1)
  permissions   Permission[]
  users         User[]
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")
  
  @@map("role")
}

model Permission {
  permissionId  String   @id @default(uuid()) @map("permission_id") @db.VarChar(36)
  code          String   @unique @db.VarChar(100)
  name          String   @db.VarChar(50)
  type          String   @db.VarChar(20)                    // menu: 菜单, button: 按钮, api: 接口
  parentCode    String?  @db.VarChar(100)
  parent        Permission?  @relation("PermissionTree", fields: [parentCode], references: [code])
  children      Permission[] @relation("PermissionTree")
  path          String?  @db.VarChar(255)
  icon          String?  @db.VarChar(50)
  sort          Int      @default(0)
  status        Int      @default(1)
  roles         Role[]
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")
  
  @@index([parentCode])
  @@map("permission")
}

model OperationLog {
  operationLogId String   @id @default(uuid()) @map("operation_log_id") @db.VarChar(36)
  userId         String   @map("user_id") @db.VarChar(36)
  username       String   @db.VarChar(50)
  module         String   @db.VarChar(50)
  action         String   @db.VarChar(50)
  method         String   @db.VarChar(10)
  url            String   @db.VarChar(255)
  ip             String   @db.VarChar(50)
  params         String?  @db.Text
  result         String?  @db.Text
  status         Int                       // 0: 失败, 1: 成功
  duration       Int?
  createdAt      DateTime @default(now()) @map("created_at")
  
  @@index([userId])
  @@index([createdAt])
  @@map("operation_log")
}

model SystemConfig {
  configId     String   @id @default(uuid()) @map("config_id") @db.VarChar(36)
  configKey    String   @unique @map("config_key") @db.VarChar(100)
  configValue  String   @map("config_value") @db.Text
  description  String?  @db.VarChar(255)
  createdAt    DateTime @default(now()) @map("created_at")
  updatedAt    DateTime @updatedAt @map("updated_at")
  
  @@map("system_config")
}

// ==================== 基础数据模块 ====================

model Category {
  categoryId    String     @id @default(uuid()) @map("category_id") @db.VarChar(36)
  code          String     @unique @db.VarChar(50)
  name          String     @db.VarChar(50)
  parentCode    String?    @db.VarChar(50)
  parent        Category?  @relation("CategoryTree", fields: [parentCode], references: [code])
  children      Category[] @relation("CategoryTree")
  sort          Int        @default(0)
  status        Int        @default(1)
  products      Product[]
  createdAt     DateTime   @default(now()) @map("created_at")
  updatedAt     DateTime   @updatedAt @map("updated_at")
  
  @@index([parentCode])
  @@map("category")
}

model Product {
  productId      String   @id @default(uuid()) @map("product_id") @db.VarChar(36)
  code           String   @unique @db.VarChar(50)
  barcode        String?  @unique @db.VarChar(50)
  name           String   @db.VarChar(100)
  categoryCode   String?  @db.VarChar(50)
  category       Category? @relation(fields: [categoryCode], references: [code])
  spec           String?  @db.VarChar(100)                    // 规格
  unit           String?  @db.VarChar(20)                     // 单位
  brand          String?  @db.VarChar(50)                     // 品牌
  origin         String?  @db.VarChar(50)                     // 产地
  price          Decimal? @db.Decimal(10, 2)
  costPrice      Decimal? @map("cost_price") @db.Decimal(10, 2)
  minStock       Int?     @map("min_stock")                   // 最低库存
  maxStock       Int?     @map("max_stock")                   // 最高库存
  shelfLife      Int?     @map("shelf_life")                  // 保质期(天)
  image          String?  @db.VarChar(255)
  description    String?  @db.Text
  status         Int      @default(1)
  stocks         Stock[]
  inboundItems   InboundItem[]
  outboundItems  OutboundItem[]
  inventoryItems InventoryItem[]
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")
  
  @@index([categoryCode])
  @@index([name])
  @@map("product")
}

model Warehouse {
  warehouseId   String     @id @default(uuid()) @map("warehouse_id") @db.VarChar(36)
  warehouseCode String     @unique @map("warehouse_code") @db.VarChar(50)
  warehouseName String     @map("warehouse_name") @db.VarChar(100)
  address       String?    @map("address") @db.VarChar(255)
  contact       String?    @map("contact") @db.VarChar(50)
  phone         String?    @map("phone") @db.VarChar(20)
  area          Decimal?   @map("area") @db.Decimal(10, 2)
  type          String?    @map("type") @db.VarChar(20)
  status        Int        @default(1) @map("status")
  zones         Zone[]
  inboundOrders InboundOrder[]
  outboundOrders OutboundOrder[]
  inventoryTasks InventoryTask[]
  moveOrders    MoveOrder[]
  createdAt     DateTime   @default(now()) @map("created_at")
  updatedAt     DateTime   @updatedAt @map("updated_at")
  
  @@map("warehouse")
}

model Zone {
  zoneId        String     @id @default(uuid()) @map("zone_id") @db.VarChar(36)
  zoneCode      String     @unique @map("zone_code") @db.VarChar(50)
  zoneName      String     @map("zone_name") @db.VarChar(100)
  warehouseCode String     @map("warehouse_code") @db.VarChar(50)
  warehouse     Warehouse  @relation(fields: [warehouseCode], references: [warehouseCode])
  type          String?    @map("type") @db.VarChar(20)
  area          Decimal?   @map("area") @db.Decimal(10, 2)
  status        Int        @default(1) @map("status")
  locations     Location[]
  createdAt     DateTime   @default(now()) @map("created_at")
  updatedAt     DateTime   @updatedAt @map("updated_at")
  
  @@index([warehouseCode])
  @@map("zone")
}

model Location {
  locationId      String     @id @default(uuid()) @map("location_id") @db.VarChar(36)
  locationCode    String     @unique @map("location_code") @db.VarChar(50)
  locationName    String?    @map("location_name") @db.VarChar(100)
  zoneCode        String     @map("zone_code") @db.VarChar(50)
  zone            Zone       @relation(fields: [zoneCode], references: [zoneCode])
  type            String?    @map("type") @db.VarChar(20)
  row             Int?       @map("row")
  column          Int?       @map("column")
  layer           Int?       @map("layer")
  capacity        Int        @default(1) @map("capacity")
  status          Int        @default(1) @map("status")
  stocks          Stock[]
  inboundItems    InboundItem[]
  outboundItems   OutboundItem[]
  inventoryItems  InventoryItem[]
  createdAt       DateTime   @default(now()) @map("created_at")
  updatedAt       DateTime   @updatedAt @map("updated_at")
  
  @@index([zoneCode])
  @@map("location")
}

model Supplier {
  supplierId    String   @id @default(uuid()) @map("supplier_id") @db.VarChar(36)
  supplierCode  String   @unique @map("supplier_code") @db.VarChar(50)
  supplierName  String   @map("supplier_name") @db.VarChar(100)
  contact       String?  @map("contact") @db.VarChar(50)
  phone         String?  @map("phone") @db.VarChar(20)
  email         String?  @map("email") @db.VarChar(100)
  address       String?  @map("address") @db.VarChar(255)
  bankName      String?  @map("bank_name") @db.VarChar(100)
  bankAccount   String?  @map("bank_account") @db.VarChar(50)
  taxNumber     String?  @map("tax_number") @db.VarChar(50)
  status        Int      @default(1) @map("status")
  inboundOrders InboundOrder[]
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")
  
  @@map("supplier")
}

model Customer {
  customerId     String   @id @default(uuid()) @map("customer_id") @db.VarChar(36)
  customerCode   String   @unique @map("customer_code") @db.VarChar(50)
  customerName   String   @map("customer_name") @db.VarChar(100)
  contact        String?  @map("contact") @db.VarChar(50)
  phone          String?  @map("phone") @db.VarChar(20)
  email          String?  @map("email") @db.VarChar(100)
  address        String?  @map("address") @db.VarChar(255)
  status         Int      @default(1) @map("status")
  outboundOrders OutboundOrder[]
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")
  
  @@map("customer")
}

// ==================== 库存管理模块 ====================

model Stock {
  stockId        String       @id @default(uuid()) @map("stock_id") @db.VarChar(36)
  productCode    String       @db.VarChar(50)
  product        Product      @relation(fields: [productCode], references: [code])
  locationCode   String       @db.VarChar(50)
  location       Location     @relation(fields: [locationCode], references: [code])
  batchNo        String?      @map("batch_no") @db.VarChar(50)     // 批次号
  productionDate DateTime?    @map("production_date")              // 生产日期
  expireDate     DateTime?    @map("expire_date")                  // 过期日期
  quantity       Int          @default(0)        // 可用数量
  frozenQty      Int          @default(0) @map("frozen_qty")  // 冻结数量
  records        StockRecord[]
  createdAt      DateTime     @default(now()) @map("created_at")
  updatedAt      DateTime     @updatedAt @map("updated_at")
  
  @@unique([productCode, locationCode, batchNo])
  @@index([productCode])
  @@index([locationCode])
  @@index([expireDate])
  @@map("stock")
}

model StockRecord {
  stockRecordId String   @id @default(uuid()) @map("stock_record_id") @db.VarChar(36)
  stockId       String   @map("stock_id") @db.VarChar(36)
  stock         Stock    @relation(fields: [stockId], references: [stockId])
  type          String   @db.VarChar(20)        // inbound: 入库, outbound: 出库, freeze: 冻结, unfreeze: 解冻
  changeQty     Int      @map("change_qty")     // 变动数量
  beforeQty     Int      @map("before_qty")     // 变动前数量
  afterQty      Int      @map("after_qty")      // 变动后数量
  orderType     String?  @map("order_type") @db.VarChar(20)         // 单据类型
  orderNo       String?  @map("order_no") @db.VarChar(50)           // 单据编号
  operatorId    String?  @map("operator_id") @db.VarChar(36)        // 操作人ID
  operatorName  String?  @map("operator_name") @db.VarChar(50)      // 操作人姓名
  remark        String?  @db.VarChar(255)
  createdAt     DateTime @default(now()) @map("created_at")
  
  @@index([stockId])
  @@index([type])
  @@index([createdAt])
  @@map("stock_record")
}

model StockFreeze {
  stockFreezeId String   @id @default(uuid()) @map("stock_freeze_id") @db.VarChar(36)
  productCode   String   @db.VarChar(50)
  locationCode  String   @db.VarChar(50)
  batchNo       String?  @map("batch_no") @db.VarChar(50)
  quantity      Int
  reason        String?  @db.VarChar(255)
  orderNo       String?  @map("order_no") @db.VarChar(50)
  orderType     String?  @map("order_type") @db.VarChar(20)
  status        Int      @default(1)          // 0: 已解冻, 1: 冻结中
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")
  
  @@index([productCode])
  @@index([locationCode])
  @@map("stock_freeze")
}

model StockAlert {
  stockAlertId  String   @id @default(uuid()) @map("stock_alert_id") @db.VarChar(36)
  productCode   String   @db.VarChar(50)
  product       Product  @relation(fields: [productCode], references: [code], onDelete: NoAction, onUpdate: NoAction)
  type          String   @db.VarChar(20)        // low: 低库存, high: 高库存, expire: 即将过期
  threshold     Int
  currentValue  Int      @map("current_value")
  status        Int      @default(0)          // 0: 未处理, 1: 已处理
  handledAt     DateTime? @map("handled_at")
  handledBy     String?  @map("handled_by") @db.VarChar(36)
  createdAt     DateTime @default(now()) @map("created_at")
  
  @@index([productCode])
  @@index([status])
  @@map("stock_alert")
}

// ==================== 入库管理模块 ====================

model InboundOrder {
  inboundOrderId String        @id @default(uuid()) @map("inbound_order_id") @db.VarChar(36)
  orderNo        String        @unique @map("order_no") @db.VarChar(50)
  type           String        @db.VarChar(20)            // purchase: 采购, return: 退货, transfer: 调拨, other: 其他
  supplierCode   String?       @map("supplier_code") @db.VarChar(50)
  supplier       Supplier?     @relation(fields: [supplierCode], references: [code])
  warehouseCode  String        @map("warehouse_code") @db.VarChar(50)
  warehouse      Warehouse     @relation(fields: [warehouseCode], references: [code])
  status         Int           @default(0)        // 0: 草稿, 1: 待审核, 2: 已审核, 3: 已入库, 4: 已取消
  totalQty       Int           @default(0) @map("total_qty")
  totalAmount    Decimal?      @map("total_amount") @db.Decimal(12, 2)
  expectDate     DateTime?     @map("expect_date")
  actualDate     DateTime?     @map("actual_date")
  remark         String?       @db.Text
  createdBy      String?       @map("created_by") @db.VarChar(36)
  reviewedBy     String?       @map("reviewed_by") @db.VarChar(36)
  reviewedAt     DateTime?     @map("reviewed_at")
  completedBy    String?       @map("completed_by") @db.VarChar(36)
  completedAt    DateTime?     @map("completed_at")
  items          InboundItem[]
  createdAt      DateTime      @default(now()) @map("created_at")
  updatedAt      DateTime      @updatedAt @map("updated_at")
  
  @@index([status])
  @@index([createdAt])
  @@index([supplierCode])
  @@index([warehouseCode])
  @@map("inbound_order")
}

model InboundItem {
  inboundItemId  String        @id @default(uuid()) @map("inbound_item_id") @db.VarChar(36)
  inboundOrderNo String        @map("inbound_order_no") @db.VarChar(50)
  order          InboundOrder  @relation(fields: [inboundOrderNo], references: [orderNo])
  productCode    String        @db.VarChar(50)
  product        Product       @relation(fields: [productCode], references: [code])
  locationCode   String?       @db.VarChar(50)
  location       Location?     @relation(fields: [locationCode], references: [code])
  expectQty      Int           @map("expect_qty")               // 预期数量
  actualQty      Int           @default(0) @map("actual_qty")   // 实际数量
  batchNo        String?       @map("batch_no") @db.VarChar(50)
  productionDate DateTime?     @map("production_date")
  expireDate     DateTime?     @map("expire_date")
  price          Decimal?      @db.Decimal(10, 2)
  amount         Decimal?      @db.Decimal(12, 2)
  remark         String?       @db.VarChar(255)
  createdAt      DateTime      @default(now()) @map("created_at")
  updatedAt      DateTime      @updatedAt @map("updated_at")
  
  @@index([inboundOrderNo])
  @@index([productCode])
  @@index([locationCode])
  @@map("inbound_item")
}

// ==================== 出库管理模块 ====================

model OutboundOrder {
  outboundOrderId String         @id @default(uuid()) @map("outbound_order_id") @db.VarChar(36)
  orderNo         String         @unique @map("order_no") @db.VarChar(50)
  type            String         @db.VarChar(20)             // sale: 销售, picking: 领料, return: 退料, transfer: 调拨, other: 其他
  customerCode    String?        @map("customer_code") @db.VarChar(50)
  customer        Customer?      @relation(fields: [customerCode], references: [code])
  warehouseCode   String         @map("warehouse_code") @db.VarChar(50)
  warehouse       Warehouse      @relation(fields: [warehouseCode], references: [code])
  status          Int            @default(0)         // 0: 草稿, 1: 待审核, 2: 已审核, 3: 已出库, 4: 已取消
  totalQty        Int            @default(0) @map("total_qty")
  totalAmount     Decimal?       @map("total_amount") @db.Decimal(12, 2)
  expectDate      DateTime?      @map("expect_date")
  actualDate      DateTime?      @map("actual_date")
  remark          String?        @db.Text
  createdBy       String?        @map("created_by") @db.VarChar(36)
  reviewedBy      String?        @map("reviewed_by") @db.VarChar(36)
  reviewedAt      DateTime?      @map("reviewed_at")
  completedBy     String?        @map("completed_by") @db.VarChar(36)
  completedAt     DateTime?      @map("completed_at")
  items           OutboundItem[]
  createdAt       DateTime       @default(now()) @map("created_at")
  updatedAt       DateTime       @updatedAt @map("updated_at")
  
  @@index([status])
  @@index([createdAt])
  @@index([customerCode])
  @@index([warehouseCode])
  @@map("outbound_order")
}

model OutboundItem {
  outboundItemId  String         @id @default(uuid()) @map("outbound_item_id") @db.VarChar(36)
  outboundOrderNo String         @map("outbound_order_no") @db.VarChar(50)
  order           OutboundOrder  @relation(fields: [outboundOrderNo], references: [orderNo])
  productCode     String         @db.VarChar(50)
  product         Product        @relation(fields: [productCode], references: [code])
  locationCode    String?        @db.VarChar(50)
  location        Location?      @relation(fields: [locationCode], references: [code])
  expectQty       Int            @map("expect_qty")              // 预期数量
  actualQty       Int            @default(0) @map("actual_qty")  // 实际数量
  batchNo         String?        @map("batch_no") @db.VarChar(50)
  price           Decimal?       @db.Decimal(10, 2)
  amount          Decimal?       @db.Decimal(12, 2)
  remark          String?        @db.VarChar(255)
  createdAt       DateTime       @default(now()) @map("created_at")
  updatedAt       DateTime       @updatedAt @map("updated_at")
  
  @@index([outboundOrderNo])
  @@index([productCode])
  @@index([locationCode])
  @@map("outbound_item")
}

// ==================== 库内作业模块 ====================

model InventoryTask {
  inventoryTaskId String          @id @default(uuid()) @map("inventory_task_id") @db.VarChar(36)
  taskNo          String          @unique @map("task_no") @db.VarChar(50)
  warehouseCode   String          @map("warehouse_code") @db.VarChar(50)
  warehouse       Warehouse       @relation(fields: [warehouseCode], references: [code])
  type            String          @db.VarChar(20)              // full: 全盘, partial: 部分盘点, cycle: 循环盘点
  status          Int             @default(0)         // 0: 草稿, 1: 进行中, 2: 已完成, 3: 已取消
  totalQty        Int             @default(0) @map("total_qty")
  diffQty         Int             @default(0) @map("diff_qty")
  startTime       DateTime?       @map("start_time")
  endTime         DateTime?       @map("end_time")
  remark          String?         @db.Text
  createdBy       String?         @map("created_by") @db.VarChar(36)
  completedBy     String?         @map("completed_by") @db.VarChar(36)
  completedAt     DateTime?       @map("completed_at")
  items           InventoryItem[]
  createdAt       DateTime        @default(now()) @map("created_at")
  updatedAt       DateTime        @updatedAt @map("updated_at")
  
  @@index([warehouseCode])
  @@index([status])
  @@map("inventory_task")
}

model InventoryItem {
  inventoryItemId String         @id @default(uuid()) @map("inventory_item_id") @db.VarChar(36)
  taskNo          String         @map("task_no") @db.VarChar(50)
  task            InventoryTask  @relation(fields: [taskNo], references: [taskNo])
  productCode     String         @db.VarChar(50)
  product         Product        @relation(fields: [productCode], references: [code])
  locationCode    String?        @db.VarChar(50)
  location        Location?      @relation(fields: [locationCode], references: [code])
  batchNo         String?        @map("batch_no") @db.VarChar(50)
  systemQty       Int            @map("system_qty")                // 系统数量
  actualQty       Int?           @map("actual_qty")                // 实盘数量
  diffQty         Int?           @map("diff_qty")                  // 差异数量
  status          Int            @default(0)         // 0: 待盘点, 1: 已盘点
  countedBy       String?        @map("counted_by") @db.VarChar(36)
  countedAt       DateTime?      @map("counted_at")
  remark          String?        @db.VarChar(255)
  createdAt       DateTime       @default(now()) @map("created_at")
  updatedAt       DateTime       @updatedAt @map("updated_at")
  
  @@index([taskNo])
  @@index([productCode])
  @@index([locationCode])
  @@map("inventory_item")
}

model MoveOrder {
  moveOrderId    String   @id @default(uuid()) @map("move_order_id") @db.VarChar(36)
  orderNo        String   @unique @map("order_no") @db.VarChar(50)
  warehouseCode  String   @map("warehouse_code") @db.VarChar(50)
  warehouse      Warehouse @relation(fields: [warehouseCode], references: [code])
  type           String   @db.VarChar(20)                       // location: 库位间移动, zone: 库区间移动
  status         Int      @default(0)         // 0: 草稿, 1: 进行中, 2: 已完成, 3: 已取消
  productCode    String   @db.VarChar(50)
  product        Product  @relation(fields: [productCode], references: [code])
  fromLocationCode String @map("from_location_code") @db.VarChar(50)
  toLocationCode   String @map("to_location_code") @db.VarChar(50)
  quantity       Int
  batchNo        String?  @map("batch_no") @db.VarChar(50)
  remark         String?  @db.VarChar(255)
  createdBy      String?  @map("created_by") @db.VarChar(36)
  completedBy    String?  @map("completed_by") @db.VarChar(36)
  completedAt    DateTime? @map("completed_at")
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")
  
  @@index([warehouseCode])
  @@index([productCode])
  @@map("move_order")
}

// ==================== 打印管理模块 ====================

model PrintTemplate {
  printTemplateId String   @id @default(uuid()) @map("print_template_id") @db.VarChar(36)
  templateCode    String   @unique @map("template_code") @db.VarChar(50)
  templateName    String   @map("template_name") @db.VarChar(100)
  type            String   @map("type") @db.VarChar(20)
  category        String?  @map("category") @db.VarChar(50)
  description     String?  @map("description") @db.VarChar(255)
  width           Int      @map("width")
  height          Int      @map("height")
  unit            String   @default("mm") @map("unit") @db.VarChar(10)
  orientation     String   @default("portrait") @map("orientation") @db.VarChar(10)
  margin          String?  @map("margin") @db.VarChar(50)
  background      String?  @map("background") @db.VarChar(255)
  elements        PrintElement[]
  isDefault       Int      @default(0) @map("is_default")
  status          Int      @default(1) @map("status")
  createdBy       String?  @map("created_by") @db.VarChar(36)
  createdAt       DateTime @default(now()) @map("created_at")
  updatedAt       DateTime @updatedAt @map("updated_at")
  
  @@index([type])
  @@index([category])
  @@map("print_template")
}

model PrintElement {
  printElementId String   @id @default(uuid()) @map("print_element_id") @db.VarChar(36)
  templateCode   String   @map("template_code") @db.VarChar(50)
  template       PrintTemplate @relation(fields: [templateCode], references: [templateCode], onDelete: Cascade)
  type           String   @map("type") @db.VarChar(20)
  name           String   @map("name") @db.VarChar(50)
  x              Float    @map("x")
  y              Float    @map("y")
  width          Float    @map("width")
  height         Float    @map("height")
  rotation       Float    @default(0) @map("rotation")
  zIndex         Int      @default(0) @map("z_index")
  dataSource     String?  @map("data_source") @db.VarChar(100)
  defaultValue   String?  @map("default_value") @db.Text
  format         String?  @map("format") @db.VarChar(50)
  styles         String?  @map("styles") @db.Text
  bindings       String?  @map("bindings") @db.Text
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")
  
  @@index([templateCode])
  @@map("print_element")
}

model PrintRecord {
  printRecordId  String   @id @default(uuid()) @map("print_record_id") @db.VarChar(36)
  templateCode   String   @map("template_code") @db.VarChar(50)
  template       PrintTemplate @relation(fields: [templateCode], references: [templateCode])
  businessType   String   @map("business_type") @db.VarChar(50)
  businessNo     String?  @map("business_no") @db.VarChar(50)
  printData      String   @map("print_data") @db.Text
  copies         Int      @default(1) @map("copies")
  printerName    String?  @map("printer_name") @db.VarChar(100)
  operatorId     String?  @map("operator_id") @db.VarChar(36)
  operatorName   String?  @map("operator_name") @db.VarChar(50)
  printedAt      DateTime @default(now()) @map("printed_at")
  createdAt      DateTime @default(now()) @map("created_at")
  
  @@index([templateCode])
  @@index([businessType])
  @@index([businessNo])
  @@index([printedAt])
  @@map("print_record")
}

model BarcodeRule {
  barcodeRuleId String   @id @default(uuid()) @map("barcode_rule_id") @db.VarChar(36)
  ruleCode      String   @unique @map("rule_code") @db.VarChar(50)
  ruleName      String   @map("rule_name") @db.VarChar(100)
  type          String   @map("type") @db.VarChar(20)
  prefix        String?  @map("prefix") @db.VarChar(20)
  sequence      Int      @default(1) @map("sequence")
  padding       Int      @default(0) @map("padding")
  suffix        String?  @map("suffix") @db.VarChar(20)
  description   String?  @map("description") @db.VarChar(255)
  status        Int      @default(1) @map("status")
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")
  
  @@map("barcode_rule")
}
```

---

## 三、数据表详细设计

### 3.1 系统管理模块

#### 3.1.1 用户表 (user)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| user_id | VARCHAR(36) | 是 | UUID | 主键 |
| username | VARCHAR(50) | 是 | - | 用户名，唯一 |
| password | VARCHAR(255) | 是 | - | 密码(bcrypt加密) |
| nickname | VARCHAR(50) | 否 | - | 昵称 |
| email | VARCHAR(100) | 否 | - | 邮箱 |
| phone | VARCHAR(20) | 否 | - | 手机号 |
| avatar | VARCHAR(255) | 否 | - | 头像URL |
| status | INT | 是 | 1 | 状态: 0禁用 1启用 |
| role_code | VARCHAR(50) | 否 | - | 角色编码 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**索引设计**:
- PRIMARY KEY (user_id)
- UNIQUE INDEX idx_username (username)
- INDEX idx_role_code (role_code)

#### 3.1.2 角色表 (role)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| role_code | VARCHAR(50) | 是 | UUID | 主键 |
| code | VARCHAR(50) | 是 | - | 角色编码，唯一 |
| name | VARCHAR(50) | 是 | - | 角色名称 |
| description | VARCHAR(255) | 否 | - | 描述 |
| status | INT | 是 | 1 | 状态 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

#### 3.1.3 权限表 (permission)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| permission_id | VARCHAR(36) | 是 | UUID | 主键 |
| code | VARCHAR(100) | 是 | - | 权限编码，唯一 |
| name | VARCHAR(50) | 是 | - | 权限名称 |
| type | VARCHAR(20) | 是 | - | 类型: menu/button/api |
| parent_code | VARCHAR(100) | 否 | - | 父级编码 |
| path | VARCHAR(255) | 否 | - | 路由路径/API路径 |
| icon | VARCHAR(50) | 否 | - | 图标 |
| sort | INT | 是 | 0 | 排序 |
| status | INT | 是 | 1 | 状态 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

#### 3.1.4 操作日志表 (operation_log)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| operation_log_id | VARCHAR(36) | 是 | UUID | 主键 |
| user_id | VARCHAR(36) | 是 | - | 用户ID |
| username | VARCHAR(50) | 是 | - | 用户名 |
| module | VARCHAR(50) | 是 | - | 模块名称 |
| action | VARCHAR(50) | 是 | - | 操作动作 |
| method | VARCHAR(10) | 是 | - | HTTP方法 |
| url | VARCHAR(255) | 是 | - | 请求URL |
| ip | VARCHAR(50) | 是 | - | IP地址 |
| params | TEXT | 否 | - | 请求参数 |
| result | TEXT | 否 | - | 响应结果 |
| status | INT | 是 | - | 状态: 0失败 1成功 |
| duration | INT | 否 | - | 耗时(ms) |
| created_at | DATETIME | 是 | now() | 创建时间 |

---

### 3.2 基础数据模块

#### 3.2.1 商品分类表 (category)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| category_id | VARCHAR(36) | 是 | UUID | 主键 |
| code | VARCHAR(50) | 是 | - | 分类编码，唯一 |
| name | VARCHAR(50) | 是 | - | 分类名称 |
| parent_code | VARCHAR(50) | 否 | - | 父级编码 |
| sort | INT | 是 | 0 | 排序 |
| status | INT | 是 | 1 | 状态 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

#### 3.2.2 商品表 (product)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| product_id | VARCHAR(36) | 是 | UUID | 主键 |
| code | VARCHAR(50) | 是 | - | 商品编码，唯一 |
| barcode | VARCHAR(50) | 否 | - | 条码，唯一 |
| name | VARCHAR(100) | 是 | - | 商品名称 |
| category_code | VARCHAR(50) | 否 | - | 分类编码 |
| spec | VARCHAR(100) | 否 | - | 规格 |
| unit | VARCHAR(20) | 否 | - | 单位 |
| brand | VARCHAR(50) | 否 | - | 品牌 |
| origin | VARCHAR(50) | 否 | - | 产地 |
| price | DECIMAL(10,2) | 否 | - | 售价 |
| cost_price | DECIMAL(10,2) | 否 | - | 成本价 |
| min_stock | INT | 否 | - | 最低库存 |
| max_stock | INT | 否 | - | 最高库存 |
| shelf_life | INT | 否 | - | 保质期(天) |
| image | VARCHAR(255) | 否 | - | 图片URL |
| description | TEXT | 否 | - | 描述 |
| status | INT | 是 | 1 | 状态 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**索引设计**:
- PRIMARY KEY (product_id)
- UNIQUE INDEX idx_code (code)
- UNIQUE INDEX idx_barcode (barcode)
- INDEX idx_category_code (category_code)
- INDEX idx_name (name)

#### 3.2.3 仓库表 (warehouse)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| warehouse_id | VARCHAR(36) | 是 | UUID | 主键 |
| warehouse_code | VARCHAR(50) | 是 | - | 仓库编码，唯一 |
| warehouse_name | VARCHAR(100) | 是 | - | 仓库名称 |
| address | VARCHAR(255) | 否 | - | 地址 |
| contact | VARCHAR(50) | 否 | - | 联系人 |
| phone | VARCHAR(20) | 否 | - | 联系电话 |
| area | DECIMAL(10,2) | 否 | - | 面积(㎡) |
| type | VARCHAR(20) | 否 | - | 仓库类型 |
| status | INT | 是 | 1 | 状态 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

#### 3.2.4 库区表 (zone)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| zone_id | VARCHAR(36) | 是 | UUID | 主键 |
| zone_code | VARCHAR(50) | 是 | - | 库区编码，唯一 |
| zone_name | VARCHAR(100) | 是 | - | 库区名称 |
| warehouse_code | VARCHAR(50) | 是 | - | 仓库编码 |
| type | VARCHAR(20) | 否 | - | 库区类型 |
| area | DECIMAL(10,2) | 否 | - | 面积 |
| status | INT | 是 | 1 | 状态 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**库区类型枚举**:
| 值 | 说明 |
|----|------|
| receiving | 收货区 |
| storage | 存储区 |
| picking | 拣货区 |
| shipping | 发货区 |
| quality | 质检区 |
| return | 退货区 |

#### 3.2.5 库位表 (location)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| location_id | VARCHAR(36) | 是 | UUID | 主键 |
| location_code | VARCHAR(50) | 是 | - | 库位编码，唯一 |
| location_name | VARCHAR(100) | 否 | - | 库位名称 |
| zone_code | VARCHAR(50) | 是 | - | 库区编码 |
| type | VARCHAR(20) | 否 | - | 库位类型 |
| row | INT | 否 | - | 行号 |
| column | INT | 否 | - | 列号 |
| layer | INT | 否 | - | 层号 |
| capacity | INT | 是 | 1 | 容量 |
| status | INT | 是 | 1 | 状态 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**库位类型枚举**:
| 值 | 说明 |
|----|------|
| shelf | 货架位 |
| pallet | 托盘位 |
| bin | 料箱位 |
| floor | 地堆位 |

**库位状态枚举**:
| 值 | 说明 |
|----|------|
| 0 | 禁用 |
| 1 | 空闲 |
| 2 | 占用 |

---

### 3.3 库存管理模块

#### 3.3.1 库存表 (stock)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| stock_id | VARCHAR(36) | 是 | UUID | 主键 |
| product_code | VARCHAR(50) | 是 | - | 商品编码 |
| location_code | VARCHAR(50) | 是 | - | 库位编码 |
| batch_no | VARCHAR(50) | 否 | - | 批次号 |
| production_date | DATE | 否 | - | 生产日期 |
| expire_date | DATE | 否 | - | 过期日期 |
| quantity | INT | 是 | 0 | 可用数量 |
| frozen_qty | INT | 是 | 0 | 冻结数量 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**索引设计**:
- PRIMARY KEY (stock_id)
- UNIQUE INDEX idx_product_location_batch (product_code, location_code, batch_no)
- INDEX idx_product (product_code)
- INDEX idx_location (location_code)
- INDEX idx_expire (expire_date)

#### 3.3.2 库存流水表 (stock_record)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| stock_record_id | VARCHAR(36) | 是 | UUID | 主键 |
| stock_id | VARCHAR(36) | 是 | - | 库存ID |
| type | VARCHAR(20) | 是 | - | 变动类型 |
| change_qty | INT | 是 | - | 变动数量 |
| before_qty | INT | 是 | - | 变动前数量 |
| after_qty | INT | 是 | - | 变动后数量 |
| order_type | VARCHAR(20) | 否 | - | 单据类型 |
| order_no | VARCHAR(50) | 否 | - | 单据编号 |
| operator_id | VARCHAR(36) | 否 | - | 操作人ID |
| operator_name | VARCHAR(50) | 否 | - | 操作人姓名 |
| remark | VARCHAR(255) | 否 | - | 备注 |
| created_at | DATETIME | 是 | now() | 创建时间 |

**变动类型枚举**:
| 值 | 说明 |
|----|------|
| inbound | 入库 |
| outbound | 出库 |
| freeze | 冻结 |
| unfreeze | 解冻 |
| inventory_gain | 盘盈 |
| inventory_loss | 盘亏 |
| move_in | 移入 |
| move_out | 移出 |

---

### 3.4 入库管理模块

#### 3.4.1 入库单主表 (inbound_order)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| inbound_order_id | VARCHAR(36) | 是 | UUID | 主键 |
| order_no | VARCHAR(50) | 是 | - | 单据编号，唯一 |
| type | VARCHAR(20) | 是 | - | 入库类型 |
| supplier_code | VARCHAR(50) | 否 | - | 供应商编码 |
| warehouse_code | VARCHAR(50) | 是 | - | 仓库编码 |
| status | INT | 是 | 0 | 状态 |
| total_qty | INT | 是 | 0 | 总数量 |
| total_amount | DECIMAL(12,2) | 否 | - | 总金额 |
| expect_date | DATE | 否 | - | 预期日期 |
| actual_date | DATE | 否 | - | 实际日期 |
| remark | TEXT | 否 | - | 备注 |
| created_by | VARCHAR(36) | 否 | - | 创建人ID |
| reviewed_by | VARCHAR(36) | 否 | - | 审核人ID |
| reviewed_at | DATETIME | 否 | - | 审核时间 |
| completed_by | VARCHAR(36) | 否 | - | 完成人ID |
| completed_at | DATETIME | 否 | - | 完成时间 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**入库类型枚举**:
| 值 | 说明 |
|----|------|
| purchase | 采购入库 |
| return | 退货入库 |
| transfer | 调拨入库 |
| inventory_gain | 盘盈入库 |
| other | 其他入库 |

**入库状态枚举**:
| 值 | 说明 |
|----|------|
| 0 | 草稿 |
| 1 | 待审核 |
| 2 | 已审核 |
| 3 | 已入库 |
| 4 | 已取消 |

#### 3.4.2 入库单明细表 (inbound_item)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| inbound_item_id | VARCHAR(36) | 是 | UUID | 主键 |
| inbound_order_no | VARCHAR(50) | 是 | - | 入库单编号 |
| product_code | VARCHAR(50) | 是 | - | 商品编码 |
| location_code | VARCHAR(50) | 否 | - | 库位编码 |
| expect_qty | INT | 是 | - | 预期数量 |
| actual_qty | INT | 是 | 0 | 实际数量 |
| batch_no | VARCHAR(50) | 否 | - | 批次号 |
| production_date | DATE | 否 | - | 生产日期 |
| expire_date | DATE | 否 | - | 过期日期 |
| price | DECIMAL(10,2) | 否 | - | 单价 |
| amount | DECIMAL(12,2) | 否 | - | 金额 |
| remark | VARCHAR(255) | 否 | - | 备注 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

---

### 3.5 出库管理模块

#### 3.5.1 出库单主表 (outbound_order)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| outbound_order_id | VARCHAR(36) | 是 | UUID | 主键 |
| order_no | VARCHAR(50) | 是 | - | 单据编号，唯一 |
| type | VARCHAR(20) | 是 | - | 出库类型 |
| customer_code | VARCHAR(50) | 否 | - | 客户编码 |
| warehouse_code | VARCHAR(50) | 是 | - | 仓库编码 |
| status | INT | 是 | 0 | 状态 |
| total_qty | INT | 是 | 0 | 总数量 |
| total_amount | DECIMAL(12,2) | 否 | - | 总金额 |
| expect_date | DATE | 否 | - | 预期日期 |
| actual_date | DATE | 否 | - | 实际日期 |
| remark | TEXT | 否 | - | 备注 |
| created_by | VARCHAR(36) | 否 | - | 创建人ID |
| reviewed_by | VARCHAR(36) | 否 | - | 审核人ID |
| reviewed_at | DATETIME | 否 | - | 审核时间 |
| completed_by | VARCHAR(36) | 否 | - | 完成人ID |
| completed_at | DATETIME | 否 | - | 完成时间 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**出库类型枚举**:
| 值 | 说明 |
|----|------|
| sale | 销售出库 |
| picking | 领料出库 |
| return | 退料出库 |
| transfer | 调拨出库 |
| inventory_loss | 盘亏出库 |
| other | 其他出库 |

**出库状态枚举**:
| 值 | 说明 |
|----|------|
| 0 | 草稿 |
| 1 | 待审核 |
| 2 | 已审核 |
| 3 | 已出库 |
| 4 | 已取消 |

---

### 3.6 库内作业模块

#### 3.6.1 盘点任务表 (inventory_task)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| inventory_task_id | VARCHAR(36) | 是 | UUID | 主键 |
| task_no | VARCHAR(50) | 是 | - | 任务编号，唯一 |
| warehouse_code | VARCHAR(50) | 是 | - | 仓库编码 |
| type | VARCHAR(20) | 是 | - | 盘点类型 |
| status | INT | 是 | 0 | 状态 |
| total_qty | INT | 是 | 0 | 总数量 |
| diff_qty | INT | 是 | 0 | 差异数量 |
| start_time | DATETIME | 否 | - | 开始时间 |
| end_time | DATETIME | 否 | - | 结束时间 |
| remark | TEXT | 否 | - | 备注 |
| created_by | VARCHAR(36) | 否 | - | 创建人ID |
| completed_by | VARCHAR(36) | 否 | - | 完成人ID |
| completed_at | DATETIME | 否 | - | 完成时间 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**盘点类型枚举**:
| 值 | 说明 |
|----|------|
| full | 全盘 |
| partial | 部分盘点 |
| cycle | 循环盘点 |

---

### 3.7 打印管理模块

#### 3.7.1 打印模板表 (print_template)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| print_template_id | VARCHAR(36) | 是 | UUID | 主键 |
| template_code | VARCHAR(50) | 是 | - | 模板编码，唯一 |
| template_name | VARCHAR(100) | 是 | - | 模板名称 |
| type | VARCHAR(20) | 是 | - | 模板类型 |
| category | VARCHAR(50) | 否 | - | 业务分类 |
| description | VARCHAR(255) | 否 | - | 描述 |
| width | INT | 是 | - | 模板宽度(mm) |
| height | INT | 是 | - | 模板高度(mm) |
| unit | VARCHAR(10) | 是 | mm | 单位 |
| orientation | VARCHAR(10) | 是 | portrait | 方向 |
| margin | VARCHAR(50) | 否 | - | 边距 |
| background | VARCHAR(255) | 否 | - | 背景图片URL |
| is_default | INT | 是 | 0 | 是否默认模板 |
| status | INT | 是 | 1 | 状态 |
| created_by | VARCHAR(36) | 否 | - | 创建人ID |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**模板类型枚举**:
| 值 | 说明 |
|----|------|
| label | 标签模板 |
| document | 单据模板 |
| barcode | 条码模板 |

**业务分类枚举**:
| 值 | 说明 |
|----|------|
| product | 商品标签 |
| inbound | 入库单据 |
| outbound | 出库单据 |
| stock | 库存标签 |
| location | 库位标签 |

---

#### 3.7.2 打印元素表 (print_element)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| print_element_id | VARCHAR(36) | 是 | UUID | 主键 |
| template_code | VARCHAR(50) | 是 | - | 模板编码 |
| type | VARCHAR(20) | 是 | - | 元素类型 |
| name | VARCHAR(50) | 是 | - | 元素名称 |
| x | FLOAT | 是 | - | X坐标 |
| y | FLOAT | 是 | - | Y坐标 |
| width | FLOAT | 是 | - | 宽度 |
| height | FLOAT | 是 | - | 高度 |
| rotation | FLOAT | 是 | 0 | 旋转角度 |
| z_index | INT | 是 | 0 | 层级 |
| data_source | VARCHAR(100) | 否 | - | 数据源字段 |
| default_value | TEXT | 否 | - | 默认值 |
| format | VARCHAR(50) | 否 | - | 格式化规则 |
| styles | TEXT | 否 | - | JSON样式配置 |
| bindings | TEXT | 否 | - | JSON数据绑定 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**元素类型枚举**:
| 值 | 说明 |
|----|------|
| text | 文本元素 |
| barcode | 一维码 |
| qrcode | 二维码 |
| image | 图片 |
| line | 线条 |
| rectangle | 矩形 |
| table | 表格 |

**样式配置示例**:
```json
{
  "fontFamily": "SimHei",
  "fontSize": 12,
  "fontWeight": "normal",
  "color": "#000000",
  "textAlign": "left",
  "verticalAlign": "middle",
  "borderWidth": 1,
  "borderColor": "#000000",
  "borderStyle": "solid",
  "backgroundColor": "transparent",
  "padding": [2, 4, 2, 4],
  "lineHeight": 1.5
}
```

**条码样式配置示例**:
```json
{
  "barcodeType": "CODE128",
  "displayValue": true,
  "textAlign": "center",
  "fontSize": 10,
  "margin": 2,
  "width": 2,
  "height": 50
}
```

**二维码样式配置示例**:
```json
{
  "qrCodeType": "QR",
  "size": 80,
  "errorCorrectionLevel": "M",
  "margin": 2,
  "colorDark": "#000000",
  "colorLight": "#ffffff"
}
```

---

#### 3.7.3 打印记录表 (print_record)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| print_record_id | VARCHAR(36) | 是 | UUID | 主键 |
| template_code | VARCHAR(50) | 是 | - | 模板编码 |
| business_type | VARCHAR(50) | 是 | - | 业务类型 |
| business_no | VARCHAR(50) | 否 | - | 业务单据编号 |
| print_data | TEXT | 是 | - | JSON打印数据 |
| copies | INT | 是 | 1 | 打印份数 |
| printer_name | VARCHAR(100) | 否 | - | 打印机名称 |
| operator_id | VARCHAR(36) | 否 | - | 操作人ID |
| operator_name | VARCHAR(50) | 否 | - | 操作人姓名 |
| printed_at | DATETIME | 是 | now() | 打印时间 |
| created_at | DATETIME | 是 | now() | 创建时间 |

---

#### 3.7.4 条码规则表 (barcode_rule)

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| barcode_rule_id | VARCHAR(36) | 是 | UUID | 主键 |
| rule_code | VARCHAR(50) | 是 | - | 规则编码，唯一 |
| rule_name | VARCHAR(100) | 是 | - | 规则名称 |
| type | VARCHAR(20) | 是 | - | 条码类型 |
| prefix | VARCHAR(20) | 否 | - | 前缀 |
| sequence | INT | 是 | 1 | 当前序号 |
| padding | INT | 是 | 0 | 序号补位长度 |
| suffix | VARCHAR(20) | 否 | - | 后缀 |
| description | VARCHAR(255) | 否 | - | 描述 |
| status | INT | 是 | 1 | 状态 |
| created_at | DATETIME | 是 | now() | 创建时间 |
| updated_at | DATETIME | 是 | now() | 更新时间 |

**条码类型枚举**:
| 值 | 说明 | 用途 |
|----|------|------|
| CODE128 | CODE128 | 通用条码 |
| EAN13 | EAN-13 | 商品条码 |
| EAN8 | EAN-8 | 小商品条码 |
| CODE39 | CODE-39 | 工业条码 |
| QR | QR Code | 二维码 |
| DATAMATRIX | DataMatrix | 二维码 |

---

## 四、数据库索引策略

### 4.1 索引设计原则

1. **主键索引**: 所有表使用业务相关的字段名作为主键（如 warehouse_id, zone_id）
2. **唯一索引**: 编码类字段使用唯一索引
3. **外键索引**: 所有外键字段（业务编码）建立索引
4. **查询索引**: 高频查询条件字段建立索引
5. **组合索引**: 多条件组合查询建立组合索引

### 4.2 关键索引列表

```sql
-- 商品表索引
CREATE UNIQUE INDEX idx_product_code ON product(code);
CREATE UNIQUE INDEX idx_product_barcode ON product(barcode);
CREATE INDEX idx_product_category ON product(category_code);
CREATE INDEX idx_product_name ON product(name);

-- 库存表索引
CREATE UNIQUE INDEX idx_stock_unique ON stock(product_code, location_code, batch_no);
CREATE INDEX idx_stock_product ON stock(product_code);
CREATE INDEX idx_stock_location ON stock(location_code);
CREATE INDEX idx_stock_expire ON stock(expire_date);

-- 库存流水索引
CREATE INDEX idx_stock_record_stock ON stock_record(stock_id);
CREATE INDEX idx_stock_record_type ON stock_record(type);
CREATE INDEX idx_stock_record_created ON stock_record(created_at);

-- 入库单索引
CREATE UNIQUE INDEX idx_inbound_order_no ON inbound_order(order_no);
CREATE INDEX idx_inbound_order_status ON inbound_order(status);
CREATE INDEX idx_inbound_order_date ON inbound_order(created_at);
CREATE INDEX idx_inbound_order_supplier ON inbound_order(supplier_code);
CREATE INDEX idx_inbound_order_warehouse ON inbound_order(warehouse_code);

-- 出库单索引
CREATE UNIQUE INDEX idx_outbound_order_no ON outbound_order(order_no);
CREATE INDEX idx_outbound_order_status ON outbound_order(status);
CREATE INDEX idx_outbound_order_date ON outbound_order(created_at);
CREATE INDEX idx_outbound_order_customer ON outbound_order(customer_code);
CREATE INDEX idx_outbound_order_warehouse ON outbound_order(warehouse_code);

-- 操作日志索引
CREATE INDEX idx_operation_log_user ON operation_log(user_id);
CREATE INDEX idx_operation_log_date ON operation_log(created_at);

-- 库区表索引
CREATE INDEX idx_zone_warehouse ON zone(warehouse_code);

-- 库位表索引
CREATE INDEX idx_location_zone ON location(zone_code);
```

---

## 五、初始数据设计

### 5.1 默认用户数据

| 用户名 | 密码 | 角色 | 说明 |
|--------|------|------|------|
| admin | admin123 | admin | 系统管理员 |
| operator | operator123 | operator | 操作员 |

### 5.2 默认角色权限

| 角色编码 | 角色名称 | 权限范围 |
|----------|----------|----------|
| admin | 系统管理员 | 全部权限 |
| operator | 操作员 | 入库、出库、库存查询 |
| viewer | 查看者 | 只读权限 |

### 5.3 系统配置数据

| 配置键 | 配置值 | 说明 |
|--------|--------|------|
| system.name | WMS-Lite | 系统名称 |
| system.logo | /logo.png | 系统Logo |
| order.prefix.inbound | RK | 入库单前缀 |
| order.prefix.outbound | CK | 出库单前缀 |
| order.prefix.inventory | PD | 盘点单前缀 |
| stock.alert.low | true | 启用低库存预警 |
| stock.alert.expire | 30 | 效期预警天数 |

---

## 六、数据迁移策略

### 6.1 字段命名变更对照表

#### 6.1.1 主键字段变更

| 表名 | 旧字段名 | 新字段名 |
|------|----------|----------|
| user | id | user_id |
| role | id | role_code |
| permission | id | permission_id |
| operation_log | id | operation_log_id |
| system_config | id | config_id |
| category | id | category_id |
| product | id | product_id |
| warehouse | id | warehouse_id |
| zone | id | zone_id |
| location | id | location_id |
| supplier | id | supplier_id |
| customer | id | customer_id |
| stock | id | stock_id |
| stock_record | id | stock_record_id |
| stock_freeze | id | stock_freeze_id |
| stock_alert | id | stock_alert_id |
| inbound_order | id | inbound_order_id |
| inbound_item | id | inbound_item_id |
| outbound_order | id | outbound_order_id |
| outbound_item | id | outbound_item_id |
| inventory_task | id | inventory_task_id |
| inventory_item | id | inventory_item_id |
| move_order | id | move_order_id |
| print_template | id | print_template_id |
| print_element | id | print_element_id |
| print_record | id | print_record_id |
| barcode_rule | id | barcode_rule_id |

#### 6.1.2 业务编码字段变更（驼峰命名）

| 表名 | 旧字段名 | 新字段名 |
|------|----------|----------|
| warehouse | code | warehouse_code |
| warehouse | name | warehouse_name |
| zone | code | zone_code |
| zone | name | zone_name |
| location | code | location_code |
| location | name | location_name |
| supplier | code | supplier_code |
| supplier | name | supplier_name |
| customer | code | customer_code |
| customer | name | customer_name |
| print_template | code | template_code |
| print_template | name | template_name |
| barcode_rule | code | rule_code |
| barcode_rule | name | rule_name |

#### 6.1.3 外键关联变更（使用业务编号）

| 表名 | 旧外键字段 | 新外键字段 | 关联表 |
|------|-------------|-------------|--------|
| user | roleId | role_code | role |
| permission | parentId | permission_id | permission |
| category | parentId | category_id | category |
| zone | warehouseId | warehouse_code | warehouse |
| location | zoneId | zone_code | zone |
| inbound_order | supplierId | supplier_code | supplier |
| inbound_order | warehouseId | warehouse_code | warehouse |
| inbound_item | orderId | inbound_order_no | inbound_order |
| inbound_item | productId | product_code | product |
| inbound_item | locationId | location_code | location |
| outbound_order | customerId | customer_code | customer |
| outbound_order | warehouseId | warehouse_code | warehouse |
| outbound_item | orderId | outbound_order_no | outbound_order |
| outbound_item | productId | product_code | product |
| outbound_item | locationId | location_code | location |
| inventory_task | warehouseId | warehouse_code | warehouse |
| inventory_item | taskId | task_no | inventory_task |
| inventory_item | productId | product_code | product |
| inventory_item | locationId | location_code | location |
| move_order | warehouseId | warehouse_code | warehouse |
| move_order | productId | product_code | product |
| move_order | fromLocationId | from_location_code | location |
| move_order | toLocationId | to_location_code | location |
| stock | productId | product_code | product |
| stock | locationId | location_code | location |
| stock_record | stockId | stock_id | stock |
| stock_freeze | productId | product_code | product |
| stock_freeze | locationId | location_code | location |
| stock_alert | productId | product_code | product |
| print_element | templateId | template_code | print_template |
| print_record | templateId | template_code | print_template |

### 6.2 数据库迁移脚本

```sql
-- ============================================
-- WMS-Lite 数据库迁移脚本
-- 版本: v1.2.0
-- 说明: 字段命名规范化迁移（驼峰命名 + 业务编号关联）
-- ============================================

-- 设置外键检查关闭
SET FOREIGN_KEY_CHECKS = 0;

-- ============================================
-- 1. 系统管理模块
-- ============================================

-- 1.1 用户表 (user)
ALTER TABLE user CHANGE COLUMN id user_id VARCHAR(36);
ALTER TABLE user CHANGE COLUMN role_id role_code VARCHAR(50);
ALTER TABLE user CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE user CHANGE COLUMN updated_at updated_at DATETIME;

-- 1.2 角色表 (role)
ALTER TABLE role CHANGE COLUMN id role_code VARCHAR(50);
ALTER TABLE role CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE role CHANGE COLUMN updated_at updated_at DATETIME;

-- 1.3 权限表 (permission)
ALTER TABLE permission CHANGE COLUMN id permission_id VARCHAR(36);
ALTER TABLE permission CHANGE COLUMN parent_id permission_id VARCHAR(100);
ALTER TABLE permission CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE permission CHANGE COLUMN updated_at updated_at DATETIME;

-- 1.4 操作日志表 (operation_log)
ALTER TABLE operation_log CHANGE COLUMN id operation_log_id VARCHAR(36);
ALTER TABLE operation_log CHANGE COLUMN user_id user_id VARCHAR(36);
ALTER TABLE operation_log CHANGE COLUMN created_at created_at DATETIME;

-- 1.5 系统配置表 (system_config)
ALTER TABLE system_config CHANGE COLUMN id config_id VARCHAR(36);
ALTER TABLE system_config CHANGE COLUMN config_key config_key VARCHAR(100);
ALTER TABLE system_config CHANGE COLUMN config_value config_value TEXT;
ALTER TABLE system_config CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE system_config CHANGE COLUMN updated_at updated_at DATETIME;

-- ============================================
-- 2. 基础数据模块
-- ============================================

-- 2.1 商品分类表 (category)
ALTER TABLE category CHANGE COLUMN id category_id VARCHAR(36);
ALTER TABLE category CHANGE COLUMN parent_id category_id VARCHAR(50);
ALTER TABLE category CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE category CHANGE COLUMN updated_at updated_at DATETIME;

-- 2.2 商品表 (product)
ALTER TABLE product CHANGE COLUMN id product_id VARCHAR(36);
ALTER TABLE product CHANGE COLUMN category_id category_code VARCHAR(50);
ALTER TABLE product CHANGE COLUMN cost_price cost_price DECIMAL(10,2);
ALTER TABLE product CHANGE COLUMN min_stock min_stock INT;
ALTER TABLE product CHANGE COLUMN max_stock max_stock INT;
ALTER TABLE product CHANGE COLUMN shelf_life shelf_life INT;
ALTER TABLE product CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE product CHANGE COLUMN updated_at updated_at DATETIME;

-- 2.3 仓库表 (warehouse)
ALTER TABLE warehouse CHANGE COLUMN id warehouse_id VARCHAR(36);
ALTER TABLE warehouse CHANGE COLUMN code warehouse_code VARCHAR(50);
ALTER TABLE warehouse CHANGE COLUMN name warehouse_name VARCHAR(100);
ALTER TABLE warehouse CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE warehouse CHANGE COLUMN updated_at updated_at DATETIME;

-- 2.4 库区表 (zone)
ALTER TABLE zone CHANGE COLUMN id zone_id VARCHAR(36);
ALTER TABLE zone CHANGE COLUMN code zone_code VARCHAR(50);
ALTER TABLE zone CHANGE COLUMN name zone_name VARCHAR(100);
ALTER TABLE zone CHANGE COLUMN warehouse_id warehouse_code VARCHAR(50);
ALTER TABLE zone CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE zone CHANGE COLUMN updated_at updated_at DATETIME;

-- 2.5 库位表 (location)
ALTER TABLE location CHANGE COLUMN id location_id VARCHAR(36);
ALTER TABLE location CHANGE COLUMN code location_code VARCHAR(50);
ALTER TABLE location CHANGE COLUMN name location_name VARCHAR(100);
ALTER TABLE location CHANGE COLUMN zone_id zone_code VARCHAR(50);
ALTER TABLE location CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE location CHANGE COLUMN updated_at updated_at DATETIME;

-- 2.6 供应商表 (supplier)
ALTER TABLE supplier CHANGE COLUMN id supplier_id VARCHAR(36);
ALTER TABLE supplier CHANGE COLUMN code supplier_code VARCHAR(50);
ALTER TABLE supplier CHANGE COLUMN name supplier_name VARCHAR(100);
ALTER TABLE supplier CHANGE COLUMN bank_name bank_name VARCHAR(100);
ALTER TABLE supplier CHANGE COLUMN bank_account bank_account VARCHAR(50);
ALTER TABLE supplier CHANGE COLUMN tax_number tax_number VARCHAR(50);
ALTER TABLE supplier CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE supplier CHANGE COLUMN updated_at updated_at DATETIME;

-- 2.7 客户表 (customer)
ALTER TABLE customer CHANGE COLUMN id customer_id VARCHAR(36);
ALTER TABLE customer CHANGE COLUMN code customer_code VARCHAR(50);
ALTER TABLE customer CHANGE COLUMN name customer_name VARCHAR(100);
ALTER TABLE customer CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE customer CHANGE COLUMN updated_at updated_at DATETIME;

-- ============================================
-- 3. 库存管理模块
-- ============================================

-- 3.1 库存表 (stock)
ALTER TABLE stock CHANGE COLUMN id stock_id VARCHAR(36);
ALTER TABLE stock CHANGE COLUMN product_id product_code VARCHAR(50);
ALTER TABLE stock CHANGE COLUMN location_id location_code VARCHAR(50);
ALTER TABLE stock CHANGE COLUMN batch_no batch_no VARCHAR(50);
ALTER TABLE stock CHANGE COLUMN production_date production_date DATE;
ALTER TABLE stock CHANGE COLUMN expire_date expire_date DATE;
ALTER TABLE stock CHANGE COLUMN frozen_qty frozen_qty INT;
ALTER TABLE stock CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE stock CHANGE COLUMN updated_at updated_at DATETIME;

-- 3.2 库存流水表 (stock_record)
ALTER TABLE stock_record CHANGE COLUMN id stock_record_id VARCHAR(36);
ALTER TABLE stock_record CHANGE COLUMN stock_id stock_id VARCHAR(36);
ALTER TABLE stock_record CHANGE COLUMN change_qty change_qty INT;
ALTER TABLE stock_record CHANGE COLUMN before_qty before_qty INT;
ALTER TABLE stock_record CHANGE COLUMN after_qty after_qty INT;
ALTER TABLE stock_record CHANGE COLUMN order_type order_type VARCHAR(20);
ALTER TABLE stock_record CHANGE COLUMN order_id order_no VARCHAR(50);
ALTER TABLE stock_record CHANGE COLUMN operator_id operator_id VARCHAR(36);
ALTER TABLE stock_record CHANGE COLUMN operator_name operator_name VARCHAR(50);
ALTER TABLE stock_record CHANGE COLUMN created_at created_at DATETIME;

-- 3.3 库存冻结表 (stock_freeze)
ALTER TABLE stock_freeze CHANGE COLUMN id stock_freeze_id VARCHAR(36);
ALTER TABLE stock_freeze CHANGE COLUMN product_id product_code VARCHAR(50);
ALTER TABLE stock_freeze CHANGE COLUMN location_id location_code VARCHAR(50);
ALTER TABLE stock_freeze CHANGE COLUMN batch_no batch_no VARCHAR(50);
ALTER TABLE stock_freeze CHANGE COLUMN order_no order_no VARCHAR(50);
ALTER TABLE stock_freeze CHANGE COLUMN order_type order_type VARCHAR(20);
ALTER TABLE stock_freeze CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE stock_freeze CHANGE COLUMN updated_at updated_at DATETIME;

-- 3.4 库存预警表 (stock_alert)
ALTER TABLE stock_alert CHANGE COLUMN id stock_alert_id VARCHAR(36);
ALTER TABLE stock_alert CHANGE COLUMN product_id product_code VARCHAR(50);
ALTER TABLE stock_alert CHANGE COLUMN current_value current_value INT;
ALTER TABLE stock_alert CHANGE COLUMN handled_at handled_at DATETIME;
ALTER TABLE stock_alert CHANGE COLUMN handled_by handled_by VARCHAR(36);
ALTER TABLE stock_alert CHANGE COLUMN created_at created_at DATETIME;

-- ============================================
-- 4. 入库管理模块
-- ============================================

-- 4.1 入库单主表 (inbound_order)
ALTER TABLE inbound_order CHANGE COLUMN id inbound_order_id VARCHAR(36);
ALTER TABLE inbound_order CHANGE COLUMN order_no order_no VARCHAR(50);
ALTER TABLE inbound_order CHANGE COLUMN supplier_id supplier_code VARCHAR(50);
ALTER TABLE inbound_order CHANGE COLUMN warehouse_id warehouse_code VARCHAR(50);
ALTER TABLE inbound_order CHANGE COLUMN total_qty total_qty INT;
ALTER TABLE inbound_order CHANGE COLUMN total_amount total_amount DECIMAL(12,2);
ALTER TABLE inbound_order CHANGE COLUMN expect_date expect_date DATE;
ALTER TABLE inbound_order CHANGE COLUMN actual_date actual_date DATE;
ALTER TABLE inbound_order CHANGE COLUMN created_by created_by VARCHAR(36);
ALTER TABLE inbound_order CHANGE COLUMN reviewed_by reviewed_by VARCHAR(36);
ALTER TABLE inbound_order CHANGE COLUMN reviewed_at reviewed_at DATETIME;
ALTER TABLE inbound_order CHANGE COLUMN completed_by completed_by VARCHAR(36);
ALTER TABLE inbound_order CHANGE COLUMN completed_at completed_at DATETIME;
ALTER TABLE inbound_order CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE inbound_order CHANGE COLUMN updated_at updated_at DATETIME;

-- 4.2 入库单明细表 (inbound_item)
ALTER TABLE inbound_item CHANGE COLUMN id inbound_item_id VARCHAR(36);
ALTER TABLE inbound_item CHANGE COLUMN order_id inbound_order_no VARCHAR(50);
ALTER TABLE inbound_item CHANGE COLUMN product_id product_code VARCHAR(50);
ALTER TABLE inbound_item CHANGE COLUMN location_id location_code VARCHAR(50);
ALTER TABLE inbound_item CHANGE COLUMN expect_qty expect_qty INT;
ALTER TABLE inbound_item CHANGE COLUMN actual_qty actual_qty INT;
ALTER TABLE inbound_item CHANGE COLUMN batch_no batch_no VARCHAR(50);
ALTER TABLE inbound_item CHANGE COLUMN production_date production_date DATE;
ALTER TABLE inbound_item CHANGE COLUMN expire_date expire_date DATE;
ALTER TABLE inbound_item CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE inbound_item CHANGE COLUMN updated_at updated_at DATETIME;

-- ============================================
-- 5. 出库管理模块
-- ============================================

-- 5.1 出库单主表 (outbound_order)
ALTER TABLE outbound_order CHANGE COLUMN id outbound_order_id VARCHAR(36);
ALTER TABLE outbound_order CHANGE COLUMN order_no order_no VARCHAR(50);
ALTER TABLE outbound_order CHANGE COLUMN customer_id customer_code VARCHAR(50);
ALTER TABLE outbound_order CHANGE COLUMN warehouse_id warehouse_code VARCHAR(50);
ALTER TABLE outbound_order CHANGE COLUMN total_qty total_qty INT;
ALTER TABLE outbound_order CHANGE COLUMN total_amount total_amount DECIMAL(12,2);
ALTER TABLE outbound_order CHANGE COLUMN expect_date expect_date DATE;
ALTER TABLE outbound_order CHANGE COLUMN actual_date actual_date DATE;
ALTER TABLE outbound_order CHANGE COLUMN created_by created_by VARCHAR(36);
ALTER TABLE outbound_order CHANGE COLUMN reviewed_by reviewed_by VARCHAR(36);
ALTER TABLE outbound_order CHANGE COLUMN reviewed_at reviewed_at DATETIME;
ALTER TABLE outbound_order CHANGE COLUMN completed_by completed_by VARCHAR(36);
ALTER TABLE outbound_order CHANGE COLUMN completed_at completed_at DATETIME;
ALTER TABLE outbound_order CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE outbound_order CHANGE COLUMN updated_at updated_at DATETIME;

-- 5.2 出库单明细表 (outbound_item)
ALTER TABLE outbound_item CHANGE COLUMN id outbound_item_id VARCHAR(36);
ALTER TABLE outbound_item CHANGE COLUMN order_id outbound_order_no VARCHAR(50);
ALTER TABLE outbound_item CHANGE COLUMN product_id product_code VARCHAR(50);
ALTER TABLE outbound_item CHANGE COLUMN location_id location_code VARCHAR(50);
ALTER TABLE outbound_item CHANGE COLUMN expect_qty expect_qty INT;
ALTER TABLE outbound_item CHANGE COLUMN actual_qty actual_qty INT;
ALTER TABLE outbound_item CHANGE COLUMN batch_no batch_no VARCHAR(50);
ALTER TABLE outbound_item CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE outbound_item CHANGE COLUMN updated_at updated_at DATETIME;

-- ============================================
-- 6. 库内作业模块
-- ============================================

-- 6.1 盘点任务表 (inventory_task)
ALTER TABLE inventory_task CHANGE COLUMN id inventory_task_id VARCHAR(36);
ALTER TABLE inventory_task CHANGE COLUMN task_no task_no VARCHAR(50);
ALTER TABLE inventory_task CHANGE COLUMN warehouse_id warehouse_code VARCHAR(50);
ALTER TABLE inventory_task CHANGE COLUMN total_qty total_qty INT;
ALTER TABLE inventory_task CHANGE COLUMN diff_qty diff_qty INT;
ALTER TABLE inventory_task CHANGE COLUMN start_time start_time DATETIME;
ALTER TABLE inventory_task CHANGE COLUMN end_time end_time DATETIME;
ALTER TABLE inventory_task CHANGE COLUMN created_by created_by VARCHAR(36);
ALTER TABLE inventory_task CHANGE COLUMN completed_by completed_by VARCHAR(36);
ALTER TABLE inventory_task CHANGE COLUMN completed_at completed_at DATETIME;
ALTER TABLE inventory_task CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE inventory_task CHANGE COLUMN updated_at updated_at DATETIME;

-- 6.2 盘点明细表 (inventory_item)
ALTER TABLE inventory_item CHANGE COLUMN id inventory_item_id VARCHAR(36);
ALTER TABLE inventory_item CHANGE COLUMN task_id task_no VARCHAR(50);
ALTER TABLE inventory_item CHANGE COLUMN product_id product_code VARCHAR(50);
ALTER TABLE inventory_item CHANGE COLUMN location_id location_code VARCHAR(50);
ALTER TABLE inventory_item CHANGE COLUMN batch_no batch_no VARCHAR(50);
ALTER TABLE inventory_item CHANGE COLUMN system_qty system_qty INT;
ALTER TABLE inventory_item CHANGE COLUMN actual_qty actual_qty INT;
ALTER TABLE inventory_item CHANGE COLUMN diff_qty diff_qty INT;
ALTER TABLE inventory_item CHANGE COLUMN counted_by counted_by VARCHAR(36);
ALTER TABLE inventory_item CHANGE COLUMN counted_at counted_at DATETIME;
ALTER TABLE inventory_item CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE inventory_item CHANGE COLUMN updated_at updated_at DATETIME;

-- 6.3 移库单表 (move_order)
ALTER TABLE move_order CHANGE COLUMN id move_order_id VARCHAR(36);
ALTER TABLE move_order CHANGE COLUMN order_no order_no VARCHAR(50);
ALTER TABLE move_order CHANGE COLUMN warehouse_id warehouse_code VARCHAR(50);
ALTER TABLE move_order CHANGE COLUMN product_id product_code VARCHAR(50);
ALTER TABLE move_order CHANGE COLUMN from_location_id from_location_code VARCHAR(50);
ALTER TABLE move_order CHANGE COLUMN to_location_id to_location_code VARCHAR(50);
ALTER TABLE move_order CHANGE COLUMN batch_no batch_no VARCHAR(50);
ALTER TABLE move_order CHANGE COLUMN created_by created_by VARCHAR(36);
ALTER TABLE move_order CHANGE COLUMN completed_by completed_by VARCHAR(36);
ALTER TABLE move_order CHANGE COLUMN completed_at completed_at DATETIME;
ALTER TABLE move_order CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE move_order CHANGE COLUMN updated_at updated_at DATETIME;

-- ============================================
-- 7. 打印管理模块
-- ============================================

-- 7.1 打印模板表 (print_template)
ALTER TABLE print_template CHANGE COLUMN id print_template_id VARCHAR(36);
ALTER TABLE print_template CHANGE COLUMN code template_code VARCHAR(50);
ALTER TABLE print_template CHANGE COLUMN name template_name VARCHAR(100);
ALTER TABLE print_template CHANGE COLUMN is_default is_default INT;
ALTER TABLE print_template CHANGE COLUMN created_by created_by VARCHAR(36);
ALTER TABLE print_template CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE print_template CHANGE COLUMN updated_at updated_at DATETIME;

-- 7.2 打印元素表 (print_element)
ALTER TABLE print_element CHANGE COLUMN id print_element_id VARCHAR(36);
ALTER TABLE print_element CHANGE COLUMN template_id template_code VARCHAR(50);
ALTER TABLE print_element CHANGE COLUMN z_index z_index INT;
ALTER TABLE print_element CHANGE COLUMN data_source data_source VARCHAR(100);
ALTER TABLE print_element CHANGE COLUMN default_value default_value TEXT;
ALTER TABLE print_element CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE print_element CHANGE COLUMN updated_at updated_at DATETIME;

-- 7.3 打印记录表 (print_record)
ALTER TABLE print_record CHANGE COLUMN id print_record_id VARCHAR(36);
ALTER TABLE print_record CHANGE COLUMN template_id template_code VARCHAR(50);
ALTER TABLE print_record CHANGE COLUMN business_type business_type VARCHAR(50);
ALTER TABLE print_record CHANGE COLUMN business_no business_no VARCHAR(50);
ALTER TABLE print_record CHANGE COLUMN print_data print_data TEXT;
ALTER TABLE print_record CHANGE COLUMN printer_name printer_name VARCHAR(100);
ALTER TABLE print_record CHANGE COLUMN operator_id operator_id VARCHAR(36);
ALTER TABLE print_record CHANGE COLUMN operator_name operator_name VARCHAR(50);
ALTER TABLE print_record CHANGE COLUMN printed_at printed_at DATETIME;
ALTER TABLE print_record CHANGE COLUMN created_at created_at DATETIME;

-- 7.4 条码规则表 (barcode_rule)
ALTER TABLE barcode_rule CHANGE COLUMN id barcode_rule_id VARCHAR(36);
ALTER TABLE barcode_rule CHANGE COLUMN code rule_code VARCHAR(50);
ALTER TABLE barcode_rule CHANGE COLUMN name rule_name VARCHAR(100);
ALTER TABLE barcode_rule CHANGE COLUMN created_at created_at DATETIME;
ALTER TABLE barcode_rule CHANGE COLUMN updated_at updated_at DATETIME;

-- ============================================
-- 8. 重建索引
-- ============================================

-- 删除旧索引
DROP INDEX idx_role_id ON user;
DROP INDEX idx_parent_id ON permission;
DROP INDEX idx_category ON product;
DROP INDEX idx_warehouse_id ON zone;
DROP INDEX idx_zone_id ON location;
DROP INDEX idx_product_id ON stock;
DROP INDEX idx_location_id ON stock;
DROP INDEX idx_supplier_id ON inbound_order;
DROP INDEX idx_warehouse_id ON inbound_order;
DROP INDEX idx_order_id ON inbound_item;
DROP INDEX idx_product_id ON inbound_item;
DROP INDEX idx_location_id ON inbound_item;
DROP INDEX idx_customer_id ON outbound_order;
DROP INDEX idx_warehouse_id ON outbound_order;
DROP INDEX idx_order_id ON outbound_item;
DROP INDEX idx_product_id ON outbound_item;
DROP INDEX idx_location_id ON outbound_item;
DROP INDEX idx_warehouse_id ON inventory_task;
DROP INDEX idx_task_id ON inventory_item;
DROP INDEX idx_product_id ON inventory_item;
DROP INDEX idx_location_id ON inventory_item;
DROP INDEX idx_warehouse_id ON move_order;
DROP INDEX idx_product_id ON move_order;
DROP INDEX idx_template_id ON print_element;
DROP INDEX idx_template_id ON print_record;

-- 创建新索引
CREATE INDEX idx_role_code ON user(role_code);
CREATE INDEX idx_permission_id ON permission(permission_id);
CREATE INDEX idx_category_id ON category(category_id);
CREATE INDEX idx_warehouse_code ON zone(warehouse_code);
CREATE INDEX idx_zone_code ON location(zone_code);
CREATE INDEX idx_product_code ON stock(product_code);
CREATE INDEX idx_location_code ON stock(location_code);
CREATE INDEX idx_supplier_code ON inbound_order(supplier_code);
CREATE INDEX idx_warehouse_code ON inbound_order(warehouse_code);
CREATE INDEX idx_inbound_order_no ON inbound_item(inbound_order_no);
CREATE INDEX idx_product_code ON inbound_item(product_code);
CREATE INDEX idx_location_code ON inbound_item(location_code);
CREATE INDEX idx_customer_code ON outbound_order(customer_code);
CREATE INDEX idx_warehouse_code ON outbound_order(warehouse_code);
CREATE INDEX idx_outbound_order_no ON outbound_item(outbound_order_no);
CREATE INDEX idx_product_code ON outbound_item(product_code);
CREATE INDEX idx_location_code ON outbound_item(location_code);
CREATE INDEX idx_warehouse_code ON inventory_task(warehouse_code);
CREATE INDEX idx_task_no ON inventory_item(task_no);
CREATE INDEX idx_product_code ON inventory_item(product_code);
CREATE INDEX idx_location_code ON inventory_item(location_code);
CREATE INDEX idx_warehouse_code ON move_order(warehouse_code);
CREATE INDEX idx_product_code ON move_order(product_code);
CREATE INDEX idx_template_code ON print_element(template_code);
CREATE INDEX idx_template_code ON print_record(template_code);

-- 设置外键检查开启
SET FOREIGN_KEY_CHECKS = 1;

-- 迁移完成
SELECT 'Database migration completed successfully!' AS status;
```

### 6.3 Prisma 迁移命令

```bash
# 创建迁移
npx prisma migrate dev --name init

# 应用迁移
npx prisma migrate deploy

# 重置数据库
npx prisma migrate reset

# 生成客户端
npx prisma generate

# 初始化数据
npx prisma db seed
```

### 6.2 数据备份策略

```bash
# MySQL 备份
mysqldump -u root -p wms_lite > backup_$(date +%Y%m%d).sql

# MySQL 恢复
mysql -u root -p wms_lite < backup.sql

# 定时备份 (crontab)
0 2 * * * mysqldump -u wms -ppassword wms_lite > /backup/wms_$(date +\%Y\%m\%d).sql
```

---

## 七、数据库性能优化

### 7.1 查询优化建议

1. **分页查询**: 大数据量必须使用分页
2. **字段选择**: 只查询需要的字段
3. **关联查询**: 合理使用 JOIN，避免 N+1 问题
4. **批量操作**: 批量插入/更新代替循环操作

### 7.2 Prisma 查询优化示例

```javascript
// 分页查询
const products = await prisma.product.findMany({
  skip: (page - 1) * pageSize,
  take: pageSize,
  select: {
    id: true,
    code: true,
    name: true,
    category: { select: { name: true } }
  }
})

// 批量插入
await prisma.inboundItem.createMany({
  data: items
})

// 事务处理
await prisma.$transaction([
  prisma.stock.update({ where: { id }, data: { quantity: { increment: qty } } }),
  prisma.stockRecord.create({ data: record })
])
```
