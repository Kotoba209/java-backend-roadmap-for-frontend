# 第三章：面向对象编程

## 3.1 本章目标

Java 是典型的面向对象语言。对于 Java 后端开发来说，面向对象不是理论概念，而是每天都会用到的基本功。

后端项目中的很多代码都依赖面向对象：

```text
User 表示用户。
Order 表示订单。
Product 表示商品。
LoginDTO 表示登录请求参数。
UserVO 表示返回给前端的用户数据。
UserService 表示用户业务逻辑。
UserMapper 表示用户数据访问。
```

如果不了解类、对象、封装、继承、多态、接口这些概念，后面看 Spring Boot、MyBatis、DTO、VO、Service、Mapper 时会非常吃力。

学完本章后，你应该能够：

- 理解类和对象的关系。
- 能定义 Java 类。
- 能创建对象并调用方法。
- 理解字段、方法、构造方法。
- 理解封装的意义。
- 掌握 getter 和 setter。
- 理解继承和方法重写。
- 理解多态的基本使用。
- 区分接口和抽象类。
- 知道 Lombok 如何减少样板代码。
- 能用面向对象方式描述用户、商品、订单等业务模型。

## 3.2 类和对象是什么

前端中经常会写对象：

```ts
const user = {
  id: 1,
  username: 'admin',
  nickname: '管理员'
}
```

这个对象描述了一个用户。

Java 中通常不会直接随手写一个对象，而是先定义类：

```java
public class User {
    private Long id;
    private String username;
    private String nickname;
}
```

然后通过类创建对象：

```java
User user = new User();
```

可以这样理解：

```text
类是对象的模板。
对象是类创建出来的具体实例。
```

类和对象的关系类似：

```text
设计图纸 -> 房子
商品模板 -> 某一个具体商品
用户类 -> 某一个具体用户
```

在后端业务中，`User` 类只是描述用户应该有哪些字段，而具体的某个用户对象才代表一条真实用户数据。

## 3.3 定义一个 Java 类

定义用户类：

```java
public class User {
    private Long id;
    private String username;
    private String nickname;
    private Integer age;
}
```

这段代码包含几个部分：

- `public class User`：定义一个公开的类，类名是 `User`。
- `private Long id`：定义用户 ID 字段。
- `private String username`：定义用户名字段。
- `private String nickname`：定义昵称字段。
- `private Integer age`：定义年龄字段。

Java 类名通常使用大驼峰：

```text
User
Product
Order
UserService
OrderController
```

字段名通常使用小驼峰：

```text
userId
username
orderNo
createTime
totalAmount
```

真实后端项目中，一个数据库表通常会对应一个实体类。

例如用户表：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50),
    nickname VARCHAR(50),
    age INT
);
```

对应 Java 实体类：

```java
public class User {
    private Long id;
    private String username;
    private String nickname;
    private Integer age;
}
```

这就是数据库数据和 Java 对象之间的基本映射关系。

## 3.4 字段和方法

类中可以包含字段和方法。

字段表示对象的数据，方法表示对象的行为。

```java
public class User {
    private String username;
    private Integer age;

    public void sayHello() {
        System.out.println("你好，我是 " + username);
    }

    public boolean isAdult() {
        return age != null && age >= 18;
    }
}
```

使用：

```java
User user = new User();
user.sayHello();
```

不过这里有一个问题：`username` 和 `age` 是 `private`，外部不能直接赋值。

下面这样写会报错：

```java
User user = new User();
user.username = "admin"; // 错误
```

这就引出了封装和 getter / setter。

## 3.5 封装是什么

封装是面向对象的核心思想之一。

简单来说，封装就是：

```text
把对象内部数据保护起来，不允许外部随意直接修改。
外部必须通过对象提供的方法访问或修改数据。
```

前端对象通常可以直接改字段：

```ts
const user = {
  username: 'admin',
  age: 18
}

user.age = -1
```

这在语法上没问题，但业务上明显不合理。年龄不应该是负数。

Java 中可以通过 `private` 字段和 setter 方法控制修改逻辑：

```java
public class User {
    private String username;
    private Integer age;

    public void setAge(Integer age) {
        if (age == null || age < 0) {
            throw new IllegalArgumentException("年龄不能小于 0");
        }
        this.age = age;
    }

    public Integer getAge() {
        return age;
    }
}
```

使用：

```java
User user = new User();
user.setAge(18);
System.out.println(user.getAge());
```

如果传入非法年龄：

```java
user.setAge(-1);
```

会抛出异常：

```text
IllegalArgumentException: 年龄不能小于 0
```

封装的意义不是让代码变复杂，而是让对象自己保护自己的状态。

## 3.6 getter 和 setter

Java 中字段通常用 `private` 修饰，然后通过 getter 和 setter 访问。

完整示例：

```java
public class Product {
    private Long id;
    private String name;
    private BigDecimal price;
    private Integer stock;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        if (price == null || price.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("价格不能小于 0");
        }
        this.price = price;
    }

    public Integer getStock() {
        return stock;
    }

    public void setStock(Integer stock) {
        if (stock == null || stock < 0) {
            throw new IllegalArgumentException("库存不能小于 0");
        }
        this.stock = stock;
    }
}
```

使用：

```java
Product product = new Product();
product.setId(1L);
product.setName("机械键盘");
product.setPrice(new BigDecimal("299.00"));
product.setStock(100);

System.out.println(product.getName());
```

后端项目中，DTO、VO、Entity 基本都会有 getter 和 setter。很多框架也依赖这些方法完成参数绑定、JSON 序列化、数据库映射。

## 3.7 this 关键字

`this` 表示当前对象。

看这个方法：

```java
public void setUsername(String username) {
    this.username = username;
}
```

左边的 `this.username` 表示当前对象的字段。右边的 `username` 表示方法参数。

如果不写 `this`：

```java
public void setUsername(String username) {
    username = username;
}
```

这行代码只是把参数赋值给参数自己，没有修改对象字段。

前端类比：

```ts
class User {
  username: string

  setUsername(username: string) {
    this.username = username
  }
}
```

Java 和 TypeScript 在这里很相似。

## 3.8 构造方法

构造方法用于创建对象时初始化数据。

无参构造：

```java
public class User {
    private String username;

    public User() {
    }
}
```

有参构造：

```java
public class User {
    private String username;
    private Integer age;

    public User(String username, Integer age) {
        this.username = username;
        this.age = age;
    }
}
```

使用：

```java
User user = new User("admin", 18);
```

如果没有写任何构造方法，Java 会默认提供一个无参构造。  
但如果你写了有参构造，Java 就不会自动提供无参构造。

例如：

```java
public class User {
    private String username;

    public User(String username) {
        this.username = username;
    }
}
```

此时下面代码会报错：

```java
User user = new User(); // 错误
```

真实后端项目中，很多框架需要无参构造来创建对象，所以实体类、DTO、VO 通常要保留无参构造。

## 3.9 方法返回对象

后端业务中，方法经常返回对象。

例如根据用户名创建用户：

```java
public User createUser(String username, String nickname) {
    User user = new User();
    user.setUsername(username);
    user.setNickname(nickname);
    return user;
}
```

前端类比：

```ts
function createUser(username: string, nickname: string) {
  return {
    username,
    nickname
  }
}
```

Java 中返回对象时，要明确返回类型：

```java
public User createUser(...)
```

这也是强类型语言的特点。返回什么类型，方法签名里必须写清楚。

## 3.10 继承

继承表示一个类可以复用另一个类的字段和方法。

父类：

```java
public class BaseEntity {
    private Long id;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public LocalDateTime getCreateTime() {
        return createTime;
    }

    public void setCreateTime(LocalDateTime createTime) {
        this.createTime = createTime;
    }

    public LocalDateTime getUpdateTime() {
        return updateTime;
    }

    public void setUpdateTime(LocalDateTime updateTime) {
        this.updateTime = updateTime;
    }
}
```

子类：

```java
public class User extends BaseEntity {
    private String username;
    private String nickname;
}
```

`User` 继承了 `BaseEntity`，所以它也拥有 `id`、`createTime`、`updateTime`。

真实项目中，很多实体类都有公共字段：

```text
id
createTime
updateTime
createBy
updateBy
deleted
```

这些字段可以抽取到父类中，减少重复代码。

## 3.11 方法重写

子类可以重写父类的方法。

父类：

```java
public class User {
    public String getRoleName() {
        return "普通用户";
    }
}
```

子类：

```java
public class AdminUser extends User {
    @Override
    public String getRoleName() {
        return "管理员";
    }
}
```

使用：

```java
AdminUser adminUser = new AdminUser();
System.out.println(adminUser.getRoleName());
```

输出：

```text
管理员
```

`@Override` 表示这个方法是重写父类方法。建议始终加上它，这样如果方法名或参数写错，编译器会提醒你。

## 3.12 super 关键字

`super` 表示父类。

调用父类构造方法：

```java
public class AdminUser extends User {

    public AdminUser(String username) {
        super(username);
    }
}
```

调用父类方法：

```java
public class AdminUser extends User {

    @Override
    public String getRoleName() {
        return super.getRoleName() + " / 管理员";
    }
}
```

`this` 表示当前对象，`super` 表示父类部分。

```text
this：当前对象。
super：父类对象能力。
```

## 3.13 多态

多态表示同一个父类引用，可以指向不同子类对象，并表现出不同的行为。

父类：

```java
public class Payment {
    public void pay() {
        System.out.println("发起支付");
    }
}
```

子类：

```java
public class AliPay extends Payment {
    @Override
    public void pay() {
        System.out.println("支付宝支付");
    }
}
```

```java
public class WeChatPay extends Payment {
    @Override
    public void pay() {
        System.out.println("微信支付");
    }
}
```

使用：

```java
Payment payment = new AliPay();
payment.pay();

payment = new WeChatPay();
payment.pay();
```

输出：

```text
支付宝支付
微信支付
```

后端项目中，多态常用于扩展业务逻辑。例如不同支付方式、不同消息发送方式、不同文件存储方式都可以通过多态处理。

## 3.14 抽象类

抽象类用 `abstract` 修饰，不能直接创建对象。

```java
public abstract class MessageSender {

    public abstract void send(String message);

    public void log(String message) {
        System.out.println("发送消息：" + message);
    }
}
```

子类实现抽象方法：

```java
public class SmsSender extends MessageSender {
    @Override
    public void send(String message) {
        System.out.println("发送短信：" + message);
    }
}
```

抽象类适合表达：

```text
有一部分通用逻辑
也有一部分必须由子类自己实现
```

例如消息发送器：

- 记录日志是通用逻辑。
- 短信、邮件、站内信的具体发送方式不同。

## 3.15 接口

接口用 `interface` 定义。

```java
public interface MessageSender {
    void send(String message);
}
```

实现类：

```java
public class SmsSender implements MessageSender {
    @Override
    public void send(String message) {
        System.out.println("发送短信：" + message);
    }
}
```

```java
public class EmailSender implements MessageSender {
    @Override
    public void send(String message) {
        System.out.println("发送邮件：" + message);
    }
}
```

使用接口：

```java
public class NotificationService {

    private final MessageSender messageSender;

    public NotificationService(MessageSender messageSender) {
        this.messageSender = messageSender;
    }

    public void notifyUser(String message) {
        messageSender.send(message);
    }
}
```

这里 `NotificationService` 不关心具体是短信还是邮件，只关心有一个对象能发送消息。

这就是接口的价值：

```text
面向接口编程，而不是面向具体实现编程。
```

## 3.16 Java interface 和 TypeScript interface 的区别

前端人员容易把 Java interface 和 TypeScript interface 混为一谈。

TypeScript 中：

```ts
interface User {
  id: number
  username: string
}
```

它主要用于描述对象结构。

Java 中接口更多用于定义行为规范：

```java
public interface UserService {
    UserVO getById(Long id);
    void create(UserCreateDTO dto);
}
```

实现类：

```java
public class UserServiceImpl implements UserService {

    @Override
    public UserVO getById(Long id) {
        return null;
    }

    @Override
    public void create(UserCreateDTO dto) {
    }
}
```

对照理解：

```text
TypeScript interface：常用于描述数据结构。
Java interface：常用于描述一组行为能力。
```

在 Java 后端中，`UserService` 通常是接口，`UserServiceImpl` 是具体实现。

## 3.17 接口和抽象类怎么选

简单规则：

```text
只定义能力规范，优先用接口。
既有公共字段或公共方法，又有子类差异，考虑抽象类。
```

接口示例：

```java
public interface FileStorageService {
    String upload(byte[] bytes, String filename);
}
```

不同实现：

```text
LocalFileStorageService：上传到本地磁盘。
OssFileStorageService：上传到对象存储。
```

抽象类示例：

```java
public abstract class AbstractImportService {

    public void importData() {
        validateFile();
        parseFile();
        saveData();
    }

    protected void validateFile() {
        System.out.println("校验文件");
    }

    protected abstract void parseFile();

    protected abstract void saveData();
}
```

这里导入流程固定，但解析和保存逻辑不同，就适合抽象类。

## 3.18 static 关键字

`static` 表示静态成员，属于类本身，不属于某一个对象。

静态变量：

```java
public class UserConstants {
    public static final Integer STATUS_ENABLED = 1;
    public static final Integer STATUS_DISABLED = 0;
}
```

使用：

```java
if (UserConstants.STATUS_ENABLED.equals(user.getStatus())) {
    System.out.println("用户启用");
}
```

静态方法：

```java
public class StringUtils {
    public static boolean isBlank(String text) {
        return text == null || text.trim().length() == 0;
    }
}
```

使用：

```java
if (StringUtils.isBlank(username)) {
    System.out.println("用户名不能为空");
}
```

工具类和常量类中经常使用 `static`。

## 3.19 final 关键字

`final` 表示最终的、不可改变的。

修饰变量：

```java
final int maxPageSize = 100;
```

变量不能再次赋值：

```java
maxPageSize = 200; // 错误
```

修饰方法：

```java
public final void validate() {
}
```

方法不能被子类重写。

修饰类：

```java
public final class StringUtils {
}
```

类不能被继承。

真实项目中最常见的是常量：

```java
public static final String TOKEN_HEADER = "Authorization";
```

## 3.20 包和 import

Java 使用 `package` 组织代码。

```java
package com.example.admin.entity;

public class User {
}
```

如果在另一个包中使用 `User`，需要导入：

```java
import com.example.admin.entity.User;

public class UserService {
    private User user;
}
```

后端项目常见包结构：

```text
com.example.admin
├── controller
├── service
├── service.impl
├── mapper
├── entity
├── dto
├── vo
├── config
├── common
└── exception
```

包名通常全部小写。

不推荐：

```text
com.example.Admin.Entity
```

推荐：

```text
com.example.admin.entity
```

## 3.21 访问修饰符

Java 常见访问修饰符：

| 修饰符 | 访问范围 |
|---|---|
| public | 任意位置可访问 |
| protected | 同包或子类可访问 |
| 默认 | 同包可访问 |
| private | 当前类内部可访问 |

最常见用法：

```java
public class User {
    private Long id;
    private String username;

    public Long getId() {
        return id;
    }
}
```

后端项目中一般遵循：

- 类通常是 `public`。
- 字段通常是 `private`。
- 对外方法通常是 `public`。
- 工具方法如果只在类内部使用，可以是 `private`。

## 3.22 Lombok 入门

手写 getter、setter、构造方法会让代码很长。

Lombok 可以通过注解减少样板代码。

没有 Lombok：

```java
public class User {
    private Long id;
    private String username;

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
}
```

使用 Lombok：

```java
import lombok.Data;

@Data
public class User {
    private Long id;
    private String username;
}
```

`@Data` 会生成：

- getter。
- setter。
- toString。
- equals。
- hashCode。

常见 Lombok 注解：

```java
@Getter
@Setter
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
```

注意：学习初期建议先手写 getter 和 setter，理解原理后再使用 Lombok。

## 3.23 面向对象综合案例：商品对象

需求：

```text
定义商品类 Product。
包含商品 ID、名称、价格、库存、是否上架。
价格不能小于 0。
库存不能小于 0。
提供一个方法计算购买指定数量时的总金额。
```

代码：

```java
import java.math.BigDecimal;

public class Product {
    private Long id;
    private String name;
    private BigDecimal price;
    private Integer stock;
    private Boolean enabled;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        if (price == null || price.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("价格不能小于 0");
        }
        this.price = price;
    }

    public Integer getStock() {
        return stock;
    }

    public void setStock(Integer stock) {
        if (stock == null || stock < 0) {
            throw new IllegalArgumentException("库存不能小于 0");
        }
        this.stock = stock;
    }

    public Boolean getEnabled() {
        return enabled;
    }

    public void setEnabled(Boolean enabled) {
        this.enabled = enabled;
    }

    public BigDecimal calculateTotal(Integer count) {
        if (count == null || count <= 0) {
            throw new IllegalArgumentException("购买数量必须大于 0");
        }
        if (count > stock) {
            throw new IllegalArgumentException("库存不足");
        }
        return price.multiply(BigDecimal.valueOf(count));
    }
}
```

测试：

```java
import java.math.BigDecimal;

public class ProductTest {
    public static void main(String[] args) {
        Product product = new Product();
        product.setId(1L);
        product.setName("机械键盘");
        product.setPrice(new BigDecimal("299.00"));
        product.setStock(100);
        product.setEnabled(true);

        BigDecimal total = product.calculateTotal(2);
        System.out.println(total);
    }
}
```

输出：

```text
598.00
```

这个案例体现了面向对象的几个重点：

- 用类描述业务对象。
- 用字段保存对象状态。
- 用 setter 控制数据合法性。
- 用方法封装业务行为。
- 使用 BigDecimal 处理金额。

## 3.24 常见错误

### 错误一：类名和文件名不一致

错误：

```text
文件名：User.java
public class Product {
}
```

正确：

```text
文件名：Product.java
public class Product {
}
```

### 错误二：忘记创建对象就调用实例方法

错误：

```java
User.setUsername("admin");
```

正确：

```java
User user = new User();
user.setUsername("admin");
```

### 错误三：字段全部 public

不推荐：

```java
public class User {
    public String username;
}
```

推荐：

```java
public class User {
    private String username;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
```

### 错误四：重写方法不加 @Override

不推荐：

```java
public String getRoleName() {
    return "管理员";
}
```

推荐：

```java
@Override
public String getRoleName() {
    return "管理员";
}
```

## 3.25 练习任务

### 任务一：定义用户类

定义 `User` 类，包含：

- id。
- username。
- password。
- nickname。
- status。

要求：

- 字段使用 `private`。
- 提供 getter 和 setter。
- status 只能是 0 或 1。

### 任务二：定义订单类

定义 `Order` 类，包含：

- orderNo。
- userId。
- totalAmount。
- status。

提供方法：

```java
public boolean canPay()
```

当订单状态为待支付时返回 `true`。

### 任务三：使用继承抽取公共字段

定义 `BaseEntity`，包含：

- id。
- createTime。
- updateTime。

让 `User` 和 `Order` 继承它。

### 任务四：定义支付接口

定义接口：

```java
public interface PaymentService {
    void pay(String orderNo);
}
```

实现两个类：

- `AliPayService`
- `WeChatPayService`

分别输出不同支付方式。

## 3.26 面试常问点

### 问题一：类和对象有什么区别？

参考回答：

```text
类是对象的模板，用来描述对象有哪些字段和行为。
对象是根据类创建出来的具体实例。
```

### 问题二：封装是什么？

参考回答：

```text
封装是把对象内部状态隐藏起来，通过公开方法控制外部访问和修改。
这样可以保护对象数据，避免外部随意修改导致非法状态。
```

### 问题三：继承有什么作用？

参考回答：

```text
继承可以让子类复用父类的字段和方法，减少重复代码。
例如多个实体类都有 id、createTime、updateTime，可以抽取到 BaseEntity 中。
```

### 问题四：接口和抽象类有什么区别？

参考回答：

```text
接口主要定义行为规范，强调能做什么。
抽象类可以包含公共字段和公共方法，也可以定义抽象方法。
如果只是定义能力，优先使用接口；如果有公共逻辑需要复用，可以考虑抽象类。
```

### 问题五：Java interface 和 TypeScript interface 一样吗？

参考回答：

```text
不完全一样。TypeScript interface 常用于描述对象结构。
Java interface 更常用于定义行为规范，由实现类完成具体逻辑。
```

## 3.27 本章总结

本章学习了 Java 面向对象基础：

```text
类
对象
字段
方法
构造方法
this
封装
getter / setter
继承
方法重写
super
多态
抽象类
接口
static
final
package
import
访问修饰符
Lombok
```

对于前端开发人员来说，理解面向对象的关键是把它和已有经验联系起来：

```text
Java 类类似 TypeScript 中 class 或接口加实现。
Java 对象类似前端运行时对象。
private 字段类似不希望外部随便修改的内部状态。
getter / setter 是对字段访问的控制入口。
接口用于定义能力规范。
Service 接口和 ServiceImpl 实现类是 Java 后端常见写法。
```

下一章将学习 Java 常用 API 和集合框架。那一章会重点讲 String、BigDecimal、LocalDateTime、List、Set、Map、泛型、Stream 和异常处理。这些内容会直接用于后续业务代码开发。

