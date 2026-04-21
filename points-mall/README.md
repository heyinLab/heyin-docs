# 积分商城 OpenAPI 文档

> 版本: v2.0.0 — 多商城架构

欢迎使用积分商城 OpenAPI。本文档提供完整的 API 接入指南。

---

## 快速链接

| 文档                                       | 说明 |
|------------------------------------------|------|
| [接入指南](../common/integration-guide.md)   | 快速开始、接入流程 |
| [认证机制](../common/authentication.md)      | HMAC-SHA256 签名算法详解 |
| [SDK 示例](../common/sdk-examples.md)      | Python、Java、Go、JavaScript、cURL 示例代码 |
| [错误码参考](../common/error-codes.md)        | 完整错误码列表及解决方案 |
| [OpenAPI 规范](../openapi/mall.yaml)       | Swagger/OpenAPI 3.0 规范文件 |

---

## 环境信息

| 环境 | Base URL | 说明 |
|------|----------|------|
| 沙箱环境 | `http://43.156.107.149:81/openapi/points` | 测试环境，用于开发调试 |
| 生产环境 | 待定 | 正式环境，数据真实有效 |

---

## 多商城架构说明

v2.0 采用**单租户多商城**架构，每个租户可创建多个独立商城：

```
租户 (Tenant)
├── 商城 A (mall-a3f8b1c2e9d4)
│   ├── API Key: pk_live_xxx (scope: mall)
│   ├── Webhook URL
│   ├── 商品分类 / 商品
│   └── 订单
├── 商城 B (mall-b7e2d9f0a1c3)
│   ├── API Key: pk_live_yyy (scope: mall)
│   ├── Webhook URL
│   ├── 商品分类 / 商品
│   └── 订单
└── ...
```

### API Key 双作用域

| Scope | 说明 | 用途 |
|-------|------|------|
| `app` | 应用级密钥 | 创建/管理商城、管理商城 API Key |
| `mall` | 商城级密钥 | 商品管理、分类管理、生成票据、下单等 |

- 创建商城时自动签发一个 `mall` 级 API Key，每个商城仅允许一个
- `mall_code` 由系统自动生成（`mall-` + 随机 hex），无需手动指定

### 对接流程

```
1. 获取租户 app-scoped API Key
      ↓
2. 创建商城（传 mallName + webhookUrl）→ 返回 mall-scoped API Key + Secret
      ↓
3. 切换到 mall-scoped API Key
      ↓
4. 创建商品分类、添加商品
      ↓
5. 生成入场票据 → 用户携带 ticketToken 进入 H5 商城
      ↓
6. 用户下单 → 系统 POST webhookUrl 通知积分扣减
      ↓
7. 商户系统处理扣减 → 调用确认接口
```

---

## API 分组概览

### 1. 商城管理 (Mall Management) — app-scoped

管理商城及其 API Key 的接口，需使用 **app-scoped** API Key 调用。

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 创建商城 | POST | `/openapi/points/malls` | 自动生成 mallCode，同时签发 mall API Key |
| 获取商城列表 | GET | `/openapi/points/malls` | 支持分页、状态筛选 |
| 获取商城详情 | GET | `/openapi/points/malls/{mall_code}` | 商城完整信息 |
| 更新商城 | PUT | `/openapi/points/malls/{mall_code}` | 修改名称、域名、Webhook 等 |
| 停用商城 | POST | `/openapi/points/malls/{mall_code}/disable` | 停用商城 |
| 签发 API Key | POST | `/openapi/points/malls/{mall_code}/api-keys` | 为商城签发新 Key（每商城仅一个） |
| API Key 列表 | GET | `/openapi/points/malls/{mall_code}/api-keys` | 查看商城 API Key |
| 重置 Secret | POST | `/openapi/points/malls/{mall_code}/api-keys/{id}/reset-secret` | 重新生成 Secret |
| 启用/禁用 Key | POST | `/openapi/points/malls/{mall_code}/api-keys/{id}/toggle-status` | 切换 Key 状态 |

**创建商城示例：**

```
POST /openapi/points/malls

{
  "mallName": "我的积分商城",
  "webhookUrl": "https://your-domain.com/webhook"
}
```

**响应：**

```json
{
  "mall": {
    "id": 1234567,
    "mallCode": "mall-a3f8b1c2e9d4",
    "mallName": "我的积分商城",
    "webhookUrl": "https://your-domain.com/webhook",
    "status": true
  },
  "apiKey": {
    "apiKey": "pk_live_xxx",
    "scope": "mall"
  },
  "secret": "sk_live_yyy",
  "message": "创建商城成功"
}
```

> **注意**: `secret` 仅在创建时返回一次，请妥善保存。丢失后只能通过重置 Secret 接口重新生成。

---

### 2. 商品分类 (Product Categories) — mall-scoped

以下接口需使用 **mall-scoped** API Key 调用。

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取分类列表 | GET | `/openapi/points/product-categories` | 分类列表 |
| 创建分类 | POST | `/openapi/points/product-categories` | 新增分类 |
| 获取分类详情 | GET | `/openapi/points/product-categories/{id}` | 分类信息 |
| 更新分类 | PUT | `/openapi/points/product-categories/{id}` | 修改分类 |
| 删除分类 | DELETE | `/openapi/points/product-categories/{id}` | 删除分类 |

---

### 3. 商品管理 (Goods Management) — mall-scoped

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取商品列表 | GET | `/openapi/points/goods` | 管理后台商品列表 |
| 创建商品 | POST | `/openapi/points/goods` | 新增商品 |
| 获取商品详情 | GET | `/openapi/points/goods/{id}` | 商品完整信息 |
| 更新商品 | PUT | `/openapi/points/goods/{id}` | 修改商品信息 |
| 删除商品 | DELETE | `/openapi/points/goods/{id}` | 删除商品 |

**虚拟卡密管理：**

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取卡密列表 | GET | `/openapi/points/goods/cards/{goodsId}` | 商品关联的卡密 |
| 批量上传卡密 | POST | `/openapi/points/good/cards/batch/{goodsId}` | 手动输入批量添加 |
| Excel导入卡密 | POST | `/openapi/points/goods/cards/import/{goodsId}` | Excel文件导入 |
| 删除卡密 | DELETE | `/openapi/points/cards/{cardId}` | 删除单个卡密 |
| 更新卡密状态 | PUT | `/openapi/points/cards/{cardId}/status` | 设置可用/无效 |

---

### 4. 认证服务 (Auth Ticket) — mall-scoped

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 生成票据 | POST | `/openapi/points/auth/ticket/generate` | 生成免登票据（需传 mallCode） |
| 验证票据 | GET | `/openapi/points/auth/ticket/verify` | 验证并建立会话（公开接口） |
| 刷新会话 | POST | `/openapi/points/auth/session/refresh` | 续期会话Token |

**生成票据示例：**

```
POST /openapi/points/auth/ticket/generate

{
  "tenantCode": "T001",
  "userCode": "USER001",
  "userName": "张三",
  "points": 5000,
  "mallCode": "mall-a3f8b1c2e9d4"
}
```

> `mallCode` 将票据绑定到指定商城，用户进入 H5 后只能浏览该商城的商品。

---

### 5. H5 商城服务 (Mall) — Session Token 认证

面向终端用户的 H5 商城接口，使用票据验证后获得的 Session Token 认证。

**所有 H5 接口必须携带以下 Header：**

```
X-Session-Token: {session_token}
```

> `session_token` 通过 `GET /openapi/points/auth/ticket/verify?ticketToken=xxx` 验证票据后获得。

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取用户信息 | GET | `/openapi/points/mall/user/info` | 用户积分等信息 |
| 获取分类列表 | GET | `/openapi/points/mall/categories` | 商品分类树 |
| 获取商品列表 | GET | `/openapi/points/mall/products` | 支持分页、分类筛选 |
| 获取商品详情 | GET | `/openapi/points/mall/products/{productId}` | 含多语言信息 |
| 创建订单 | POST | `/openapi/points/mall/orders` | 积分兑换下单 |
| 获取订单列表 | GET | `/openapi/points/mall/orders` | 用户订单记录 |
| 获取订单详情 | GET | `/openapi/points/mall/orders/{orderId}` | 含物流、卡密信息 |
| 确认收货 | PUT | `/openapi/points/mall/orders/{orderId}/confirm` | 完成订单 |

**收货地址管理：**

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取地址列表 | GET | `/openapi/points/mall/addresses` | 用户收货地址 |
| 创建地址 | POST | `/openapi/points/mall/addresses` | 新增收货地址 |
| 更新地址 | PUT | `/openapi/points/mall/addresses/{id}` | 修改地址信息 |
| 删除地址 | DELETE | `/openapi/points/mall/addresses/{id}` | 删除地址 |
| 获取默认地址 | GET | `/openapi/points/mall/addresses/default` | 默认收货地址 |
| 设置默认地址 | PUT | `/openapi/points/mall/addresses/{id}/default` | 设为默认 |

---

### 6. 订单管理 (Order Management) — mall-scoped

后台管理接口，用于订单处理。

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取订单列表 | GET | `/openapi/points/orders` | 支持多条件筛选 |
| 获取订单详情 | GET | `/openapi/points/orders/{id}` | 订单完整信息 |
| 更新订单 | PUT | `/openapi/points/orders/{id}` | 修改订单信息（取消等） |
| 删除订单 | DELETE | `/openapi/points/orders/{id}` | 删除订单记录 |
| 发货 | POST | `/openapi/points/orders/{id}/ship` | 填写物流信息 |

---

### 7. 物流管理 (Logistics) — mall-scoped

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取物流公司列表 | GET | `/openapi/points/logistics-companies` | 可选物流公司 |
| 创建物流公司 | POST | `/openapi/points/logistics-companies` | 新增物流公司 |
| 获取物流公司详情 | GET | `/openapi/points/logistics-companies/{id}` | 物流公司信息 |
| 更新物流公司 | PUT | `/openapi/points/logistics-companies/{id}` | 修改物流公司 |
| 删除物流公司 | DELETE | `/openapi/points/logistics-companies/{id}` | 删除物流公司 |

---

### 8. 租户管理 (Tenant Management) — app-scoped

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取租户列表 | GET | `/openapi/points/tenants` | 租户配置列表 |
| 创建租户 | POST | `/openapi/points/tenants` | 新增租户配置 |
| 获取租户详情 | GET | `/openapi/points/tenants/{tenantCode}` | 租户完整配置 |
| 更新租户 | PUT | `/openapi/points/tenants/{tenantCode}` | 修改租户配置 |
| 删除租户 | DELETE | `/openapi/points/tenants/{tenantCode}` | 删除租户 |
| 重置密钥 | POST | `/openapi/points/tenants/{tenantCode}/reset-secret` | 重新生成API密钥 |

---

### 9. Webhook 服务

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 确认Webhook | POST | `/openapi/points/webhook/confirm` | 商户确认处理结果 |
| 获取事件列表 | GET | `/openapi/points/webhook/events` | 查询Webhook事件 |
| 获取事件详情 | GET | `/openapi/points/webhook/events/{eventId}` | 单个事件信息 |
| 手动重试 | POST | `/openapi/points/webhook/events/{eventId}/retry` | 重新发送Webhook |

#### Webhook 回调说明

用户下单后，系统向商城配置的 `webhookUrl` 发起 POST 请求通知积分扣减/退还。

**事件类型：**

| 事件 | 触发时机 |
|------|---------|
| `points_deduct` | 用户下单消费积分 |
| `points_refund` | 订单取消退还积分 |

**请求示例（积分扣减）：**

```
POST {webhookUrl}
Content-Type: application/json
X-Webhook-Event-Id: 459f4306-0c0d-42b8-a753-8b42c2199dff
X-Webhook-Event-Type: points_deduct
X-Webhook-Timestamp: 1776136557
X-Webhook-Signature: 05430b0c59ae3dca5a16600c00de560133a8c82eacd544fe4496ce1cb27032b7
X-Webhook-Signature-Algorithm: HMAC-SHA256
```

```json
{
  "event_id": "459f4306-0c0d-42b8-a753-8b42c2199dff",
  "event_type": "points_deduct",
  "tenant_code": "AMGvfOgFmaWqRYDr4KRYl",
  "timestamp": 1776136557,
  "data": {
    "deduct_points": 100,
    "goods_id": 1323730640,
    "goods_name": "商品4",
    "order_id": 776648204,
    "order_no": "ZFD202604140315571100810",
    "user_id": "usercode1"
  },
  "signature": "05430b0c59ae3dca5a16600c00de560133a8c82eacd544fe4496ce1cb27032b7"
}
```

**商户系统响应要求：**

返回 HTTP 2xx 状态码即表示接收成功，响应体内容不限：

```json
{
  "success": true,
  "message": "received"
}
```

> 收到后请先返回 2xx，再异步处理业务逻辑，避免请求超时（默认 30 秒）。
> 处理完成后需调用 `POST /openapi/points/webhook/confirm` 确认处理结果。

**重试策略（指数退避）：**

```
1min → 2min → 4min → 8min → 16min → 32min → 60min → 60min → 60min → 60min
```

最多重试 10 次。确认超时默认 30 分钟。

**URL 协议处理：** `webhookUrl` 无协议前缀时，系统自动先尝试 HTTPS，失败后 fallback 到 HTTP。

详见 [Webhook 指南](../common/webhook-guide.md)。

---

## 通用规范

### 请求格式

- **协议**: HTTP/HTTPS
- **数据格式**: JSON
- **字符编码**: UTF-8
- **字段命名**: snake_case（下划线命名法）

### 认证方式

所有 API 请求必须携带以下 HTTP Headers：

```
X-API-Key: {your_api_key}
X-Timestamp: {unix_timestamp}
X-Signature: {hmac_sha256_signature}
```

**签名算法：**

```
SignString = Timestamp + ApiKey
Signature = HMAC-SHA256(Secret, SignString)
```

时间戳有效期为 5 分钟。详见 [认证机制](../common/authentication.md)。

### 响应格式

**成功响应：**
```json
{
    "code": 0,
    "message": "success",
    "data": { ... }
}
```

**错误响应：**
```json
{
    "code": 40001,
    "message": "Invalid API Key",
    "reason": "INVALID_API_KEY"
}
```


## 联系方式

如有问题，请联系技术支持。
