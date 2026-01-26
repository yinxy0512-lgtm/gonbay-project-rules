# gonbay-redisson 组件使用规则

> 强制性约束 · 组件使用规范 · 全局适用

---

## 1. 规则目的

本规则定义了 gonbay-redisson 组件的使用规范。gonbay-redisson 是基于 redisson 的封装，提供一系列缓存实现、分布式锁、缓存注解等功能。AI 编码助手在使用 Redis 相关功能时，必须严格遵守本规则，优先使用 gonbay-redisson 组件提供的功能。

---

## 2. 组件介绍

### 2.1 组件特性
- ✅ 所有 Redis 相关操作都要使用本规范中的 API
- ✅ 基于 Spring SPI 实现，Maven 引入后自动装配
- ✅ 当配置环境没有 redisson 相关配置时，不会自动装配到 Spring 上下文中
- ✅ 提供一系列对 Redis 操作的 API
- ✅ 提供缓存注解 `@Cacheable`
- ✅ 提供表单重复提交注解 `@NoResubmit`

### 2.2 必须遵守的规则

**必须使用**
- ✅ 所有 Redis 操作必须使用 `FastRedisClient` 或 `RedissonClientContext`
- ✅ 缓存功能必须使用 `@Cacheable` 注解
- ✅ 表单重复提交防护必须使用 `@NoResubmit` 注解
- ✅ 分布式锁必须使用组件提供的锁功能

**禁止行为**
- ❌ 禁止直接使用 Redisson 原生 API（除非 `FastRedisClient` 不满足需求）
- ❌ 禁止使用其他 Redis 客户端（如 Jedis、Lettuce）
- ❌ 禁止绕过组件封装直接操作 Redis
- ❌ 禁止在业务代码中直接创建 Redis 连接

---

## 3. Maven 依赖配置

### 3.1 依赖引入

```xml
<dependency>
    <groupId>net.gonbay</groupId>
    <artifactId>gonbay-redisson</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

**规则**
- ✅ 必须在 `gonbay-{project}-service` 模块中引入
- ✅ 版本号使用项目统一版本管理
- ❌ 禁止使用其他版本的 redisson 依赖

---

## 4. 配置规范

### 4.1 配置结构

组件支持多种 Redis 部署模式，包括单节点、集群、主从、复制集群等。

### 4.2 单节点配置

```yaml
spring:
  redisson:
    configs:
      - key-group: "TEST"
        defaulted: true
        default-expire-seconds: 1000
        config: |
          singleServerConfig:
            address: "redis://127.0.0.1:6379"
            database: 0
            username: 
            password:
```

### 4.3 集群模式配置

```yaml
spring:
  redisson:
    configs:
      - key-group: "TEST"
        defaulted: true
        default-expire-seconds: 1000
        config: |
          clusterServersConfig:
            nodeAddresses: 
              - "redis://127.0.0.1:6379"
              - "redis://127.0.0.1:6379"
              - "redis://127.0.0.1:6379"
```

### 4.4 配置规则

**必须遵守**
- ✅ `spring.redisson.configs.config` 必须是 YAML 格式的字符串（使用 `|` 标记）
- ✅ 必须通过 `key-group` 区分不同的缓存环境
- ✅ 必须指定一个 `defaulted: true` 的环境作为默认环境
- ✅ `default-expire-seconds` 必须设置合理的默认失效时间（不指定默认 1200 秒）

**禁止行为**
- ❌ 禁止在配置中硬编码 Redis 地址
- ❌ 禁止使用不安全的 Redis 连接（生产环境必须设置密码）

---

## 5. 使用方式规范

### 5.1 FastRedisClient 使用

#### 5.1.1 注入方式

```java
import net.gonbay.redisson.core.FastRedisClient;

@Autowired
private FastRedisClient fastRedisClient;
```

**规则**
- ✅ 必须使用 `@Autowired` 注入，使用默认环境
- ✅ 如需指定环境，使用 `RedissonClientContext.client("key-group")`

#### 5.1.2 常用操作

```java
// 插入缓存，不指定有效期默认使用 defaultExpireSeconds
fastRedisClient.set("key", "value");

// 指定有效期插入缓存
fastRedisClient.set("key", "value", Duration.ofSeconds(100));

// 获取缓存
String value = fastRedisClient.get("key");

// 设置有效期
fastRedisClient.expire("key", Duration.ofSeconds(100));
```

**规则**
- ✅ 所有缓存操作必须指定合理的过期时间
- ✅ 缓存 key 必须使用有意义的命名规范
- ❌ 禁止使用过长的过期时间（建议不超过 7 天）

#### 5.1.3 链表结构操作

```java
// 向链表插入值
fastRedisClient.ladd("key", "value1");
fastRedisClient.ladd("key", "value2", Duration.ofMinutes(1));

// 批量插入
fastRedisClient.laddAll("key", Arrays.asList("v1", "v2"));

// 获取链表元素
String element = fastRedisClient.lget("key", 0);
int size = fastRedisClient.llen("key");

// 栈操作
String top = fastRedisClient.lpop("key");
fastRedisClient.lpush("key", "value");
fastRedisClient.rpush("key", "value");
String bottom = fastRedisClient.rpop("key");
```

#### 5.1.4 Set 结构操作

```java
// 插入值
fastRedisClient.sadd("key", "value");
fastRedisClient.sadd("key", "value", Duration.ofSeconds(10));

// 批量插入
fastRedisClient.saddAll("key", Set.of("value2", "value3"));

// 获取所有元素
Set<String> members = fastRedisClient.smembers("key");

// 随机删除
String value = fastRedisClient.sremoveRandom("key");
Set<String> values = fastRedisClient.sremoveRandom("key", 3);
```

#### 5.1.5 Hash 结构操作

```java
// 插入键值对
fastRedisClient.hset("key", "field", "value");

// 批量插入
fastRedisClient.hsetAll("key", Map.of("f1", "v1", "f2", "v2"));

// 删除 field
fastRedisClient.hdel("key", "f1");

// 获取值
String value = fastRedisClient.hget("key", "f1");
Map<String, String> all = fastRedisClient.hgetAll("key");
Set<String> fields = fastRedisClient.hkeys("key");
int size = fastRedisClient.hlen("key");
```

### 5.2 分布式锁使用

#### 5.2.1 重入锁

```java
RLock lock = fastRedisClient.getLock("biz_no");
try {
    lock.lock(30, TimeUnit.SECONDS);
    // 业务逻辑
} finally {
    lock.unlock();
}
```

**规则**
- ✅ 必须在 `finally` 块中释放锁
- ✅ 必须设置合理的锁超时时间
- ❌ 禁止在业务逻辑中捕获异常后不释放锁

#### 5.2.2 公平锁

```java
RLock lock = fastRedisClient.getFairLock("biz_no");
try {
    lock.lock(30, TimeUnit.SECONDS);
    // 业务逻辑
} finally {
    lock.unlock();
}
```

#### 5.2.3 读写锁

```java
ReadWriteLock lock = fastRedisClient.getReadWriteLock("biz_no");
Lock readLock = lock.readLock();
Lock writeLock = lock.writeLock();
```

**规则**
- ✅ 读写锁适用于多读少写场景
- ✅ 读锁可以并发，写锁互斥
- ❌ 禁止在持有读锁时尝试获取写锁

### 5.3 Redisson 原生操作

```java

// 获取指定环境的 RedissonClient
RedissonClient client = redissonClientContext.client("TEST");

// 获取默认环境的 RedissonClient
RedissonClient defaultClient = redissonClientContext.client();

// 使用原生 API（仅在 FastRedisClient 不满足需求时使用）
defaultClient.getBucket("key", JsonJacksonCodec.INSTANCE).set(user);
```

**规则**
- ✅ 仅在 `FastRedisClient` 不满足需求时使用原生 API
- ✅ 使用原生 API 时必须指定合适的编解码器
- ❌ 禁止在业务代码中直接创建 `RedissonClient` 实例

---

## 6. 缓存注解 @Cacheable

### 6.1 配置

```yaml
spring:
  cacheable:
    cache-type: REDIS  # 指定缓存实现为 LOCAL 或 REDIS
    key-group: "TEST"  # 指定 redis key-group
    expire-time: 1000  # 失效时间: 默认秒
    limit-count: 512   # 缓存数量: cacheType=REDIS 时失效
```

### 6.2 使用方式

```java
/**
 * 增加业务缓存
 * key: 为缓存 key，支持动态脚本解析
 * putOrDelete: true: 加缓存  false: 清除缓存
 * isGroupMode: 是否为分组模式, true: 存储结构为 hash  false: 存储为 k-v
 * group: 当 isGroupMode=true 时生效，作为 hash 的 key
 */
@Cacheable(putOrDelete = true, key = "$user:get:${args[0]}")
public ShopUser getUserById(Long id) {
    return get(id);
}
```

### 6.3 规则

**必须遵守**
- ✅ 缓存 key 必须使用有意义的命名，支持动态脚本解析
- ✅ `putOrDelete` 在业务层必须成对出现（get 方法加缓存，update 方法清除缓存）
- ✅ 成对操作的 key 必须保持一致
- ✅ 当 `cache-type=LOCAL` 时，`limit-count` 生效，遵循 LRU 淘汰策略
- ✅ 当 `cache-type=REDIS` 时，`key-group` 必须指定 redisson 上下文中的环境

**禁止行为**
- ❌ 禁止缓存 key 使用随机值或时间戳
- ❌ 禁止只加缓存不清除缓存（导致数据不一致）
- ❌ 禁止在事务方法中使用缓存注解（可能导致数据不一致）

---

## 7. 表单重复提交 @NoResubmit

### 7.1 配置

```yaml
spring:
  no-resubmit:
    key-group: "TEST"
    expire-time: 1000
    limit-count: 512
    cache-type: REDIS
```

### 7.2 使用方式

```java
@NoResubmit(name = "R_T")
public void saveUser(ShopUser shopUser) {
    shopUserService.save(shopUser);
}
```

### 7.3 规则

**必须遵守**
- ✅ `@NoResubmit` 注解必须单独进行配置
- ✅ `name` 属性对应 Request Header 的 key，前端必须传入唯一身份信息
- ✅ 配置参数与 `@Cacheable` 一致
- ✅ 失效时间必须设置足够大，避免在用户会话期间失效

**禁止行为**
- ❌ 禁止在 GET 请求上使用 `@NoResubmit`
- ❌ 禁止不配置 `name` 属性
- ❌ 禁止设置过短的失效时间

### 7.4 实现逻辑说明

1. 前端在准备提交表单时，生成唯一的身份信息，由 header 携带到服务端
2. 服务端从缓存中判断是否提交过，检测到提交过则直接抛异常
3. 如果没有提交过，会借助缓存组件缓存这次提交的身份信息

**注意**：表单重复提交主要控制点击请求时穿透前端限制的情况，如果等待身份信息失效后再次提交，可以将失效时间设置得足够大。

---

## 8. 使用场景

### 8.1 适用场景

- ✅ 缓存业务数据，减少数据库访问
- ✅ 分布式锁控制并发访问
- ✅ 表单重复提交防护
- ✅ 计数器、排行榜等 Redis 数据结构操作
- ✅ 会话存储、临时数据存储

### 8.2 禁止场景

- ❌ 不适合存储大量数据（应使用数据库）
- ❌ 不适合存储敏感信息（除非加密）
- ❌ 不适合作为唯一数据源（必须有持久化备份）

---

## 9. 违规处理

如果发现违反本规则的情况：
1. 立即停止当前操作
2. 提示用户规则冲突
3. 等待用户明确指示

---

## 10. 规则优先级

本规则优先级：**高**
- 与其他规则冲突时，以本规则为准
- 除非用户明确要求，否则不得违反本规则
