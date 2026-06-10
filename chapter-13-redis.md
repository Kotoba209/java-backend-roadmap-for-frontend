# 第十三章：Redis 缓存与常见应用

## 13.1 本章目标

Redis 是 Java 后端项目中非常常见的中间件。它不是替代 MySQL 的数据库，而是常用于缓存、验证码、登录状态、分布式锁、防重复提交等场景。

本章要解决：

```text
Redis 是什么？
Redis 和 MySQL 有什么区别？
为什么后端项目需要缓存？
Spring Boot 如何集成 Redis？
如何缓存用户信息？
如何保存验证码？
如何防止重复提交？
缓存穿透、击穿、雪崩是什么？
```

学完本章后，你应该能够：

- 理解 Redis 的作用。
- 掌握 Redis 常见数据类型。
- 在 Spring Boot 中使用 Redis。
- 使用 Redis 缓存登录用户。
- 使用 Redis 保存验证码。
- 使用 Redis 实现简单防重复提交。
- 理解常见缓存问题和解决思路。

## 13.2 Redis 是什么

Redis 是一个基于内存的键值数据库。

可以先这样理解：

```text
MySQL：主要负责长期保存业务数据。
Redis：主要负责快速读取临时数据或热点数据。
```

MySQL 中的数据保存在磁盘，适合持久化保存：

```text
用户表
订单表
商品表
角色表
菜单表
```

Redis 中的数据通常保存在内存，访问速度快，适合：

```text
验证码
Token
登录用户信息
热点配置
临时计数
防重复提交标记
```

前端类比并不完全准确，但可以把 Redis 暂时理解成后端的“高速缓存区”。

## 13.3 Redis 常见数据类型

常见数据类型：

| 类型 | 说明 | 常见场景 |
|---|---|---|
| String | 字符串 | Token、验证码、计数 |
| Hash | 哈希对象 | 用户信息、对象字段 |
| List | 列表 | 消息队列简单场景 |
| Set | 不重复集合 | 标签、去重 |
| ZSet | 有序集合 | 排行榜 |

最常用的是 String。

命令示例：

```bash
SET login:token:abc userId:1
GET login:token:abc
DEL login:token:abc
EXPIRE login:token:abc 3600
```

带过期时间：

```bash
SETEX captcha:abc 300 9527
```

表示验证码 300 秒后过期。

## 13.4 Spring Boot 集成 Redis

添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

配置：

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password:
      database: 0
```

使用 `StringRedisTemplate`：

```java
@Service
@RequiredArgsConstructor
public class RedisDemoService {

    private final StringRedisTemplate stringRedisTemplate;

    public void setValue() {
        stringRedisTemplate.opsForValue().set("name", "admin");
    }

    public String getValue() {
        return stringRedisTemplate.opsForValue().get("name");
    }
}
```

设置过期时间：

```java
stringRedisTemplate.opsForValue().set("captcha:abc", "9527", Duration.ofMinutes(5));
```

## 13.5 Redis Key 命名规范

Redis key 要有业务前缀。

推荐：

```text
login:token:{token}
captcha:{uuid}
user:info:{userId}
repeat:submit:{userId}:{uri}
```

不推荐：

```text
token
user
abc
```

好的 key 命名应该表达：

```text
业务模块
数据类型
唯一标识
```

示例：

```java
String key = "captcha:" + uuid;
```

这样排查 Redis 数据时更清楚。

## 13.6 缓存登录用户信息

登录成功后，可以把用户信息存入 Redis：

```java
String token = jwtUtil.generateToken(user.getId());
String key = "login:token:" + token;

stringRedisTemplate.opsForValue().set(
        key,
        String.valueOf(user.getId()),
        Duration.ofHours(2)
);
```

请求时：

```java
String key = "login:token:" + token;
String userId = stringRedisTemplate.opsForValue().get(key);

if (userId == null) {
    throw new UnauthorizedException("登录已过期");
}
```

这样可以实现：

- Token 主动失效。
- 退出登录。
- 登录状态集中管理。

退出登录：

```java
stringRedisTemplate.delete("login:token:" + token);
```

如果只用纯 JWT，Token 未过期前无法主动失效。配合 Redis 后，可以删除 Redis key 让 Token 立即失效。

## 13.7 验证码缓存

验证码通常有有效期，例如 5 分钟。

生成验证码：

```java
String uuid = UUID.randomUUID().toString();
String code = "9527";

stringRedisTemplate.opsForValue().set(
        "captcha:" + uuid,
        code,
        Duration.ofMinutes(5)
);
```

返回给前端：

```json
{
  "uuid": "abc-xxx",
  "image": "base64..."
}
```

登录时前端提交：

```json
{
  "username": "admin",
  "password": "123456",
  "uuid": "abc-xxx",
  "code": "9527"
}
```

后端校验：

```java
String key = "captcha:" + dto.getUuid();
String realCode = stringRedisTemplate.opsForValue().get(key);

if (realCode == null) {
    throw new BusinessException("验证码已过期");
}

if (!realCode.equalsIgnoreCase(dto.getCode())) {
    throw new BusinessException("验证码错误");
}

stringRedisTemplate.delete(key);
```

校验成功后删除验证码，避免重复使用。

## 13.8 防重复提交

有些按钮用户可能连续点击多次，例如：

```text
提交订单
支付
新增表单
导入数据
```

可以使用 Redis 做简单防重复提交。

思路：

```text
用户请求到来
生成 key
尝试写入 Redis，并设置短过期时间
如果写入成功，说明首次提交
如果 key 已存在，说明重复提交
```

示例：

```java
String key = "repeat:submit:" + userId + ":" + request.getRequestURI();

Boolean success = stringRedisTemplate.opsForValue()
        .setIfAbsent(key, "1", Duration.ofSeconds(3));

if (Boolean.FALSE.equals(success)) {
    throw new BusinessException("请勿重复提交");
}
```

这适合处理短时间重复点击。

更复杂的业务幂等，需要结合业务唯一号，例如订单号、支付流水号。

## 13.9 缓存用户详情

用户详情如果访问频繁，可以缓存。

查询逻辑：

```text
先查 Redis。
Redis 有，直接返回。
Redis 没有，查 MySQL。
MySQL 查到后写入 Redis。
```

代码示例：

```java
public UserVO getUserById(Long id) {
    String key = "user:info:" + id;
    String json = stringRedisTemplate.opsForValue().get(key);

    if (json != null) {
        return objectMapper.readValue(json, UserVO.class);
    }

    User user = userMapper.selectById(id);
    if (user == null) {
        throw new BusinessException("用户不存在");
    }

    UserVO vo = toVO(user);
    stringRedisTemplate.opsForValue().set(key, objectMapper.writeValueAsString(vo), Duration.ofMinutes(30));
    return vo;
}
```

修改用户时，要删除缓存：

```java
stringRedisTemplate.delete("user:info:" + id);
```

否则前端可能看到旧数据。

## 13.10 缓存一致性

缓存和数据库同时存在时，会有一致性问题。

常见策略：

```text
读：先查缓存，缓存没有再查数据库。
写：先更新数据库，再删除缓存。
```

为什么不是更新缓存？

```text
很多时候缓存数据是复杂计算结果。
直接删除缓存，让下次查询重新加载，更简单可靠。
```

更新用户：

```java
userMapper.updateById(user);
stringRedisTemplate.delete("user:info:" + user.getId());
```

这不能解决所有极端并发问题，但适合大多数普通业务场景。

## 13.11 缓存穿透、击穿、雪崩

缓存穿透：

```text
查询一个数据库中不存在的数据。
缓存没有，数据库也没有。
大量这种请求会一直打到数据库。
```

解决思路：

- 缓存空值。
- 参数校验。
- 布隆过滤器。

缓存击穿：

```text
某个热点 key 过期，大量请求同时访问数据库。
```

解决思路：

- 热点 key 不过期。
- 加锁重建缓存。

缓存雪崩：

```text
大量 key 同时过期，导致请求集中打到数据库。
```

解决思路：

- 过期时间加随机值。
- 分散 key 过期时间。
- 限流降级。

学习阶段先理解概念，真实高并发项目再深入。

## 13.12 Redis 常见错误

### 错误一：把 Redis 当 MySQL 用

Redis 适合缓存和临时数据，不适合替代主要业务数据库。

### 错误二：key 没有过期时间

验证码、Token、重复提交标记都应该设置过期时间。

### 错误三：更新数据库后不删缓存

会导致用户看到旧数据。

### 错误四：Redis key 命名混乱

排查问题困难，也容易 key 冲突。

## 13.13 练习任务

### 任务一：集成 Redis

在 Spring Boot 项目中添加 Redis 依赖和配置，使用 `StringRedisTemplate` 写入并读取一个 key。

### 任务二：实现验证码缓存

生成验证码并存入 Redis，设置 5 分钟过期。

### 任务三：实现退出登录

登录时把 Token 存入 Redis，退出时删除 Token。

### 任务四：实现防重复提交

对新增商品接口增加 3 秒内防重复提交。

### 任务五：缓存用户详情

实现用户详情缓存，修改用户后删除缓存。

## 13.14 面试常问点

### 问题一：Redis 和 MySQL 有什么区别？

参考回答：

```text
MySQL 是关系型数据库，主要用于持久化保存业务数据。
Redis 是内存键值数据库，访问速度快，常用于缓存、验证码、Token 和临时数据。
```

### 问题二：为什么验证码适合放 Redis？

参考回答：

```text
验证码是临时数据，有过期时间，不需要长期保存。
Redis 支持快速读写和过期时间，非常适合保存验证码。
```

### 问题三：缓存更新常见策略是什么？

参考回答：

```text
常见策略是先更新数据库，再删除缓存。
下次查询时如果缓存不存在，再从数据库加载并写入缓存。
```

## 13.15 本章总结

本章学习了 Redis：

```text
Redis 作用
Redis 和 MySQL 区别
StringRedisTemplate
key 命名规范
登录用户缓存
验证码缓存
防重复提交
用户详情缓存
缓存一致性
缓存穿透
缓存击穿
缓存雪崩
```

Redis 不是必须一开始就深入到底层，但后端开发必须掌握常见应用场景。  
下一章将学习接口文档、异常、日志和工程规范，这些内容决定项目是否具备企业级可维护性。

