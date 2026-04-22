# RESTful API设计 - 技术参考文档

## 参考标准

### 国际标准
- **REST** — Representational State Transfer (Roy Fielding, 2000)
- **HTTP/1.1** — RFC 7231 (Hypertext Transfer Protocol)
- **OpenAPI 3.0** — API Documentation Standard
- **JSON:API** — JSON API规范

### 权威参考
- [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines)
- [Google Cloud API Design Guide](https://cloud.google.com/apis/design)
- [PayPal API Design Guidelines](https://github.com/paypal/api-standards)

## HTTP状态码详解

### 2xx 成功
| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 200 | OK | 操作成功 |
| 201 | Created | 资源创建成功 |
| 202 | Accepted | 异步操作已接受 |
| 204 | No Content | 删除成功，无返回 |

### 4xx 客户端错误
| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 405 | Method Not Allowed | HTTP方法不支持 |
| 409 | Conflict | 资源冲突 |
| 422 | Unprocessable Entity | 验证失败 |
| 429 | Too Many Requests | 请求过于频繁 |

### 5xx 服务器错误
| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 500 | Internal Server Error | 服务器内部错误 |
| 502 | Bad Gateway | 网关错误 |
| 503 | Service Unavailable | 服务不可用 |
| 504 | Gateway Timeout | 网关超时 |

## 认证授权方案

### JWT (JSON Web Token)
```python
import jwt
from datetime import datetime, timedelta

# 创建Token
def create_token(user_id):
    payload = {
        'user_id': user_id,
        'exp': datetime.utcnow() + timedelta(hours=2)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm='HS256')

# 验证Token
def verify_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        return payload
    except jwt.ExpiredSignatureError:
        return None  # Token过期
```

### OAuth 2.0 流程
```
1. 用户点击授权
2. 跳转授权页面
3. 用户同意授权
4. 返回授权码
5. 后端用授权码换Token
6. 用Token访问资源
```

## API文档工具

### Swagger/OpenAPI
```bash
# Python Flask
from flasgger import Swagger

app = Flask(__name__)
swagger = Swagger(app)

@app.route('/api/users', methods=['GET'])
def get_users():
    """
    获取用户列表
    ---
    tags:
      - 用户管理
    parameters:
      - name: page
        in: query
        type: integer
        default: 1
    responses:
      200:
        description: 成功
    """
    pass
```

### API管理平台
- **Postman** — API开发和测试
- **Apifox** — 国产API管理
- **Apiflow** — API设计协作
- **Insomnia** — API客户端

## 错误码规范示例

### 错误码结构
```python
{
    "error": {
        "code": "USER_001",
        "message": "用户不存在",
        "details": {...},
        "help_url": "https://api.example.com/docs/errors/USER_001"
    }
}
```

### 错误码命名规范
```
模块_序号
例如：
USER_001, USER_002
ORDER_001, ORDER_002
AUTH_001, AUTH_002
```

## 性能优化

### 缓存策略
```bash
# 客户端缓存
Cache-Control: max-age=3600

# 条件请求
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

### 压缩传输
```bash
# 启用Gzip压缩
Accept-Encoding: gzip, deflate
Content-Encoding: gzip
```

### 分页策略
```python
# 游标分页（适合大数据量）
GET /api/users?cursor=eyJpZCI6MTAwfQ&limit=20

# 偏移分页（适合小数据量）
GET /api/users?offset=0&limit=20
```

## 安全最佳实践

### 输入验证
```python
# 使用marshmallow进行参数验证
from marshmallow import Schema, fields, validate

class UserSchema(Schema):
    email = fields.Email(required=True)
    age = fields.Int(validate=validate.Range(min=0, max=150))
```

### 限流策略
```python
# Redis +滑动窗口限流
def rate_limit(key, max_requests, window):
    current = redis.incr(key)
    if current == 1:
        redis.expire(key, window)
    return current <= max_requests
```

### CORS配置
```python
# Flask CORS
from flask_cors import CORS
CORS(app, resources={r"/api/*": {"origins": "*"}})
```

## 推荐阅读

### 书籍
- 《RESTful Web APIs》— Leonard Richardson
- 《API Design Patterns》— Geertjan Tost

### 文章
- [REST API Design Best Practices](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)

---

**最后更新**: 2024年
**维护者**: 萝卜飞
**版本**: v1.0
