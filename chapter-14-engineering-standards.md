# 第十四章：接口文档、异常、日志与工程规范

## 14.1 本章目标

能写接口不等于能写好后端项目。真实项目还需要统一规范。

如果没有规范，项目很容易变成：

```text
有的接口返回字符串，有的返回对象。
有的错误返回 500，有的返回中文字符串。
Controller 里到处 try catch。
日志没有上下文，线上问题无法排查。
接口文档靠口头沟通。
类名、字段名、提交信息混乱。
```

本章目标是建立企业级后端项目的基础规范。

学完本章后，你应该能够：

- 设计统一返回结果。
- 设计统一错误码。
- 实现全局异常处理。
- 使用参数校验。
- 使用日志记录关键信息。
- 使用 Knife4j / Swagger 生成接口文档。
- 掌握项目目录规范。
- 掌握命名规范。
- 掌握数据库设计规范。
- 掌握 Git 提交规范。

## 14.2 项目目录结构规范

推荐目录：

```text
com.example.admin
├── controller
├── service
│   └── impl
├── mapper
├── entity
├── dto
├── vo
├── common
├── config
├── exception
├── interceptor
├── annotation
├── aspect
└── util
```

说明：

| 目录 | 作用 |
|---|---|
| controller | 接口入口 |
| service | 业务接口 |
| service.impl | 业务实现 |
| mapper | 数据库访问 |
| entity | 数据库实体 |
| dto | 请求参数 |
| vo | 响应数据 |
| common | 通用结果、常量 |
| config | 配置类 |
| exception | 异常类和异常处理 |
| interceptor | 拦截器 |
| annotation | 自定义注解 |
| aspect | AOP 切面 |
| util | 工具类 |

前端类比：

```text
api、components、utils、stores、router 各有职责。
后端 controller、service、mapper、dto、vo 也要各司其职。
```

## 14.3 Controller 规范

Controller 只负责：

```text
接收请求。
接收参数。
调用 Service。
返回 Result。
```

推荐：

```java
@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public Result<PageResult<UserVO>> page(UserQueryDTO queryDTO) {
        return Result.success(userService.page(queryDTO));
    }
}
```

不推荐：

```java
@GetMapping
public Result<?> page(UserQueryDTO queryDTO) {
    // 拼 SQL
    // 查数据库
    // 处理业务
    // 转换对象
}
```

Controller 越薄，项目越容易维护。

## 14.4 Service 规范

Service 负责业务逻辑。

应该放在 Service 的内容：

- 参数业务校验。
- 数据是否存在判断。
- 唯一性校验。
- 事务控制。
- 多表数据组合。
- Entity 和 VO 转换。
- 调用 Mapper。
- 调用 Redis、文件服务、第三方服务。

示例：

```java
@Transactional
public void assignRoles(Long userId, AssignUserRoleDTO dto) {
    User user = userMapper.selectById(userId);
    if (user == null) {
        throw new BusinessException("用户不存在");
    }

    userRoleMapper.deleteByUserId(userId);
    userRoleMapper.batchInsert(userId, dto.getRoleIds());
}
```

事务通常放在 Service 方法上，而不是 Controller 或 Mapper。

## 14.5 Mapper 规范

Mapper 只负责数据库访问。

推荐：

```java
public interface UserMapper extends BaseMapper<User> {
    List<UserRoleVO> selectUserRoles(Long userId);
}
```

不推荐在 Mapper 中写业务判断：

```java
boolean canDeleteUser(Long userId);
```

这种业务规则更适合放在 Service。

XML SQL 建议：

- SQL 格式清晰。
- 字段明确，不滥用 `SELECT *`。
- 复杂 SQL 加必要注释。
- 参数命名清楚。

## 14.6 统一返回结果

统一响应结构：

```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

Java 类：

```java
@Data
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

    public static Result<Void> fail(Integer code, String message) {
        Result<Void> result = new Result<>();
        result.setCode(code);
        result.setMessage(message);
        return result;
    }
}
```

好处：

- 前端处理统一。
- 错误信息统一。
- 接口文档统一。
- 全局异常处理方便。

## 14.7 统一错误码

可以定义错误码枚举：

```java
@Getter
@AllArgsConstructor
public enum ErrorCode {
    SUCCESS(200, "success"),
    BAD_REQUEST(400, "请求参数错误"),
    UNAUTHORIZED(401, "未登录"),
    FORBIDDEN(403, "无权限"),
    NOT_FOUND(404, "资源不存在"),
    SYSTEM_ERROR(500, "系统异常");

    private final Integer code;
    private final String message;
}
```

业务异常：

```java
@Getter
public class BusinessException extends RuntimeException {
    private final Integer code;

    public BusinessException(String message) {
        super(message);
        this.code = 400;
    }

    public BusinessException(Integer code, String message) {
        super(message);
        this.code = code;
    }
}
```

这样可以让错误语义更稳定。

## 14.8 全局异常处理

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusinessException(BusinessException e) {
        return Result.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<Void> handleValidException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().get(0).getDefaultMessage();
        return Result.fail(400, message);
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return Result.fail(500, "系统异常");
    }
}
```

Controller 不需要到处写：

```java
try {
} catch (Exception e) {
}
```

Service 里直接抛业务异常：

```java
throw new BusinessException("用户不存在");
```

全局异常处理负责统一返回。

## 14.9 参数校验规范

DTO 中写校验规则：

```java
@Data
public class UserCreateDTO {

    @NotBlank(message = "用户名不能为空")
    private String username;

    @NotBlank(message = "密码不能为空")
    @Size(min = 6, message = "密码长度不能小于 6 位")
    private String password;

    @Email(message = "邮箱格式不正确")
    private String email;
}
```

Controller 使用 `@Valid`：

```java
@PostMapping("/users")
public Result<Void> create(@Valid @RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

前端校验是体验，后端校验是安全。

常见校验：

```text
@NotNull
@NotBlank
@NotEmpty
@Size
@Min
@Max
@Email
@Pattern
```

## 14.10 日志规范

使用 SLF4J：

```java
private static final Logger log = LoggerFactory.getLogger(UserServiceImpl.class);
```

记录关键业务：

```java
log.info("创建用户成功，userId={}, username={}", user.getId(), user.getUsername());
```

记录异常：

```java
log.error("创建用户失败，username={}", dto.getUsername(), e);
```

不要这样写：

```java
System.out.println("创建用户成功");
```

日志级别：

| 级别 | 场景 |
|---|---|
| debug | 调试信息 |
| info | 关键业务流程 |
| warn | 可恢复异常或风险 |
| error | 系统错误 |

日志要有上下文，例如用户 ID、订单号、请求 ID。

## 14.11 操作日志 AOP

定义注解：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OperationLog {
    String value();
}
```

使用：

```java
@OperationLog("新增用户")
@PostMapping("/users")
public Result<Void> create(@RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

AOP：

```java
@Aspect
@Component
public class OperationLogAspect {

    @Around("@annotation(operationLog)")
    public Object around(ProceedingJoinPoint joinPoint, OperationLog operationLog) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            Object result = joinPoint.proceed();
            long cost = System.currentTimeMillis() - start;
            log.info("操作成功：{}，耗时={}ms", operationLog.value(), cost);
            return result;
        } catch (Throwable e) {
            log.error("操作失败：{}", operationLog.value(), e);
            throw e;
        }
    }
}
```

后续可以扩展为写入操作日志表。

## 14.12 Knife4j / Swagger 接口文档

接口文档用于前后端联调。

添加 Knife4j 依赖：

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.5.0</version>
</dependency>
```

Controller 注解：

```java
@Tag(name = "用户管理")
@RestController
@RequestMapping("/users")
public class UserController {

    @Operation(summary = "分页查询用户")
    @GetMapping
    public Result<PageResult<UserVO>> page(UserQueryDTO queryDTO) {
        return Result.success(userService.page(queryDTO));
    }
}
```

DTO 字段：

```java
@Schema(description = "新增用户请求")
public class UserCreateDTO {

    @Schema(description = "用户名", example = "admin")
    private String username;
}
```

前端可以通过接口文档查看：

- 请求地址。
- 请求方法。
- 请求参数。
- 响应结构。
- 调试接口。

## 14.13 命名规范

类名：

```text
UserController
UserService
UserServiceImpl
UserMapper
UserCreateDTO
UserVO
BusinessException
```

方法名：

```text
create
update
delete
getById
page
assignRoles
updateStatus
```

变量名：

```text
userId
roleId
orderNo
totalAmount
createTime
```

接口路径：

```text
/users
/users/{id}
/roles/{id}/menus
/auth/login
```

权限标识：

```text
user:list
user:add
user:update
user:delete
```

命名要稳定、可读、统一。

## 14.14 数据库设计规范

表名：

```text
sys_user
sys_role
sys_menu
biz_order
biz_product
```

字段名：

```text
user_id
order_no
create_time
update_time
```

推荐字段：

```text
id
create_time
update_time
deleted
```

金额：

```sql
DECIMAL(10, 2)
```

状态字段要写注释：

```sql
status TINYINT COMMENT '状态：0禁用，1启用'
```

唯一字段加唯一索引：

```sql
UNIQUE KEY uk_username (username)
```

查询字段加合理索引：

```sql
INDEX idx_create_time (create_time)
```

## 14.15 Git 提交规范

推荐格式：

```text
type: subject
```

常见 type：

| type | 说明 |
|---|---|
| feat | 新功能 |
| fix | 修复问题 |
| docs | 文档 |
| style | 格式调整 |
| refactor | 重构 |
| test | 测试 |
| chore | 构建或工具 |

示例：

```text
feat: add user management APIs
fix: handle expired JWT token
docs: add deployment guide
refactor: move login logic to auth service
```

不要写：

```text
update
fix bug
修改了一下
```

好的提交记录能帮助团队追踪变更。

## 14.16 常见错误

### 错误一：没有统一响应

导致前端每个接口都要单独适配。

### 错误二：异常直接暴露堆栈

生产环境不能把 Java 异常堆栈直接返回给前端。

### 错误三：日志只写“出错了”

没有用户 ID、订单号、请求参数，就无法排查问题。

### 错误四：接口文档不更新

前后端联调会反复沟通，浪费时间。

### 错误五：命名随意

短期看起来快，长期维护成本很高。

## 14.17 练习任务

### 任务一：实现统一 Result

支持：

```text
success()
success(data)
fail(code, message)
```

### 任务二：实现全局异常

处理：

- BusinessException。
- 参数校验异常。
- 系统异常。

### 任务三：接入 Knife4j

为用户管理接口添加接口文档注解。

### 任务四：添加操作日志注解

实现 `@OperationLog`，记录方法执行耗时。

### 任务五：整理 Git 提交规范

为当前项目写一份提交规范说明。

## 14.18 面试常问点

### 问题一：为什么需要统一返回结果？

参考回答：

```text
统一返回结果可以让前端统一处理响应数据、错误码和错误信息，也便于全局异常处理和接口文档维护。
```

### 问题二：全局异常处理有什么作用？

参考回答：

```text
全局异常处理可以统一捕获业务异常、参数异常和系统异常，避免 Controller 到处写 try catch，并保证错误响应格式一致。
```

### 问题三：日志中应该记录什么？

参考回答：

```text
日志应记录关键业务上下文，例如用户 ID、订单号、请求地址、耗时、异常堆栈等，便于线上问题排查。
```

## 14.19 本章总结

本章学习了工程规范：

```text
项目目录结构
Controller 规范
Service 规范
Mapper 规范
统一返回
错误码
全局异常
参数校验
日志
操作日志 AOP
Knife4j / Swagger
命名规范
数据库规范
Git 提交规范
```

这些内容决定一个后端项目是否可维护。  
下一章将进入完整项目实战，把前面所有知识整合成一套权限管理后台系统。

