# PWA OpenAPI 文档

> 版本: v1.0.0

欢迎使用PWA OpenAPI。本文档提供完整的 API 接入指南。

---

## 快速链接

| 文档 | 说明 |
|------|------|
| [接入指南](../common/integration-guide.md) | 快速开始、接入流程 |
| [认证机制](../common/authentication.md) | HMAC-SHA256 签名算法详解 |
| [SDK 示例](../common/sdk-examples.md) | Python、Java、Go、JavaScript、cURL 示例代码 |
| [错误码参考](../common/error-codes.md) | 完整错误码列表及解决方案 |
| [Webhook 指南](../common/webhook-guide.md) | 异步通知对接说明 |
| [OpenAPI 规范](../openapi/openapi.external.yaml) | Swagger/OpenAPI 3.0 规范文件 |

---

## 环境信息

| 环境 | Base URL | 说明 |
|------|----------|------|
| 沙箱环境 | `http://43.156.107.149:8888/?urls.primaryName=pwa` | 测试环境，用于开发调试 |
| 生产环境 | 待定 | 正式环境，数据真实有效 |

---

## API 分组概览

### 1. 应用管理服务 (App)

PWA应用管理接口。

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取应用列表 | GET | `/openapi/pwa/apps` | 支持分页、状态筛选、搜索 |
| 创建应用 | POST | `/openapi/pwa/apps` | 创建新的PWA应用 |
| 获取应用详情 | GET | `/openapi/pwa/apps/{appId}` | 获取指定应用完整信息 |
| 更新应用 | PATCH | `/openapi/pwa/apps/{appId}` | 更新应用配置信息 |
| 删除应用 | DELETE | `/openapi/pwa/apps/{appId}` | 删除指定应用 |

### 2. 应用举报/投诉管理服务 (Complaint)

举报与投诉处理接口。

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 列出举报记录 | GET | `/openapi/pwa/complaints` | 支持多条件筛选、分页、排序 |
| 提交举报 | POST | `/openapi/pwa/complaints` | 用户提交新的举报 |
| 获取举报统计 | GET | `/openapi/pwa/complaints/stats` | 获取举报数据统计 |
| 获取举报详情 | GET | `/openapi/pwa/complaints/{id}` | 查看单个举报详细信息 |
| 更新举报信息 | PATCH | `/openapi/pwa/complaints/{id}` | 更新举报记录 |
| 删除举报 | DELETE | `/openapi/pwa/complaints/{id}` | 删除指定举报记录 |
| 处理举报 | POST | `/openapi/pwa/complaints/{id}/handle` | 处理举报并更新状态 |

### 3. PWA落地页服务 (LandingPageService)

PWA应用落地页相关接口。

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 根据域名获取应用标识 | GET | `/openapi/pwa/landing/app` | 根据域名获取应用标识（PWA落地页第一步） |
| 获取应用详情 | GET | `/openapi/pwa/landing/detail` | 根据app_code获取应用详情（PWA落地页第二步） |
| 记录安装行为 | POST | `/openapi/pwa/landing/stat/install` | 记录PWA应用安装行为 |
| 记录打开行为 | POST | `/openapi/pwa/landing/stat/open` | 记录PWA应用打开行为 |

### 4. 推广链接管理服务 (PromotionLink)

推广链接管理接口。

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 列出推广链接 | GET | `/openapi/pwa/promotion-links` | 支持多条件筛选、分页、排序 |
| 创建推广链接 | POST | `/openapi/pwa/promotion-links` | 创建新的推广链接 |
| 获取推广链接统计 | GET | `/openapi/pwa/promotion-links/stats` | 获取推广链接整体统计 |
| 获取推广链接详情 | GET | `/openapi/pwa/promotion-links/{id}` | 查看单个推广链接详细信息 |
| 更新推广链接 | PATCH | `/openapi/pwa/promotion-links/{id}` | 更新推广链接配置 |
| 删除推广链接 | DELETE | `/openapi/pwa/promotion-links/{id}` | 删除指定推广链接 |

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

详见 [认证机制](../common/authentication.md)。

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

---

## 使用 Swagger UI 查看接口

1. 打开 [Swagger Editor](https://editor.swagger.io/)
2. 点击 `File` -> `Import URL`
3. 导入 `openapi.external.yaml` 文件
4. 即可查看完整的接口文档和请求/响应示例

---

## 使用 Postman 测试

1. 下载 [Postman Collection](./pwa-server.postman_collection.json)
2. 在 Postman 中导入 Collection
3. 配置环境变量：
   - `baseUrl`: `http://43.133.43.2/openapi/points`
   - `apiKey`: 您的 API Key
   - `apiSecret`: 您的 Secret
4. 手动添加认证头（或配置 Pre-request Script）

---

## 联系方式

如有问题，请联系技术支持。
