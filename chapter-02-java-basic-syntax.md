# 第二章：Java 基础语法

## 2.1 本章目标

本章开始正式学习 Java 基础语法。

前端开发人员学习 Java 时，最容易出现的问题是：能看懂一点代码，但不知道为什么必须这样写。例如：

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

很多人第一次看到这段代码，会有这些疑问：

```text
为什么必须有 class？
public 是什么？
static 是什么？
void 是什么？
main 方法为什么这么写？
String[] args 是什么？
System.out.println 是什么？
```

本章不要求你一次性掌握所有底层原理，但要先建立 Java 程序的基本语法认知。

学完本章后，你应该能够：

- 看懂一个最简单的 Java 程序。
- 理解 Java 的变量声明方式。
- 区分基本数据类型和引用数据类型。
- 掌握八种基本数据类型。
- 理解 `String` 是引用类型。
- 掌握常见运算符和流程控制。
- 能定义和调用简单方法。
- 能使用数组保存一组数据。
- 能写出简单控制台练习程序。

## 2.2 Java 程序的基本结构

Java 程序通常从一个类开始。

最简单的 Java 程序：

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

逐行理解：

```java
public class HelloWorld {
```

这一行定义了一个类，类名叫 `HelloWorld`。Java 中大多数代码都必须写在类里面。

前端中你可以直接写：

```js
console.log('Hello JavaScript')
```

但 Java 不允许在类外面直接写业务代码。Java 更强调结构化和面向对象。

继续看：

```java
public static void main(String[] args) {
```

这是 Java 程序的入口方法。运行 Java 程序时，JVM 会从 `main` 方法开始执行。

暂时可以这样记：

```text
main 方法类似一个程序的启动入口。
```

最后：

```java
System.out.println("Hello Java");
```

这行代码表示向控制台输出一段文本。

前端类比：

```js
console.log('Hello JavaScript')
```

Java 对应：

```java
System.out.println("Hello Java");
```

注意 Java 每条语句末尾通常需要分号：

```java
System.out.println("Hello Java"); // 正确
System.out.println("Hello Java")  // 错误
```

## 2.3 class 是什么

`class` 表示类。Java 是面向对象语言，类是 Java 程序的基本组织单位。

前端中，你可以把一个模块写成：

```ts
export function getUserName(user) {
  return user.username
}
```

Java 中，方法通常要放在类里面：

```java
public class UserUtils {

    public static String getUserName(User user) {
        return user.getUsername();
    }
}
```

也就是说，Java 不鼓励把函数随便散落在文件中，而是通过类来组织代码。

类名通常使用大驼峰命名：

```text
HelloWorld
UserService
OrderController
ProductMapper
```

文件名通常要和 public 类名一致。

例如：

```java
public class HelloWorld {
}
```

文件名应该是：

```text
HelloWorld.java
```

这是 Java 初学者非常容易忽略的规则。

## 2.4 main 方法是什么

`main` 方法是 Java 程序的入口。

标准写法：

```java
public static void main(String[] args) {
}
```

先不用深入纠结每个关键字的底层含义，可以先这样理解：

- `public`：公开的，JVM 可以访问。
- `static`：静态的，不需要创建对象就能运行。
- `void`：没有返回值。
- `main`：固定的方法名，程序入口。
- `String[] args`：命令行参数。

一个完整示例：

```java
public class MainDemo {
    public static void main(String[] args) {
        System.out.println("程序启动了");
    }
}
```

运行结果：

```text
程序启动了
```

在 Spring Boot 项目中，也有入口方法：

```java
@SpringBootApplication
public class AdminApplication {
    public static void main(String[] args) {
        SpringApplication.run(AdminApplication.class, args);
    }
}
```

所以现在学习 `main` 方法，是为了后面看懂 Spring Boot 启动类。

## 2.5 注释

Java 注释有三种常见写法。

单行注释：

```java
// 这是单行注释
int age = 18;
```

多行注释：

```java
/*
  这是多行注释
  可以写多行说明
 */
```

文档注释：

```java
/**
 * 计算商品总价
 *
 * @param price 商品单价
 * @param count 商品数量
 * @return 商品总价
 */
public BigDecimal calculateTotal(BigDecimal price, Integer count) {
    return price.multiply(BigDecimal.valueOf(count));
}
```

文档注释通常用于类、方法、字段说明。真实项目中，不建议写大量无意义注释。

不好的注释：

```java
// 设置用户名
user.setUsername(username);
```

这类注释没有价值，因为代码已经说明了含义。

更有价值的注释应该解释业务原因：

```java
// 新用户默认禁用，需要管理员审核后才能登录
user.setStatus(0);
```

## 2.6 变量是什么

变量用于保存数据。

前端中：

```ts
let username = 'admin'
const age = 18
```

Java 中：

```java
String username = "admin";
int age = 18;
```

Java 声明变量时必须明确类型：

```java
类型 变量名 = 值;
```

例如：

```java
String nickname = "张三";
Integer score = 90;
Boolean enabled = true;
```

Java 是强类型语言，变量类型确定后，不能随意改变：

```java
int age = 18;
age = "18"; // 错误
```

TypeScript 中也类似：

```ts
let age: number = 18
age = '18' // 错误
```

但 Java 比 TypeScript 更严格，因为 Java 编译后运行在 JVM 上，类型系统是语言运行机制的重要部分。

## 2.7 变量命名规范

Java 变量通常使用小驼峰命名：

```java
String username = "admin";
Integer pageSize = 10;
Boolean loginEnabled = true;
```

不推荐：

```java
String UserName = "admin";
String user_name = "admin";
String yhxm = "admin";
```

真实项目中的命名应该清晰表达业务含义：

```java
Long userId;
String orderNo;
BigDecimal totalAmount;
LocalDateTime createTime;
```

前端人员要特别注意，Java 后端项目中命名不只是风格问题，还会影响数据库字段、接口参数、JSON 字段和代码可读性。

例如前端传参：

```json
{
  "pageNum": 1,
  "pageSize": 10
}
```

后端 DTO 通常这样写：

```java
public class PageQueryDTO {
    private Integer pageNum;
    private Integer pageSize;
}
```

字段名保持一致，可以减少前后端联调成本。

## 2.8 Java 数据类型总览

Java 数据类型分为两大类：

```text
基本数据类型
引用数据类型
```

基本数据类型有八种：

```text
byte
short
int
long
float
double
char
boolean
```

引用数据类型包括：

```text
类
对象
String
数组
集合
接口
枚举
```

示例：

```java
int age = 18;
long userId = 10001L;
double price = 99.9;
boolean enabled = true;
char level = 'A';

String username = "admin";
User user = new User();
int[] scores = {90, 80, 70};
```

其中：

- `int`、`long`、`double`、`boolean`、`char` 是基本类型。
- `String`、`User`、数组是引用类型。

简单理解：

```text
基本类型变量保存的是具体值。
引用类型变量保存的是对象的引用。
```

这句话暂时不用从内存层面完全理解，但要先记住：`String` 不是基本类型。

## 2.9 八种基本数据类型

Java 的八种基本数据类型如下：

| 类型 | 含义 | 示例 |
|---|---|---|
| byte | 很小的整数 | `byte b = 1;` |
| short | 短整数 | `short s = 100;` |
| int | 常用整数 | `int age = 18;` |
| long | 长整数 | `long id = 10001L;` |
| float | 单精度小数 | `float score = 95.5F;` |
| double | 双精度小数 | `double price = 99.9;` |
| char | 单个字符 | `char level = 'A';` |
| boolean | 布尔值 | `boolean enabled = true;` |

后端项目中最常见的是：

```java
int pageNum = 1;
long userId = 10001L;
double rate = 0.8;
boolean enabled = true;
```

但真实业务项目中，更常用包装类型：

```java
Integer pageNum;
Long userId;
BigDecimal amount;
Boolean enabled;
```

原因是包装类型可以表示 `null`，更适合接收前端参数和数据库字段。

例如前端查询参数可能没有传：

```text
GET /users?pageNum=1
```

后端 DTO：

```java
public class UserQueryDTO {
    private Integer pageNum;
    private Integer pageSize;
}
```

如果 `pageSize` 没传，它可以是 `null`。如果使用基本类型 `int`，默认值会是 `0`，有时会混淆“没传”和“传了 0”。

## 2.10 整数类型 byte、short、int、long

整数类型用于表示没有小数的数字。

最常用的是 `int` 和 `long`。

```java
int age = 18;
long userId = 10000000001L;
```

为什么 `long` 后面要加 `L`？

```java
long id = 10000000001; // 错误
```

Java 中整数默认按 `int` 处理，如果数字超过 `int` 范围，就会报错。加上 `L` 表示这是一个 `long` 类型字面量：

```java
long id = 10000000001L; // 正确
```

后端项目中，数据库主键 ID 通常使用 `Long`：

```java
private Long id;
private Long userId;
private Long roleId;
```

原因是：

- 数据库主键可能增长到很大。
- 分布式 ID 通常超过 `int` 范围。
- MySQL 中常用 `BIGINT` 对应 Java 的 `Long`。

示例：

```sql
id BIGINT PRIMARY KEY AUTO_INCREMENT
```

对应 Java：

```java
private Long id;
```

## 2.11 浮点类型 float 和 double

浮点类型用于表示小数。

```java
float score = 95.5F;
double price = 99.9;
```

`float` 后面通常要加 `F`：

```java
float score = 95.5; // 错误
float score = 95.5F; // 正确
```

因为 Java 中小数字面量默认是 `double`。

前端人员要特别注意：金额不要用 `float` 或 `double`。

例如：

```java
System.out.println(0.1 + 0.2);
```

输出可能不是精确的 `0.3`。

这和 JavaScript 中类似：

```js
console.log(0.1 + 0.2) // 0.30000000000000004
```

后端涉及金额时，应该使用 `BigDecimal`：

```java
BigDecimal price = new BigDecimal("99.90");
BigDecimal count = new BigDecimal("3");
BigDecimal total = price.multiply(count);
```

真实项目中商品价格、订单金额、支付金额都应该用 `BigDecimal`。

## 2.12 字符类型 char

`char` 表示单个字符。

```java
char level = 'A';
char gender = '男';
```

注意 `char` 使用单引号：

```java
char level = 'A'; // 正确
char level = "A"; // 错误
```

`String` 使用双引号：

```java
String level = "A";
```

区别：

```text
char：单个字符。
String：字符串，可以包含零个、一个或多个字符。
```

真实后端项目中，`char` 用得不多。状态、类型、编码通常更推荐使用 `String`、`Integer` 或枚举。

例如：

```java
private Integer status; // 0 禁用，1 启用
```

或者：

```java
private String orderStatus; // PENDING, PAID, CANCELED
```

## 2.13 布尔类型 boolean

`boolean` 只有两个值：

```java
boolean enabled = true;
boolean deleted = false;
```

业务中常用于判断：

```java
if (enabled) {
    System.out.println("账号已启用");
} else {
    System.out.println("账号已禁用");
}
```

前端中：

```ts
const enabled = true
if (enabled) {
  console.log('账号已启用')
}
```

Java 中基本一致。

真实项目中，布尔字段命名要清晰：

```java
private Boolean enabled;
private Boolean deleted;
private Boolean admin;
```

但数据库中不一定直接使用 boolean，MySQL 常用 `TINYINT`：

```sql
enabled TINYINT DEFAULT 1 COMMENT '是否启用：0 否，1 是'
```

对应 Java：

```java
private Integer enabled;
```

或者：

```java
private Boolean enabled;
```

具体选哪种，要看团队规范。

## 2.14 引用数据类型

引用数据类型不是八种基本类型。常见引用类型包括：

- 类。
- 对象。
- `String`。
- 数组。
- 集合。
- 接口。
- 枚举。

示例：

```java
String username = "admin";
User user = new User();
List<String> roles = new ArrayList<>();
int[] scores = {90, 80, 70};
```

其中 `User` 是自定义类：

```java
public class User {
    private Long id;
    private String username;
}
```

创建对象：

```java
User user = new User();
```

前端类比：

```ts
const user = {
  id: 1,
  username: 'admin'
}
```

区别是 Java 更强调先定义类，再创建对象。

## 2.15 String 字符串

`String` 是 Java 中最常用的引用类型之一。

```java
String username = "admin";
String nickname = "管理员";
```

注意：`String` 首字母大写。

错误写法：

```java
string username = "admin"; // 错误
```

正确写法：

```java
String username = "admin";
```

字符串拼接：

```java
String username = "admin";
String message = "欢迎你，" + username;
System.out.println(message);
```

输出：

```text
欢迎你，admin
```

字符串比较是初学者最容易出错的地方。

错误示例：

```java
String username = new String("admin");

if (username == "admin") {
    System.out.println("是管理员");
}
```

Java 中比较字符串内容应该使用 `equals`：

```java
if ("admin".equals(username)) {
    System.out.println("是管理员");
}
```

推荐把常量写在前面：

```java
"admin".equals(username)
```

这样即使 `username` 是 `null`，也不会出现空指针异常。

## 2.16 基本类型和引用类型的区别

这是 Java 基础中非常重要的概念。

示例：

```java
int age = 18;
String username = "admin";
User user = new User();
```

其中：

- `int` 是基本类型。
- `String` 是引用类型。
- `User` 是引用类型。

简单理解：

```text
基本类型变量直接保存值。
引用类型变量保存对象的引用。
```

再看一个例子：

```java
User user1 = new User();
user1.setUsername("admin");

User user2 = user1;
user2.setUsername("root");

System.out.println(user1.getUsername());
```

输出：

```text
root
```

为什么？

因为 `user1` 和 `user2` 指向的是同一个对象。

前端中也有类似情况：

```ts
const user1 = { username: 'admin' }
const user2 = user1
user2.username = 'root'
console.log(user1.username) // root
```

所以前端人员理解引用类型并不难，只要把 Java 对象和 JavaScript 对象的引用关系联系起来即可。

## 2.17 类型转换

Java 类型转换分为：

```text
自动类型转换
强制类型转换
字符串和数字转换
```

自动类型转换：

```java
int num = 100;
long bigNum = num;
```

小范围类型可以自动转成大范围类型。

强制类型转换：

```java
double price = 99.9;
int intPrice = (int) price;
System.out.println(intPrice);
```

输出：

```text
99
```

注意小数部分会丢失。

字符串转数字：

```java
String pageNumText = "1";
Integer pageNum = Integer.valueOf(pageNumText);
```

数字转字符串：

```java
Integer pageNum = 1;
String pageNumText = String.valueOf(pageNum);
```

前端类比：

```ts
const pageNum = Number('1')
const text = String(1)
```

Java 中如果字符串不是合法数字，会报错：

```java
Integer.valueOf("abc");
```

运行时报：

```text
NumberFormatException
```

所以真实项目中，接口参数一般交给 Spring Boot 自动转换和校验，不建议手动随意转换。

## 2.18 常量 final

Java 中可以用 `final` 定义常量。

```java
final int MAX_PAGE_SIZE = 100;
```

常量一旦赋值，不能再次修改：

```java
final int MAX_PAGE_SIZE = 100;
MAX_PAGE_SIZE = 200; // 错误
```

前端类比：

```ts
const MAX_PAGE_SIZE = 100
```

真实后端项目中，常量通常写成：

```java
public class UserConstants {
    public static final Integer STATUS_ENABLED = 1;
    public static final Integer STATUS_DISABLED = 0;
}
```

使用：

```java
if (UserConstants.STATUS_DISABLED.equals(user.getStatus())) {
    throw new BusinessException("账号已被禁用");
}
```

常量的意义是避免魔法值。

不推荐：

```java
if (user.getStatus() == 0) {
    throw new BusinessException("账号已被禁用");
}
```

更推荐：

```java
if (UserConstants.STATUS_DISABLED.equals(user.getStatus())) {
    throw new BusinessException("账号已被禁用");
}
```

## 2.19 运算符

Java 常见运算符包括：

- 算术运算符。
- 赋值运算符。
- 比较运算符。
- 逻辑运算符。
- 三元运算符。

算术运算：

```java
int a = 10;
int b = 3;

System.out.println(a + b); // 13
System.out.println(a - b); // 7
System.out.println(a * b); // 30
System.out.println(a / b); // 3
System.out.println(a % b); // 1
```

注意整数相除：

```java
System.out.println(10 / 3);
```

输出：

```text
3
```

如果想得到小数：

```java
System.out.println(10.0 / 3);
```

比较运算：

```java
int age = 18;

System.out.println(age > 18);
System.out.println(age >= 18);
System.out.println(age == 18);
System.out.println(age != 18);
```

逻辑运算：

```java
int age = 20;
boolean enabled = true;

if (age >= 18 && enabled) {
    System.out.println("允许登录");
}
```

三元运算：

```java
Integer status = 1;
String statusText = status == 1 ? "启用" : "禁用";
```

前端中也类似：

```ts
const statusText = status === 1 ? '启用' : '禁用'
```

## 2.20 流程控制 if

`if` 用于条件判断。

```java
int age = 20;

if (age >= 18) {
    System.out.println("成年人");
} else {
    System.out.println("未成年人");
}
```

业务示例：判断账号是否可登录。

```java
Integer status = 1;

if (status == 1) {
    System.out.println("账号正常，可以登录");
} else {
    System.out.println("账号禁用，不能登录");
}
```

多条件判断：

```java
Integer orderStatus = 2;

if (orderStatus == 0) {
    System.out.println("待支付");
} else if (orderStatus == 1) {
    System.out.println("已支付");
} else if (orderStatus == 2) {
    System.out.println("已发货");
} else {
    System.out.println("未知状态");
}
```

真实项目中，如果状态很多，不建议大量堆 `if else`，后面会学习枚举和策略优化。

## 2.21 流程控制 switch

`switch` 适合多个固定值分支。

```java
int status = 1;

switch (status) {
    case 0:
        System.out.println("待支付");
        break;
    case 1:
        System.out.println("已支付");
        break;
    case 2:
        System.out.println("已发货");
        break;
    default:
        System.out.println("未知状态");
}
```

`break` 很重要。如果忘记写，可能继续执行后面的分支。

Java 新版本支持更简洁的写法：

```java
int status = 1;

String text = switch (status) {
    case 0 -> "待支付";
    case 1 -> "已支付";
    case 2 -> "已发货";
    default -> "未知状态";
};
```

学习初期先掌握传统写法即可，因为很多公司项目仍然能看到传统 `switch`。

## 2.22 循环 for

`for` 循环用于重复执行代码。

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}
```

输出：

```text
0
1
2
3
4
```

遍历数组：

```java
String[] names = {"张三", "李四", "王五"};

for (int i = 0; i < names.length; i++) {
    System.out.println(names[i]);
}
```

增强 for：

```java
for (String name : names) {
    System.out.println(name);
}
```

前端类比：

```ts
const names = ['张三', '李四', '王五']

for (const name of names) {
  console.log(name)
}
```

后端业务中，循环常用于处理列表数据：

```java
for (User user : users) {
    System.out.println(user.getUsername());
}
```

## 2.23 循环 while 和 do while

`while` 先判断条件，再执行：

```java
int count = 0;

while (count < 3) {
    System.out.println(count);
    count++;
}
```

`do while` 先执行一次，再判断条件：

```java
int count = 0;

do {
    System.out.println(count);
    count++;
} while (count < 3);
```

真实后端业务中，`for` 和增强 `for` 更常见，`while` 常用于不确定循环次数的场景。

例如分页同步数据：

```java
int pageNum = 1;
boolean hasNext = true;

while (hasNext) {
    System.out.println("同步第 " + pageNum + " 页数据");
    pageNum++;

    if (pageNum > 5) {
        hasNext = false;
    }
}
```

## 2.24 break 和 continue

`break` 用于结束整个循环。

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;
    }
    System.out.println(i);
}
```

输出：

```text
0
1
2
3
4
```

`continue` 用于跳过本次循环，继续下一次。

```java
for (int i = 0; i < 5; i++) {
    if (i == 2) {
        continue;
    }
    System.out.println(i);
}
```

输出：

```text
0
1
3
4
```

业务示例：跳过禁用用户。

```java
for (User user : users) {
    if (user.getStatus() == 0) {
        continue;
    }
    System.out.println("发送通知给：" + user.getUsername());
}
```

## 2.25 方法基础

方法用于封装一段可复用逻辑。

前端函数：

```ts
function add(a: number, b: number): number {
  return a + b
}
```

Java 方法：

```java
public int add(int a, int b) {
    return a + b;
}
```

方法基本结构：

```text
访问修饰符 返回值类型 方法名(参数列表) {
    方法体
}
```

示例：

```java
public String buildWelcomeMessage(String username) {
    return "欢迎你，" + username;
}
```

调用：

```java
String message = buildWelcomeMessage("admin");
System.out.println(message);
```

没有返回值的方法使用 `void`：

```java
public void printUserName(String username) {
    System.out.println(username);
}
```

业务示例：判断用户是否成年。

```java
public boolean isAdult(Integer age) {
    return age != null && age >= 18;
}
```

## 2.26 方法重载

方法重载指同一个类中，方法名相同，但参数不同。

```java
public int add(int a, int b) {
    return a + b;
}

public int add(int a, int b, int c) {
    return a + b + c;
}

public double add(double a, double b) {
    return a + b;
}
```

Java 会根据参数类型和参数数量选择调用哪个方法。

注意，只有返回值不同不算重载：

```java
public int getValue() {
    return 1;
}

public String getValue() {
    return "1";
}
```

这是错误的，因为方法名和参数列表相同。

真实项目中，重载常用于工具类方法，例如：

```java
public class UserUtils {

    public static String getDisplayName(String username) {
        return username;
    }

    public static String getDisplayName(String username, String nickname) {
        return nickname == null ? username : nickname;
    }
}
```

## 2.27 数组基础

数组用于保存一组相同类型的数据。

```java
String[] names = {"张三", "李四", "王五"};
```

访问数组元素：

```java
System.out.println(names[0]);
```

输出：

```text
张三
```

数组下标从 0 开始，这一点和前端数组一样。

前端：

```ts
const names = ['张三', '李四', '王五']
console.log(names[0])
```

Java 数组长度：

```java
System.out.println(names.length);
```

遍历数组：

```java
for (String name : names) {
    System.out.println(name);
}
```

数组越界错误：

```java
System.out.println(names[3]);
```

因为数组只有 3 个元素，下标是 `0`、`1`、`2`。访问 `3` 会报：

```text
ArrayIndexOutOfBoundsException
```

真实后端项目中，数组用得没有集合多。后面会重点学习 `List`，它更适合处理业务列表数据。

## 2.28 控制台输入 Scanner

`Scanner` 可以从控制台接收用户输入。

```java
import java.util.Scanner;

public class ScannerDemo {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("请输入用户名：");
        String username = scanner.nextLine();

        System.out.println("你好，" + username);
    }
}
```

运行示例：

```text
请输入用户名：
admin
你好，admin
```

接收数字：

```java
System.out.println("请输入年龄：");
int age = scanner.nextInt();
```

学习控制台输入的意义不是以后都写控制台项目，而是帮助你练习基础语法。

真实后端项目中，数据更多来自 HTTP 请求：

```java
@PostMapping("/users")
public Result<Void> create(@RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

这里的 `dto` 就类似从外部接收输入，只是来源从控制台变成了 HTTP 请求。

## 2.29 综合案例：用户注册信息校验

需求：

```text
接收用户名、密码、年龄。
用户名不能为空。
密码长度不能小于 6 位。
年龄必须大于等于 18。
全部满足则输出注册成功。
```

代码：

```java
import java.util.Scanner;

public class RegisterDemo {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("请输入用户名：");
        String username = scanner.nextLine();

        System.out.println("请输入密码：");
        String password = scanner.nextLine();

        System.out.println("请输入年龄：");
        int age = scanner.nextInt();

        if (username == null || username.length() == 0) {
            System.out.println("用户名不能为空");
            return;
        }

        if (password.length() < 6) {
            System.out.println("密码长度不能小于 6 位");
            return;
        }

        if (age < 18) {
            System.out.println("年龄必须大于等于 18");
            return;
        }

        System.out.println("注册成功");
    }
}
```

这个案例虽然简单，但已经包含后端常见思想：

- 接收外部输入。
- 校验参数。
- 不满足条件时提前返回。
- 满足条件后继续处理业务。

后面进入 Spring Boot 后，这段逻辑会演变为：

```java
@PostMapping("/register")
public Result<Void> register(@RequestBody RegisterDTO dto) {
    userService.register(dto);
    return Result.success();
}
```

参数校验也会升级为：

```java
public class RegisterDTO {
    @NotBlank(message = "用户名不能为空")
    private String username;

    @Size(min = 6, message = "密码长度不能小于 6 位")
    private String password;

    @Min(value = 18, message = "年龄必须大于等于 18")
    private Integer age;
}
```

所以基础语法不是孤立知识，它会逐步演变成真实后端项目代码。

## 2.30 常见错误

### 错误一：String 写成 string

错误：

```java
string username = "admin";
```

正确：

```java
String username = "admin";
```

### 错误二：long 忘记加 L

错误：

```java
long id = 10000000001;
```

正确：

```java
long id = 10000000001L;
```

### 错误三：字符串比较使用 ==

错误：

```java
if (username == "admin") {
    System.out.println("管理员");
}
```

正确：

```java
if ("admin".equals(username)) {
    System.out.println("管理员");
}
```

### 错误四：数组越界

错误：

```java
String[] names = {"张三", "李四"};
System.out.println(names[2]);
```

正确：

```java
System.out.println(names[1]);
```

### 错误五：忘记分号

错误：

```java
System.out.println("Hello")
```

正确：

```java
System.out.println("Hello");
```

## 2.31 本章练习任务

### 任务一：基础变量练习

定义一个商品信息，包括：

- 商品 ID。
- 商品名称。
- 商品价格。
- 商品库存。
- 是否上架。

参考：

```java
Long productId = 1L;
String productName = "机械键盘";
BigDecimal price = new BigDecimal("299.00");
Integer stock = 100;
Boolean enabled = true;
```

输出商品信息。

### 任务二：订单金额计算

定义商品单价和购买数量，计算订单总金额。

要求使用 `BigDecimal`，不要使用 `double`。

### 任务三：用户状态判断

定义用户状态：

```text
0 禁用
1 启用
2 锁定
```

根据状态输出对应文字。

要求分别使用 `if else` 和 `switch` 实现。

### 任务四：数组遍历

定义一个用户名称数组：

```java
String[] usernames = {"admin", "test", "guest"};
```

遍历并输出每个用户名。

### 任务五：注册校验

基于本章综合案例，增加手机号校验：

```text
手机号不能为空。
手机号长度必须是 11 位。
```

## 2.32 面试常问点

### 问题一：Java 有哪些基本数据类型？

参考回答：

```text
Java 有八种基本数据类型：byte、short、int、long、float、double、char、boolean。
```

### 问题二：基本类型和引用类型有什么区别？

参考回答：

```text
基本类型变量保存的是具体值，例如 int、long、boolean。
引用类型变量保存的是对象的引用，例如 String、数组、自定义类对象。
引用类型可以为 null，基本类型不能为 null。
```

### 问题三：String 是基本类型吗？

参考回答：

```text
不是。String 是引用类型，是 Java 标准库提供的字符串类。
```

### 问题四：为什么字符串比较不能随便用 ==？

参考回答：

```text
== 比较的是引用地址，equals 比较的是字符串内容。
判断两个字符串内容是否相等，应该使用 equals。
```

### 问题五：为什么金额计算推荐使用 BigDecimal？

参考回答：

```text
float 和 double 是浮点数，存在精度问题。
金额计算需要精确结果，因此推荐使用 BigDecimal。
```

## 2.33 本章总结

本章学习了 Java 基础语法，包括：

```text
Java 程序结构
class
main 方法
注释
变量
基本数据类型
引用数据类型
String
类型转换
final 常量
运算符
if / switch
for / while
break / continue
方法
数组
Scanner
```

前端人员学习本章时，最重要的是建立几组核心认知：

```text
JavaScript 可以直接写脚本，Java 代码通常写在类中。
TypeScript 有类型，Java 类型更严格。
String 不是基本类型，而是引用类型。
Java 字符串比较要用 equals。
金额计算不要用 double，要用 BigDecimal。
方法是后端业务逻辑封装的基础。
流程控制会在业务判断中大量出现。
```

下一章将进入面向对象编程。那一章会详细讲解类、对象、字段、方法、构造方法、封装、继承、多态、接口、抽象类和 Lombok。面向对象是理解 Entity、DTO、VO、Service、Mapper 等后端结构的基础。

