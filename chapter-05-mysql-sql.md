# 第五章：MySQL 与 SQL 基础

## 5.1 本章目标

前端开发人员平时看到的是接口返回的数据：

```json
{
  "id": 1,
  "username": "admin",
  "nickname": "管理员"
}
```

但后端必须回答这些问题：

```text
这些数据存在哪里？
用户表怎么设计？
字段用什么类型？
用户名是否允许重复？
密码怎么保存？
列表怎么分页？
多个表之间怎么关联？
删除数据是真删还是逻辑删除？
多个 SQL 操作如何保证一致性？
```

这些问题都属于数据库和 SQL 的范围。

学完本章后，你应该能够：

- 理解数据库、表、字段、记录的关系。
- 掌握 MySQL 常见字段类型。
- 能创建用户、角色、订单等常见业务表。
- 能编写 `INSERT`、`SELECT`、`UPDATE`、`DELETE`。
- 能进行条件查询、模糊查询、排序和分页。
- 能理解一对一、一对多、多对多关系。
- 能编写基础 JOIN 查询。
- 理解主键、唯一索引、普通索引。
- 理解事务的基本作用。
- 掌握基础数据库设计规范。

## 5.2 前端为什么必须学习数据库

前端开发人员经常调用接口：

```ts
const res = await request.get('/api/users', {
  params: {
    pageNum: 1,
    pageSize: 10,
    keyword: 'admin'
  }
})
```

前端拿到的是 JSON：

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": 1,
    "records": [
      {
        "id": 1,
        "username": "admin",
        "nickname": "管理员"
      }
    ]
  }
}
```

但后端要做的是：

```text
接收 pageNum、pageSize、keyword
根据 keyword 拼接查询条件
查询 sys_user 表
统计总条数
分页查询当前页数据
把数据库记录转换成 VO
返回给前端
```

对应 SQL 可能是：

```sql
SELECT id, username, nickname
FROM sys_user
WHERE deleted = 0
  AND username LIKE '%admin%'
ORDER BY create_time DESC
LIMIT 0, 10;
```

所以数据库不是后端的附加知识，而是后端开发的核心能力。

## 5.3 数据库、表、字段、记录

可以先用 Excel 类比数据库。

```text
一个数据库：类似一个 Excel 文件。
一张表：类似一个 Sheet。
一个字段：类似一列。
一条记录：类似一行。
```

例如用户表：

| id | username | nickname | status |
|---|---|---|---|
| 1 | admin | 管理员 | 1 |
| 2 | test | 测试用户 | 1 |

对应概念：

- `sys_user` 是表。
- `id`、`username`、`nickname`、`status` 是字段。
- 每一行是一条用户记录。

建表 SQL：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    nickname VARCHAR(50),
    status TINYINT DEFAULT 1
);
```

这张表表示系统用户数据。

## 5.4 MySQL 常见数据类型

后端设计表时，必须为每个字段选择合适的数据类型。

常见类型：

| MySQL 类型 | 说明 | Java 对应类型 |
|---|---|---|
| BIGINT | 大整数，常用于主键 ID | Long |
| INT | 普通整数 | Integer |
| TINYINT | 小整数，常用于状态 | Integer / Boolean |
| VARCHAR | 变长字符串 | String |
| TEXT | 长文本 | String |
| DECIMAL | 精确小数，常用于金额 | BigDecimal |
| DATETIME | 日期时间 | LocalDateTime |
| DATE | 日期 | LocalDate |

用户表示例：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '用户ID',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    password VARCHAR(100) NOT NULL COMMENT '密码',
    nickname VARCHAR(50) COMMENT '昵称',
    status TINYINT DEFAULT 1 COMMENT '状态：0禁用，1启用',
    create_time DATETIME COMMENT '创建时间',
    update_time DATETIME COMMENT '更新时间'
);
```

对应 Java：

```java
public class User {
    private Long id;
    private String username;
    private String password;
    private String nickname;
    private Integer status;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

这里要建立映射意识：

```text
BIGINT -> Long
VARCHAR -> String
TINYINT -> Integer
DATETIME -> LocalDateTime
DECIMAL -> BigDecimal
```

## 5.5 创建数据库和数据表

创建数据库：

```sql
CREATE DATABASE java_admin DEFAULT CHARACTER SET utf8mb4;
```

使用数据库：

```sql
USE java_admin;
```

创建用户表：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '用户ID',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    password VARCHAR(100) NOT NULL COMMENT '密码',
    nickname VARCHAR(50) COMMENT '昵称',
    phone VARCHAR(20) COMMENT '手机号',
    email VARCHAR(100) COMMENT '邮箱',
    status TINYINT DEFAULT 1 COMMENT '状态：0禁用，1启用',
    deleted TINYINT DEFAULT 0 COMMENT '逻辑删除：0未删除，1已删除',
    create_time DATETIME COMMENT '创建时间',
    update_time DATETIME COMMENT '更新时间'
) COMMENT='用户表';
```

字段说明：

- `id`：主键，自增。
- `username`：用户名，不能为空。
- `password`：密码，不能为空。
- `status`：用户状态。
- `deleted`：逻辑删除标记。
- `create_time`：创建时间。
- `update_time`：更新时间。

真实项目中，建议每张业务表都保留：

```text
id
create_time
update_time
deleted
```

这样后续审计、排序、逻辑删除都会更方便。

## 5.6 INSERT 新增数据

新增用户：

```sql
INSERT INTO sys_user (
    username,
    password,
    nickname,
    phone,
    email,
    status,
    deleted,
    create_time,
    update_time
) VALUES (
    'admin',
    '123456',
    '管理员',
    '13800138000',
    'admin@example.com',
    1,
    0,
    NOW(),
    NOW()
);
```

查询：

```sql
SELECT * FROM sys_user;
```

前端新增用户时，可能提交：

```json
{
  "username": "admin",
  "password": "123456",
  "nickname": "管理员",
  "phone": "13800138000"
}
```

后端最终会把这些数据转换成 SQL 插入数据库。

在 MyBatis 中，可能写成：

```java
userMapper.insert(user);
```

但底层本质仍然是执行 `INSERT`。

## 5.7 SELECT 查询数据

查询所有用户：

```sql
SELECT * FROM sys_user;
```

实际项目中不建议总是使用 `*`，更推荐明确字段：

```sql
SELECT id, username, nickname, status, create_time
FROM sys_user;
```

根据 ID 查询：

```sql
SELECT id, username, nickname, status
FROM sys_user
WHERE id = 1;
```

根据用户名查询：

```sql
SELECT id, username, password, status
FROM sys_user
WHERE username = 'admin';
```

登录时后端通常会先根据用户名查询用户：

```sql
SELECT id, username, password, status
FROM sys_user
WHERE username = 'admin'
  AND deleted = 0;
```

然后在 Java 中校验密码。

## 5.8 UPDATE 修改数据

修改用户昵称：

```sql
UPDATE sys_user
SET nickname = '超级管理员',
    update_time = NOW()
WHERE id = 1;
```

修改用户状态：

```sql
UPDATE sys_user
SET status = 0,
    update_time = NOW()
WHERE id = 1;
```

一定要注意 `WHERE` 条件。

危险 SQL：

```sql
UPDATE sys_user
SET status = 0;
```

这会把所有用户都禁用。

后端开发必须养成习惯：

```text
写 UPDATE 和 DELETE 时，先确认 WHERE 条件。
```

## 5.9 DELETE 删除数据和逻辑删除

物理删除：

```sql
DELETE FROM sys_user
WHERE id = 1;
```

物理删除会真正删除数据库记录，恢复困难。

后台管理系统中更常用逻辑删除：

```sql
UPDATE sys_user
SET deleted = 1,
    update_time = NOW()
WHERE id = 1;
```

查询时过滤：

```sql
SELECT id, username, nickname
FROM sys_user
WHERE deleted = 0;
```

逻辑删除的好处：

- 数据可以恢复。
- 可以保留操作记录。
- 避免误删导致业务数据丢失。

但逻辑删除也有代价：所有查询都要记得加 `deleted = 0`。

## 5.10 WHERE 条件查询

条件查询是后端接口最常见的需求。

根据状态查询：

```sql
SELECT id, username, nickname
FROM sys_user
WHERE status = 1;
```

根据用户名模糊查询：

```sql
SELECT id, username, nickname
FROM sys_user
WHERE username LIKE '%admin%';
```

多条件查询：

```sql
SELECT id, username, nickname
FROM sys_user
WHERE deleted = 0
  AND status = 1
  AND username LIKE '%admin%';
```

前端查询表单：

```ts
const query = {
  username: 'admin',
  status: 1
}
```

后端需要根据参数动态生成 SQL 条件。后面学习 MyBatis 动态 SQL 时会重点讲。

## 5.11 ORDER BY 排序

按创建时间倒序：

```sql
SELECT id, username, nickname, create_time
FROM sys_user
WHERE deleted = 0
ORDER BY create_time DESC;
```

按 ID 升序：

```sql
SELECT id, username, nickname
FROM sys_user
ORDER BY id ASC;
```

后台管理系统列表通常按创建时间倒序，这样最新数据排在前面。

```sql
ORDER BY create_time DESC
```

## 5.12 LIMIT 分页查询

前端列表通常有分页：

```ts
{
  pageNum: 1,
  pageSize: 10
}
```

MySQL 使用 `LIMIT` 分页。

第一页：

```sql
SELECT id, username, nickname
FROM sys_user
WHERE deleted = 0
ORDER BY create_time DESC
LIMIT 0, 10;
```

第二页：

```sql
LIMIT 10, 10;
```

计算公式：

```text
offset = (pageNum - 1) * pageSize
```

Java 中：

```java
Integer offset = (pageNum - 1) * pageSize;
```

分页接口通常还需要查总数：

```sql
SELECT COUNT(*)
FROM sys_user
WHERE deleted = 0;
```

返回给前端：

```json
{
  "total": 100,
  "records": []
}
```

## 5.13 聚合函数

常见聚合函数：

| 函数 | 作用 |
|---|---|
| COUNT | 统计数量 |
| SUM | 求和 |
| AVG | 平均值 |
| MAX | 最大值 |
| MIN | 最小值 |

统计用户数量：

```sql
SELECT COUNT(*)
FROM sys_user
WHERE deleted = 0;
```

统计订单总金额：

```sql
SELECT SUM(total_amount)
FROM biz_order
WHERE status = 1;
```

按状态分组统计：

```sql
SELECT status, COUNT(*) AS total
FROM sys_user
WHERE deleted = 0
GROUP BY status;
```

返回结果可能是：

| status | total |
|---|---|
| 0 | 3 |
| 1 | 20 |

这类 SQL 常用于后台首页统计、报表、仪表盘。

## 5.14 表关系：一对一、一对多、多对多

后端系统不是只有单表。

常见关系有三种。

一对一：

```text
一个用户对应一个用户详情。
```

一对多：

```text
一个用户可以有多个订单。
```

多对多：

```text
一个用户可以有多个角色。
一个角色也可以分配给多个用户。
```

用户和角色是典型多对多关系：

```text
sys_user
sys_role
sys_user_role
```

用户表：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL
);
```

角色表：

```sql
CREATE TABLE sys_role (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    role_name VARCHAR(50) NOT NULL,
    role_code VARCHAR(50) NOT NULL
);
```

用户角色关联表：

```sql
CREATE TABLE sys_user_role (
    user_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL
);
```

为什么需要关联表？  
因为用户和角色不是简单的一对一关系，一个用户可能有多个角色，一个角色也可能属于多个用户。

## 5.15 JOIN 多表查询

查询用户及其角色：

```sql
SELECT
    u.id,
    u.username,
    r.role_name
FROM sys_user u
LEFT JOIN sys_user_role ur ON u.id = ur.user_id
LEFT JOIN sys_role r ON ur.role_id = r.id
WHERE u.id = 1;
```

解释：

- `sys_user u`：给用户表起别名 `u`。
- `LEFT JOIN sys_user_role ur`：关联用户角色表。
- `LEFT JOIN sys_role r`：关联角色表。
- `ON`：指定关联条件。

前端最终可能看到：

```json
{
  "id": 1,
  "username": "admin",
  "roles": ["管理员", "编辑"]
}
```

后端需要通过多表查询组装这些数据。

`LEFT JOIN` 表示即使右表没有匹配数据，左表数据也保留。  
后台业务中 `LEFT JOIN` 很常见，因为用户没有角色时，也可能仍然要显示用户。

## 5.16 主键、唯一索引和普通索引

主键用于唯一标识一条记录。

```sql
id BIGINT PRIMARY KEY AUTO_INCREMENT
```

用户名通常不允许重复，可以加唯一索引：

```sql
ALTER TABLE sys_user
ADD UNIQUE KEY uk_username (username);
```

普通索引用于提高查询效率：

```sql
ALTER TABLE sys_user
ADD INDEX idx_status (status);
```

如果经常根据手机号查询用户：

```sql
SELECT id, username
FROM sys_user
WHERE phone = '13800138000';
```

可以给 `phone` 加索引：

```sql
ALTER TABLE sys_user
ADD INDEX idx_phone (phone);
```

但索引不是越多越好。索引会提升查询速度，但会增加写入成本和存储空间。

## 5.17 事务

事务用于保证一组数据库操作要么全部成功，要么全部失败。

经典案例：转账。

```text
A 扣 100 元
B 加 100 元
```

如果 A 扣款成功，但 B 加款失败，数据就不一致。

事务要求：

```text
两个操作都成功，事务提交。
任何一个失败，事务回滚。
```

SQL 示例：

```sql
START TRANSACTION;

UPDATE account SET balance = balance - 100 WHERE id = 1;
UPDATE account SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

如果出现错误：

```sql
ROLLBACK;
```

Java Spring 中通常使用：

```java
@Transactional
public void transfer(Long fromId, Long toId, BigDecimal amount) {
    accountMapper.decrease(fromId, amount);
    accountMapper.increase(toId, amount);
}
```

后面学习 Spring 事务时会详细讲。

## 5.18 数据库设计规范

真实项目中，表设计建议遵循以下规范。

表名使用小写加下划线：

```text
sys_user
sys_role
biz_order
biz_product
```

字段名使用小写加下划线：

```text
user_id
order_no
create_time
update_time
```

每张表建议有主键：

```sql
id BIGINT PRIMARY KEY AUTO_INCREMENT
```

业务表建议有：

```text
create_time
update_time
deleted
```

金额字段使用 `DECIMAL`：

```sql
total_amount DECIMAL(10, 2)
```

状态字段要有注释：

```sql
status TINYINT DEFAULT 1 COMMENT '状态：0禁用，1启用'
```

用户名、手机号等唯一字段要加唯一索引：

```sql
UNIQUE KEY uk_username (username)
```

不要在数据库中保存明文密码。学习阶段可以先用明文理解流程，但真实项目必须加密保存。

## 5.19 综合案例：用户权限基础表设计

设计一个最小权限系统，需要五张表：

```text
sys_user：用户表
sys_role：角色表
sys_menu：菜单权限表
sys_user_role：用户角色关联表
sys_role_menu：角色菜单关联表
```

用户表：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '用户ID',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    password VARCHAR(100) NOT NULL COMMENT '密码',
    nickname VARCHAR(50) COMMENT '昵称',
    status TINYINT DEFAULT 1 COMMENT '状态：0禁用，1启用',
    deleted TINYINT DEFAULT 0 COMMENT '逻辑删除：0未删除，1已删除',
    create_time DATETIME COMMENT '创建时间',
    update_time DATETIME COMMENT '更新时间',
    UNIQUE KEY uk_username (username)
) COMMENT='用户表';
```

角色表：

```sql
CREATE TABLE sys_role (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '角色ID',
    role_name VARCHAR(50) NOT NULL COMMENT '角色名称',
    role_code VARCHAR(50) NOT NULL COMMENT '角色编码',
    status TINYINT DEFAULT 1 COMMENT '状态：0禁用，1启用',
    create_time DATETIME COMMENT '创建时间',
    update_time DATETIME COMMENT '更新时间',
    UNIQUE KEY uk_role_code (role_code)
) COMMENT='角色表';
```

用户角色表：

```sql
CREATE TABLE sys_user_role (
    user_id BIGINT NOT NULL COMMENT '用户ID',
    role_id BIGINT NOT NULL COMMENT '角色ID',
    PRIMARY KEY (user_id, role_id)
) COMMENT='用户角色关联表';
```

菜单表：

```sql
CREATE TABLE sys_menu (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '菜单ID',
    parent_id BIGINT DEFAULT 0 COMMENT '父菜单ID',
    menu_name VARCHAR(50) NOT NULL COMMENT '菜单名称',
    permission VARCHAR(100) COMMENT '权限标识',
    menu_type TINYINT COMMENT '类型：1目录，2菜单，3按钮',
    path VARCHAR(200) COMMENT '路由路径',
    create_time DATETIME COMMENT '创建时间',
    update_time DATETIME COMMENT '更新时间'
) COMMENT='菜单权限表';
```

角色菜单表：

```sql
CREATE TABLE sys_role_menu (
    role_id BIGINT NOT NULL COMMENT '角色ID',
    menu_id BIGINT NOT NULL COMMENT '菜单ID',
    PRIMARY KEY (role_id, menu_id)
) COMMENT='角色菜单关联表';
```

这套表结构会在后续权限章节继续使用。

## 5.20 常见错误

### 错误一：表没有主键

不推荐：

```sql
CREATE TABLE sys_user (
    username VARCHAR(50)
);
```

推荐：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50)
);
```

### 错误二：金额使用 FLOAT 或 DOUBLE

不推荐：

```sql
price DOUBLE
```

推荐：

```sql
price DECIMAL(10, 2)
```

### 错误三：UPDATE 忘记 WHERE

危险：

```sql
UPDATE sys_user SET status = 0;
```

推荐先写：

```sql
SELECT * FROM sys_user WHERE id = 1;
```

确认条件后再写：

```sql
UPDATE sys_user SET status = 0 WHERE id = 1;
```

### 错误四：明文保存密码

学习阶段可以暂时理解流程，但真实项目不能这样：

```sql
password VARCHAR(100) DEFAULT '123456'
```

真实项目中应该保存加密后的密码，例如 BCrypt 哈希值。

### 错误五：逻辑删除后查询忘记过滤

逻辑删除：

```sql
UPDATE sys_user SET deleted = 1 WHERE id = 1;
```

查询时要加：

```sql
WHERE deleted = 0
```

## 5.21 练习任务

### 任务一：创建商品表

设计 `biz_product` 表，包含：

- id。
- product_name。
- price。
- stock。
- status。
- create_time。
- update_time。
- deleted。

要求金额使用 `DECIMAL`。

### 任务二：编写用户 CRUD SQL

基于 `sys_user` 表，写出：

- 新增用户。
- 根据 ID 查询用户。
- 根据用户名模糊查询用户。
- 修改用户昵称。
- 逻辑删除用户。

### 任务三：分页查询

查询第 3 页用户列表，每页 10 条。

计算 `LIMIT` 的 offset。

### 任务四：设计订单表

设计 `biz_order` 表，包含：

- id。
- order_no。
- user_id。
- total_amount。
- status。
- create_time。

### 任务五：用户角色查询

基于 `sys_user`、`sys_role`、`sys_user_role`，写 SQL 查询某个用户拥有的角色。

## 5.22 面试常问点

### 问题一：主键有什么作用？

参考回答：

```text
主键用于唯一标识表中的一条记录。
每张业务表通常都应该有主键，常见设计是 BIGINT 自增 ID。
```

### 问题二：VARCHAR 和 TEXT 有什么区别？

参考回答：

```text
VARCHAR 适合存储较短字符串，例如用户名、手机号、标题。
TEXT 适合存储较长文本，例如文章内容、备注。
```

### 问题三：为什么金额字段使用 DECIMAL？

参考回答：

```text
DECIMAL 是精确小数，适合金额计算。
FLOAT 和 DOUBLE 存在精度问题，不适合保存金额。
```

### 问题四：什么是逻辑删除？

参考回答：

```text
逻辑删除不是物理删除数据，而是通过 deleted 字段标记数据已删除。
查询时过滤 deleted = 0。这样可以保留历史数据，避免误删无法恢复。
```

### 问题五：事务有什么作用？

参考回答：

```text
事务用于保证一组数据库操作的一致性。
要么全部成功提交，要么出现异常全部回滚。
例如转账、下单扣库存等场景都需要事务。
```

## 5.23 本章总结

本章学习了 MySQL 和 SQL 基础：

```text
数据库、表、字段、记录
MySQL 常见数据类型
建库建表
INSERT
SELECT
UPDATE
DELETE
逻辑删除
WHERE 条件查询
LIKE 模糊查询
ORDER BY 排序
LIMIT 分页
聚合函数
一对一、一对多、多对多
JOIN 多表查询
主键、唯一索引、普通索引
事务
数据库设计规范
```

对前端开发人员来说，最重要的是完成视角转换：

```text
前端看到的是 JSON。
后端看到的是表结构、SQL、业务关系和数据一致性。
```

一个用户列表接口背后并不只是返回数组，而是包含：

```text
表设计
查询条件
分页
排序
逻辑删除
字段过滤
数据转换
```

下一章将学习 JDBC 与后端分层思想。你会看到 Java 如何直接连接 MySQL，也会理解为什么真实项目需要 MyBatis，以及 Controller、Service、Mapper 分层到底解决什么问题。

