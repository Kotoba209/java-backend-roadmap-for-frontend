# 第四章：Java 常用 API 与集合框架

## 4.1 本章目标

前面已经学习了 Java 基础语法和面向对象。本章开始学习真实业务代码中最常用的一批 Java API。

后端开发不是只会写类和方法，还需要熟练处理：

- 字符串。
- 金额。
- 时间。
- 列表。
- 键值对。
- 泛型。
- 数据转换。
- 异常。

例如一个用户列表接口，后端可能需要做这些事情：

```text
判断查询关键字是否为空。
根据用户状态过滤数据。
把 User 实体转换成 UserVO。
格式化创建时间。
返回分页列表。
参数错误时抛出业务异常。
```

这些能力都离不开 Java 常用 API 和集合框架。

学完本章后，你应该能够：

- 熟练使用 `String` 处理字符串。
- 理解 `StringBuilder` 的使用场景。
- 理解包装类和自动装箱拆箱。
- 使用 `BigDecimal` 处理金额。
- 使用 `LocalDate`、`LocalDateTime` 处理日期时间。
- 使用 `List` 保存列表数据。
- 使用 `Set` 处理去重。
- 使用 `Map` 保存键值对数据。
- 理解泛型的意义。
- 使用 Stream API 处理集合。
- 理解 Java 异常处理机制。

## 4.2 String 常用方法

`String` 是后端项目中最常用的类型之一。

前端中你经常处理字符串：

```ts
const username = ' admin '
console.log(username.trim())
console.log(username.includes('adm'))
```

Java 中也有类似方法：

```java
String username = " admin ";

System.out.println(username.trim());
System.out.println(username.contains("adm"));
```

常用方法：

```java
String text = "hello java";

System.out.println(text.length());          // 长度
System.out.println(text.contains("java"));  // 是否包含
System.out.println(text.startsWith("he"));  // 是否以 he 开头
System.out.println(text.endsWith("va"));    // 是否以 va 结尾
System.out.println(text.toUpperCase());     // 转大写
System.out.println(text.toLowerCase());     // 转小写
System.out.println(text.replace("java", "spring"));
```

业务示例：判断用户名是否为空。

```java
public boolean isBlank(String text) {
    return text == null || text.trim().length() == 0;
}
```

使用：

```java
if (isBlank(username)) {
    throw new IllegalArgumentException("用户名不能为空");
}
```

很多项目会使用工具类，例如 Spring 的 `StringUtils` 或 Apache Commons Lang 的 `StringUtils`，但初学阶段先掌握原理。

## 4.3 字符串比较

Java 中字符串内容比较要使用 `equals`。

错误写法：

```java
String status = new String("ENABLE");

if (status == "ENABLE") {
    System.out.println("启用");
}
```

正确写法：

```java
if ("ENABLE".equals(status)) {
    System.out.println("启用");
}
```

为什么推荐常量写前面？

```java
"ENABLE".equals(status)
```

如果 `status` 是 `null`，这段代码会返回 `false`，不会报错。

如果写成：

```java
status.equals("ENABLE")
```

当 `status` 为 `null` 时，会抛出：

```text
NullPointerException
```

业务中常见写法：

```java
if ("ADMIN".equals(user.getRoleCode())) {
    System.out.println("管理员");
}
```

## 4.4 StringBuilder

如果需要频繁拼接字符串，推荐使用 `StringBuilder`。

普通拼接：

```java
String sql = "";
sql = sql + "SELECT * ";
sql = sql + "FROM sys_user ";
sql = sql + "WHERE deleted = 0";
```

更推荐：

```java
StringBuilder sql = new StringBuilder();
sql.append("SELECT * ");
sql.append("FROM sys_user ");
sql.append("WHERE deleted = 0");

System.out.println(sql.toString());
```

在循环中尤其不要频繁用 `+` 拼接大量字符串。

示例：拼接用户名列表。

```java
List<String> usernames = List.of("admin", "test", "guest");

StringBuilder builder = new StringBuilder();
for (String username : usernames) {
    builder.append(username).append(",");
}

String result = builder.toString();
System.out.println(result);
```

真实项目中，如果只是少量字符串拼接，用 `+` 没问题。只有在循环、大量拼接、动态构建文本时，优先考虑 `StringBuilder`。

## 4.5 包装类

Java 的基本类型都有对应包装类：

| 基本类型 | 包装类 |
|---|---|
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
| float | Float |
| double | Double |
| char | Character |
| boolean | Boolean |

示例：

```java
int age = 18;
Integer score = 90;
```

包装类可以为 `null`，基本类型不可以。

```java
int pageNum = 0;
Integer pageSize = null;
```

在后端 DTO 中，更推荐使用包装类：

```java
public class UserQueryDTO {
    private Integer pageNum;
    private Integer pageSize;
    private Long roleId;
}
```

原因是前端可以不传某个参数，不传时对应字段应该是 `null`。

例如：

```text
GET /users?pageNum=1
```

如果 `pageSize` 使用 `Integer`，值是 `null`，可以判断用户没传。  
如果使用 `int`，默认是 `0`，可能分不清是没传还是传了 0。

## 4.6 自动装箱和拆箱

自动装箱：基本类型自动转包装类。

```java
Integer age = 18;
```

等价于：

```java
Integer age = Integer.valueOf(18);
```

自动拆箱：包装类自动转基本类型。

```java
Integer age = 18;
int realAge = age;
```

需要注意空指针问题：

```java
Integer count = null;
int realCount = count; // NullPointerException
```

因为拆箱时需要从 `Integer` 取出 `int` 值，但 `count` 是 `null`。

真实项目中常见错误：

```java
if (user.getStatus() == 1) {
    System.out.println("启用");
}
```

如果 `getStatus()` 返回 `Integer` 且值为 `null`，可能触发拆箱空指针。

更稳妥：

```java
if (Integer.valueOf(1).equals(user.getStatus())) {
    System.out.println("启用");
}
```

## 4.7 BigDecimal 金额计算

金额计算不要使用 `double`。

错误示例：

```java
double total = 0.1 + 0.2;
System.out.println(total);
```

输出可能是：

```text
0.30000000000000004
```

正确使用 `BigDecimal`：

```java
BigDecimal price = new BigDecimal("99.90");
BigDecimal count = new BigDecimal("3");
BigDecimal total = price.multiply(count);

System.out.println(total);
```

输出：

```text
299.70
```

注意不要这样创建：

```java
BigDecimal price = new BigDecimal(99.90);
```

因为传入 double 时，精度问题已经产生。

推荐：

```java
BigDecimal price = new BigDecimal("99.90");
```

或者：

```java
BigDecimal count = BigDecimal.valueOf(3);
```

常用方法：

```java
BigDecimal a = new BigDecimal("10.00");
BigDecimal b = new BigDecimal("3.00");

a.add(b);       // 加
a.subtract(b);  // 减
a.multiply(b);  // 乘
a.divide(b, 2, RoundingMode.HALF_UP); // 除
```

比较大小不要用 `>`：

```java
if (a.compareTo(b) > 0) {
    System.out.println("a 大于 b");
}
```

业务示例：订单金额计算。

```java
public BigDecimal calculateTotal(BigDecimal price, Integer count) {
    if (price == null || count == null || count <= 0) {
        throw new IllegalArgumentException("参数错误");
    }
    return price.multiply(BigDecimal.valueOf(count));
}
```

## 4.8 日期和时间 API

Java 8 之后推荐使用：

- `LocalDate`：日期，例如 `2026-06-10`。
- `LocalTime`：时间，例如 `15:30:00`。
- `LocalDateTime`：日期时间，例如 `2026-06-10T15:30:00`。

示例：

```java
LocalDate today = LocalDate.now();
LocalDateTime now = LocalDateTime.now();

System.out.println(today);
System.out.println(now);
```

真实项目中常见字段：

```java
private LocalDateTime createTime;
private LocalDateTime updateTime;
```

数据库中通常对应：

```sql
create_time DATETIME,
update_time DATETIME
```

格式化时间：

```java
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String text = now.format(formatter);
System.out.println(text);
```

解析字符串：

```java
String text = "2026-06-10 15:30:00";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime time = LocalDateTime.parse(text, formatter);
```

前端对接时，要注意时间格式统一。后端返回：

```json
{
  "createTime": "2026-06-10 15:30:00"
}
```

前端才能稳定展示。

## 4.9 List 列表

`List` 用于保存有序、可重复的数据。

前端数组：

```ts
const users = ['admin', 'test', 'guest']
```

Java List：

```java
List<String> users = new ArrayList<>();
users.add("admin");
users.add("test");
users.add("guest");
```

读取：

```java
System.out.println(users.get(0));
```

遍历：

```java
for (String username : users) {
    System.out.println(username);
}
```

保存对象：

```java
List<User> userList = new ArrayList<>();

User user = new User();
user.setId(1L);
user.setUsername("admin");

userList.add(user);
```

后端接口中，列表非常常见：

```java
public Result<List<UserVO>> list() {
    List<UserVO> users = userService.list();
    return Result.success(users);
}
```

返回给前端后就是 JSON 数组：

```json
[
  {
    "id": 1,
    "username": "admin"
  },
  {
    "id": 2,
    "username": "test"
  }
]
```

## 4.10 Set 集合

`Set` 用于保存不重复的数据。

```java
Set<String> roles = new HashSet<>();
roles.add("admin");
roles.add("editor");
roles.add("admin");

System.out.println(roles);
```

虽然添加了两次 `admin`，但 Set 中只会保留一个。

业务场景：

- 去重角色编码。
- 去重权限标识。
- 去重用户 ID。

示例：

```java
List<Long> userIds = List.of(1L, 2L, 2L, 3L);
Set<Long> uniqueUserIds = new HashSet<>(userIds);

System.out.println(uniqueUserIds);
```

如果业务要求保持插入顺序，可以使用 `LinkedHashSet`：

```java
Set<String> permissions = new LinkedHashSet<>();
```

## 4.11 Map 键值对

`Map` 用于保存键值对。

前端对象：

```ts
const userMap = {
  admin: '管理员',
  test: '测试用户'
}
```

Java Map：

```java
Map<String, String> userMap = new HashMap<>();
userMap.put("admin", "管理员");
userMap.put("test", "测试用户");

System.out.println(userMap.get("admin"));
```

业务场景：根据状态码获取状态名称。

```java
Map<Integer, String> statusMap = new HashMap<>();
statusMap.put(0, "禁用");
statusMap.put(1, "启用");
statusMap.put(2, "锁定");

String statusText = statusMap.get(1);
```

遍历 Map：

```java
for (Map.Entry<Integer, String> entry : statusMap.entrySet()) {
    System.out.println(entry.getKey() + ":" + entry.getValue());
}
```

真实项目中，Map 常用于：

- 数据分组。
- ID 到对象的映射。
- 配置项。
- 临时聚合结果。

例如把用户列表转成 `userId -> User`：

```java
Map<Long, User> userMap = new HashMap<>();
for (User user : users) {
    userMap.put(user.getId(), user);
}
```

## 4.12 泛型

泛型用于限制集合中元素的类型。

```java
List<String> names = new ArrayList<>();
```

表示这个 List 只能放字符串。

正确：

```java
names.add("admin");
```

错误：

```java
names.add(123);
```

如果没有泛型：

```java
List list = new ArrayList();
list.add("admin");
list.add(123);
```

这样会让集合里什么类型都能放，后续取数据时容易出错。

后端项目中泛型非常常见：

```java
List<UserVO>
Result<UserVO>
Result<List<UserVO>>
PageResult<OrderVO>
Map<Long, User>
```

例如统一返回结果：

```java
public class Result<T> {
    private Integer code;
    private String message;
    private T data;
}
```

`T` 表示返回数据类型可以变化。

使用：

```java
Result<UserVO> userResult;
Result<List<UserVO>> listResult;
Result<Boolean> booleanResult;
```

泛型让代码既灵活，又保持类型安全。

## 4.13 Stream API 入门

Stream API 用于处理集合数据，类似前端数组的 `map`、`filter`、`reduce`。

前端：

```ts
const enabledUsers = users.filter(user => user.enabled)
const usernames = users.map(user => user.username)
```

Java：

```java
List<User> enabledUsers = users.stream()
        .filter(user -> Boolean.TRUE.equals(user.getEnabled()))
        .toList();

List<String> usernames = users.stream()
        .map(User::getUsername)
        .toList();
```

过滤：

```java
List<User> enabledUsers = users.stream()
        .filter(user -> Integer.valueOf(1).equals(user.getStatus()))
        .toList();
```

映射：

```java
List<UserVO> voList = users.stream()
        .map(user -> {
            UserVO vo = new UserVO();
            vo.setId(user.getId());
            vo.setUsername(user.getUsername());
            return vo;
        })
        .toList();
```

排序：

```java
List<User> sortedUsers = users.stream()
        .sorted(Comparator.comparing(User::getCreateTime))
        .toList();
```

分组：

```java
Map<Integer, List<User>> groupByStatus = users.stream()
        .collect(Collectors.groupingBy(User::getStatus));
```

注意：Stream 很方便，但不要为了炫技写得过度复杂。业务代码首先要清晰。

## 4.14 Optional

`Optional` 用于表达一个值可能存在，也可能不存在。

```java
Optional<User> optionalUser = findUserById(1L);
```

判断：

```java
if (optionalUser.isPresent()) {
    User user = optionalUser.get();
}
```

更常见写法：

```java
User user = optionalUser.orElseThrow(() -> new BusinessException("用户不存在"));
```

示例方法：

```java
public Optional<User> findUserById(Long id) {
    if (id == null) {
        return Optional.empty();
    }

    User user = new User();
    user.setId(id);
    user.setUsername("admin");
    return Optional.of(user);
}
```

`Optional` 可以减少一部分空值判断，但不建议所有字段都用 Optional。真实项目中，DTO、Entity 字段通常不使用 Optional。

## 4.15 异常处理

异常表示程序运行过程中出现了不正常情况。

例如：

```java
int result = 10 / 0;
```

会抛出：

```text
ArithmeticException
```

数组越界：

```java
String[] names = {"admin"};
System.out.println(names[1]);
```

会抛出：

```text
ArrayIndexOutOfBoundsException
```

空指针：

```java
User user = null;
System.out.println(user.getUsername());
```

会抛出：

```text
NullPointerException
```

异常不是坏事。异常的价值是告诉你程序哪里出现了不符合预期的情况。

## 4.16 try catch

可以使用 `try catch` 捕获异常。

```java
try {
    int result = 10 / 0;
    System.out.println(result);
} catch (ArithmeticException e) {
    System.out.println("除数不能为 0");
}
```

输出：

```text
除数不能为 0
```

多个异常：

```java
try {
    Integer num = Integer.valueOf("abc");
    System.out.println(num);
} catch (NumberFormatException e) {
    System.out.println("数字格式错误");
} catch (Exception e) {
    System.out.println("系统异常");
}
```

注意：不要什么异常都简单吞掉。

不推荐：

```java
try {
    doSomething();
} catch (Exception e) {
}
```

这样错误被隐藏，排查问题会很困难。

真实项目中，通常使用全局异常处理统一返回错误信息，而不是在每个 Controller 中写大量 `try catch`。

## 4.17 throw 和 throws

`throw` 用于主动抛出异常。

```java
if (username == null || username.trim().length() == 0) {
    throw new IllegalArgumentException("用户名不能为空");
}
```

业务异常示例：

```java
public User getUser(Long id) {
    User user = userMapper.selectById(id);
    if (user == null) {
        throw new BusinessException("用户不存在");
    }
    return user;
}
```

`throws` 用于声明方法可能抛出的异常：

```java
public void readFile(String path) throws IOException {
}
```

初学阶段更常见的是 `throw new RuntimeException(...)` 或自定义业务异常。后续会在工程规范章节详细设计 `BusinessException` 和全局异常处理。

## 4.18 自定义业务异常

真实后端项目中，不建议直接到处抛 `RuntimeException`。

可以定义业务异常：

```java
public class BusinessException extends RuntimeException {

    public BusinessException(String message) {
        super(message);
    }
}
```

使用：

```java
if (user == null) {
    throw new BusinessException("用户不存在");
}
```

在 Spring Boot 中，可以通过全局异常处理统一返回：

```json
{
  "code": 400,
  "message": "用户不存在",
  "data": null
}
```

这比直接让程序报 500 错误更友好，也更适合前后端联调。

## 4.19 综合案例：用户列表转换

需求：

```text
有一组 User 对象。
过滤掉禁用用户。
把 User 转成 UserVO。
只返回 id、username、nickname。
```

实体类：

```java
public class User {
    private Long id;
    private String username;
    private String password;
    private String nickname;
    private Integer status;

    // getter / setter 省略
}
```

返回对象：

```java
public class UserVO {
    private Long id;
    private String username;
    private String nickname;

    // getter / setter 省略
}
```

转换代码：

```java
List<UserVO> voList = users.stream()
        .filter(user -> Integer.valueOf(1).equals(user.getStatus()))
        .map(user -> {
            UserVO vo = new UserVO();
            vo.setId(user.getId());
            vo.setUsername(user.getUsername());
            vo.setNickname(user.getNickname());
            return vo;
        })
        .toList();
```

为什么不直接返回 `User`？

因为 `User` 中有密码字段：

```java
private String password;
```

如果直接返回实体对象，可能把密码暴露给前端。  
所以后端项目中经常会把 Entity 转成 VO。

这就是集合、Stream、DTO / VO 在真实项目中的结合使用。

## 4.20 常见错误

### 错误一：字符串比较用 ==

错误：

```java
if (status == "ENABLE") {
}
```

正确：

```java
if ("ENABLE".equals(status)) {
}
```

### 错误二：BigDecimal 使用 double 构造

错误：

```java
BigDecimal price = new BigDecimal(99.90);
```

正确：

```java
BigDecimal price = new BigDecimal("99.90");
```

### 错误三：包装类拆箱导致空指针

错误：

```java
Integer status = null;
if (status == 1) {
}
```

正确：

```java
if (Integer.valueOf(1).equals(status)) {
}
```

### 错误四：直接返回 Entity 给前端

不推荐：

```java
public List<User> list() {
    return userMapper.selectList();
}
```

推荐：

```java
public List<UserVO> list() {
    return userMapper.selectList().stream()
            .map(this::toVO)
            .toList();
}
```

## 4.21 练习任务

### 任务一：字符串校验

写一个方法：

```java
public boolean isValidUsername(String username)
```

要求：

- 不能为空。
- 去除空格后长度至少 3 位。
- 不能包含空格。

### 任务二：订单金额计算

定义商品价格列表：

```java
List<BigDecimal> prices = List.of(
    new BigDecimal("99.90"),
    new BigDecimal("199.00"),
    new BigDecimal("50.50")
);
```

计算总金额。

### 任务三：用户角色去重

定义角色列表：

```java
List<String> roles = List.of("admin", "editor", "admin", "viewer");
```

使用 Set 去重。

### 任务四：用户列表转换

定义 `User` 和 `UserVO`，使用 Stream 把 `List<User>` 转成 `List<UserVO>`。

### 任务五：业务异常

定义 `BusinessException`，当用户不存在时抛出：

```java
throw new BusinessException("用户不存在");
```

## 4.22 面试常问点

### 问题一：String 为什么不能用 == 比较内容？

参考回答：

```text
== 比较的是引用地址，equals 比较的是字符串内容。
判断字符串内容是否相同应该使用 equals。
```

### 问题二：ArrayList 和数组有什么区别？

参考回答：

```text
数组长度固定，创建后不能改变。
ArrayList 长度可变，更适合业务列表数据处理。
```

### 问题三：List、Set、Map 有什么区别？

参考回答：

```text
List 是有序可重复集合。
Set 是不重复集合。
Map 是键值对集合，通过 key 查找 value。
```

### 问题四：为什么金额要使用 BigDecimal？

参考回答：

```text
float 和 double 有浮点精度问题，金额计算要求精确。
BigDecimal 可以更准确地处理金额加减乘除。
```

### 问题五：什么是泛型？

参考回答：

```text
泛型用于在编译期约束数据类型，例如 List<String> 表示列表中只能存放 String。
泛型可以提高类型安全，减少强制类型转换。
```

## 4.23 本章总结

本章学习了 Java 常用 API 和集合框架：

```text
String
StringBuilder
包装类
自动装箱和拆箱
BigDecimal
LocalDate / LocalDateTime
List
Set
Map
泛型
Stream API
Optional
异常处理
自定义业务异常
```

对前端开发人员来说，可以这样建立类比：

```text
Java List 类似前端数组。
Java Map 类似前端对象或 Map。
Java Stream 类似数组的 map、filter、reduce。
Java BigDecimal 用于解决金额精度问题。
Java LocalDateTime 用于处理后端时间字段。
Java 异常机制类似统一错误处理，但更严格。
```

这些内容会在后续章节反复出现。尤其是 `List<User>`、`Result<List<UserVO>>`、`BigDecimal`、`LocalDateTime`、`BusinessException`，都是后端项目高频内容。

下一章将进入 MySQL 与 SQL。那一章会从数据库、表、字段开始，讲到增删改查、条件查询、分页、多表关系、JOIN、索引和事务。

