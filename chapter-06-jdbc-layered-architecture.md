# 第六章：JDBC 与后端分层思想

## 6.1 本章目标

前面已经学习了 MySQL 和 SQL。现在要解决一个关键问题：

```text
Java 程序如何连接 MySQL？
Java 代码如何执行 SQL？
数据库查询结果如何变成 Java 对象？
为什么真实项目不直接在 Controller 里写 SQL？
Controller、Service、Mapper 到底分别负责什么？
```

JDBC 是 Java 访问数据库的基础。虽然真实项目中更常用 MyBatis 或 MyBatis-Plus，但先理解 JDBC，可以帮助你看清后端数据访问的底层流程。

学完本章后，你应该能够：

- 理解 JDBC 的作用。
- 知道 Java 连接数据库需要哪些信息。
- 能看懂 JDBC 查询、新增、修改、删除代码。
- 理解 JDBC 的痛点。
- 理解为什么需要 MyBatis。
- 掌握 Controller、Service、Mapper 分层思想。
- 区分 Entity、DTO、VO。
- 能用分层思维拆解一个用户管理模块。

## 6.2 JDBC 是什么

JDBC 全称是 Java Database Connectivity，即 Java 数据库连接。

可以这样理解：

```text
JDBC 是 Java 官方提供的一套访问数据库的标准 API。
```

前端类比：

```text
前端通过 axios 发送 HTTP 请求。
Java 通过 JDBC 发送 SQL 请求。
```

前端请求接口：

```ts
const res = await axios.get('/api/users')
```

Java JDBC 查询数据库：

```java
ResultSet rs = statement.executeQuery("SELECT * FROM sys_user");
```

两者都属于“从外部系统获取数据”：

```text
前端 -> HTTP -> 后端接口
Java 后端 -> JDBC -> MySQL 数据库
```

JDBC 不是某个具体数据库独有的技术，而是一套标准。MySQL、Oracle、PostgreSQL 等数据库都会提供自己的 JDBC 驱动。

## 6.3 Java 连接 MySQL 需要什么

Java 连接 MySQL 通常需要四类信息：

```text
数据库地址
数据库用户名
数据库密码
数据库驱动
```

示例：

```java
String url = "jdbc:mysql://localhost:3306/java_admin?useSSL=false&serverTimezone=Asia/Shanghai";
String username = "root";
String password = "123456";
```

解释：

- `localhost`：数据库所在服务器地址。
- `3306`：MySQL 默认端口。
- `java_admin`：数据库名称。
- `root`：数据库用户名。
- `123456`：数据库密码。

前端人员可以类比后端接口地址：

```text
http://localhost:8080/api/users
```

数据库连接地址：

```text
jdbc:mysql://localhost:3306/java_admin
```

一个是 HTTP 接口地址，一个是数据库连接地址。

## 6.4 JDBC 查询用户列表

先看一段完整 JDBC 查询代码。

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class JdbcQueryDemo {
    public static void main(String[] args) throws Exception {
        String url = "jdbc:mysql://localhost:3306/java_admin?useSSL=false&serverTimezone=Asia/Shanghai";
        String username = "root";
        String password = "123456";

        Connection connection = DriverManager.getConnection(url, username, password);

        String sql = "SELECT id, username, nickname, status FROM sys_user WHERE deleted = 0";
        PreparedStatement statement = connection.prepareStatement(sql);

        ResultSet rs = statement.executeQuery();

        while (rs.next()) {
            Long id = rs.getLong("id");
            String userName = rs.getString("username");
            String nickname = rs.getString("nickname");
            Integer status = rs.getInt("status");

            System.out.println(id + " - " + userName + " - " + nickname + " - " + status);
        }

        rs.close();
        statement.close();
        connection.close();
    }
}
```

这段代码的流程：

```text
1. 准备数据库连接信息。
2. 获取数据库连接 Connection。
3. 编写 SQL。
4. 创建 PreparedStatement。
5. 执行查询。
6. 遍历 ResultSet。
7. 从结果集中取字段值。
8. 关闭资源。
```

这里的 `ResultSet` 可以理解为数据库查询结果集。  
每次 `rs.next()` 都移动到下一行记录。

## 6.5 把查询结果封装成对象

直接打印数据不够真实。后端项目中通常会把数据库记录封装成 Java 对象。

先定义实体类：

```java
public class User {
    private Long id;
    private String username;
    private String nickname;
    private Integer status;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getNickname() {
        return nickname;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    public Integer getStatus() {
        return status;
    }

    public void setStatus(Integer status) {
        this.status = status;
    }
}
```

JDBC 查询并封装：

```java
List<User> users = new ArrayList<>();

while (rs.next()) {
    User user = new User();
    user.setId(rs.getLong("id"));
    user.setUsername(rs.getString("username"));
    user.setNickname(rs.getString("nickname"));
    user.setStatus(rs.getInt("status"));
    users.add(user);
}
```

这一步非常关键：

```text
数据库行记录 -> Java 对象
```

后面 MyBatis 做的很多事情，本质上就是帮你自动完成这类映射。

## 6.6 JDBC 新增用户

新增用户使用 `INSERT`。

```java
String sql = "INSERT INTO sys_user (username, password, nickname, status, deleted) VALUES (?, ?, ?, ?, ?)";

PreparedStatement statement = connection.prepareStatement(sql);
statement.setString(1, "zhangsan");
statement.setString(2, "123456");
statement.setString(3, "张三");
statement.setInt(4, 1);
statement.setInt(5, 0);

int rows = statement.executeUpdate();
System.out.println("影响行数：" + rows);
```

这里的 `?` 是占位符。

不要直接拼接 SQL：

```java
String sql = "INSERT INTO sys_user (username) VALUES ('" + username + "')";
```

这种写法容易产生 SQL 注入风险。

推荐使用 `PreparedStatement`：

```java
statement.setString(1, username);
```

前端人员可以理解为：不要把用户输入直接拼进最终执行语句里，要使用参数绑定。

## 6.7 JDBC 修改和删除

修改用户状态：

```java
String sql = "UPDATE sys_user SET status = ? WHERE id = ?";

PreparedStatement statement = connection.prepareStatement(sql);
statement.setInt(1, 0);
statement.setLong(2, 1L);

int rows = statement.executeUpdate();
System.out.println("影响行数：" + rows);
```

逻辑删除用户：

```java
String sql = "UPDATE sys_user SET deleted = 1 WHERE id = ?";

PreparedStatement statement = connection.prepareStatement(sql);
statement.setLong(1, 1L);

int rows = statement.executeUpdate();
```

`executeUpdate()` 用于执行：

```text
INSERT
UPDATE
DELETE
```

`executeQuery()` 用于执行：

```text
SELECT
```

## 6.8 JDBC 的问题

JDBC 能用，但真实项目很少直接这样写业务，原因是代码太繁琐。

主要问题：

```text
数据库连接代码重复。
PreparedStatement 设置参数繁琐。
ResultSet 转对象繁琐。
资源关闭繁琐。
SQL 和 Java 代码混在一起。
异常处理复杂。
事务管理麻烦。
```

例如每次查询都要写：

```java
Connection connection = DriverManager.getConnection(url, username, password);
PreparedStatement statement = connection.prepareStatement(sql);
ResultSet rs = statement.executeQuery();
```

每次还要手动转换对象：

```java
User user = new User();
user.setId(rs.getLong("id"));
user.setUsername(rs.getString("username"));
```

如果有几十张表、几百个接口，代码会非常重复。

这就是为什么真实项目会使用 MyBatis 或 MyBatis-Plus。

## 6.9 为什么需要 MyBatis

MyBatis 的作用可以简单理解为：

```text
帮你管理 JDBC 的重复工作，让你更专注于 SQL 和业务。
```

JDBC 中你要写：

```java
PreparedStatement statement = connection.prepareStatement(sql);
statement.setLong(1, id);
ResultSet rs = statement.executeQuery();
```

MyBatis 中可能只需要：

```java
User user = userMapper.selectById(id);
```

Mapper 接口：

```java
public interface UserMapper {
    User selectById(Long id);
}
```

XML SQL：

```xml
<select id="selectById" resultType="com.example.entity.User">
    SELECT id, username, nickname, status
    FROM sys_user
    WHERE id = #{id}
</select>
```

MyBatis 会帮你：

- 获取数据库连接。
- 设置 SQL 参数。
- 执行 SQL。
- 把结果映射成 Java 对象。
- 释放资源。

所以学习 JDBC 的目的不是以后都手写 JDBC，而是理解 MyBatis 背后解决了什么问题。

## 6.10 后端为什么要分层

初学者容易把所有代码写到一个方法里：

```java
@GetMapping("/users")
public List<User> list() throws Exception {
    Connection connection = DriverManager.getConnection(url, username, password);
    PreparedStatement statement = connection.prepareStatement("SELECT * FROM sys_user");
    ResultSet rs = statement.executeQuery();

    List<User> users = new ArrayList<>();
    while (rs.next()) {
        User user = new User();
        user.setId(rs.getLong("id"));
        user.setUsername(rs.getString("username"));
        users.add(user);
    }

    return users;
}
```

这段代码能跑，但不适合真实项目。

问题：

- Controller 太臃肿。
- SQL 和接口代码混在一起。
- 业务逻辑无法复用。
- 单元测试困难。
- 后续维护成本高。

所以后端项目通常分层：

```text
Controller：接收请求，返回响应。
Service：处理业务逻辑。
Mapper / DAO：访问数据库。
Entity：映射数据库表。
DTO：接收前端参数。
VO：返回给前端数据。
```

## 6.11 Controller 层

Controller 是接口入口。

前端请求：

```ts
request.get('/users')
```

后端 Controller：

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public Result<List<UserVO>> list(UserQueryDTO queryDTO) {
        List<UserVO> users = userService.list(queryDTO);
        return Result.success(users);
    }
}
```

Controller 的职责：

- 接收请求。
- 接收参数。
- 调用 Service。
- 返回结果。

Controller 不应该写大量 SQL，也不应该堆复杂业务。

## 6.12 Service 层

Service 负责业务逻辑。

```java
public interface UserService {
    List<UserVO> list(UserQueryDTO queryDTO);
}
```

实现类：

```java
@Service
public class UserServiceImpl implements UserService {

    private final UserMapper userMapper;

    public UserServiceImpl(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public List<UserVO> list(UserQueryDTO queryDTO) {
        List<User> users = userMapper.selectList(queryDTO);

        return users.stream()
                .map(this::toVO)
                .toList();
    }

    private UserVO toVO(User user) {
        UserVO vo = new UserVO();
        vo.setId(user.getId());
        vo.setUsername(user.getUsername());
        vo.setNickname(user.getNickname());
        return vo;
    }
}
```

Service 的职责：

- 校验业务规则。
- 调用 Mapper 查询或修改数据库。
- 处理事务。
- 转换 Entity 和 VO。
- 组合多个数据来源。

例如登录逻辑、下单逻辑、分配角色逻辑，都应该放在 Service。

## 6.13 Mapper / DAO 层

Mapper 负责访问数据库。

```java
public interface UserMapper {
    List<User> selectList(UserQueryDTO queryDTO);
    User selectById(Long id);
    int insert(User user);
    int updateById(User user);
    int deleteById(Long id);
}
```

Mapper 的职责很明确：

```text
把 Java 方法映射成 SQL 操作。
```

例如：

```java
User selectById(Long id);
```

对应 SQL：

```sql
SELECT id, username, nickname, status
FROM sys_user
WHERE id = ?
```

Mapper 不应该处理复杂业务。  
例如“用户是否可以登录”“订单是否可以支付”不应该放在 Mapper 中。

## 6.14 Entity、DTO、VO 的区别

这是前端转后端必须掌握的概念。

### Entity

Entity 对应数据库表。

```java
public class User {
    private Long id;
    private String username;
    private String password;
    private String nickname;
    private Integer status;
}
```

它通常包含数据库字段。

### DTO

DTO 用于接收前端请求参数。

```java
public class UserCreateDTO {
    private String username;
    private String password;
    private String nickname;
}
```

前端提交：

```json
{
  "username": "zhangsan",
  "password": "123456",
  "nickname": "张三"
}
```

后端用 DTO 接收。

### VO

VO 用于返回给前端。

```java
public class UserVO {
    private Long id;
    private String username;
    private String nickname;
    private Integer status;
}
```

注意：VO 中不应该包含密码。

### 三者对比

| 类型 | 作用 | 面向谁 |
|---|---|---|
| Entity | 映射数据库表 | 数据库 |
| DTO | 接收前端参数 | 前端请求 |
| VO | 返回前端数据 | 前端响应 |

不要一个对象到处用。这样短期省事，长期会导致安全和维护问题。

## 6.15 一个用户列表接口的完整分层流程

前端请求：

```ts
getUserList({
  username: 'admin',
  status: 1
})
```

Controller：

```java
@GetMapping
public Result<List<UserVO>> list(UserQueryDTO queryDTO) {
    return Result.success(userService.list(queryDTO));
}
```

Service：

```java
public List<UserVO> list(UserQueryDTO queryDTO) {
    List<User> users = userMapper.selectList(queryDTO);
    return users.stream().map(this::toVO).toList();
}
```

Mapper：

```java
List<User> selectList(UserQueryDTO queryDTO);
```

SQL：

```sql
SELECT id, username, nickname, status
FROM sys_user
WHERE deleted = 0
  AND username LIKE '%admin%'
  AND status = 1;
```

返回给前端：

```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "id": 1,
      "username": "admin",
      "nickname": "管理员",
      "status": 1
    }
  ]
}
```

完整链路：

```text
前端页面
-> Controller
-> Service
-> Mapper
-> MySQL
-> Mapper
-> Service 转 VO
-> Controller 包装 Result
-> 前端页面
```

这就是后端分层的核心。

## 6.16 常见错误

### 错误一：Controller 里直接写 SQL

不推荐：

```java
@GetMapping("/users")
public List<User> list() {
    // JDBC 查询数据库
}
```

推荐：

```java
@GetMapping("/users")
public Result<List<UserVO>> list(UserQueryDTO queryDTO) {
    return Result.success(userService.list(queryDTO));
}
```

### 错误二：直接返回 Entity

不推荐：

```java
public List<User> list() {
    return userMapper.selectList();
}
```

如果 `User` 包含密码字段，可能泄露敏感数据。

推荐：

```java
public List<UserVO> list() {
    return userMapper.selectList().stream()
            .map(this::toVO)
            .toList();
}
```

### 错误三：Service 只做转发

不好的 Service：

```java
public User getById(Long id) {
    return userMapper.selectById(id);
}
```

如果只是简单查询可以接受，但复杂业务中 Service 应该承担业务规则：

```java
public UserVO getById(Long id) {
    User user = userMapper.selectById(id);
    if (user == null) {
        throw new BusinessException("用户不存在");
    }
    return toVO(user);
}
```

### 错误四：DTO、VO、Entity 混用

不推荐：

```java
@PostMapping("/users")
public Result<Void> create(@RequestBody User user) {
}
```

推荐：

```java
@PostMapping("/users")
public Result<Void> create(@RequestBody UserCreateDTO dto) {
}
```

## 6.17 练习任务

### 任务一：写 JDBC 查询

使用 JDBC 查询 `sys_user` 表，输出：

- id。
- username。
- nickname。

### 任务二：封装 User 对象

把 JDBC 查询结果封装成 `List<User>`。

### 任务三：设计分层结构

为用户管理模块设计：

```text
UserController
UserService
UserServiceImpl
UserMapper
User
UserCreateDTO
UserQueryDTO
UserVO
```

### 任务四：区分 DTO 和 VO

设计新增用户 DTO 和用户详情 VO。

要求：

- DTO 包含 password。
- VO 不包含 password。

### 任务五：画出调用链路

画出用户列表接口从前端到数据库再返回前端的完整链路。

## 6.18 面试常问点

### 问题一：JDBC 是什么？

参考回答：

```text
JDBC 是 Java 官方提供的数据库连接 API，用于让 Java 程序连接数据库并执行 SQL。
MyBatis 等框架底层也是基于 JDBC 完成数据库访问。
```

### 问题二：为什么真实项目不直接使用 JDBC？

参考回答：

```text
直接使用 JDBC 代码重复较多，需要手动管理连接、设置参数、处理结果集和关闭资源。
MyBatis 可以简化这些重复工作，让开发者更专注于 SQL 和业务逻辑。
```

### 问题三：Controller、Service、Mapper 分别负责什么？

参考回答：

```text
Controller 负责接收请求和返回响应。
Service 负责业务逻辑处理。
Mapper 负责数据库访问。
```

### 问题四：DTO、VO、Entity 有什么区别？

参考回答：

```text
Entity 对应数据库表。
DTO 用于接收前端请求参数。
VO 用于返回给前端展示数据。
三者不要随意混用，避免安全和维护问题。
```

### 问题五：为什么不建议直接返回 Entity？

参考回答：

```text
Entity 可能包含数据库中的敏感字段，例如密码、删除标记、内部状态。
直接返回给前端可能造成数据泄露，也会让接口和数据库结构强耦合。
```

## 6.19 本章总结

本章学习了 JDBC 和后端分层思想：

```text
JDBC 是 Java 访问数据库的基础。
JDBC 需要数据库连接、SQL、PreparedStatement、ResultSet。
JDBC 能完成增删改查，但代码重复较多。
MyBatis 用于简化 JDBC 数据访问。
后端项目通常分为 Controller、Service、Mapper。
Entity 映射数据库表。
DTO 接收前端请求。
VO 返回给前端展示。
```

对前端开发人员来说，本章最重要的是理解后端完整链路：

```text
前端请求
-> Controller 接口入口
-> Service 业务逻辑
-> Mapper 数据库访问
-> MySQL
-> Entity
-> VO
-> JSON 响应
-> 前端页面
```

下一章将进入 Maven 与 Spring Boot。你会学习如何创建一个真正的 Spring Boot 后端项目，理解 `pom.xml`、项目目录、配置文件、启动类、依赖注入和第一个接口。

