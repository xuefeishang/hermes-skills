# Spring Boot后端开发 - 技术参考文档

## Spring Boot 核心注解

### 核心注解
| 注解 | 说明 |
|------|------|
| @SpringBootApplication | 启动类注解 |
| @Configuration | 配置类 |
| @Bean | Bean定义 |
| @ComponentScan | 组件扫描 |
| @Conditional | 条件配置 |

### Web注解
| 注解 | 说明 |
|------|------|
| @RestController | REST控制器 |
| @Controller | MVC控制器 |
| @RequestMapping | 请求映射 |
| @GetMapping | GET请求 |
| @PostMapping | POST请求 |
| @PutMapping | PUT请求 |
| @DeleteMapping | DELETE请求 |
| @PathVariable | 路径参数 |
| @RequestParam | 请求参数 |
| @RequestBody | 请求体 |

### 依赖注入注解
| 注解 | 说明 |
|------|------|
| @Autowired | 自动注入 |
| @Resource | 资源注入 |
| @Inject | 依赖注入 |
| @Primary | 优先使用 |
| @Qualifier | 指定Bean |

### 数据访问注解
| 注解 | 说明 |
|------|------|
| @Repository | 数据仓库 |
| @Mapper | MyBatis映射 |
| @TableName | 表名映射 |
| @TableField | 字段映射 |
| @TableId | 主键映射 |
| @Version | 乐观锁 |

### AOP注解
| 注解 | 说明 |
|------|------|
| @Aspect | 切面类 |
| @Before | 前置通知 |
| @After | 后置通知 |
| @Around | 环绕通知 |
| @Pointcut | 切入点 |

## MyBatis Plus 常用功能

### 条件构造器
```java
// 简单条件
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(User::getStatus, 1);
List<User> list = userRepository.selectList(wrapper);

// 复杂条件
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.like(StrUtil.isNotBlank(name), User::getUserName, name)
       .eq(status != null, User::getStatus, status)
       .between(startTime != null, User::getCreateTime, startTime, endTime)
       .orderByDesc(User::getCreateTime);
```

### 分页查询
```java
// 配置分页插件（见SKILL.md）

// 分页查询
Page<User> page = new Page<>(current, size);
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.like(User::getUserName, keyword);
IPage<User> result = userRepository.selectPage(page, wrapper);

// 返回分页结果
List<User> records = result.getRecords();
long total = result.getTotal();
```

### 逻辑删除
```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
```

### 自动填充
```java
@Component
public class MetaObjectHandlerImpl implements MetaObjectHandler {
    
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "createBy", Long.class, getCurrentUserId());
    }
    
    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
        this.strictUpdateFill(metaObject, "updateBy", Long.class, getCurrentUserId());
    }
}
```

## Spring 事务详解

### 事务传播行为
| 传播行为 | 说明 |
|----------|------|
| REQUIRED | 默认，存在事务则加入，否则创建新事务 |
| REQUIRES_NEW | 创建新事务，挂起当前事务 |
| SUPPORTS | 存在事务则加入，否则非事务执行 |
| MANDATORY | 必须在事务中执行，否则抛异常 |
| NOT_SUPPORTED | 非事务执行，挂起当前事务 |
| NEVER | 非事务执行，存在事务则抛异常 |
| NESTED | 嵌套事务（Savepoint） |

### 事务隔离级别
| 隔离级别 | 说明 | 问题 |
|----------|------|------|
| DEFAULT | 使用数据库默认 | - |
| READ_UNCOMMITTED | 读未提交 | 脏读、不可重复读、幻读 |
| READ_COMMITTED | 读已提交 | 不可重复读、幻读 |
| REPEATABLE_READ | 可重复读 | 幻读 |
| SERIALIZABLE | 串行化 | 性能低 |

### 事务超时设置
```java
@Transactional(timeout = 30)  // 30秒超时
public void someOperation() {
    // 操作
}
```

## Redis 在 Spring Boot 中的应用

### Redis 配置
```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: ${REDIS_PASSWORD:}
      database: 0
      timeout: 5000ms
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 2
```

### RedisTemplate 使用
```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;

// 存储
redisTemplate.opsForValue().set("user:1", user, 30, TimeUnit.MINUTES);

// 获取
User user = (User) redisTemplate.opsForValue().get("user:1");

// 删除
redisTemplate.delete("user:1");

// Hash操作
redisTemplate.opsForHash().put("user:hash", "name", "张三");

// 设置过期
redisTemplate.expire("user:1", 30, TimeUnit.MINUTES);
```

### 分布式锁
```java
public boolean tryLock(String key, String requestId, long expireTime) {
    Boolean result = redisTemplate.opsForValue()
            .setIfAbsent(key, requestId, expireTime, TimeUnit.SECONDS);
    return Boolean.TRUE.equals(result);
}

public void unlock(String key, String requestId) {
    String currentValue = (String) redisTemplate.opsForValue().get(key);
    if (requestId.equals(currentValue)) {
        redisTemplate.delete(key);
    }
}
```

## 常用工具类

### BeanUtils
```java
// 对象属性复制
UserDTO dto = new UserDTO();
BeanUtils.copyProperties(user, dto);

// 深拷贝
BeanUtils.copyProperties(source, target, new ConstantAwareConverter());
```

### ObjectUtils
```java
// 判断非空
ObjectUtils.isNotEmpty(list)
ObjectUtils.isNotEmpty(str)

// 转换为字符串
ObjectUtils.toString(obj, "default")
```

### CollUtil
```java
// 判断非空
CollUtil.isNotEmpty(list)

// 集合操作
CollUtil.newArrayList(args);
CollUtil.union(list1, list2);
CollUtil.subtract(list1, list2);
```

### StrUtil
```java
// 判断非空
StrUtil.isNotBlank(str);

// 字符串操作
StrUtil.format("user:{}", id);
StrUtil.join(list, ",");
StrUtil.substring(str, 0, 10);
```

## 性能优化建议

### 数据库优化
1. 使用索引优化查询
2. 批量操作减少数据库交互
3. 分页查询避免全表扫描
4. 使用连接池管理连接

### 缓存优化
1. 热点数据放入Redis
2. 合理设置缓存过期时间
3. 缓存穿透、击穿、雪崩防护

### 代码优化
1. 避免N+1查询问题
2. 异步处理耗时操作
3. 使用线程池并发处理
4. 减少不必要的对象创建

## 推荐阅读

### 书籍
- 《Spring Boot实战》— Craig Walls
- 《Spring实战》— Craig Walls
- 《高性能MySQL》— Baron Schwartz

### 在线资源
- [Spring Boot官方文档](https://spring.io/projects/spring-boot)
- [MyBatis Plus文档](https://baomidou.com/pages/24112f/)
- [Redis中文文档](http://www.redis.cn/)

---

**最后更新**: 2024年
**维护者**: 萝卜飞
**版本**: v1.0
