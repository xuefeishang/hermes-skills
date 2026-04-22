---
name: Spring Boot后端开发
description: "当使用Java/Spring Boot进行后端开发、创建REST API、实现业务逻辑、处理数据库操作时，使用本技能确保开发规范和代码质量。"
license: MIT
---

# Spring Boot后端开发实战技能

## 概述

Spring Boot是企业级Java开发的主流框架。本技能涵盖从项目初始化到生产部署的完整流程。

**核心原则**: 好的Spring Boot代码应该分层清晰、事务安全、日志完善、异常处理规范。坏的代码会导致内存泄漏、事务问题、排查困难。

## 何时使用

**始终：**
- 创建新的Spring Boot项目
- 开发REST API接口
- 实现业务逻辑层
- 配置数据源和事务
- 部署应用到生产环境

**触发短语：**
- "Spring Boot"
- "Java后端"
- "REST接口"
- "业务逻辑"
- "增删改查"

## 项目结构规范

### 标准分层结构
```
com.example.project
├── config/              # 配置类
│   ├── WebConfig.java
│   ├── RedisConfig.java
│   └── SecurityConfig.java
├── controller/           # 控制器层
│   ├── UserController.java
│   └── OrderController.java
├── service/              # 服务层
│   ├── UserService.java
│   ├── impl/
│   │   └── UserServiceImpl.java
├── repository/           # 数据访问层
│   ├── UserRepository.java
│   └── OrderRepository.java
├── entity/               # 实体类
│   ├── User.java
│   └── Order.java
├── dto/                  # 数据传输对象
│   ├── UserDTO.java
│   └── OrderDTO.java
├── vo/                   # 视图对象
│   └── ResultVO.java
├── common/               # 通用类
│   ├── constants/
│   ├── enums/
│   ├── exception/
│   └── util/
└── ProjectApplication.java
```

### Maven pom.xml 模板
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>project-name</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
        <mybatis-plus.version>3.5.5</mybatis-plus.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Boot Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- MySQL Driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- MyBatis Plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>

        <!-- Redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Spring Boot Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 实体类设计

### User 实体
```java
package com.example.project.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@TableName("t_sys_user")
public class User {
    
    @TableId(type = IdType.AUTO)
    private Long id;
    
    private String userNo;
    
    private String userName;
    
    private String email;
    
    private String phone;
    
    private String password;
    
    private Integer status;
    
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
    
    private Long createBy;
    
    private Long updateBy;
    
    @TableLogic
    private Integer deleted;
    
    @Version
    private Integer version;
}
```

### 枚举类定义
```java
package com.example.project.common.enums;

public enum UserStatus {
    DISABLED(0, "禁用"),
    ENABLED(1, "启用"),
    LOCKED(2, "锁定");
    
    private final int code;
    private final String desc;
    
    UserStatus(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }
    
    public int getCode() {
        return code;
    }
    
    public String getDesc() {
        return desc;
    }
}
```

## Repository层开发

### UserRepository
```java
package com.example.project.repository;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.project.entity.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import java.util.List;

@Mapper
public interface UserRepository extends BaseMapper<User> {
    
    // 根据用户名查询
    User selectByUserName(@Param("userName") String userName);
    
    // 根据邮箱查询
    User selectByEmail(@Param("email") String email);
    
    // 条件查询用户列表
    List<User> selectUserList(@Param("userName") String userName, 
                               @Param("status") Integer status);
    
    // 统计部门用户数
    int countByDeptId(@Param("deptId") Long deptId);
}
```

### XML映射文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.project.repository.UserRepository">

    <select id="selectByUserName" resultType="User">
        SELECT * FROM t_sys_user 
        WHERE f_user_name = #{userName} AND f_deleted = 0
    </select>

    <select id="selectUserList" resultType="User">
        SELECT * FROM t_sys_user
        WHERE f_deleted = 0
        <if test="userName != null and userName != ''">
            AND f_user_name LIKE CONCAT('%', #{userName}, '%')
        </if>
        <if test="status != null">
            AND f_status = #{status}
        </if>
        ORDER BY f_create_time DESC
    </select>

</mapper>
```

## Service层开发

### Service接口
```java
package com.example.project.service;

import com.example.project.common.Result;
import com.example.project.dto.UserDTO;
import com.example.project.entity.User;
import java.util.List;

public interface UserService {
    
    Result<User> getUserById(Long id);
    
    Result<List<User>> getUserList(String userName, Integer status);
    
    Result<Long> createUser(UserDTO dto);
    
    Result<Void> updateUser(Long id, UserDTO dto);
    
    Result<Void> deleteUser(Long id);
    
    Result<Void> updatePassword(Long id, String oldPassword, String newPassword);
}
```

### Service实现
```java
package com.example.project.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.example.project.common.enums.UserStatus;
import com.example.project.common.exception.BusinessException;
import com.example.project.common.util.SecurityUtils;
import com.example.project.dto.UserDTO;
import com.example.project.entity.User;
import com.example.project.repository.UserRepository;
import com.example.project.service.UserService;
import lombok.RequiredArgsConstructor;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    @Override
    public Result<User> getUserById(Long id) {
        User user = userRepository.selectById(id);
        if (user == null) {
            throw new BusinessException("用户不存在");
        }
        return Result.success(user);
    }
    
    @Override
    public Result<List<User>> getUserList(String userName, Integer status) {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.like(userName != null, User::getUserName, userName)
               .eq(status != null, User::getStatus, status)
               .orderByDesc(User::getCreateTime);
        List<User> list = userRepository.selectList(wrapper);
        return Result.success(list);
    }
    
    @Override
    @Transactional(rollbackFor = Exception.class)
    public Result<Long> createUser(UserDTO dto) {
        // 校验用户名唯一
        User existUser = userRepository.selectByUserName(dto.getUserName());
        if (existUser != null) {
            throw new BusinessException("用户名已存在");
        }
        
        // 校验邮箱唯一
        User existEmail = userRepository.selectByEmail(dto.getEmail());
        if (existEmail != null) {
            throw new BusinessException("邮箱已被使用");
        }
        
        // 创建用户
        User user = new User();
        user.setUserName(dto.getUserName());
        user.setEmail(dto.getEmail());
        user.setPhone(dto.getPhone());
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user.setStatus(UserStatus.ENABLED.getCode());
        user.setCreateBy(SecurityUtils.getCurrentUserId());
        
        userRepository.insert(user);
        return Result.success(user.getId());
    }
    
    @Override
    @Transactional(rollbackFor = Exception.class)
    public Result<Void> updateUser(Long id, UserDTO dto) {
        User user = userRepository.selectById(id);
        if (user == null) {
            throw new BusinessException("用户不存在");
        }
        
        user.setUserName(dto.getUserName());
        user.setEmail(dto.getEmail());
        user.setPhone(dto.getPhone());
        user.setUpdateBy(SecurityUtils.getCurrentUserId());
        
        userRepository.updateById(user);
        return Result.success();
    }
    
    @Override
    @Transactional(rollbackFor = Exception.class)
    public Result<Void> deleteUser(Long id) {
        User user = userRepository.selectById(id);
        if (user == null) {
            throw new BusinessException("用户不存在");
        }
        
        // 逻辑删除
        userRepository.deleteById(id);
        return Result.success();
    }
}
```

## Controller层开发

### 统一响应封装
```java
package com.example.project.vo;

import lombok.Data;
import java.time.LocalDateTime;

@Data
public class ResultVO<T> {
    
    private int code;
    private String message;
    private T data;
    private LocalDateTime timestamp;
    
    public static <T> ResultVO<T> success(T data) {
        ResultVO<T> result = new ResultVO<>();
        result.setCode(200);
        result.setMessage("操作成功");
        result.setData(data);
        result.setTimestamp(LocalDateTime.now());
        return result;
    }
    
    public static <T> ResultVO<T> error(String message) {
        ResultVO<T> result = new ResultVO<>();
        result.setCode(500);
        result.setMessage(message);
        result.setTimestamp(LocalDateTime.now());
        return result;
    }
    
    public static <T> ResultVO<T> error(int code, String message) {
        ResultVO<T> result = new ResultVO<>();
        result.setCode(code);
        result.setMessage(message);
        result.setTimestamp(LocalDateTime.now());
        return result;
    }
}
```

### UserController
```java
package com.example.project.controller;

import com.example.project.common.annotations.OperationLog;
import com.example.project.common.enums.OperationType;
import com.example.project.common.util.PageUtils;
import com.example.project.common.util.Result;
import com.example.project.dto.UserDTO;
import com.example.project.entity.User;
import com.example.project.service.UserService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {
    
    private final UserService userService;
    
    @GetMapping("/{id}")
    @OperationLog(title = "用户管理", action = OperationType.QUERY)
    public Result<User> getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }
    
    @GetMapping
    @OperationLog(title = "用户管理", action = OperationType.QUERY)
    public Result<List<User>> getUserList(
            @RequestParam(required = false) String userName,
            @RequestParam(required = false) Integer status) {
        return userService.getUserList(userName, status);
    }
    
    @PostMapping
    @OperationLog(title = "用户管理", action = OperationType.CREATE)
    public Result<Long> createUser(@Valid @RequestBody UserDTO dto) {
        return userService.createUser(dto);
    }
    
    @PutMapping("/{id}")
    @OperationLog(title = "用户管理", action = OperationType.UPDATE)
    public Result<Void> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UserDTO dto) {
        return userService.updateUser(id, dto);
    }
    
    @DeleteMapping("/{id}")
    @OperationLog(title = "用户管理", action = OperationType.DELETE)
    public Result<Void> deleteUser(@PathVariable Long id) {
        return userService.deleteUser(id);
    }
}
```

## 统一异常处理

### 全局异常处理器
```java
package com.example.project.common.exception;

import com.example.project.vo.ResultVO;
import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.BindException;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(BusinessException.class)
    public ResultVO<Void> handleBusinessException(BusinessException e) {
        log.warn("业务异常: {}", e.getMessage());
        return ResultVO.error(400, e.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResultVO<Void> handleValidationException(MethodArgumentNotValidException e) {
        StringBuilder message = new StringBuilder();
        for (FieldError error : e.getBindingResult().getFieldErrors()) {
            message.append(error.getField())
                   .append(": ")
                   .append(error.getDefaultMessage())
                   .append("; ");
        }
        log.warn("参数验证异常: {}", message);
        return ResultVO.error(400, message.toString());
    }
    
    @ExceptionHandler(Exception.class)
    public ResultVO<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return ResultVO.error(500, "系统异常，请稍后重试");
    }
}
```

### 自定义业务异常
```java
package com.example.project.common.exception;

public class BusinessException extends RuntimeException {
    
    private static final long serialVersionUID = 1L;
    
    private int code = 400;
    
    public BusinessException(String message) {
        super(message);
    }
    
    public BusinessException(int code, String message) {
        super(message);
        this.code = code;
    }
    
    public int getCode() {
        return code;
    }
}
```

## 配置类

### 分页插件配置
```java
package com.example.project.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyBatisConfig {
    
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

### 统一响应包装配置
```java
package com.example.project.config;

import com.example.project.common.interceptor.ResultInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new ResultInterceptor())
                .addPathPatterns("/api/**");
    }
}
```

## application.yml 配置

```yaml
server:
  port: 8080
  servlet:
    context-path: /

spring:
  application:
    name: project-name
  
  datasource:
    url: jdbc:mysql://localhost:3306/project_db?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai&useSSL=false
    username: root
    password: ${DB_PASSWORD:your_password}
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      minimum-idle: 5
      maximum-pool-size: 20
      idle-timeout: 30000
      connection-timeout: 30000
      max-lifetime: 1800000
  
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

mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*.xml
  type-aliases-package: com.example.project.entity
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0

logging:
  level:
    com.example.project: DEBUG
    org.springframework: INFO
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

## 最佳实践清单

### 代码规范
- [ ] 分层清晰（Controller/Service/Repository）
- [ ] 使用Lombok简化代码
- [ ] 统一异常处理
- [ ] 统一响应格式
- [ ] 参数校验使用@Valid
- [ ] 操作日志记录

### 事务处理
- [ ] 增删改使用@Transactional
- [ ] 设置rollbackFor
- [ ] 避免长事务
- [ ] 读写分离分离事务

### 安全考虑
- [ ] 密码加密存储
- [ ] SQL注入防护
- [ ] XSS防护
- [ ] 敏感数据脱敏

## 相关资源

- [RESTful API设计](../architecture/api-design/) — 接口设计
- [数据库设计](../architecture/database-design/) — 数据模型
- [单元测试设计](../testing/unit-testing/) — 测试编写
- [Spring Boot官方文档](https://spring.io/projects/spring-boot)
