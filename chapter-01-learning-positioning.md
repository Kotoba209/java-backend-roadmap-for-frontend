# 第一章：前端开发人员转型 Java 后端的学习定位

## 1.1 这份资料适合谁

这份资料面向已经具备前端开发基础，希望系统转型 Java 后端开发的学习者。

你可能已经熟悉 HTML、CSS、JavaScript、TypeScript、Vue、React、Axios、Fetch、前后端分离、接口联调、Token 登录、路由守卫、权限菜单、表单提交、列表分页、文件上传、npm、pnpm、Vite、Webpack 和 Git。

如果你现在的状态是：

```text
我会写页面
我会调用接口
我知道登录后要保存 Token
我知道列表数据来自后端
我知道新增、编辑、删除都要请求接口
但我不知道这些接口在后端是怎么写出来的
```

那么这套资料就是为你准备的。

这套资料的目标不是让你只会写几个简单接口，而是让你最终具备完整 Java 后端开发能力，包括：

- 使用 Java 编写业务代码。
- 使用 Spring Boot 开发 RESTful 接口。
- 使用 MySQL 设计数据表。
- 使用 MyBatis / MyBatis-Plus 操作数据库。
- 实现登录认证。
- 实现角色权限控制。
- 实现业务模块 CRUD。
- 实现文件上传。
- 实现统一返回结果。
- 实现全局异常处理。
- 编写接口文档。
- 使用 Redis 做缓存。
- 使用 Docker / Nginx 部署项目。

最终目标可以概括为一句话：

```text
从会调用接口的前端开发人员，成长为能独立设计、开发、部署接口的 Java 后端开发人员。
```

## 1.2 前端转 Java 后端不是从零开始

很多前端开发人员一听到 Java 后端，会觉得自己要完全重新开始。其实不是。

以前你可能写过这样的前端请求：

```ts
export function getUserList(params) {
  return request({
    url: '/api/users',
    method: 'get',
    params
  })
}
```

你已经知道：

- 接口有 URL。
- 接口有请求方式。
- GET 请求通常用于查询。
- `params` 是查询参数。
- 后端会返回用户列表数据。

转到 Java 后端以后，你要学会写出这个接口的后端实现：

```java
@GetMapping("/users")
public Result<List<UserVO>> getUserList(UserQueryDTO queryDTO) {
    List<UserVO> list = userService.getUserList(queryDTO);
    return Result.success(list);
}
```

这段 Java 代码暂时看不懂没关系。先建立对应关系：

```text
前端调用 /api/users
后端提供 /users 接口

前端传 params
后端用 queryDTO 接收参数

前端拿到用户列表
后端从数据库查询用户列表并返回
```

前端转后端的学习过程，本质上是从：

```text
我知道怎么调用接口
```

转变为：

```text
我知道怎么设计接口、实现接口、保护接口、部署接口
```

## 1.3 你已经理解接口是什么

前端开发中，接口是非常常见的概念。

例如商品列表页面中，前端可能这样请求数据：

```ts
const res = await axios.get('/api/products', {
  params: {
    pageNum: 1,
    pageSize: 10,
    keyword: '键盘'
  }
})
```

这段代码表达了几个后端接口要素：

- 请求地址：`/api/products`
- 请求方式：`GET`
- 查询参数：`pageNum`、`pageSize`、`keyword`
- 业务含义：分页查询商品列表
- 返回结果：商品分页数据

对应到 Java 后端，接口可能这样写：

```java
@GetMapping("/products")
public Result<PageResult<ProductVO>> list(ProductQueryDTO queryDTO) {
    PageResult<ProductVO> pageResult = productService.list(queryDTO);
    return Result.success(pageResult);
}
```

这里的重点不是马上记住所有注解，而是理解职责变化：

```text
前端：发起请求，消费数据。
后端：接收请求，处理业务，返回数据。
```

前端人员学习后端时，最大的优势就是你不是第一次接触接口，只是以前站在调用方，现在要站在提供方。

## 1.4 你已经理解 JSON 数据结构

前端开发人员天然熟悉 JSON。

例如后端返回用户信息：

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "username": "admin",
    "nickname": "管理员",
    "roles": ["admin", "editor"]
  }
}
```

前端会这样使用：

```ts
const user = res.data
console.log(user.username)
console.log(user.roles)
```

Java 后端中，这个 `data` 通常来自一个 Java 对象：

```java
public class UserVO {
    private Long id;
    private String username;
    private String nickname;
    private List<String> roles;
}
```

Spring Boot 会把 Java 对象自动转换成 JSON：

```text
Java 对象 -> JSON -> 前端对象
```

这就是为什么后端项目中经常出现 DTO、VO、Entity 这些类。它们本质上都是用于描述数据结构，只是使用场景不同。

- DTO：接收前端传来的数据。
- VO：返回给前端看的数据。
- Entity：和数据库表对应的数据。

例如创建用户接口中，前端提交：

```json
{
  "username": "zhangsan",
  "password": "123456",
  "nickname": "张三"
}
```

后端可以用 DTO 接收：

```java
public class UserCreateDTO {
    private String username;
    private String password;
    private String nickname;
}
```

返回给前端时，不能把密码返回出去，所以用 VO：

```java
public class UserVO {
    private Long id;
    private String username;
    private String nickname;
}
```

这个区别非常重要。前端人员经常会问：为什么不能一个对象到处用？原因是接口入参、数据库实体和接口出参的安全性、字段范围、业务含义都不同。

## 1.5 你已经有业务流程理解能力

前端并不是只写页面。一个登录功能，前端需要理解完整流程：

```text
用户输入账号密码
点击登录按钮
前端校验表单
调用登录接口
后端返回 Token
前端保存 Token
跳转后台首页
后续请求携带 Token
Token 失效后跳回登录页
```

以前你主要负责前端部分：

```ts
const res = await loginApi({
  username: form.username,
  password: form.password
})

localStorage.setItem('token', res.data.token)
router.push('/dashboard')
```

后端要负责的是：

```text
接收账号密码
查询用户是否存在
校验密码是否正确
判断用户是否被禁用
生成 Token
返回用户信息和权限信息
```

Java 后端业务代码大概会是：

```java
public LoginVO login(LoginDTO loginDTO) {
    User user = userMapper.selectByUsername(loginDTO.getUsername());

    if (user == null) {
        throw new BusinessException("用户不存在");
    }

    if (!passwordEncoder.matches(loginDTO.getPassword(), user.getPassword())) {
        throw new BusinessException("密码错误");
    }

    if (user.getStatus() == 0) {
        throw new BusinessException("账号已被禁用");
    }

    String token = jwtUtil.generateToken(user.getId());

    return new LoginVO(token, user.getUsername(), user.getNickname());
}
```

这段代码体现了后端业务开发的核心：

- 不是简单查数据库。
- 不是简单返回数据。
- 而是要在接口背后执行完整业务规则。

前端人员转后端时，这种业务流程理解能力非常重要，因为后端开发的重点就是把业务规则稳定、准确地落到代码里。

## 1.6 你已经具备工程化思维

前端项目通常不是一个 HTML 文件，而是完整工程。

你可能熟悉这样的目录：

```text
src
├── api
├── components
├── router
├── stores
├── utils
├── views
└── main.ts
```

Java 后端项目也有类似分层目录：

```text
src/main/java/com/example/project
├── controller
├── service
├── mapper
├── entity
├── dto
├── vo
├── config
├── common
└── exception
```

前端中的 `api/user.ts` 可能这样写：

```ts
export function getUserList(params) {
  return request.get('/users', { params })
}
```

后端中的 `UserController.java` 可能这样写：

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    @GetMapping
    public Result<List<UserVO>> list(UserQueryDTO queryDTO) {
        return Result.success(userService.list(queryDTO));
    }
}
```

你会发现，前端和后端虽然技术不同，但工程化思想是相通的：

```text
前端拆组件、拆 API、拆状态管理。
后端拆 Controller、Service、Mapper、DTO、VO。
```

会工程化的前端人员学习 Java 后端时，最重要的是把已有的“模块拆分意识”迁移过来。

## 1.7 前端知识和 Java 后端知识的对应关系

前端人员学习 Java 后端时，不要孤立地记概念。更好的方式是把新概念和已有经验建立映射。

| 前端概念 | Java 后端概念 | 理解方式 |
|---|---|---|
| npm / pnpm | Maven | 都是依赖管理工具 |
| package.json | pom.xml | 都描述项目依赖和构建配置 |
| Vite 配置 | application.yml | 都是项目运行配置 |
| axios 请求 | Controller 接口 | 前端发请求，Controller 接请求 |
| API 模块 | Controller 层 | 定义接口入口 |
| hooks / service | Service 层 | 封装业务逻辑 |
| TypeScript interface | DTO / VO | 描述数据结构 |
| 路由守卫 | 拦截器 / 过滤器 | 控制访问权限 |
| localStorage Token | JWT / Redis Token | 保存或校验登录状态 |
| Mock 数据 | Swagger / Knife4j | 接口说明和调试 |
| 前端错误提示 | 全局异常处理 | 统一返回错误信息 |
| 打包 dist | Maven 打 jar 包 | 构建可部署产物 |
| Nginx 静态部署 | Nginx 反向代理后端 | 线上访问入口 |

例如前端中，`package.json` 管理依赖：

```json
{
  "dependencies": {
    "axios": "^1.6.0",
    "vue": "^3.4.0"
  }
}
```

Java 后端中，`pom.xml` 管理依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.7</version>
    </dependency>
</dependencies>
```

可以先这样理解：

```text
npm install axios
类似于
在 pom.xml 中加入 Maven 依赖
```

## 1.8 需要补齐的能力一：Java 语法和类型系统

前端常用 JavaScript / TypeScript。Java 和 TypeScript 都有类型，但 Java 更严格。

TypeScript 中：

```ts
let id: number = 1
let username: string = 'admin'
let enabled: boolean = true
```

Java 中：

```java
Long id = 1L;
String username = "admin";
Boolean enabled = true;
```

需要注意：

```java
Long id = 1; // 错误
```

因为 `1` 默认是 `int`，而 `Long` 类型通常要写成：

```java
Long id = 1L;
```

前端人员刚学 Java 时，常见问题包括：

- `String` 写成 `string`。
- 忘记每行末尾的分号。
- 不理解 `public static void main`。
- 不理解 `Long` 和 `long`。
- 不理解对象为什么要 `new`。
- 不理解字段为什么要 `private`。

这些都需要在 Java 基础阶段补齐。

## 1.9 需要补齐的能力二：面向对象能力

前端中也有对象：

```ts
const user = {
  id: 1,
  username: 'admin'
}
```

Java 中通常要定义类：

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

然后创建对象：

```java
User user = new User();
user.setId(1L);
user.setUsername("admin");
```

Java 后端大量代码都围绕对象展开：

- 用户对象。
- 商品对象。
- 订单对象。
- 登录请求对象。
- 接口返回对象。
- 数据库实体对象。

所以面向对象不是可选内容，而是 Java 后端的基础。

## 1.10 需要补齐的能力三：数据库和 SQL

前端看到的是接口返回数据：

```json
{
  "id": 1,
  "username": "admin",
  "roleName": "管理员"
}
```

后端要考虑的是这些数据从哪里来。

可能需要三张表：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(100) NOT NULL
);

CREATE TABLE sys_role (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    role_name VARCHAR(50) NOT NULL
);

CREATE TABLE sys_user_role (
    user_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL
);
```

然后通过 SQL 查询：

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

前端人员必须补齐数据库能力，因为后端接口的核心流程之一就是：

```text
接收请求 -> 查询数据库 -> 处理业务 -> 返回数据
```

不会数据库，就很难真正理解后端。

## 1.11 需要补齐的能力四：Spring Boot 框架

Java 语法只是基础。真正开发后端项目，主要使用 Spring Boot。

Spring Boot 可以理解为 Java 后端项目的主框架，类似前端中的 Vue / React 项目框架。

一个最简单的接口：

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @GetMapping
    public String hello() {
        return "Hello Spring Boot";
    }
}
```

浏览器访问：

```text
GET http://localhost:8080/hello
```

返回：

```text
Hello Spring Boot
```

后续所有接口开发，都会围绕 Spring Boot 展开。你需要掌握：

- Controller。
- Service。
- 配置文件。
- 依赖注入。
- 参数接收。
- JSON 返回。
- 异常处理。
- 参数校验。
- 文件上传。
- 拦截器。
- 跨域配置。

## 1.12 需要补齐的能力五：登录认证和权限控制

前端常见登录逻辑：

```ts
localStorage.setItem('token', token)
```

但真正判断用户有没有登录，不能只靠前端。

后端需要校验 Token：

```java
String token = request.getHeader("Authorization");

if (token == null || !jwtUtil.validateToken(token)) {
    throw new UnauthorizedException("登录已过期");
}
```

权限控制也是一样。前端可以隐藏按钮：

```vue
<button v-if="hasPermission('user:add')">新增用户</button>
```

但这只是用户体验层面的控制。真正安全的权限控制必须在后端：

```java
@PreAuthorize("hasAuthority('user:add')")
@PostMapping("/users")
public Result<Void> addUser(@RequestBody UserCreateDTO dto) {
    userService.addUser(dto);
    return Result.success();
}
```

否则用户可以绕过前端，直接调用接口。

后端必须掌握：

- 登录认证。
- JWT。
- Token 校验。
- 密码加密。
- 角色权限。
- 菜单权限。
- 按钮权限。
- 接口权限。

## 1.13 需要补齐的能力六：部署和运行

前端部署通常是：

```text
npm run build -> 生成 dist -> Nginx 部署静态文件
```

Java 后端部署通常是：

```text
mvn package -> 生成 jar -> java -jar 运行
```

示例：

```bash
java -jar admin-system.jar
```

后端部署还需要考虑：

- JDK 是否安装。
- MySQL 是否可连接。
- Redis 是否可连接。
- 端口是否开放。
- 日志文件在哪里。
- 配置文件是否正确。
- Nginx 是否正确代理。
- 服务挂了怎么排查。

这部分能力决定你能不能把项目真正上线，而不是只在本地运行。

## 1.14 建议的学习路线

完整学习路线如下：

```text
第一阶段：Java 基础语法
第二阶段：面向对象编程
第三阶段：Java 常用 API 和集合
第四阶段：MySQL 与 SQL
第五阶段：JDBC 与分层思想
第六阶段：Maven 与 Spring Boot
第七阶段：Spring MVC 接口开发
第八阶段：MyBatis / MyBatis-Plus
第九阶段：统一返回、异常、参数校验
第十阶段：登录认证与 JWT
第十一阶段：权限控制
第十二阶段：Redis
第十三阶段：文件上传、日志、接口文档
第十四阶段：完整项目实战
第十五阶段：Docker、Nginx、部署上线
```

这条路线的设计原则是：

```text
先学语言
再学数据库
再学框架
再学业务开发
再学安全和权限
最后做完整项目和部署
```

不建议一开始就直接做 Spring Boot 项目。原因是你可能能照着视频写出代码，但一旦出现错误，就不知道问题在哪里。

比如下面这个错误：

```text
Cannot resolve symbol 'UserService'
```

如果 Java 基础和项目结构没学好，你可能不知道是：

- 类没创建。
- 包名写错。
- import 错误。
- Maven 没刷新。
- Spring Bean 没注入。

基础扎实以后，排错能力会明显提升。

## 1.15 学习时应该避免的误区

### 误区一：只看视频，不写代码

后端学习必须写代码。尤其是 Java、SQL、Spring Boot、MyBatis，只看不写基本学不会。

正确方式是：

```text
看一个知识点
写一个小例子
改一个参数试试
故意制造一个错误
观察报错信息
修复错误
总结规律
```

### 误区二：只会写 CRUD，不理解分层

很多初学者会把所有逻辑写在 Controller 中：

```java
@GetMapping("/users")
public List<User> list() {
    // 查询数据库
    // 处理业务
    // 拼接返回值
    // 全部写在这里
}
```

这在真实项目中不规范。

更合理的方式是：

```text
Controller：接收请求。
Service：处理业务。
Mapper：访问数据库。
```

示例：

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    @GetMapping
    public Result<List<UserVO>> list() {
        return Result.success(userService.list());
    }
}
```

```java
@Service
public class UserService {

    private final UserMapper userMapper;

    public List<UserVO> list() {
        List<User> users = userMapper.selectList();
        return users.stream()
                .map(user -> new UserVO(user.getId(), user.getUsername()))
                .toList();
    }
}
```

### 误区三：认为前端做了权限控制就安全

前端权限只能改善体验，不能保证安全。

例如前端隐藏了删除按钮：

```vue
<button v-if="hasPermission('user:delete')">删除</button>
```

但用户仍然可以通过 Postman 或浏览器控制台直接请求：

```text
DELETE /api/users/1
```

所以后端必须再次校验权限。后端才是安全边界。

### 误区四：忽视数据库设计

很多后端问题不是代码问题，而是表设计问题。

例如用户和角色关系，如果一开始设计成：

```text
用户表里直接放 role_name
```

后面一个用户需要多个角色时，就很难扩展。

更合理的是：

```text
用户表 sys_user
角色表 sys_role
用户角色关联表 sys_user_role
```

后端开发必须具备数据库建模思维。

## 1.16 本资料的学习方式

后续每一章都会按照固定结构展开：

```text
1. 本章目标
2. 核心概念讲解
3. 前端类比理解
4. Java 代码示例
5. 真实业务场景
6. 常见错误
7. 练习任务
8. 面试常问点
9. 本章总结
```

例如讲 Controller，不会只写：

```text
Controller 用于接收请求。
```

而是会具体讲：

- 前端是怎么请求接口的。
- 后端 Controller 如何接收。
- GET 参数怎么接。
- POST JSON 怎么接。
- 路径参数怎么接。
- 返回 JSON 怎么封装。
- 出错时怎么返回。
- 怎么写接口文档。
- 前后端怎么联调。

示例代码会尽量接近真实项目，而不是只写玩具代码。

## 1.17 本章练习任务

### 任务一：画出一个接口调用流程

请根据你熟悉的前端项目，画出一个列表查询流程：

```text
页面点击查询按钮
前端收集查询条件
调用后端接口
后端接收参数
后端查询数据库
后端返回 JSON
前端渲染表格
```

你可以用商品列表、用户列表、订单列表任意一个场景。

### 任务二：把一个前端接口反推成后端接口

假设前端有如下请求：

```ts
export function getOrderList(params) {
  return request({
    url: '/api/orders',
    method: 'get',
    params
  })
}
```

请尝试写出它对应的后端接口形式：

```java
@GetMapping("/orders")
public Result<?> list(OrderQueryDTO queryDTO) {
    return Result.success();
}
```

暂时不要求你写完整业务，只需要理解接口对应关系。

### 任务三：列出你目前最薄弱的三项能力

例如：

```text
1. Java 语法
2. MySQL 表设计
3. 登录认证和权限控制
```

这样后续学习时可以更有重点。

## 1.18 本章面试常问点

### 问题一：前后端分离项目中，前端和后端分别负责什么？

参考回答：

```text
前端主要负责页面展示、用户交互、表单校验、接口调用和状态管理。
后端主要负责接口设计、业务逻辑、数据库操作、登录认证、权限控制、数据安全和系统部署。
前后端通过 HTTP 接口进行交互，通常使用 JSON 作为数据格式。
```

### 问题二：前端做了按钮权限控制，后端还需要做权限控制吗？

参考回答：

```text
需要。前端权限控制只能控制页面显示，不能真正保证接口安全。
用户可以绕过前端页面，直接通过 Postman 或脚本请求后端接口。
因此后端必须在接口层做权限校验，确保没有权限的用户无法执行敏感操作。
```

### 问题三：为什么后端开发必须学习数据库？

参考回答：

```text
因为后端接口返回的数据大多来自数据库。
后端不仅要查询数据，还要设计表结构、维护数据关系、保证数据一致性、处理事务和优化查询性能。
不会数据库，就无法独立完成完整后端业务开发。
```

## 1.19 本章总结

前端开发人员转 Java 后端，并不是完全从零开始。你已经具备很多优势：

```text
接口理解能力
JSON 数据理解能力
前后端联调经验
业务流程理解能力
工程化思维
模块化开发习惯
```

但你也必须补齐后端核心能力：

```text
Java 语法
面向对象
MySQL 和 SQL
Spring Boot
MyBatis
事务
登录认证
权限控制
Redis
日志
异常处理
接口文档
部署上线
```

学习 Java 后端的正确方向不是只会写几个接口，而是逐步建立完整后端能力：

```text
能建表
能写接口
能处理业务
能做权限
能处理异常
能写文档
能部署项目
能排查问题
```

从下一章开始，将正式进入 Java 基础语法学习。重点不是死记语法，而是用前端人员熟悉的 TypeScript / JavaScript 经验，逐步理解 Java 的强类型、类、对象、方法和运行机制。

