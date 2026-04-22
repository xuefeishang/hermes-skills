---
name: RESTful API设计
description: "当设计API接口、制定接口规范、实现REST服务、处理HTTP请求响应时，使用本技能确保API设计的规范性和可用性。"
license: MIT
---

# RESTful API设计技能

## 概述

API是系统间交互的桥梁。不合理的API设计会导致：
- 前端开发效率低下
- 接口调用混乱
- 版本升级困难
- 安全漏洞

**核心原则**: 好的API应该直观、一致、易于理解、向后兼容。坏的API会导致集成困难、维护成本高。

## 何时使用

**始终：**
- 设计新API接口时
- 重构现有API时
- 前后端接口联调时
- 开放平台接口设计时

**触发短语：**
- "API设计"
- "接口规范"
- "REST API"
- "HTTP接口"
- "请求响应格式"

## API设计核心规范

### 1. URL设计规范

#### 资源命名
```bash
# ✅ 正确示例：使用名词，复数形式
GET    /api/v1/users          # 获取用户列表
GET    /api/v1/users/{id}     # 获取单个用户
POST   /api/v1/users          # 创建用户
PUT    /api/v1/users/{id}    # 更新用户
DELETE /api/v1/users/{id}    # 删除用户

# ❌ 错误示例：使用动词
GET    /api/v1/getUsers
POST   /api/v1/createUser
POST   /api/v1/deleteUser
```

#### 层级结构（限制在3层以内）
```bash
# ✅ 合理层级
GET /api/v1/users/{userId}/orders

# ❌ 层级过深（超过3层）
GET /api/v1/users/{userId}/orders/{orderId}/items/{itemId}
```

### 2. HTTP方法规范

| 方法 | 用途 | 幂等性 | 安全性 |
|------|------|--------|--------|
| GET | 查询资源 | ✅ | ✅ |
| POST | 创建资源 | ❌ | ❌ |
| PUT | 更新资源（完整） | ✅ | ❌ |
| PATCH | 更新资源（部分） | ❌ | ❌ |
| DELETE | 删除资源 | ✅ | ❌ |

### 3. 状态码规范

```python
# 成功响应
200 OK              # 操作成功
201 Created         # 资源创建成功
204 No Content      # 删除成功，无返回内容

# 客户端错误
400 Bad Request     # 请求参数错误
401 Unauthorized    # 未认证
403 Forbidden       # 无权限
404 Not Found       # 资源不存在
409 Conflict        # 资源冲突
422 Unprocessable   # 验证错误

# 服务器错误
500 Internal Error  # 服务器内部错误
503 Service Unavailable # 服务不可用
```

### 4. 统一响应格式

```python
# Python Flask 响应格式
{
    "success": true,
    "data": {
        "id": 1,
        "name": "张三",
        "email": "zhangsan@example.com"
    },
    "message": "操作成功",
    "meta": {
        "page": 1,
        "limit": 20,
        "total": 100
    },
    "timestamp": "2024-01-15T10:30:00Z"
}

# 错误响应格式
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "参数验证失败",
        "details": [
            {"field": "email", "message": "邮箱格式不正确"}
        ]
    },
    "timestamp": "2024-01-15T10:30:00Z"
}
```

### 5. 分页规范

```bash
# 请求参数
GET /api/v1/users?page=1&limit=20&sort=name&order=asc

# 响应格式
{
    "data": [...],
    "pagination": {
        "page": 1,
        "limit": 20,
        "total": 100,
        "total_pages": 5,
        "has_next": true,
        "has_prev": false
    }
}
```

### 6. 过滤、排序、搜索

```bash
# 过滤条件
GET /api/v1/users?status=active&role=admin

# 搜索
GET /api/v1/users?search=张三

# 排序
GET /api/v1/users?sort=created_at&order=desc

# 字段选择
GET /api/v1/users?fields=id,name,email
```

## API版本控制

### URL路径版本控制（推荐）
```bash
# 版本在URL中
GET /api/v1/users
GET /api/v2/users

# 优点：直观、易于调试
# 缺点：版本变更时URL变化
```

### Header版本控制
```bash
# 请求头中指定版本
GET /api/users
Accept: application/vnd.api+json; version=2

# 优点：URL保持稳定
# 缺点：不直观，调试困难
```

## 认证授权

### OAuth 2.0 + JWT
```bash
# 获取Token
POST /api/v1/auth/login
Body: { "username": "xxx", "password": "xxx" }

# 响应
{
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "token_type": "Bearer",
    "expires_in": 7200
}

# 使用Token
GET /api/v1/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

## API设计常见问题

### 问题1：接口职责不清
```
❌ 一个接口做太多事情
POST /api/v1/user/operation
    - 既创建用户又发送邮件又初始化数据

✅ 一个接口只做一件事
POST /api/v1/users          # 创建用户
POST /api/v1/users/{id}/welcome-email  # 发送欢迎邮件
```

### 问题2：异常处理不规范
```
❌ 所有错误都返回500
✅ 正确使用状态码，错误信息结构化
```

### 问题3：缺少文档
```
❌ 接口上线后无文档
✅ 使用Swagger/OpenAPI自动生成文档
```

## API文档规范

### Swagger/OpenAPI 示例
```yaml
openapi: 3.0.0
info:
  title: 用户管理系统API
  version: 1.0.0
paths:
  /users:
    get:
      summary: 获取用户列表
      tags:
        - 用户管理
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
```

## 最佳实践清单

### 设计阶段
- [ ] 使用RESTful风格
- [ ] URL使用名词复数形式
- [ ] 正确使用HTTP方法
- [ ] 统一响应格式
- [ ] 设计版本控制策略
- [ ] 考虑安全性（认证授权）
- [ ] 编写API文档

### 实现阶段
- [ ] 参数验证
- [ ] 错误处理
- [ ] 日志记录
- [ ] 限流保护
- [ ] 幂等性处理

### 测试阶段
- [ ] 接口功能测试
- [ ] 参数边界测试
- [ ] 权限测试
- [ ] 性能测试
- [ ] 文档同步更新

## 相关资源

- [后端开发实战](../coding/backend-dev/) — 后端API实现
- [集成测试实施](../testing/integration-testing/) — API集成测试
- [Spring Boot应用开发](../frameworks/spring-boot/) — Java API框架
