---
name: 数据库设计
description: "当设计数据库表结构、创建数据模型、进行ER图建模、优化SQL查询时，使用本技能确保数据库设计的规范性和性能。"
license: MIT
---

# 数据库设计技能

## 概述

数据库是信息系统的核心。不合理的数据库设计会导致：
- 数据冗余和更新异常
- 查询性能低下
- 扩展困难
- 数据完整性问题

**核心原则**: 好的数据库设计应该遵循范式要求、索引合理、表结构清晰。坏的数据库设计会导致数据混乱、性能问题、维护困难。

## 何时使用

**始终：**
- 新系统数据库设计时
- 现有数据库重构时
- 新功能需要新增表时
- SQL性能优化时

**触发短语：**
- "数据库设计"
- "表结构"
- "ER图"
- "SQL优化"
- "索引设计"
- "数据库范式"

## 数据库设计流程

### 1. 需求分析阶段
```
1. 识别实体（Entity）
   - 用户、订单、商品、供应商等
   
2. 识别属性（Attribute）
   - 用户：姓名、邮箱、密码、创建时间
   
3. 识别关系（Relationship）
   - 用户 1:N 订单
   - 订单 N:M 商品
```

### 2. 概念设计阶段（ER图）
```bash
+-------------+       +-------------+
|    用户      |       |    订单      |
+-------------+       +-------------+
| PK id       |<--1:N->| FK user_id  |
| name        |       | PK id       |
| email       |       | order_no    |
| password    |       | total_amount|
| created_at  |       | status      |
+-------------+       | created_at  |
                      +-------------+
```

### 3. 逻辑设计阶段

#### 范式要求

| 范式 | 要求 | 适用场景 |
|------|------|----------|
| 1NF | 字段原子性，不可再分 | 所有场景 |
| 2NF | 消除部分依赖 | 有复合主键的表 |
| 3NF | 消除传递依赖 | 所有场景 |
| BCNF | 消除主属性对候选码的部分/传递依赖 | 特殊场景 |

#### 命名规范
```sql
-- 表命名：t_业务模块_实体名
t_sys_user          -- 系统用户表
t_biz_order          -- 业务订单表
t_biz_order_item     -- 订单明细表

-- 字段命名：f_实体_属性名
f_user_name          -- 用户名
f_user_email         -- 用户邮箱
f_order_amount       -- 订单金额

-- 索引命名：idx_表名_字段名
idx_sys_user_email   -- 用户邮箱索引
idx_biz_order_status -- 订单状态索引

-- 主键命名：pk_表名
pk_sys_user          -- 用户表主键

-- 外键命名：fk_表名_引用表名
fk_order_user        -- 订单表外键引用用户表
```

## 表结构设计规范

### 通用字段设计
```sql
-- 每张表必备字段
CREATE TABLE t_sys_user (
    f_id              BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '主键ID',
    f_user_no         VARCHAR(32) NOT NULL UNIQUE COMMENT '用户编号',
    f_user_name       VARCHAR(64) NOT NULL COMMENT '用户名',
    f_email           VARCHAR(128) NOT NULL COMMENT '邮箱',
    f_password        VARCHAR(256) NOT NULL COMMENT '密码（加密存储）',
    f_status          TINYINT DEFAULT 1 COMMENT '状态：0禁用 1启用',
    f_create_time     DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    f_update_time     DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    f_create_by       BIGINT COMMENT '创建人',
    f_update_by       BIGINT COMMENT '更新人',
    f_deleted         TINYINT DEFAULT 0 COMMENT '删除标记：0未删 1已删',
    f_version         INT DEFAULT 0 COMMENT '乐观锁版本号',
    f_remark          VARCHAR(256) COMMENT '备注',
    
    -- 索引
    INDEX idx_email (f_email),
    INDEX idx_status (f_status),
    INDEX idx_create_time (f_create_time)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统用户表';
```

### 主键设计原则
```sql
-- ✅ 推荐：使用BIGINT自增主键
f_id BIGINT PRIMARY KEY AUTO_INCREMENT

-- ✅ 推荐：使用UUID（分布式场景）
f_id VARCHAR(36) PRIMARY KEY DEFAULT (UUID())

-- ❌ 不推荐：使用业务字段作为主键
f_user_no VARCHAR(32) PRIMARY KEY  -- 用户编号可能变化
```

### 字段类型选择

| 数据类型 | 使用场景 | 注意事项 |
|----------|----------|----------|
| BIGINT | 主键、金额 | 避免溢出 |
| VARCHAR | 字符串 | 指定长度 |
| TEXT | 大文本 | 避免滥用 |
| DATETIME | 时间 | 推荐使用 |
| DECIMAL | 金额 | 不用FLOAT |
| TINYINT | 状态、枚举 | 明确含义 |

### 金额字段设计
```sql
-- ✅ 正确：使用DECIMAL
f_amount DECIMAL(15,2) COMMENT '金额'

-- ❌ 错误：使用DOUBLE/FLOAT
f_amount DOUBLE COMMENT '金额'  -- 精度问题
```

## 索引设计规范

### 索引类型
```sql
-- 主键索引
PRIMARY KEY (f_id)

-- 唯一索引
UNIQUE INDEX idx_email (f_email)

-- 普通索引
INDEX idx_status (f_status)

-- 联合索引
INDEX idx_user_status_time (f_user_id, f_status, f_create_time)
```

### 索引设计原则
```
✅ 应该创建索引的场景：
- WHERE条件经常使用的字段
- JOIN连接的字段
- ORDER BY排序的字段
- SELECT中频繁使用的字段

❌ 不应该创建索引的场景：
- 基数很低的字段（如性别）
- 更新频繁的字段
- 参与计算的字段
- 字符串前缀长度过长的字段
```

### 联合索引设计
```sql
-- 场景：经常查询「某用户的状态为启用的订单」
-- 联合索引顺序：区分度高的字段在前
INDEX idx_user_status (f_user_id, f_status)

-- SQL优化
SELECT * FROM t_biz_order 
WHERE f_user_id = 1 AND f_status = 1;

-- 覆盖索引优化（不需要回表）
SELECT f_id, f_order_no FROM t_biz_order 
WHERE f_user_id = 1 AND f_status = 1;
```

## 常见表设计模板

### 用户表
```sql
CREATE TABLE t_sys_user (
    f_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    f_user_no VARCHAR(32) NOT NULL UNIQUE COMMENT '用户编号',
    f_user_name VARCHAR(64) NOT NULL COMMENT '用户姓名',
    f_phone VARCHAR(20) COMMENT '手机号',
    f_email VARCHAR(128) COMMENT '邮箱',
    f_password VARCHAR(256) NOT NULL COMMENT '密码',
    f_avatar VARCHAR(512) COMMENT '头像URL',
    f_dept_id BIGINT COMMENT '部门ID',
    f_post VARCHAR(64) COMMENT '岗位',
    f_status TINYINT DEFAULT 1 COMMENT '状态',
    f_login_time DATETIME COMMENT '最后登录时间',
    f_login_ip VARCHAR(64) COMMENT '最后登录IP',
    f_create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    f_update_time DATETIME ON UPDATE CURRENT_TIMESTAMP,
    f_create_by BIGINT,
    f_update_by BIGINT,
    f_deleted TINYINT DEFAULT 0,
    f_version INT DEFAULT 0,
    f_remark VARCHAR(256),
    
    INDEX idx_phone (f_phone),
    INDEX idx_email (f_email),
    INDEX idx_dept (f_dept_id),
    INDEX idx_status (f_status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

### 部门表（树形结构）
```sql
CREATE TABLE t_sys_dept (
    f_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    f_parent_id BIGINT DEFAULT 0 COMMENT '父部门ID，0为根',
    f_dept_path VARCHAR(512) COMMENT '部门路径，如：/1/2/3/',
    f_dept_level INT DEFAULT 1 COMMENT '部门层级',
    f_dept_name VARCHAR(64) NOT NULL COMMENT '部门名称',
    f_leader_name VARCHAR(32) COMMENT '负责人姓名',
    f_leader_phone VARCHAR(20) COMMENT '负责人电话',
    f_sort_order INT DEFAULT 0 COMMENT '排序',
    f_status TINYINT DEFAULT 1 COMMENT '状态',
    f_create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    f_update_time DATETIME ON UPDATE CURRENT_TIMESTAMP,
    f_deleted TINYINT DEFAULT 0,
    
    INDEX idx_parent (f_parent_id),
    INDEX idx_path (f_dept_path)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='部门表';
```

### 订单表
```sql
CREATE TABLE t_biz_order (
    f_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    f_order_no VARCHAR(32) NOT NULL UNIQUE COMMENT '订单号',
    f_user_id BIGINT NOT NULL COMMENT '用户ID',
    f_order_status TINYINT DEFAULT 1 COMMENT '订单状态：1待支付 2已支付 3已完成 4已取消',
    f_total_amount DECIMAL(15,2) NOT NULL COMMENT '订单总金额',
    f_pay_amount DECIMAL(15,2) COMMENT '实付金额',
    f_pay_time DATETIME COMMENT '支付时间',
    f_pay_method TINYINT COMMENT '支付方式：1微信 2支付宝 3银行卡',
    f_shipping_address_id BIGINT COMMENT '收货地址ID',
    f_shipping_time DATETIME COMMENT '发货时间',
    f_receive_time DATETIME COMMENT '收货时间',
    f_cancel_time DATETIME COMMENT '取消时间',
    f_cancel_reason VARCHAR(256) COMMENT '取消原因',
    f_create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    f_update_time DATETIME ON UPDATE CURRENT_TIMESTAMP,
    f_deleted TINYINT DEFAULT 0,
    f_version INT DEFAULT 0,
    f_remark VARCHAR(512) COMMENT '备注',
    
    INDEX idx_order_no (f_order_no),
    INDEX idx_user (f_user_id),
    INDEX idx_status (f_order_status),
    INDEX idx_create_time (f_create_time)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';
```

### 订单明细表
```sql
CREATE TABLE t_biz_order_item (
    f_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    f_order_id BIGINT NOT NULL COMMENT '订单ID',
    f_product_id BIGINT NOT NULL COMMENT '商品ID',
    f_product_name VARCHAR(128) NOT NULL COMMENT '商品名称（冗余）',
    f_sku_code VARCHAR(64) COMMENT 'SKU编码',
    f_price DECIMAL(15,2) NOT NULL COMMENT '单价',
    f_quantity INT NOT NULL DEFAULT 1 COMMENT '数量',
    f_amount DECIMAL(15,2) NOT NULL COMMENT '小计金额',
    f_discount_amount DECIMAL(15,2) DEFAULT 0 COMMENT '优惠金额',
    f_create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_order (f_order_id),
    INDEX idx_product (f_product_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单明细表';
```

## SQL编写规范

### INSERT语句
```sql
-- ✅ 批量插入
INSERT INTO t_sys_user (f_user_name, f_email, f_password) VALUES
('张三', 'zhangsan@example.com', 'hashed_pwd'),
('李四', 'lisi@example.com', 'hashed_pwd'),
('王五', 'wangwu@example.com', 'hashed_pwd');

-- ✅ INSERT后获取ID
INSERT INTO t_sys_user (f_user_name, f_email, f_password) 
VALUES ('张三', 'zhangsan@example.com', 'hashed_pwd');
SELECT LAST_INSERT_ID();
```

### UPDATE语句
```sql
-- ✅ 使用乐观锁更新
UPDATE t_biz_order 
SET f_order_status = 2, 
    f_pay_time = NOW(),
    f_version = f_version + 1
WHERE f_id = 1 AND f_version = 0;

-- ✅ 验证影响行数
UPDATE t_biz_order 
SET f_order_status = 2 
WHERE f_id = 1 AND f_order_status = 1;
-- 检查 affected_rows = 1
```

### 查询优化
```sql
-- ✅ 使用EXPLAIN分析查询
EXPLAIN SELECT * FROM t_biz_order WHERE f_user_id = 1;

-- ✅ 避免SELECT *
SELECT f_id, f_order_no, f_total_amount FROM t_biz_order;

-- ✅ 使用LIMIT限制结果集
SELECT * FROM t_sys_user LIMIT 100;

-- ✅ 分页查询优化（大数据量）
-- 低效：
SELECT * FROM t_biz_order ORDER BY f_id LIMIT 1000000, 20;
-- 高效（使用ID游标）：
SELECT * FROM t_biz_order WHERE f_id > 1000000 ORDER BY f_id LIMIT 20;
```

## 数据库设计常见问题

### 问题1：没有主键
```
❌ CREATE TABLE t_order (...);  -- 没有主键

✅ CREATE TABLE t_order (
    f_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    ...
);
```

### 问题2：字段长度设计不合理
```
❌ f_phone VARCHAR(1000)  -- 过长，浪费空间
❌ f_name VARCHAR(10)     -- 过短，可能不够用

✅ f_phone VARCHAR(20)   -- 手机号最长15位
✅ f_name VARCHAR(64)    -- 中文姓名64位足够
```

### 问题3：没有索引或索引过多
```
❌ 频繁查询的字段没有索引

❌ 每个字段都建索引（更新变慢）
CREATE INDEX idx_col1 ON t_table(col1);
CREATE INDEX idx_col2 ON t_table(col2);
...
```

### 问题4：金额使用浮点数
```
❌ f_amount DOUBLE  -- 精度问题

✅ f_amount DECIMAL(15,2)  -- 精确存储
```

## 最佳实践清单

### 设计阶段
- [ ] 遵循命名规范
- [ ] 合理使用范式（3NF）
- [ ] 每表必备审计字段
- [ ] 主键选择合适类型
- [ ] 字段长度设计合理
- [ ] 考虑未来扩展性

### 索引阶段
- [ ] 根据查询设计索引
- [ ] 考虑联合索引顺序
- [ ] 避免过多索引
- [ ] 考虑覆盖索引

### SQL阶段
- [ ] 避免SELECT *
- [ ] 使用参数化查询
- [ ] 批量操作优化
- [ ] 分页查询优化
- [ ] 使用EXPLAIN分析

## 相关资源

- [数据库实现](../coding/database-impl/) — SQL脚本开发
- [SQL优化与索引](../database/sql-optimization/) — 性能优化
- [系统架构设计](../system-design/) — 架构设计
