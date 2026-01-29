# CFService API 文档

> 版本: v1.0.0

欢迎使用 CFService API。本文档提供自定义域名管理和 Cloudflare 集成功能的完整 API 接入指南。

---

## 快速链接

| 文档 | 说明 |
|------|------|
| [接入指南](../common/integration-guide.md) | 快速开始、接入流程 |
| [认证机制](../common/authentication.md) | HMAC-SHA256 签名算法详解 |
| [SDK 示例](../common/sdk-examples.md) | Python、Java、Go、JavaScript、cURL 示例代码 |
| [错误码参考](../common/error-codes.md) | 完整错误码列表及解决方案 |
| [Webhook 指南](../common/webhook-guide.md) | 异步通知对接说明 |
| [OpenAPI 规范](../openapi/openapi.cfservice.yaml) | Swagger/OpenAPI 3.0 规范文件 |

---

## 环境信息

| 环境 | Base URL | 说明 |
|------|----------|------|
| 沙箱环境 | `http://43.156.107.149:8888/?urls.primaryName=cf` | 测试环境，用于开发调试 |
| 生产环境 | 待定 | 正式环境，数据真实有效 |

---

## API 分组概览

### 1. 域名管理集成 (Domains)

自定义域名管理和 Cloudflare 集成功能。

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 获取域名列表 | GET | `/openapi/cf/domains` | 支持分页、搜索和状态过滤 |
| 创建域名配置 | POST | `/openapi/cf/domains` | 创建新的域名配置 |
| 根据名称获取域名 | GET | `/openapi/cf/domains/by-name/{domainName}` | 根据域名名称获取信息 |
| 隐藏域名 | POST | `/openapi/cf/domains/hide` | 隐藏指定域名 |
| 更新域名状态 | PUT | `/openapi/cf/domains/status` | 更新状态（如标记为使用中、封禁） |
| 取消隐藏域名 | POST | `/openapi/cf/domains/unhide` | 取消隐藏域名 |
| 验证域名 | POST | `/openapi/cf/domains/verify` | 检查DNS配置和SSL状态 |
| 获取域名详情 | GET | `/openapi/cf/domains/{domainId}` | 根据ID获取域名完整信息 |
| 删除域名 | DELETE | `/openapi/cf/domains/{domainId}` | 硬删除域名及相关资源 |
| 获取DNS配置说明 | GET | `/openapi/cf/domains/{domainId}/dns-instructions` | 获取NS和CNAME配置说明 |
| 获取域名选项列表 | GET | `/openapi/cf/options/domains` | 用于下拉框的域名选项 |

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

1. 下载 [Postman Collection](./cf-server.postman_collection.json)
2. 在 Postman 中导入 Collection
3. 配置环境变量：
   - `baseUrl`: `http://43.133.43.2/openapi/points`
   - `apiKey`: 您的 API Key
   - `apiSecret`: 您的 Secret
4. 手动添加认证头（或配置 Pre-request Script）

---

## 联系方式

如有问题，请联系技术支持。