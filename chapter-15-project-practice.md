# 第十五章：完整项目实战：权限管理后台系统

## 15.1 项目目标

本章设计一个完整 Java 后端实战项目：

```text
项目名称：权限管理后台系统
```

这个项目适合前端开发人员转 Java 后端练习，因为它覆盖了后台系统中最常见的能力：

```text
登录认证
用户管理
角色管理
菜单权限
接口权限
文件上传
操作日志
字典管理
统一返回
全局异常
参数校验
接口文档
Redis
Docker 部署
```

项目目标：

```text
能独立开发一套前后端分离的 Java 后端系统。
前端可以通过接口完成登录、菜单渲染、用户管理、角色分配和权限控制。
后端具备清晰分层、统一规范和可部署能力。
```

## 15.2 技术栈

后端技术：

```text
Java 17
Maven
Spring Boot
Spring MVC
MyBatis-Plus
MySQL
Redis
JWT
Sa-Token 或自定义拦截器
Lombok
Knife4j
Docker
Nginx
Git
```

技术作用：

| 技术 | 作用 |
|---|---|
| Java | 编写后端业务代码 |
| Maven | 管理依赖和打包 |
| Spring Boot | 后端应用框架 |
| Spring MVC | 开发 HTTP 接口 |
| MyBatis-Plus | 数据库访问 |
| MySQL | 保存业务数据 |
| Redis | 缓存 Token、验证码等 |
| JWT | 登录令牌 |
| Knife4j | 接口文档 |
| Docker | 容器化部署 |
| Nginx | 反向代理 |

前端可以使用 Vue 或 React 调用接口，本项目重点是后端。

## 15.3 功能模块

核心模块：

```text
认证模块
用户管理
角色管理
菜单管理
权限控制
文件上传
操作日志
字典管理
系统配置
```

认证模块：

- 登录。
- 退出。
- 获取当前用户。
- 获取当前用户菜单。
- 获取当前用户权限。

用户管理：

- 用户分页。
- 用户详情。
- 新增用户。
- 修改用户。
- 删除用户。
- 启用禁用。
- 重置密码。
- 分配角色。

角色管理：

- 角色分页。
- 新增角色。
- 修改角色。
- 删除角色。
- 分配菜单权限。

菜单管理：

- 菜单树。
- 新增菜单。
- 修改菜单。
- 删除菜单。
- 按钮权限标识。

## 15.4 数据库表设计

核心表：

```text
sys_user
sys_role
sys_menu
sys_user_role
sys_role_menu
sys_file
sys_operation_log
sys_dict_type
sys_dict_data
```

用户表：

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '用户ID',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    password VARCHAR(100) NOT NULL COMMENT '密码',
    nickname VARCHAR(50) COMMENT '昵称',
    phone VARCHAR(20) COMMENT '手机号',
    email VARCHAR(100) COMMENT '邮箱',
    status TINYINT DEFAULT 1 COMMENT '状态：0禁用，1启用',
    deleted TINYINT DEFAULT 0 COMMENT '逻辑删除',
    create_time DATETIME COMMENT '创建时间',
    update_time DATETIME COMMENT '更新时间',
    UNIQUE KEY uk_username (username)
) COMMENT='用户表';
```

文件表：

```sql
CREATE TABLE sys_file (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '文件ID',
    original_name VARCHAR(255) COMMENT '原始文件名',
    file_name VARCHAR(255) COMMENT '存储文件名',
    url VARCHAR(500) COMMENT '访问地址',
    size BIGINT COMMENT '文件大小',
    content_type VARCHAR(100) COMMENT '文件类型',
    create_time DATETIME COMMENT '创建时间'
) COMMENT='文件表';
```

字典类型表：

```sql
CREATE TABLE sys_dict_type (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    dict_name VARCHAR(100) NOT NULL COMMENT '字典名称',
    dict_code VARCHAR(100) NOT NULL COMMENT '字典编码',
    status TINYINT DEFAULT 1 COMMENT '状态',
    UNIQUE KEY uk_dict_code (dict_code)
) COMMENT='字典类型表';
```

字典数据表：

```sql
CREATE TABLE sys_dict_data (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    dict_code VARCHAR(100) NOT NULL COMMENT '字典编码',
    label VARCHAR(100) NOT NULL COMMENT '显示名称',
    value VARCHAR(100) NOT NULL COMMENT '字典值',
    sort INT DEFAULT 0 COMMENT '排序',
    status TINYINT DEFAULT 1 COMMENT '状态'
) COMMENT='字典数据表';
```

## 15.5 后端项目结构

推荐结构：

```text
admin-system
├── pom.xml
└── src/main
    ├── java/com/example/admin
    │   ├── AdminApplication.java
    │   ├── controller
    │   ├── service
    │   │   └── impl
    │   ├── mapper
    │   ├── entity
    │   ├── dto
    │   ├── vo
    │   ├── common
    │   ├── config
    │   ├── exception
    │   ├── interceptor
    │   ├── annotation
    │   ├── aspect
    │   └── util
    └── resources
        ├── application.yml
        ├── application-dev.yml
        └── mapper
```

包设计要稳定。不要把所有类堆在一个包里。

## 15.6 接口设计

认证接口：

```text
POST /auth/login
POST /auth/logout
GET  /auth/me
GET  /auth/menus
GET  /auth/permissions
```

用户接口：

```text
GET    /users
GET    /users/{id}
POST   /users
PUT    /users/{id}
DELETE /users/{id}
PATCH  /users/{id}/status
PUT    /users/{id}/roles
PUT    /users/{id}/password
```

角色接口：

```text
GET    /roles
GET    /roles/{id}
POST   /roles
PUT    /roles/{id}
DELETE /roles/{id}
PUT    /roles/{id}/menus
```

菜单接口：

```text
GET    /menus/tree
POST   /menus
PUT    /menus/{id}
DELETE /menus/{id}
```

文件接口：

```text
POST /files/upload
GET  /files
```

日志接口：

```text
GET /operation-logs
```

## 15.7 开发顺序

建议开发顺序：

```text
1. 创建 Spring Boot 项目。
2. 配置 Maven 依赖。
3. 配置 MySQL 和 MyBatis-Plus。
4. 创建统一 Result。
5. 创建全局异常处理。
6. 创建用户表和 UserMapper。
7. 实现登录接口。
8. 实现 JWT 拦截器。
9. 实现用户管理。
10. 实现角色管理。
11. 实现菜单管理。
12. 实现权限查询。
13. 实现接口权限校验。
14. 实现文件上传。
15. 实现操作日志。
16. 接入 Knife4j。
17. 前后端联调。
18. Docker 部署。
```

不要一开始就写所有模块。先打通登录和用户管理，再扩展其他模块。

## 15.8 登录模块实现重点

登录模块核心：

```text
LoginDTO
LoginVO
AuthController
AuthService
JwtUtil
LoginInterceptor
LoginUserContext
```

关键流程：

```text
查询用户
校验密码
检查状态
生成 Token
返回登录信息
拦截器校验 Token
ThreadLocal 保存当前用户
```

验收标准：

- 正确账号密码可以登录。
- 错误账号密码返回错误信息。
- 禁用账号不能登录。
- 未携带 Token 不能访问受保护接口。
- Token 过期返回 401。

## 15.9 用户模块实现重点

重点能力：

- 分页条件查询。
- 用户名唯一校验。
- 新增用户密码加密。
- 修改用户资料。
- 启用禁用。
- 逻辑删除。
- 分配角色。

分页返回：

```json
{
  "total": 100,
  "records": []
}
```

新增用户时要校验：

```text
用户名不能为空。
密码长度不少于 6 位。
用户名不能重复。
```

## 15.10 角色和菜单模块实现重点

角色模块重点：

```text
角色编码唯一。
角色删除前检查是否分配给用户。
角色分配菜单需要事务。
```

菜单模块重点：

```text
菜单树结构。
parent_id 父子关系。
菜单类型区分目录、菜单、按钮。
权限标识用于按钮和接口权限。
```

树构建：

```java
private List<MenuVO> buildTree(List<MenuVO> menus, Long parentId) {
    return menus.stream()
            .filter(menu -> parentId.equals(menu.getParentId()))
            .peek(menu -> menu.setChildren(buildTree(menus, menu.getId())))
            .toList();
}
```

## 15.11 文件上传实现重点

上传接口：

```java
@PostMapping("/upload")
public Result<FileVO> upload(@RequestParam("file") MultipartFile file) {
    return Result.success(fileService.upload(file));
}
```

需要处理：

- 空文件。
- 文件大小。
- 文件类型。
- 文件名重复。
- 保存路径。
- 访问 URL。
- 文件记录入库。

学习阶段可以先上传到本地磁盘，后续再扩展对象存储。

## 15.12 操作日志实现重点

操作日志建议使用 AOP：

```java
@OperationLog("新增用户")
@PostMapping("/users")
public Result<Void> create(@RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

记录内容：

```text
当前用户
操作名称
请求路径
请求方法
IP 地址
执行耗时
执行结果
创建时间
```

日志模块的价值是线上排查问题和审计操作。

## 15.13 前后端联调方式

联调前先确认：

```text
后端端口。
接口前缀。
跨域配置。
Token Header 名称。
统一返回结构。
错误码规则。
分页字段名。
时间格式。
```

前端请求拦截器：

```ts
request.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})
```

响应拦截器：

```ts
if (res.code !== 200) {
  message.error(res.message)
  return Promise.reject(res)
}
```

联调顺序：

```text
登录
获取当前用户
获取菜单
用户列表
新增用户
修改用户
删除用户
角色分配
菜单权限
```

## 15.14 项目验收标准

项目完成后，应满足：

```text
后端服务可以启动。
接口文档可以访问。
登录接口可用。
Token 拦截生效。
用户管理完整可用。
角色管理完整可用。
菜单树返回正确。
当前用户权限返回正确。
按钮权限可联动前端。
接口权限后端可校验。
文件上传可用。
操作日志可记录。
全局异常返回统一格式。
参数校验生效。
Docker 可以部署运行。
```

最低验收闭环：

```text
登录 -> 获取菜单 -> 用户分页 -> 新增用户 -> 分配角色 -> 分配菜单 -> 权限生效 -> 部署访问
```

## 15.15 常见错误

### 错误一：一开始就追求大而全

正确做法是先打通主流程，再补模块。

### 错误二：只实现前端菜单权限

后端接口权限必须做。

### 错误三：Entity 直接返回前端

容易泄露密码、删除标记等字段。

### 错误四：没有统一异常和日志

联调和排错成本会很高。

### 错误五：部署前没有环境配置

本地能跑不代表服务器能跑。数据库、Redis、文件路径、端口都要配置。

## 15.16 本章总结

本章设计了一套完整权限管理后台系统：

```text
项目目标
技术栈
功能模块
数据库表
项目结构
接口设计
开发顺序
登录模块
用户模块
角色菜单模块
文件上传
操作日志
前后端联调
验收标准
```

这不是简单 CRUD 项目，而是一套覆盖真实后端开发核心能力的综合项目。

下一章将学习项目部署上线，包括 Maven 打包、jar 运行、Docker、MySQL、Redis、Nginx 和线上日志排查。

