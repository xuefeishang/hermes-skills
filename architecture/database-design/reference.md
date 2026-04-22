# 数据库设计 - 技术参考文档

## 数据库范式详解

### 第一范式（1NF）
**要求**：字段具有原子性，不可再分

```
❌ 违反1NF：字段可再分
f_address VARCHAR(200)  -- "北京市朝阳区XXX"

✅ 符合1NF：
f_province VARCHAR(32)   -- 省
f_city VARCHAR(32)        -- 市
f_district VARCHAR(32)   -- 区
f_detail VARCHAR(200)    -- 详细地址
```

### 第二范式（2NF）
**要求**：消除部分依赖（复合主键情况下，非主键字段不能只依赖主键的一部分）

```
❌ 违反2NF（假设订单号+商品ID是复合主键）
t_order_item (f_order_no, f_product_id, f_product_name, ...)
-- f_product_name只依赖f_product_id，不依赖f_order_no

✅ 符合2NF：
-- 商品表单独存储商品信息
t_product (f_product_id, f_product_name, ...)

-- 订单明细表只存储关系和数量
t_order_item (f_order_no, f_product_id, f_quantity, f_price, ...)
```

### 第三范式（3NF）
**要求**：消除传递依赖（非主键字段不能依赖于另一个非主键字段）

```
❌ 违反3NF：
t_user (f_id, f_user_name, f_dept_name, f_dept_leader)
-- f_dept_name和f_dept_leader通过部门传递依赖于f_id

✅ 符合3NF：
t_user (f_id, f_user_name, f_dept_id, ...)
t_dept (f_dept_id, f_dept_name, f_dept_leader, ...)
```

## ER图绘制规范

### 符号说明
```
┌──────────────┐
│    实体名     │  -- 矩形表示实体
├──────────────┤
│  属性1 (PK)   │  -- PK表示主键
│  属性2        │
│  属性3 (FK)   │  -- FK表示外键
└──────────────┘

1 ────── N    -- 一对多
N ────── M    -- 多对多
1 ────── 1    -- 一对一
```

### 常用ER图工具
- **draw.io** — 免费在线绘图
- **PowerDesigner** — 专业建模工具
- **Navicat** — 数据库设计
- **PDMan** — 国产免费建模工具

## 字段类型选择指南

### 整数类型
| 类型 | 范围 | 字节 | 适用场景 |
|------|------|------|----------|
| TINYINT | -128~127 | 1 | 状态、枚举 |
| SMALLINT | -32768~32767 | 2 | 小数量 |
| INT | -21亿~21亿 | 4 | 主键、常规数量 |
| BIGINT | 极大 | 8 | 主键、金额 |

### 字符串类型
| 类型 | 最大长度 | 适用场景 |
|------|----------|----------|
| CHAR | 255 | 固定长度（性别、状态码） |
| VARCHAR | 65535 | 可变长度（用户名、地址） |
| TEXT | 65535 | 大文本（日志、备注） |
| LONGTEXT | 4GB | 超大文本 |

### 时间类型
| 类型 | 格式 | 适用场景 |
|------|------|----------|
| DATE | 2024-01-15 | 只关心日期 |
| DATETIME | 2024-01-15 10:30:00 | 时间戳 |
| TIMESTAMP | 时间戳 | UTC自动转换 |

## 索引原理

### B+树索引
```
- 是一种平衡树
- 所有数据都在叶子节点
- 叶子节点有序链表
- 查询复杂度O(log n)
```

### 索引失效场景
```sql
-- ❌ 使用函数
SELECT * FROM t_user WHERE YEAR(f_create_time) = 2024;

-- ❌ 类型转换
SELECT * FROM t_user WHERE f_phone = 13800138000;  -- f_phone是VARCHAR

-- ❌ 前缀模糊查询
SELECT * FROM t_user WHERE f_name LIKE '%三%';

-- ❌ OR条件
SELECT * FROM t_user WHERE f_user_name = '张三' OR f_email = 'zhangsan@example.com';
-- 如果有联合索引idx_name_email(user_name, email)，使用OR会导致索引失效
```

## 分库分表策略

### 垂直拆分
```
将大表的字段拆分到多个表

t_user --> t_user_base (id, name, email, phone)
        --> t_user_profile (id, user_id, avatar, bio, ...)
```

### 水平拆分
```
将大表的记录拆分到多个表

t_order_2024_01
t_order_2024_02
t_order_2024_03
```

### 分片键选择
```
选择原则：
1. 业务经常用到的字段
2. 尽量均匀分布的字段
3. 不经常更新的字段

例如：订单表按user_id分片，因为经常需要查询某用户的所有订单
```

## 数据库选型

### 关系型数据库
| 数据库 | 适用场景 | 特点 |
|--------|----------|------|
| MySQL | 互联网应用 | 开源、成熟、生态丰富 |
| PostgreSQL | 复杂查询 | 功能强大、标准支持好 |
| Oracle | 企业核心系统 | 商业、高可靠 |
| SQL Server | Windows环境 | 微软生态、集成度高 |

### NoSQL数据库
| 数据库 | 类型 | 适用场景 |
|--------|------|----------|
| Redis | KV/缓存 | 高性能缓存、Session |
| MongoDB | 文档 | 灵活 schema、日志 |
| Elasticsearch | 搜索 | 全文检索、日志分析 |

## 数据库工具

### 设计工具
- **PowerDesigner** — 专业数据库建模
- **Navicat Data Modeler** — 可视化建模
- **PDMan** — 国产免费建模工具
- **dbdiagram.io** — 在线ER图

### 管理工具
- **DBeaver** — 通用数据库客户端
- **Navicat** — 功能全面
- **DataGrip** — JetBrains出品
- **HeidiSQL** — 轻量级MySQL客户端

### 监控工具
- **Prometheus + Grafana** — 性能监控
- **Percona Monitoring** — MySQL监控
- **pgAdmin** — PostgreSQL管理

## 推荐阅读

### 书籍
- 《数据库系统概论》— 王珊
- 《高性能MySQL》— Baron Schwartz
- 《SQL权威指南》— Joe Celko

### 在线资源
- [MySQL官方文档](https://dev.mysql.com/doc/)
- [PostgreSQL官方文档](https://www.postgresql.org/docs/)
- [SQL Tutorial](https://www.w3schools.com/sql/)

---

**最后更新**: 2024年
**维护者**: 萝卜飞
**版本**: v1.0
