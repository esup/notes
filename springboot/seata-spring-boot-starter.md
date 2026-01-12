# Seata Spring Boot Starter 整合实战

## 目录
- [什么是 Seata](#什么是-seata)
- [分布式事务场景](#分布式事务场景)
- [快速开始](#快速开始)
- [Maven 依赖配置](#maven-依赖配置)
- [配置文件](#配置文件)
- [代码示例](#代码示例)
- [三种事务模式](#三种事务模式)
- [常见问题](#常见问题)
- [最佳实践](#最佳实践)

---

## 什么是 Seata

Seata（Simple Extensible Autonomous Transaction Architecture）是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。

### 核心特性
- **高性能**：低侵入性，性能损耗小
- **易用性**：支持 Spring Cloud、Dubbo 等主流微服务框架
- **多种模式**：支持 AT、TCC、SAGA、XA 四种事务模式
- **零侵入**：AT 模式下业务代码无需修改

### 核心组件
- **TC（Transaction Coordinator）**：事务协调器，维护全局事务和分支事务的状态，驱动全局事务提交或回滚
- **TM（Transaction Manager）**：事务管理器，定义全局事务的范围，开始、提交或回滚全局事务
- **RM（Resource Manager）**：资源管理器，管理分支事务处理的资源，与 TC 交互注册分支事务和报告分支事务状态

---

## 分布式事务场景

### 典型应用场景

**电商订单场景**
```
用户下单 → 订单服务创建订单 → 库存服务扣减库存 → 积分服务增加积分
```

如果任何一个环节失败，整个事务都需要回滚。

**转账场景**
```
用户 A 转账给用户 B → A 账户扣款 → B 账户加款
```

两个操作必须同时成功或同时失败。

---

## 快速开始

### 前置条件

1. **JDK 1.8+**
2. **Spring Boot 2.x 或 3.x**
3. **MySQL 5.7+**（或其他支持的数据库）
4. **Seata Server 1.5.0+**

### 安装 Seata Server

#### 方式一：下载安装包

1. 从 [Seata Releases](https://github.com/seata/seata/releases) 下载最新版本
2. 解压到本地目录
3. 修改配置文件 `conf/application.yml`
4. 启动服务：
   ```bash
   # Linux/macOS
   sh bin/seata-server.sh
   
   # Windows
   bin\seata-server.bat
   ```

#### 方式二：使用 Docker

```bash
docker run -d --name seata-server \
  -p 8091:8091 \
  -p 7091:7091 \
  seataio/seata-server:latest
```

---

## Maven 依赖配置

### Spring Boot 2.x

```xml
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Seata Spring Boot Starter -->
    <dependency>
        <groupId>io.seata</groupId>
        <artifactId>seata-spring-boot-starter</artifactId>
        <version>1.7.1</version>
    </dependency>

    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>

    <!-- MyBatis Plus（可选） -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.3.1</version>
    </dependency>

    <!-- Druid 数据源（推荐） -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.2.18</version>
    </dependency>
</dependencies>
```

### Spring Boot 3.x

```xml
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Seata Spring Boot Starter -->
    <dependency>
        <groupId>io.seata</groupId>
        <artifactId>seata-spring-boot-starter</artifactId>
        <version>2.0.0</version>
    </dependency>

    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.0.33</version>
    </dependency>

    <!-- MyBatis Plus -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.5</version>
    </dependency>
</dependencies>
```

---

## 配置文件

### application.yml

```yaml
spring:
  application:
    name: order-service  # 服务名称
  
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata_order?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource

# Seata 配置
seata:
  enabled: true
  application-id: ${spring.application.name}
  tx-service-group: default_tx_group  # 事务分组名称
  
  # 注册中心配置
  registry:
    type: file  # 可选：file、nacos、eureka、redis、zk、consul、etcd3、sofa
    file:
      name: file.conf
  
  # 配置中心配置
  config:
    type: file  # 可选：file、nacos、apollo、zk、consul、etcd3
    file:
      name: file.conf
  
  # 服务端配置
  service:
    vgroup-mapping:
      default_tx_group: default  # 事务分组与集群的映射关系
    grouplist:
      default: 127.0.0.1:8091  # Seata Server 地址
  
  # 客户端配置
  client:
    rm:
      report-success-enable: false
      table-meta-check-enable: false
      report-retry-count: 5
      async-commit-buffer-limit: 10000
      lock:
        retry-interval: 10
        retry-times: 30
        retry-policy-branch-rollback-on-conflict: true
    tm:
      commit-retry-count: 5
      rollback-retry-count: 5
      degrade-check: false
      degrade-check-allow-times: 10
      degrade-check-period: 2000
    undo:
      data-validation: true
      log-serialization: jackson
      log-table: undo_log
      only-care-update-columns: true

# MyBatis Plus 配置（可选）
mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true
```

### 使用 Nacos 作为注册中心和配置中心

```yaml
seata:
  enabled: true
  application-id: ${spring.application.name}
  tx-service-group: default_tx_group
  
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      namespace: ""
      group: SEATA_GROUP
      username: nacos
      password: nacos
  
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace: ""
      group: SEATA_GROUP
      username: nacos
      password: nacos
      data-id: seataServer.properties
```

---

## 代码示例

### 1. 创建数据库表

#### undo_log 表（AT 模式必需）

```sql
-- 在每个业务数据库中创建
CREATE TABLE IF NOT EXISTS `undo_log`
(
    `branch_id`     BIGINT       NOT NULL COMMENT 'branch transaction id',
    `xid`           VARCHAR(128) NOT NULL COMMENT 'global transaction id',
    `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
    `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
    `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
    `log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
    `log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`),
    KEY `idx_log_created` (`log_created`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4 COMMENT ='AT transaction mode undo table';
```

#### 业务表示例

```sql
-- 订单服务数据库
CREATE DATABASE seata_order;
USE seata_order;

CREATE TABLE `t_order` (
    `id` BIGINT NOT NULL AUTO_INCREMENT,
    `user_id` BIGINT NOT NULL COMMENT '用户ID',
    `product_id` BIGINT NOT NULL COMMENT '产品ID',
    `count` INT NOT NULL COMMENT '数量',
    `money` DECIMAL(10,2) NOT NULL COMMENT '金额',
    `status` INT DEFAULT 0 COMMENT '订单状态：0创建中，1已完成',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 库存服务数据库
CREATE DATABASE seata_storage;
USE seata_storage;

CREATE TABLE `t_storage` (
    `id` BIGINT NOT NULL AUTO_INCREMENT,
    `product_id` BIGINT NOT NULL COMMENT '产品ID',
    `total` INT NOT NULL COMMENT '总库存',
    `used` INT DEFAULT 0 COMMENT '已用库存',
    `residue` INT NOT NULL COMMENT '剩余库存',
    PRIMARY KEY (`id`),
    UNIQUE KEY `idx_product_id` (`product_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 账户服务数据库
CREATE DATABASE seata_account;
USE seata_account;

CREATE TABLE `t_account` (
    `id` BIGINT NOT NULL AUTO_INCREMENT,
    `user_id` BIGINT NOT NULL COMMENT '用户ID',
    `total` DECIMAL(10,2) NOT NULL COMMENT '总额度',
    `used` DECIMAL(10,2) DEFAULT 0 COMMENT '已用额度',
    `residue` DECIMAL(10,2) NOT NULL COMMENT '剩余额度',
    PRIMARY KEY (`id`),
    UNIQUE KEY `idx_user_id` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 2. 实体类

```java
package com.example.order.entity;

import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import lombok.Data;
import java.math.BigDecimal;

@Data
@TableName("t_order")
public class Order {
    @TableId(type = IdType.AUTO)
    private Long id;
    private Long userId;
    private Long productId;
    private Integer count;
    private BigDecimal money;
    private Integer status;
}
```

#### Storage 实体类

```java
package com.example.storage.entity;

import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import lombok.Data;

@Data
@TableName("t_storage")
public class Storage {
    @TableId(type = IdType.AUTO)
    private Long id;
    private Long productId;
    private Integer total;
    private Integer used;
    private Integer residue;
}
```

#### Account 实体类

```java
package com.example.account.entity;

import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import lombok.Data;
import java.math.BigDecimal;

@Data
@TableName("t_account")
public class Account {
    @TableId(type = IdType.AUTO)
    private Long id;
    private Long userId;
    private BigDecimal total;
    private BigDecimal used;
    private BigDecimal residue;
}
```

### 3. Mapper 接口

```java
package com.example.order.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.order.entity.Order;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface OrderMapper extends BaseMapper<Order> {
}
```

### 4. Service 实现

#### OrderService

```java
package com.example.order.service;

import com.example.order.entity.Order;
import com.example.order.mapper.OrderMapper;
import io.seata.spring.annotation.GlobalTransactional;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j
@Service
@RequiredArgsConstructor
public class OrderService {
    
    private final OrderMapper orderMapper;
    private final StorageService storageService;
    private final AccountService accountService;
    
    /**
     * 创建订单 -> 减库存 -> 扣余额 -> 修改订单状态
     * 使用 @GlobalTransactional 注解开启分布式事务
     */
    @GlobalTransactional(name = "create-order", rollbackFor = Exception.class)
    public void create(Order order) {
        log.info("开始创建订单");
        
        // 1. 创建订单
        order.setStatus(0);
        orderMapper.insert(order);
        log.info("订单创建成功，订单ID：{}", order.getId());
        
        // 2. 扣减库存
        log.info("开始扣减库存");
        storageService.decrease(order.getProductId(), order.getCount());
        log.info("库存扣减成功");
        
        // 3. 扣减账户余额
        log.info("开始扣减账户余额");
        accountService.decrease(order.getUserId(), order.getMoney());
        log.info("账户余额扣减成功");
        
        // 4. 修改订单状态
        log.info("开始修改订单状态");
        order.setStatus(1);
        orderMapper.updateById(order);
        log.info("订单状态修改成功");
        
        log.info("订单创建完成");
    }
}
```

#### StorageService（远程调用）

```java
package com.example.order.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(name = "storage-service", url = "http://localhost:8082")
public interface StorageService {
    
    @PostMapping("/storage/decrease")
    void decrease(@RequestParam("productId") Long productId, 
                  @RequestParam("count") Integer count);
}
```

#### AccountService（远程调用）

```java
package com.example.order.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import java.math.BigDecimal;

@FeignClient(name = "account-service", url = "http://localhost:8083")
public interface AccountService {
    
    @PostMapping("/account/decrease")
    void decrease(@RequestParam("userId") Long userId, 
                  @RequestParam("money") BigDecimal money);
}
```

### 5. 库存服务实现

```java
package com.example.storage.service;

import com.baomidou.mybatisplus.core.conditions.update.LambdaUpdateWrapper;
import com.example.storage.entity.Storage;
import com.example.storage.mapper.StorageMapper;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Slf4j
@Service
@RequiredArgsConstructor
public class StorageService {
    
    private final StorageMapper storageMapper;
    
    @Transactional(rollbackFor = Exception.class)
    public void decrease(Long productId, Integer count) {
        log.info("库存服务：开始扣减库存，产品ID：{}，数量：{}", productId, count);
        
        Storage storage = storageMapper.selectById(productId);
        if (storage == null) {
            throw new RuntimeException("产品不存在");
        }
        
        if (storage.getResidue() < count) {
            throw new RuntimeException("库存不足");
        }
        
        // 扣减库存
        LambdaUpdateWrapper<Storage> wrapper = new LambdaUpdateWrapper<>();
        wrapper.setSql("used = used + " + count)
               .setSql("residue = residue - " + count)
               .eq(Storage::getId, productId);
        
        int result = storageMapper.update(null, wrapper);
        if (result == 0) {
            throw new RuntimeException("库存扣减失败");
        }
        
        log.info("库存服务：库存扣减成功");
    }
}
```

### 6. 账户服务实现

```java
package com.example.account.service;

import com.baomidou.mybatisplus.core.conditions.update.LambdaUpdateWrapper;
import com.example.account.entity.Account;
import com.example.account.mapper.AccountMapper;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.math.BigDecimal;

@Slf4j
@Service
@RequiredArgsConstructor
public class AccountService {
    
    private final AccountMapper accountMapper;
    
    @Transactional(rollbackFor = Exception.class)
    public void decrease(Long userId, BigDecimal money) {
        log.info("账户服务：开始扣减账户余额，用户ID：{}，金额：{}", userId, money);
        
        Account account = accountMapper.selectById(userId);
        if (account == null) {
            throw new RuntimeException("账户不存在");
        }
        
        if (account.getResidue().compareTo(money) < 0) {
            throw new RuntimeException("账户余额不足");
        }
        
        // 扣减余额
        LambdaUpdateWrapper<Account> wrapper = new LambdaUpdateWrapper<>();
        wrapper.setSql("used = used + " + money)
               .setSql("residue = residue - " + money)
               .eq(Account::getId, userId);
        
        int result = accountMapper.update(null, wrapper);
        if (result == 0) {
            throw new RuntimeException("账户余额扣减失败");
        }
        
        log.info("账户服务：账户余额扣减成功");
    }
}
```

### 7. Controller

```java
package com.example.order.controller;

import com.example.order.entity.Order;
import com.example.order.service.OrderService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/order")
@RequiredArgsConstructor
public class OrderController {
    
    private final OrderService orderService;
    
    @PostMapping("/create")
    public String create(@RequestBody Order order) {
        try {
            orderService.create(order);
            return "订单创建成功";
        } catch (Exception e) {
            return "订单创建失败：" + e.getMessage();
        }
    }
}
```

---

## 三种事务模式

### 1. AT 模式（推荐）

**特点：**
- 无侵入性，业务代码几乎无需修改
- 基于支持本地 ACID 事务的关系型数据库
- 自动生成回滚日志
- 性能好，适用于大部分场景

**使用方式：**
```java
@GlobalTransactional
public void businessMethod() {
    // 业务代码
}
```

**适用场景：**
- 对性能要求较高
- 业务逻辑相对简单
- 使用关系型数据库

### 2. TCC 模式

**特点：**
- 需要业务代码实现 Try、Confirm、Cancel 三个方法
- 不依赖底层数据库事务支持
- 性能更高，但开发复杂度高

**使用方式：**
```java
import io.seata.rm.tcc.api.BusinessActionContext;
import io.seata.rm.tcc.api.BusinessActionContextParameter;
import io.seata.rm.tcc.api.LocalTCC;
import io.seata.rm.tcc.api.TwoPhaseBusinessAction;
import java.math.BigDecimal;

@LocalTCC
public interface AccountTccService {
    
    @TwoPhaseBusinessAction(name = "deduct", commitMethod = "commit", rollbackMethod = "rollback")
    boolean deduct(@BusinessActionContextParameter(paramName = "userId") Long userId,
                   @BusinessActionContextParameter(paramName = "money") BigDecimal money);
    
    boolean commit(BusinessActionContext context);
    
    boolean rollback(BusinessActionContext context);
}
```

**适用场景：**
- 对性能要求极高
- 需要精细控制事务过程
- 涉及非关系型数据库

### 3. SAGA 模式

**特点：**
- 长事务解决方案
- 每个服务提供正向和补偿操作
- 适用于业务流程长、时间跨度大的场景

**使用方式：**
通过状态机定义业务流程和补偿流程。

**适用场景：**
- 业务流程复杂，涉及多个服务
- 事务时间跨度大
- 对最终一致性要求高

---

## 常见问题

### 1. 连接不上 Seata Server

**问题：**
```
io.seata.common.exception.FrameworkException: can not register RM,err:can not connect to services-server
```

**解决方案：**
- 检查 Seata Server 是否启动
- 检查 `seata.service.grouplist.default` 配置的地址是否正确
- 检查网络连接和防火墙设置
- 查看 Seata Server 日志

### 2. 事务不回滚

**问题：**
分布式事务中某个服务抛出异常，但数据没有回滚。

**解决方案：**
- 确保 `@GlobalTransactional` 注解的 `rollbackFor` 包含了抛出的异常类型
- 检查异常是否被捕获但没有重新抛出
- 确保所有参与事务的服务都正确配置了 Seata
- 检查数据库是否创建了 `undo_log` 表

### 3. undo_log 表数据堆积

**问题：**
`undo_log` 表数据越来越多，占用大量空间。

**解决方案：**
Seata 会自动清理成功的事务日志，如果堆积可能是：
- 存在长时间未提交的事务
- 清理机制配置不当
- 可以手动清理已完成事务的日志：
  ```sql
  DELETE FROM undo_log WHERE log_status = 1 AND log_created < DATE_SUB(NOW(), INTERVAL 7 DAY);
  ```

### 4. 数据源代理问题

**问题：**
```
io.seata.common.exception.ShouldNeverHappenException: DataSourceProxy not found
```

**解决方案：**
确保使用的数据源被 Seata 代理，配置数据源代理：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.boot.context.properties.ConfigurationProperties;
import javax.sql.DataSource;
import com.alibaba.druid.pool.DruidDataSource;
import io.seata.rm.datasource.DataSourceProxy;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;

@Configuration
public class DataSourceProxyConfig {
    
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource dataSource() {
        return new DruidDataSource();
    }
    
    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }
    
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        return sqlSessionFactoryBean.getObject();
    }
}
```

### 5. 超时问题

**问题：**
分布式事务执行时间过长导致超时。

**解决方案：**
调整超时配置：

```yaml
seata:
  client:
    tm:
      commit-retry-count: 5
      rollback-retry-count: 5
      default-global-transaction-timeout: 60000  # 全局事务超时时间（毫秒）
    rm:
      report-retry-count: 5
      branch-report-timeout: 60000  # 分支事务超时时间（毫秒）
```

### 6. 跨数据库类型事务

**问题：**
需要在 MySQL 和 PostgreSQL 之间执行分布式事务。

**解决方案：**
AT 模式支持跨数据库类型，但需要注意：
- 确保所有数据库都创建了 `undo_log` 表
- 根据不同数据库类型调整 SQL 语法
- 测试兼容性

---

## 最佳实践

### 1. 事务边界控制

**原则：**
- 事务范围尽可能小
- 避免在事务中进行耗时操作（如外部 API 调用）
- 合理拆分大事务

**示例：**
```java
// 不推荐：事务中包含外部调用
@GlobalTransactional
public void badExample() {
    createOrder();
    callExternalAPI();  // 外部调用，可能很慢
    updateInventory();
}

// 推荐：外部调用放在事务外
public void goodExample() {
    try {
        createOrderWithTransaction();
    } finally {
        callExternalAPI();  // 异步或事务完成后调用
    }
}

@GlobalTransactional
private void createOrderWithTransaction() {
    createOrder();
    updateInventory();
}
```

### 2. 异常处理

**原则：**
- 明确指定需要回滚的异常类型
- 不要捕获异常后不抛出
- 使用自定义业务异常

**示例：**
```java
// 自定义业务异常类
public class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
    
    public BusinessException(String message, Throwable cause) {
        super(message, cause);
    }
}

@GlobalTransactional(rollbackFor = Exception.class)
public void createOrder(Order order) {
    try {
        orderMapper.insert(order);
        storageService.decrease(order.getProductId(), order.getCount());
        accountService.decrease(order.getUserId(), order.getMoney());
    } catch (BusinessException e) {
        log.error("业务异常", e);
        throw e;  // 必须重新抛出异常，否则不会回滚
    } catch (Exception e) {
        log.error("系统异常", e);
        throw new BusinessException("订单创建失败", e);
    }
}
```

### 3. 性能优化

**建议：**
- 使用数据库连接池（Druid、HikariCP）
- 开启批量提交
- 合理设置 undo_log 保留时间
- 使用异步提交模式（如果业务允许）

```yaml
seata:
  client:
    rm:
      async-commit-buffer-limit: 10000  # 异步提交缓冲区大小
      report-success-enable: false  # 关闭成功上报（提高性能）
```

### 4. 监控和日志

**建议：**
- 启用详细日志记录分布式事务执行过程
- 监控全局事务和分支事务的成功率
- 监控 `undo_log` 表大小
- 设置告警机制

```java
import io.seata.core.context.RootContext;
import io.seata.spring.annotation.GlobalTransactional;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@GlobalTransactional
public void createOrder(Order order) {
    String xid = RootContext.getXID();
    log.info("开始全局事务，XID：{}", xid);
    
    try {
        // 业务逻辑
        log.info("全局事务执行成功，XID：{}", xid);
    } catch (Exception e) {
        log.error("全局事务执行失败，XID：{}，错误：{}", xid, e.getMessage(), e);
        throw e;
    }
}
```

### 5. 幂等性设计

**原则：**
- 所有分支事务操作都应该是幂等的
- 使用唯一键防止重复执行
- 使用状态机控制业务流程

**示例：**
```java
@Transactional(rollbackFor = Exception.class)
public void decrease(Long productId, Integer count, String requestId) {
    // 检查是否已经执行过（幂等性）
    if (storageRecordMapper.exists(requestId)) {
        log.info("请求已处理，requestId：{}", requestId);
        return;
    }
    
    // 执行业务逻辑
    Storage storage = storageMapper.selectById(productId);
    storage.setResidue(storage.getResidue() - count);
    storageMapper.updateById(storage);
    
    // 记录请求
    StorageRecord record = new StorageRecord();
    record.setRequestId(requestId);
    record.setProductId(productId);
    record.setCount(count);
    storageRecordMapper.insert(record);
}
```

### 6. 高可用部署

**建议：**
- Seata Server 集群部署（至少 2 个节点）
- 使用注册中心（Nacos、Eureka）实现服务发现
- 使用配置中心统一管理配置
- 数据库主从架构

### 7. 降级策略

**场景：**
当 Seata Server 不可用时，需要有降级方案。

**方案：**
```java
@Service
public class OrderService {
    
    @Value("${seata.degraded:false}")
    private boolean degraded;
    
    public void createOrder(Order order) {
        if (degraded) {
            // 降级模式：使用本地事务或队列异步处理
            createOrderWithLocalTransaction(order);
        } else {
            // 正常模式：使用分布式事务
            createOrderWithGlobalTransaction(order);
        }
    }
    
    @GlobalTransactional
    private void createOrderWithGlobalTransaction(Order order) {
        // 分布式事务逻辑
    }
    
    @Transactional
    private void createOrderWithLocalTransaction(Order order) {
        // 本地事务逻辑或消息队列
    }
}
```

---

## 总结

Seata Spring Boot Starter 为微服务架构下的分布式事务提供了简单易用的解决方案。通过本文的实战指南，你应该能够：

1. ✅ 理解 Seata 的核心概念和工作原理
2. ✅ 快速集成 Seata 到 Spring Boot 项目
3. ✅ 掌握 AT、TCC、SAGA 三种事务模式的使用
4. ✅ 解决常见问题和性能优化
5. ✅ 遵循最佳实践构建高可用的分布式事务系统

### 下一步学习

- 深入学习 Seata 源码和实现原理
- 探索 TCC 和 SAGA 模式的高级用法
- 学习分布式事务在实际业务中的应用
- 研究分布式事务的性能调优

### 参考资源

- [Seata 官方文档](https://seata.io/zh-cn/)
- [Seata GitHub](https://github.com/seata/seata)
- [Spring Cloud Alibaba 文档](https://github.com/alibaba/spring-cloud-alibaba)

---

**最后更新：** 2024-01-12
