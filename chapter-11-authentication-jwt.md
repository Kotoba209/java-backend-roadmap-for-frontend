# 第十一章：登录认证与 JWT

## 11.1 本章目标

后台系统不能让所有接口都匿名访问。用户必须先登录，后续请求携带 Token，后端校验 Token 后才能放行。

前端人员通常熟悉这套流程：

```text
登录页输入账号密码
调用登录接口
后端返回 Token
前端保存 Token
axios 请求拦截器携带 Token
Token 过期跳转登录页
```

本章要从后端角度完整实现这套机制。

学完本章后，你应该能够：

- 理解认证和授权的区别。
- 实现账号密码登录。
- 使用 BCrypt 加密密码。
- 理解 JWT 的结构和作用。
- 生成 JWT Token。
- 校验 JWT Token。
- 使用拦截器保护接口。
- 获取当前登录用户。
- 理解 Token 过期和 401。
- 完成前后端登录联调。

## 11.2 认证和授权的区别

认证 Authentication：

```text
你是谁？
```

例如登录时校验账号密码：

```text
username = admin
password = 123456
```

后端确认这个用户存在，并且密码正确，这就是认证。

授权 Authorization：

```text
你能做什么？
```

例如：

```text
admin 可以新增用户。
test 只能查看用户。
guest 不能删除用户。
```

本章重点讲认证，也就是登录和 Token。下一章会重点讲授权，也就是角色权限和 RBAC。

## 11.3 登录流程总览

完整登录流程：

```text
1. 前端提交账号密码。
2. 后端根据用户名查询用户。
3. 判断用户是否存在。
4. 判断用户是否被禁用。
5. 校验密码是否正确。
6. 生成 JWT Token。
7. 返回 Token 和用户基本信息。
8. 前端保存 Token。
9. 后续请求在 Header 中携带 Token。
10. 后端拦截器校验 Token。
11. 校验通过，放行接口。
12. 校验失败，返回 401。
```

前端登录请求：

```ts
request.post('/auth/login', {
  username: 'admin',
  password: '123456'
})
```

后端接口：

```java
@PostMapping("/login")
public Result<LoginVO> login(@Valid @RequestBody LoginDTO dto) {
    return Result.success(authService.login(dto));
}
```

返回：

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiJ9...",
    "userId": 1,
    "username": "admin",
    "nickname": "管理员"
  }
}
```

## 11.4 登录请求和响应对象

登录 DTO：

```java
@Data
public class LoginDTO {

    @NotBlank(message = "用户名不能为空")
    private String username;

    @NotBlank(message = "密码不能为空")
    private String password;
}
```

登录 VO：

```java
@Data
@AllArgsConstructor
public class LoginVO {
    private String token;
    private Long userId;
    private String username;
    private String nickname;
}
```

为什么登录返回 VO 而不是 User？

```text
User 包含 password、deleted 等数据库字段。
LoginVO 只返回前端登录后需要的数据。
```

前端拿到 `token` 后通常保存：

```ts
localStorage.setItem('token', res.data.token)
```

或者使用状态管理：

```ts
userStore.setToken(res.data.token)
```

## 11.5 密码为什么不能明文保存

学习阶段为了理解流程，可能会看到：

```sql
password = '123456'
```

真实项目中绝对不能明文保存密码。

原因：

```text
数据库泄露后，所有用户密码都会直接暴露。
很多用户会在多个网站使用相同密码。
明文密码属于严重安全风险。
```

正确做法是保存密码哈希。

例如用户输入：

```text
123456
```

数据库保存：

```text
$2a$10$O8uW2xI0w3NwH2q...
```

登录时不是把明文密码解密出来，而是使用算法比对：

```text
用户输入密码 + 哈希值 -> 判断是否匹配
```

## 11.6 BCrypt 密码加密

Spring Security 提供了 `BCryptPasswordEncoder`。

配置：

```java
@Configuration
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

注册或创建用户时加密：

```java
String rawPassword = "123456";
String encodedPassword = passwordEncoder.encode(rawPassword);
user.setPassword(encodedPassword);
```

登录时校验：

```java
boolean matched = passwordEncoder.matches(dto.getPassword(), user.getPassword());
```

完整判断：

```java
if (!passwordEncoder.matches(dto.getPassword(), user.getPassword())) {
    throw new BusinessException("用户名或密码错误");
}
```

注意：错误提示通常不要区分“用户不存在”和“密码错误”，避免泄露账号是否存在。

更安全提示：

```text
用户名或密码错误
```

## 11.7 JWT 是什么

JWT 全称 JSON Web Token。

可以简单理解为：

```text
JWT 是一种可以携带用户信息、并且能被后端校验真伪的 Token。
```

JWT 通常长这样：

```text
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjF9.xxxxx
```

它由三部分组成：

```text
Header.Payload.Signature
```

- Header：算法信息。
- Payload：载荷，例如用户 ID、过期时间。
- Signature：签名，用于防止篡改。

注意：

```text
JWT 的 Payload 不是加密，只是编码。
不要把密码、手机号、身份证等敏感信息放进 JWT。
```

通常只放：

```text
userId
username
过期时间
```

## 11.8 JWT 工具类

示例依赖：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
```

工具类示例：

```java
@Component
public class JwtUtil {

    private static final String SECRET = "replace-with-a-long-secure-secret-key-123456";
    private static final long EXPIRE_MILLIS = 24 * 60 * 60 * 1000L;

    private SecretKey getSecretKey() {
        return Keys.hmacShaKeyFor(SECRET.getBytes(StandardCharsets.UTF_8));
    }

    public String generateToken(Long userId) {
        Date now = new Date();
        Date expireAt = new Date(now.getTime() + EXPIRE_MILLIS);

        return Jwts.builder()
                .subject(String.valueOf(userId))
                .issuedAt(now)
                .expiration(expireAt)
                .signWith(getSecretKey())
                .compact();
    }

    public Long parseUserId(String token) {
        Claims claims = Jwts.parser()
                .verifyWith(getSecretKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();

        return Long.valueOf(claims.getSubject());
    }
}
```

真实项目中，密钥和过期时间应该放在配置文件中，不要硬编码。

## 11.9 登录 Service 实现

Mapper 查询：

```java
User selectByUsername(String username);
```

使用 MyBatis-Plus：

```java
private User getByUsername(String username) {
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(User::getUsername, username)
           .eq(User::getDeleted, 0);
    return userMapper.selectOne(wrapper);
}
```

登录实现：

```java
@Service
@RequiredArgsConstructor
public class AuthServiceImpl implements AuthService {

    private final UserMapper userMapper;
    private final BCryptPasswordEncoder passwordEncoder;
    private final JwtUtil jwtUtil;

    @Override
    public LoginVO login(LoginDTO dto) {
        User user = getByUsername(dto.getUsername());

        if (user == null) {
            throw new BusinessException("用户名或密码错误");
        }

        if (Integer.valueOf(0).equals(user.getStatus())) {
            throw new BusinessException("账号已被禁用");
        }

        if (!passwordEncoder.matches(dto.getPassword(), user.getPassword())) {
            throw new BusinessException("用户名或密码错误");
        }

        String token = jwtUtil.generateToken(user.getId());

        return new LoginVO(token, user.getId(), user.getUsername(), user.getNickname());
    }

    private User getByUsername(String username) {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getUsername, username)
               .eq(User::getDeleted, 0);
        return userMapper.selectOne(wrapper);
    }
}
```

这个流程体现了认证逻辑的核心。

## 11.10 登录 Controller

```java
@RestController
@RequestMapping("/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthService authService;

    @PostMapping("/login")
    public Result<LoginVO> login(@Valid @RequestBody LoginDTO dto) {
        return Result.success(authService.login(dto));
    }
}
```

前端请求：

```ts
export function login(data) {
  return request.post('/auth/login', data)
}
```

登录成功后：

```ts
const res = await login(form)
localStorage.setItem('token', res.data.token)
```

## 11.11 前端请求携带 Token

前端通常在请求拦截器中统一携带 Token。

Axios 示例：

```ts
request.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})
```

后端从 Header 读取：

```java
String authorization = request.getHeader("Authorization");
```

常见格式：

```text
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

后端需要去掉 `Bearer ` 前缀：

```java
String token = authorization.substring(7);
```

## 11.12 拦截器校验 Token

定义拦截器：

```java
@Component
@RequiredArgsConstructor
public class LoginInterceptor implements HandlerInterceptor {

    private final JwtUtil jwtUtil;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String authorization = request.getHeader("Authorization");

        if (authorization == null || !authorization.startsWith("Bearer ")) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }

        String token = authorization.substring(7);

        try {
            Long userId = jwtUtil.parseUserId(token);
            LoginUserContext.setUserId(userId);
            return true;
        } catch (Exception e) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        LoginUserContext.clear();
    }
}
```

配置拦截器：

```java
@Configuration
@RequiredArgsConstructor
public class WebConfig implements WebMvcConfigurer {

    private final LoginInterceptor loginInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/auth/login");
    }
}
```

这样除了登录接口，其他接口都需要 Token。

## 11.13 当前登录用户 ThreadLocal

后端经常需要知道当前是谁在操作。

例如：

```text
创建订单时记录 userId。
操作日志记录 username。
修改数据时记录 updateBy。
```

可以用 ThreadLocal 保存当前请求用户 ID。

```java
public class LoginUserContext {

    private static final ThreadLocal<Long> USER_ID_HOLDER = new ThreadLocal<>();

    public static void setUserId(Long userId) {
        USER_ID_HOLDER.set(userId);
    }

    public static Long getUserId() {
        return USER_ID_HOLDER.get();
    }

    public static void clear() {
        USER_ID_HOLDER.remove();
    }
}
```

拦截器校验 Token 后：

```java
LoginUserContext.setUserId(userId);
```

Service 中使用：

```java
Long currentUserId = LoginUserContext.getUserId();
```

请求结束后必须清理：

```java
LoginUserContext.clear();
```

否则在线程复用场景下可能产生数据污染。

## 11.14 获取当前用户信息

接口：

```text
GET /auth/me
```

Controller：

```java
@GetMapping("/me")
public Result<UserVO> me() {
    Long userId = LoginUserContext.getUserId();
    return Result.success(userService.getById(userId));
}
```

前端刷新页面时，可以调用这个接口恢复用户信息：

```ts
const userInfo = await request.get('/auth/me')
userStore.setUserInfo(userInfo.data)
```

这样即使页面刷新，只要 Token 还在，就能重新获取当前用户。

## 11.15 Token 过期处理

JWT 中通常包含过期时间。

Token 过期后，后端返回：

```text
401 Unauthorized
```

前端响应拦截器：

```ts
request.interceptors.response.use(
  response => response.data,
  error => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token')
      router.push('/login')
    }
    return Promise.reject(error)
  }
)
```

后端返回 401 的场景：

- 没有 Token。
- Token 格式错误。
- Token 过期。
- Token 签名无效。

前端收到 401 后，通常清除登录状态并跳转登录页。

## 11.16 Refresh Token 思路

简单项目可以只使用一个 Access Token。

更完整的系统可能使用：

```text
Access Token：短期有效，用于访问接口。
Refresh Token：长期有效，用于刷新 Access Token。
```

流程：

```text
Access Token 过期
前端使用 Refresh Token 请求刷新接口
后端校验 Refresh Token
返回新的 Access Token
```

学习阶段可以先不实现 Refresh Token，先掌握单 Token 登录流程。

后续项目复杂度提升后，再考虑：

- Refresh Token。
- Redis 存储 Token。
- 主动踢下线。
- 多端登录控制。

## 11.17 退出登录

如果只使用纯 JWT，后端不保存 Token 状态，那么退出登录通常是前端删除 Token：

```ts
localStorage.removeItem('token')
router.push('/login')
```

但这种方式有一个问题：

```text
Token 在过期前理论上仍然有效。
```

如果想实现后端主动失效，需要配合 Redis：

- 登录时把 Token 存入 Redis。
- 每次请求校验 JWT 后，再校验 Redis 中是否存在。
- 退出登录时删除 Redis Token。

学习阶段可以先实现前端删除 Token。Redis 方案会在 Redis 章节继续讲。

## 11.18 常见错误

### 错误一：明文保存密码

不推荐：

```java
user.setPassword(dto.getPassword());
```

推荐：

```java
user.setPassword(passwordEncoder.encode(dto.getPassword()));
```

### 错误二：JWT 中放敏感信息

不推荐：

```text
password
phone
idCard
```

推荐只放：

```text
userId
username
expireTime
```

### 错误三：拦截器没有排除登录接口

如果 `/auth/login` 也被拦截，那么用户还没登录就无法登录。

需要：

```java
excludePathPatterns("/auth/login")
```

### 错误四：ThreadLocal 不清理

请求结束后必须：

```java
LoginUserContext.clear();
```

### 错误五：前端 Token 格式和后端不一致

前端传：

```text
Authorization: Bearer token
```

后端就要按 Bearer 解析。

如果前端直接传 token，后端也要对应调整。

## 11.19 练习任务

### 任务一：实现登录接口

实现：

```text
POST /auth/login
```

请求：

```json
{
  "username": "admin",
  "password": "123456"
}
```

响应：

```json
{
  "token": "...",
  "userId": 1,
  "username": "admin"
}
```

### 任务二：使用 BCrypt

创建用户时加密密码。登录时使用 `matches` 校验。

### 任务三：实现 JWT 工具类

要求：

- 生成 Token。
- 解析用户 ID。
- 支持过期时间。

### 任务四：实现登录拦截器

要求：

- `/auth/login` 放行。
- 其他接口必须携带 Token。
- Token 无效返回 401。

### 任务五：实现当前用户接口

实现：

```text
GET /auth/me
```

返回当前登录用户信息。

## 11.20 面试常问点

### 问题一：认证和授权有什么区别？

参考回答：

```text
认证是确认用户是谁，例如账号密码登录。
授权是确认用户能做什么，例如是否有 user:add 权限。
```

### 问题二：为什么密码不能明文保存？

参考回答：

```text
明文密码一旦数据库泄露，用户真实密码会直接暴露。
应该使用 BCrypt 等哈希算法保存密码哈希，登录时使用 matches 校验。
```

### 问题三：JWT 由哪几部分组成？

参考回答：

```text
JWT 由 Header、Payload、Signature 三部分组成。
Header 描述算法，Payload 保存载荷，Signature 用于校验是否被篡改。
```

### 问题四：JWT 中能放密码吗？

参考回答：

```text
不能。JWT Payload 只是编码，不是加密。
不应该放密码、身份证、手机号等敏感信息。
```

### 问题五：为什么 ThreadLocal 使用后要清理？

参考回答：

```text
Web 服务器线程会复用。如果不清理 ThreadLocal，可能导致上一个请求的数据影响下一个请求。
因此请求结束后必须 remove。
```

## 11.21 本章总结

本章学习了登录认证与 JWT：

```text
认证和授权
登录流程
LoginDTO
LoginVO
密码加密
BCrypt
JWT 结构
JWT 工具类
登录 Service
登录 Controller
前端携带 Token
后端拦截器校验 Token
ThreadLocal 当前用户
Token 过期
退出登录
```

现在你已经可以完成后台系统的登录认证闭环：

```text
前端登录
-> 后端校验账号密码
-> 生成 Token
-> 前端保存 Token
-> 请求拦截器携带 Token
-> 后端拦截器校验 Token
-> 获取当前用户
-> 访问受保护接口
```

下一章将学习权限控制与 RBAC。登录只能说明用户是谁，权限控制要解决用户能访问哪些菜单、哪些按钮、哪些接口。

