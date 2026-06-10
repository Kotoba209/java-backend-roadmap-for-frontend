# 第八章：Spring MVC 与 RESTful 接口开发

## 8.1 本章目标

前端开发人员最熟悉的是调接口。本章要把“调接口”的经验反过来，变成“写接口”的能力。

前端常见请求：

```ts
request.get('/users', { params: { pageNum: 1, pageSize: 10 } })
request.post('/users', data)
request.put('/users/1', data)
request.delete('/users/1')
```

后端要提供对应接口：

```java
@GetMapping("/users")
@PostMapping("/users")
@PutMapping("/users/{id}")
@DeleteMapping("/users/{id}")
```

学完本章后，你应该能够：

- 理解 Spring MVC 的作用。
- 使用 Controller 开发 RESTful 接口。
- 掌握 GET、POST、PUT、DELETE。
- 使用 `@RequestParam` 接收查询参数。
- 使用 `@PathVariable` 接收路径参数。
- 使用 `@RequestBody` 接收 JSON 请求体。
- 设计统一返回结果 `Result<T>`。
- 理解 HTTP 状态码和业务状态码。
- 使用参数校验。
- 理解跨域问题和 CORS 配置。
- 能开发标准 CRUD 接口。

## 8.2 Spring MVC 是什么

Spring MVC 是 Spring 体系中用于 Web 开发的模块。

可以简单理解为：

```text
Spring MVC 负责接收 HTTP 请求，并把请求分发给对应的 Controller 方法。
```

前端发请求：

```ts
await request.get('/users/1')
```

Spring MVC 找到后端方法：

```java
@GetMapping("/users/{id}")
public Result<UserVO> getById(@PathVariable Long id) {
    return Result.success(userService.getById(id));
}
```

完整流程：

```text
浏览器 / 前端应用
-> HTTP 请求
-> Spring MVC
-> Controller 方法
-> Service
-> 返回 Java 对象
-> 转成 JSON
-> 前端接收
```

## 8.3 Controller 基本写法

Controller 是接口入口。

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping
    public String list() {
        return "用户列表";
    }
}
```

解释：

- `@RestController`：声明这是 REST 接口控制器。
- `@RequestMapping("/users")`：当前类下的接口都以 `/users` 开头。
- `@GetMapping`：处理 GET 请求。

访问：

```text
GET http://localhost:8080/users
```

返回：

```text
用户列表
```

如果方法返回对象，Spring Boot 会自动转 JSON。

## 8.4 GET 查询接口

GET 通常用于查询。

前端：

```ts
request.get('/users', {
  params: {
    pageNum: 1,
    pageSize: 10,
    username: 'admin'
  }
})
```

后端方式一：使用 `@RequestParam`。

```java
@GetMapping("/users")
public Result<String> list(
        @RequestParam Integer pageNum,
        @RequestParam Integer pageSize,
        @RequestParam(required = false) String username) {
    return Result.success("查询成功");
}
```

如果参数很多，不推荐一个个写在方法参数里。

更推荐 DTO：

```java
public class UserQueryDTO {
    private Integer pageNum;
    private Integer pageSize;
    private String username;
    private Integer status;
}
```

Controller：

```java
@GetMapping("/users")
public Result<List<UserVO>> list(UserQueryDTO queryDTO) {
    List<UserVO> users = userService.list(queryDTO);
    return Result.success(users);
}
```

Spring MVC 会自动把查询参数绑定到 DTO 字段。

## 8.5 路径参数 @PathVariable

查询用户详情时，接口通常这样设计：

```text
GET /users/1
```

这里的 `1` 是路径参数。

后端写法：

```java
@GetMapping("/users/{id}")
public Result<UserVO> getById(@PathVariable Long id) {
    UserVO user = userService.getById(id);
    return Result.success(user);
}
```

前端：

```ts
request.get(`/users/${id}`)
```

路径参数适合表示资源 ID：

```text
GET /users/1
GET /orders/1001
DELETE /products/20
```

查询条件则更适合放在 query：

```text
GET /users?pageNum=1&pageSize=10&username=admin
```

## 8.6 POST 新增接口

POST 通常用于新增。

前端：

```ts
request.post('/users', {
  username: 'zhangsan',
  password: '123456',
  nickname: '张三'
})
```

后端 DTO：

```java
public class UserCreateDTO {
    private String username;
    private String password;
    private String nickname;
}
```

Controller：

```java
@PostMapping("/users")
public Result<Void> create(@RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

`@RequestBody` 表示从请求体中读取 JSON，并转换成 Java 对象。

请求体：

```json
{
  "username": "zhangsan",
  "password": "123456",
  "nickname": "张三"
}
```

会被转换成：

```java
UserCreateDTO
```

## 8.7 PUT 修改接口

PUT 通常用于修改。

前端：

```ts
request.put('/users/1', {
  nickname: '张三丰',
  status: 1
})
```

后端 DTO：

```java
public class UserUpdateDTO {
    private String nickname;
    private Integer status;
}
```

Controller：

```java
@PutMapping("/users/{id}")
public Result<Void> update(@PathVariable Long id, @RequestBody UserUpdateDTO dto) {
    userService.update(id, dto);
    return Result.success();
}
```

这里同时使用了：

- `@PathVariable`：接收路径中的用户 ID。
- `@RequestBody`：接收 JSON 请求体。

Service 中可以：

```java
public void update(Long id, UserUpdateDTO dto) {
    User user = userMapper.selectById(id);
    if (user == null) {
        throw new BusinessException("用户不存在");
    }

    user.setNickname(dto.getNickname());
    user.setStatus(dto.getStatus());
    userMapper.updateById(user);
}
```

## 8.8 DELETE 删除接口

DELETE 通常用于删除。

前端：

```ts
request.delete('/users/1')
```

后端：

```java
@DeleteMapping("/users/{id}")
public Result<Void> delete(@PathVariable Long id) {
    userService.delete(id);
    return Result.success();
}
```

Service：

```java
public void delete(Long id) {
    User user = userMapper.selectById(id);
    if (user == null) {
        throw new BusinessException("用户不存在");
    }
    userMapper.logicDeleteById(id);
}
```

注意：接口使用 DELETE，不代表数据库一定物理删除。真实项目中通常做逻辑删除：

```sql
UPDATE sys_user SET deleted = 1 WHERE id = ?
```

## 8.9 RESTful 接口风格

RESTful 是一种接口设计风格，强调资源和 HTTP 方法。

用户资源：

```text
GET    /users       查询用户列表
GET    /users/{id}  查询用户详情
POST   /users       新增用户
PUT    /users/{id}  修改用户
DELETE /users/{id}  删除用户
```

前端人员可能习惯这样的接口：

```text
/getUserList
/addUser
/updateUser
/deleteUser
```

RESTful 更推荐：

```text
/users
/users/{id}
```

用 HTTP 方法表达动作。

对比：

| 操作 | 非 RESTful | RESTful |
|---|---|---|
| 查询列表 | `/getUserList` | `GET /users` |
| 查询详情 | `/getUserById?id=1` | `GET /users/1` |
| 新增 | `/addUser` | `POST /users` |
| 修改 | `/updateUser` | `PUT /users/1` |
| 删除 | `/deleteUser?id=1` | `DELETE /users/1` |

企业项目不一定完全严格 RESTful，但理解这种风格非常重要。

## 8.10 统一返回结果

真实项目中不建议有的接口返回字符串，有的返回对象，有的返回数组。

应该统一响应结构：

```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

Java 泛型类：

```java
public class Result<T> {
    private Integer code;
    private String message;
    private T data;

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        result.setData(data);
        return result;
    }

    public static Result<Void> success() {
        Result<Void> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        return result;
    }

    public static Result<Void> fail(String message) {
        Result<Void> result = new Result<>();
        result.setCode(400);
        result.setMessage(message);
        return result;
    }

    // getter / setter 省略
}
```

Controller：

```java
@GetMapping("/users/{id}")
public Result<UserVO> getById(@PathVariable Long id) {
    return Result.success(userService.getById(id));
}
```

前端处理起来更统一：

```ts
if (res.code === 200) {
  return res.data
}
```

## 8.11 HTTP 状态码和业务状态码

HTTP 状态码是协议层状态：

| 状态码 | 含义 |
|---|---|
| 200 | 请求成功 |
| 400 | 请求参数错误 |
| 401 | 未登录 |
| 403 | 无权限 |
| 404 | 资源不存在 |
| 500 | 服务器异常 |

业务状态码是接口响应体中的 `code`：

```json
{
  "code": 40001,
  "message": "用户名或密码错误",
  "data": null
}
```

有些团队 HTTP 永远返回 200，通过业务 code 判断。也有团队会合理使用 HTTP 状态码。

学习阶段可以先使用统一响应：

```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

后续权限认证章节再细化 401、403。

## 8.12 参数校验

不要只靠前端校验。

前端校验：

```ts
if (!form.username) {
  message.error('用户名不能为空')
}
```

后端仍然必须校验：

```java
public class UserCreateDTO {

    @NotBlank(message = "用户名不能为空")
    private String username;

    @NotBlank(message = "密码不能为空")
    @Size(min = 6, message = "密码长度不能小于 6 位")
    private String password;

    private String nickname;
}
```

Controller：

```java
@PostMapping("/users")
public Result<Void> create(@Valid @RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

常见校验注解：

| 注解 | 作用 |
|---|---|
| `@NotNull` | 不能为 null |
| `@NotBlank` | 字符串不能为空白 |
| `@NotEmpty` | 集合或字符串不能为空 |
| `@Size` | 长度或数量限制 |
| `@Min` | 最小值 |
| `@Max` | 最大值 |
| `@Email` | 邮箱格式 |
| `@Pattern` | 正则校验 |

## 8.13 全局异常处理

如果参数校验失败，或者业务抛出异常，不能让接口返回一堆后端堆栈。

定义业务异常：

```java
public class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
}
```

全局异常处理：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusinessException(BusinessException e) {
        return Result.fail(e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        return Result.fail("系统异常");
    }
}
```

Service：

```java
if (user == null) {
    throw new BusinessException("用户不存在");
}
```

前端收到：

```json
{
  "code": 400,
  "message": "用户不存在",
  "data": null
}
```

这样前后端联调更稳定。

## 8.14 跨域问题 CORS

前端开发时常见：

```text
前端：http://localhost:5173
后端：http://localhost:8080
```

端口不同，就是不同源。浏览器会触发跨域限制。

后端可以配置 CORS：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:5173")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true);
    }
}
```

学习阶段也可以临时允许所有来源：

```java
allowedOriginPatterns("*")
```

但生产环境不建议随意开放所有来源。

前端也可以通过 Vite 代理解决开发环境跨域：

```ts
server: {
  proxy: {
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true
    }
  }
}
```

## 8.15 标准 CRUD 示例

用户管理接口设计：

```text
GET    /users       查询用户列表
GET    /users/{id}  查询用户详情
POST   /users       新增用户
PUT    /users/{id}  修改用户
DELETE /users/{id}  删除用户
```

Controller：

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
        return Result.success(userService.list(queryDTO));
    }

    @GetMapping("/{id}")
    public Result<UserVO> getById(@PathVariable Long id) {
        return Result.success(userService.getById(id));
    }

    @PostMapping
    public Result<Void> create(@Valid @RequestBody UserCreateDTO dto) {
        userService.create(dto);
        return Result.success();
    }

    @PutMapping("/{id}")
    public Result<Void> update(@PathVariable Long id, @Valid @RequestBody UserUpdateDTO dto) {
        userService.update(id, dto);
        return Result.success();
    }

    @DeleteMapping("/{id}")
    public Result<Void> delete(@PathVariable Long id) {
        userService.delete(id);
        return Result.success();
    }
}
```

这就是后端业务模块最常见的接口结构。

## 8.16 前后端联调方式

前端通常封装 API：

```ts
export function getUserList(params) {
  return request.get('/users', { params })
}

export function getUserById(id) {
  return request.get(`/users/${id}`)
}

export function createUser(data) {
  return request.post('/users', data)
}

export function updateUser(id, data) {
  return request.put(`/users/${id}`, data)
}

export function deleteUser(id) {
  return request.delete(`/users/${id}`)
}
```

后端对应：

```java
@GetMapping
@GetMapping("/{id}")
@PostMapping
@PutMapping("/{id}")
@DeleteMapping("/{id}")
```

联调时重点检查：

- 请求地址是否一致。
- 请求方法是否一致。
- 参数位置是否一致。
- JSON 字段名是否一致。
- Token 是否携带。
- 返回结构是否统一。
- 错误信息是否可读。

## 8.17 常见错误

### 错误一：POST JSON 忘记 @RequestBody

错误：

```java
@PostMapping("/users")
public Result<Void> create(UserCreateDTO dto) {
}
```

前端传 JSON 时，后端可能接收不到。

正确：

```java
@PostMapping("/users")
public Result<Void> create(@RequestBody UserCreateDTO dto) {
}
```

### 错误二：路径参数忘记 @PathVariable

错误：

```java
@GetMapping("/users/{id}")
public Result<UserVO> getById(Long id) {
}
```

正确：

```java
@GetMapping("/users/{id}")
public Result<UserVO> getById(@PathVariable Long id) {
}
```

### 错误三：GET 查询参数使用 @RequestBody

不推荐：

```java
@GetMapping("/users")
public Result<List<UserVO>> list(@RequestBody UserQueryDTO dto) {
}
```

GET 请求通常使用 query 参数：

```java
@GetMapping("/users")
public Result<List<UserVO>> list(UserQueryDTO dto) {
}
```

### 错误四：没有统一返回结构

不推荐：

```java
return "success";
return user;
return list;
```

推荐：

```java
return Result.success(data);
```

## 8.18 练习任务

### 任务一：开发商品 CRUD 接口

设计接口：

```text
GET    /products
GET    /products/{id}
POST   /products
PUT    /products/{id}
DELETE /products/{id}
```

### 任务二：设计 DTO 和 VO

创建：

```text
ProductCreateDTO
ProductUpdateDTO
ProductQueryDTO
ProductVO
```

要求：

- 创建 DTO 包含商品名称、价格、库存。
- VO 包含商品 ID、名称、价格、库存、状态。

### 任务三：实现统一返回 Result

实现：

```java
Result.success()
Result.success(data)
Result.fail(message)
```

### 任务四：添加参数校验

商品新增要求：

- 商品名称不能为空。
- 价格不能小于 0。
- 库存不能小于 0。

### 任务五：处理业务异常

当商品不存在时，返回：

```json
{
  "code": 400,
  "message": "商品不存在",
  "data": null
}
```

## 8.19 面试常问点

### 问题一：@RequestParam、@PathVariable、@RequestBody 有什么区别？

参考回答：

```text
@RequestParam 用于接收 query 参数，例如 ?pageNum=1。
@PathVariable 用于接收路径参数，例如 /users/{id}。
@RequestBody 用于接收请求体 JSON，常用于 POST、PUT。
```

### 问题二：GET 和 POST 有什么区别？

参考回答：

```text
GET 通常用于查询，参数一般放在 URL query 中。
POST 通常用于新增或提交数据，参数一般放在请求体中。
```

### 问题三：为什么需要统一返回结果？

参考回答：

```text
统一返回结果可以让前端以一致方式处理成功、失败、错误信息和业务数据。
也有利于接口规范和全局异常处理。
```

### 问题四：前端已经校验参数了，后端还需要校验吗？

参考回答：

```text
需要。前端校验可以被绕过，后端才是真正的安全边界。
所有关键参数都应该在后端再次校验。
```

### 问题五：什么是跨域？

参考回答：

```text
跨域是浏览器同源策略导致的问题。
当前端和后端协议、域名或端口不一致时，浏览器可能阻止请求。
可以通过后端 CORS 配置或前端开发代理解决。
```

## 8.20 本章总结

本章学习了 Spring MVC 与 RESTful 接口开发：

```text
Controller
@RestController
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@RequestParam
@PathVariable
@RequestBody
RESTful
统一返回 Result
参数校验
全局异常处理
跨域 CORS
CRUD 接口设计
前后端联调
```

对前端开发人员来说，本章的关键转换是：

```text
以前你写 request.get('/users')。
现在你要写 @GetMapping('/users')。

以前你传 params。
现在你要用 DTO 接收查询参数。

以前你传 JSON。
现在你要用 @RequestBody 接收 JSON。

以前你处理接口错误。
现在你要设计统一错误返回。
```

下一章将学习 MyBatis / MyBatis-Plus。你会把本章写出来的接口真正连接到 MySQL，实现完整的数据库增删改查。

