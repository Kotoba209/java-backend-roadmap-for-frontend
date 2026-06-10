# 第九章：MyBatis / MyBatis-Plus 数据访问

## 9.1 本章目标

前面已经学习了 Spring MVC 接口开发，但接口如果不连接数据库，只能返回写死的数据。

本章开始解决：

```text
Controller 如何查询 MySQL？
Service 如何调用数据库？
Mapper 是什么？
MyBatis 如何把 SQL 结果变成 Java 对象？
MyBatis-Plus 如何简化 CRUD？
分页查询怎么做？
条件查询怎么写？
逻辑删除和自动填充怎么配置？
```

学完本章后，你应该能够：

- 理解 MyBatis 的作用。
- 掌握 Mapper 接口和 XML SQL。
- 使用 MyBatis 完成基础 CRUD。
- 理解参数绑定和结果映射。
- 使用动态 SQL 编写条件查询。
- 理解 MyBatis-Plus 的作用。
- 使用 BaseMapper 快速完成 CRUD。
- 使用 LambdaQueryWrapper 编写条件查询。
- 使用分页插件完成分页。
- 配置逻辑删除和自动填充。

## 9.2 MyBatis 是什么

MyBatis 是 Java 后端常用的数据访问框架。

前面 JDBC 查询用户需要写很多代码：

```java
Connection connection = DriverManager.getConnection(url, username, password);
PreparedStatement statement = connection.prepareStatement(sql);
ResultSet rs = statement.executeQuery();
```

还要手动把结果集转成对象：

```java
User user = new User();
user.setId(rs.getLong("id"));
user.setUsername(rs.getString("username"));
```

MyBatis 帮你简化这些重复工作。

你只需要定义 Mapper 方法：

```java
User selectById(Long id);
```

再写 SQL：

```xml
<select id="selectById" resultType="com.example.admin.entity.User">
    SELECT id, username, nickname, status
    FROM sys_user
    WHERE id = #{id}
</select>
```

调用：

```java
User user = userMapper.selectById(1L);
```

MyBatis 会负责执行 SQL，并把查询结果映射成 `User` 对象。

## 9.3 添加 MyBatis-Plus 依赖

真实项目中可以直接使用 MyBatis，也可以使用 MyBatis-Plus。

MyBatis-Plus 是 MyBatis 的增强工具，能简化大量单表 CRUD。

Maven 依赖：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.7</version>
</dependency>

<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

数据库配置：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/java_admin?useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

启动类添加 Mapper 扫描：

```java
@SpringBootApplication
@MapperScan("com.example.admin.mapper")
public class AdminApplication {
    public static void main(String[] args) {
        SpringApplication.run(AdminApplication.class, args);
    }
}
```

`@MapperScan` 告诉 Spring 去哪里扫描 Mapper 接口。

## 9.4 Entity 实体类

数据库表：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(100) NOT NULL,
    nickname VARCHAR(50),
    status TINYINT DEFAULT 1,
    deleted TINYINT DEFAULT 0,
    create_time DATETIME,
    update_time DATETIME
);
```

Java 实体类：

```java
@Data
@TableName("sys_user")
public class User {
    private Long id;
    private String username;
    private String password;
    private String nickname;
    private Integer status;
    private Integer deleted;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

说明：

- `@Data`：Lombok 生成 getter、setter。
- `@TableName("sys_user")`：指定数据库表名。
- `create_time` 会自动映射到 `createTime`。
- `update_time` 会自动映射到 `updateTime`。

数据库字段使用下划线，Java 字段使用小驼峰，这是后端项目常见规范。

## 9.5 Mapper 接口

使用 MyBatis-Plus，可以让 Mapper 继承 `BaseMapper`：

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

这样就自动拥有很多基础方法：

```java
insert
deleteById
updateById
selectById
selectList
selectPage
```

Service 中使用：

```java
@Service
public class UserServiceImpl implements UserService {

    private final UserMapper userMapper;

    public UserServiceImpl(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    public UserVO getById(Long id) {
        User user = userMapper.selectById(id);
        if (user == null) {
            throw new BusinessException("用户不存在");
        }
        return toVO(user);
    }
}
```

这比手写 JDBC 简洁很多。

## 9.6 新增数据

DTO：

```java
@Data
public class UserCreateDTO {
    private String username;
    private String password;
    private String nickname;
}
```

Service：

```java
public void create(UserCreateDTO dto) {
    User user = new User();
    user.setUsername(dto.getUsername());
    user.setPassword(dto.getPassword());
    user.setNickname(dto.getNickname());
    user.setStatus(1);
    user.setDeleted(0);
    user.setCreateTime(LocalDateTime.now());
    user.setUpdateTime(LocalDateTime.now());

    userMapper.insert(user);
}
```

底层对应 SQL：

```sql
INSERT INTO sys_user (...)
VALUES (...);
```

注意：学习阶段可以暂时直接保存密码，真实项目必须加密保存。登录认证章节会讲 BCrypt。

## 9.7 根据 ID 查询

```java
public UserVO getById(Long id) {
    User user = userMapper.selectById(id);

    if (user == null) {
        throw new BusinessException("用户不存在");
    }

    return toVO(user);
}
```

Entity 转 VO：

```java
private UserVO toVO(User user) {
    UserVO vo = new UserVO();
    vo.setId(user.getId());
    vo.setUsername(user.getUsername());
    vo.setNickname(user.getNickname());
    vo.setStatus(user.getStatus());
    return vo;
}
```

为什么不直接返回 `User`？

```text
User 里有 password、deleted 等字段。
这些字段不应该返回给前端。
```

## 9.8 修改数据

DTO：

```java
@Data
public class UserUpdateDTO {
    private String nickname;
    private Integer status;
}
```

Service：

```java
public void update(Long id, UserUpdateDTO dto) {
    User user = userMapper.selectById(id);
    if (user == null) {
        throw new BusinessException("用户不存在");
    }

    user.setNickname(dto.getNickname());
    user.setStatus(dto.getStatus());
    user.setUpdateTime(LocalDateTime.now());

    userMapper.updateById(user);
}
```

修改前先查询，是为了确认数据存在，并且可以做业务判断。

例如：

```java
if (Integer.valueOf(0).equals(user.getStatus())) {
    throw new BusinessException("禁用用户不能修改");
}
```

## 9.9 删除数据

物理删除：

```java
userMapper.deleteById(id);
```

真实项目更常用逻辑删除。

如果配置 MyBatis-Plus 逻辑删除：

```java
@TableLogic
private Integer deleted;
```

配置：

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
```

调用：

```java
userMapper.deleteById(id);
```

MyBatis-Plus 会把它转成类似：

```sql
UPDATE sys_user SET deleted = 1 WHERE id = ?
```

而不是物理删除。

## 9.10 条件查询 LambdaQueryWrapper

查询启用用户：

```java
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(User::getStatus, 1);
wrapper.eq(User::getDeleted, 0);

List<User> users = userMapper.selectList(wrapper);
```

模糊查询：

```java
wrapper.like(User::getUsername, "admin");
```

排序：

```java
wrapper.orderByDesc(User::getCreateTime);
```

组合：

```java
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(User::getDeleted, 0)
       .eq(queryDTO.getStatus() != null, User::getStatus, queryDTO.getStatus())
       .like(queryDTO.getUsername() != null, User::getUsername, queryDTO.getUsername())
       .orderByDesc(User::getCreateTime);

List<User> users = userMapper.selectList(wrapper);
```

这里的条件写法：

```java
eq(condition, column, value)
```

只有 `condition` 为 `true` 时才拼接条件。

这很适合前端查询表单：

```ts
{
  username: 'admin',
  status: 1
}
```

某些参数不传，就不加对应 SQL 条件。

## 9.11 分页查询

分页插件配置：

```java
@Configuration
public class MyBatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

DTO：

```java
@Data
public class UserQueryDTO {
    private Integer pageNum = 1;
    private Integer pageSize = 10;
    private String username;
    private Integer status;
}
```

Service：

```java
public PageResult<UserVO> page(UserQueryDTO queryDTO) {
    Page<User> page = new Page<>(queryDTO.getPageNum(), queryDTO.getPageSize());

    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(User::getDeleted, 0)
           .like(queryDTO.getUsername() != null, User::getUsername, queryDTO.getUsername())
           .eq(queryDTO.getStatus() != null, User::getStatus, queryDTO.getStatus())
           .orderByDesc(User::getCreateTime);

    Page<User> result = userMapper.selectPage(page, wrapper);

    List<UserVO> records = result.getRecords().stream()
            .map(this::toVO)
            .toList();

    return new PageResult<>(result.getTotal(), records);
}
```

返回对象：

```java
@Data
@AllArgsConstructor
public class PageResult<T> {
    private Long total;
    private List<T> records;
}
```

返回给前端：

```json
{
  "total": 100,
  "records": []
}
```

## 9.12 XML 自定义 SQL

MyBatis-Plus 适合单表 CRUD，但复杂 SQL 仍然可以写 XML。

Mapper：

```java
public interface UserMapper extends BaseMapper<User> {
    List<UserRoleVO> selectUserRoles(Long userId);
}
```

XML 文件：

```xml
<mapper namespace="com.example.admin.mapper.UserMapper">

    <select id="selectUserRoles" resultType="com.example.admin.vo.UserRoleVO">
        SELECT
            u.id AS userId,
            u.username,
            r.role_name AS roleName,
            r.role_code AS roleCode
        FROM sys_user u
        LEFT JOIN sys_user_role ur ON u.id = ur.user_id
        LEFT JOIN sys_role r ON ur.role_id = r.id
        WHERE u.id = #{userId}
    </select>

</mapper>
```

XML 路径通常放在：

```text
src/main/resources/mapper/UserMapper.xml
```

配置：

```yaml
mybatis-plus:
  mapper-locations: classpath:/mapper/**/*.xml
```

## 9.13 动态 SQL

MyBatis XML 支持动态 SQL。

```xml
<select id="selectListByCondition" resultType="com.example.admin.entity.User">
    SELECT id, username, nickname, status
    FROM sys_user
    WHERE deleted = 0
    <if test="username != null and username != ''">
        AND username LIKE CONCAT('%', #{username}, '%')
    </if>
    <if test="status != null">
        AND status = #{status}
    </if>
    ORDER BY create_time DESC
</select>
```

如果前端传了 username，就加用户名条件。  
如果没传，就不加。

这和前端动态拼接查询参数类似：

```ts
const params = {}
if (form.username) {
  params.username = form.username
}
```

后端动态 SQL 是根据参数决定是否拼接 SQL 条件。

## 9.14 自动填充

创建时间、更新时间不应该每次都手写。

实体类：

```java
@TableField(fill = FieldFill.INSERT)
private LocalDateTime createTime;

@TableField(fill = FieldFill.INSERT_UPDATE)
private LocalDateTime updateTime;
```

填充处理器：

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
}
```

这样新增时自动填充 `createTime` 和 `updateTime`，修改时自动更新 `updateTime`。

## 9.15 常见错误

### 错误一：忘记 @MapperScan

如果没有扫描 Mapper，启动时可能找不到 Mapper Bean。

正确：

```java
@MapperScan("com.example.admin.mapper")
```

### 错误二：表名和实体名不一致

数据库表是：

```text
sys_user
```

实体类是：

```java
public class User {
}
```

需要：

```java
@TableName("sys_user")
```

### 错误三：直接返回 Entity

不推荐：

```java
return userMapper.selectById(id);
```

推荐：

```java
return toVO(userMapper.selectById(id));
```

### 错误四：条件查询没有判断 null

不推荐：

```java
wrapper.eq(User::getStatus, queryDTO.getStatus());
```

如果 status 是 null，可能生成不符合预期的条件。

推荐：

```java
wrapper.eq(queryDTO.getStatus() != null, User::getStatus, queryDTO.getStatus());
```

## 9.16 练习任务

### 任务一：用户 Mapper

创建 `UserMapper extends BaseMapper<User>`，实现：

- 新增用户。
- 根据 ID 查询。
- 修改用户。
- 逻辑删除用户。

### 任务二：用户分页查询

实现：

```text
GET /users?pageNum=1&pageSize=10&username=admin&status=1
```

返回：

```json
{
  "total": 1,
  "records": []
}
```

### 任务三：商品管理 Mapper

创建商品表和 `ProductMapper`，完成商品 CRUD。

### 任务四：动态 SQL

使用 XML 写一个用户查询 SQL，支持：

- username 模糊查询。
- status 精确查询。
- createTime 时间排序。

### 任务五：自动填充

配置 `createTime` 和 `updateTime` 自动填充。

## 9.17 面试常问点

### 问题一：MyBatis 和 JDBC 有什么关系？

参考回答：

```text
JDBC 是 Java 原生数据库访问 API。
MyBatis 底层基于 JDBC，但封装了连接管理、参数设置、结果映射等重复工作。
```

### 问题二：MyBatis-Plus 有什么作用？

参考回答：

```text
MyBatis-Plus 是 MyBatis 的增强工具，提供 BaseMapper、条件构造器、分页、逻辑删除、自动填充等能力，可以简化单表 CRUD。
```

### 问题三：为什么复杂 SQL 仍然需要 XML？

参考回答：

```text
MyBatis-Plus 适合常见单表 CRUD。
复杂多表关联、动态 SQL、统计查询等场景，XML 更清晰，也更方便维护 SQL。
```

### 问题四：什么是逻辑删除？

参考回答：

```text
逻辑删除是把 deleted 字段更新为删除状态，而不是物理删除数据库记录。
这样可以保留历史数据，方便恢复和审计。
```

### 问题五：为什么要 Entity 转 VO？

参考回答：

```text
Entity 对应数据库表，可能包含密码、删除标记等内部字段。
VO 面向前端展示，可以隐藏敏感字段，避免接口和数据库结构强耦合。
```

## 9.18 本章总结

本章学习了 MyBatis / MyBatis-Plus：

```text
MyBatis 作用
MyBatis-Plus 依赖
数据源配置
@MapperScan
Entity
Mapper
BaseMapper
新增
查询
修改
删除
LambdaQueryWrapper
分页插件
XML 自定义 SQL
动态 SQL
逻辑删除
自动填充
```

现在你已经能完成一个接口的完整链路：

```text
前端请求
-> Controller
-> Service
-> Mapper
-> MySQL
-> Entity
-> VO
-> Result
-> JSON 返回
```

下一章将学习标准业务模块开发，把用户管理、角色管理、菜单管理、文件上传、日志模块这些真实后台系统模块按规范拆解出来。

