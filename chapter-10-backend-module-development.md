# 第十章：标准业务模块开发

## 10.1 本章目标

前面已经学会了接口开发和数据库访问。本章要把这些知识组合起来，形成“开发一个标准后端业务模块”的能力。

真实后台系统中，大部分业务模块都有类似结构：

```text
用户管理
角色管理
菜单管理
商品管理
订单管理
字典管理
文件管理
日志管理
```

这些模块通常都包含：

```text
新增
删除
修改
查询详情
分页列表
状态切换
参数校验
业务异常
接口文档
前后端联调
```

学完本章后，你应该能够：

- 理解一个标准业务模块的开发流程。
- 从数据库表开始设计模块。
- 设计 Entity、DTO、VO。
- 编写 Controller、Service、Mapper。
- 实现分页查询、详情、新增、修改、删除。
- 处理唯一性校验和业务异常。
- 理解状态切换、逻辑删除和数据转换。
- 能独立开发用户管理、角色管理、商品管理这类模块。

## 10.2 标准模块开发流程

开发一个后端业务模块，建议按这个顺序：

```text
1. 分析业务需求
2. 设计数据库表
3. 创建 Entity
4. 创建 DTO / VO
5. 创建 Mapper
6. 创建 Service 接口和实现类
7. 创建 Controller
8. 实现分页查询
9. 实现新增
10. 实现修改
11. 实现详情
12. 实现删除或状态切换
13. 添加参数校验
14. 添加业务异常
15. 联调接口
```

不要一上来就写 Controller。

后端开发要先搞清楚：

```text
数据怎么存？
字段有哪些？
哪些字段前端提交？
哪些字段返回给前端？
哪些规则必须后端校验？
```

## 10.3 以用户管理模块为例

用户管理模块常见功能：

```text
分页查询用户
查看用户详情
新增用户
修改用户
删除用户
启用 / 禁用用户
重置密码
分配角色
```

本章先实现基础功能：

```text
分页查询
详情
新增
修改
逻辑删除
状态切换
```

接口设计：

```text
GET    /users           分页查询用户
GET    /users/{id}      查询用户详情
POST   /users           新增用户
PUT    /users/{id}      修改用户
DELETE /users/{id}      删除用户
PATCH  /users/{id}/status 修改用户状态
```

前端 API：

```ts
export function getUserPage(params) {
  return request.get('/users', { params })
}

export function createUser(data) {
  return request.post('/users', data)
}

export function updateUser(id, data) {
  return request.put(`/users/${id}`, data)
}
```

## 10.4 用户表设计

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
    update_time DATETIME COMMENT '更新时间',
    UNIQUE KEY uk_username (username)
) COMMENT='用户表';
```

设计说明：

- `username` 加唯一索引，避免重复账号。
- `password` 真实项目必须加密保存。
- `status` 用于启用或禁用用户。
- `deleted` 用于逻辑删除。
- `create_time`、`update_time` 用于排序和审计。

前端用户列表通常展示：

```text
用户名
昵称
手机号
邮箱
状态
创建时间
操作按钮
```

密码不应该出现在列表返回中。

## 10.5 Entity 设计

```java
@Data
@TableName("sys_user")
public class User {
    private Long id;
    private String username;
    private String password;
    private String nickname;
    private String phone;
    private String email;
    private Integer status;
    private Integer deleted;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

Entity 对应数据库表。它可以包含 `password` 和 `deleted`，因为这些是数据库字段。

但注意：

```text
Entity 不应该直接返回给前端。
```

返回给前端时应该转换成 VO。

## 10.6 DTO 设计

分页查询 DTO：

```java
@Data
public class UserQueryDTO {
    private Integer pageNum = 1;
    private Integer pageSize = 10;
    private String username;
    private String phone;
    private Integer status;
}
```

新增用户 DTO：

```java
@Data
public class UserCreateDTO {

    @NotBlank(message = "用户名不能为空")
    private String username;

    @NotBlank(message = "密码不能为空")
    @Size(min = 6, message = "密码长度不能小于 6 位")
    private String password;

    private String nickname;
    private String phone;
    private String email;
}
```

修改用户 DTO：

```java
@Data
public class UserUpdateDTO {
    private String nickname;
    private String phone;
    private String email;
    private Integer status;
}
```

状态修改 DTO：

```java
@Data
public class UserStatusDTO {

    @NotNull(message = "状态不能为空")
    private Integer status;
}
```

DTO 的作用：

```text
明确前端允许提交哪些字段。
防止前端提交不该修改的字段。
为参数校验提供载体。
```

## 10.7 VO 设计

用户列表 VO：

```java
@Data
public class UserVO {
    private Long id;
    private String username;
    private String nickname;
    private String phone;
    private String email;
    private Integer status;
    private LocalDateTime createTime;
}
```

用户详情 VO 可以根据需要增加角色信息：

```java
@Data
public class UserDetailVO {
    private Long id;
    private String username;
    private String nickname;
    private String phone;
    private String email;
    private Integer status;
    private List<String> roles;
    private LocalDateTime createTime;
}
```

VO 的核心原则：

```text
只返回前端需要展示的数据。
不返回密码。
不返回 deleted。
不返回内部敏感字段。
```

## 10.8 Mapper 设计

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

使用 MyBatis-Plus 后，基础 CRUD 可以直接使用 `BaseMapper`。

如果需要复杂查询，可以扩展方法：

```java
List<UserDetailVO> selectUserWithRoles(Long userId);
```

简单模块优先使用 Wrapper，复杂多表查询再写 XML。

## 10.9 Service 设计

接口：

```java
public interface UserService {
    PageResult<UserVO> page(UserQueryDTO queryDTO);
    UserDetailVO getById(Long id);
    void create(UserCreateDTO dto);
    void update(Long id, UserUpdateDTO dto);
    void delete(Long id);
    void updateStatus(Long id, UserStatusDTO dto);
}
```

实现类：

```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final UserMapper userMapper;

    @Override
    public PageResult<UserVO> page(UserQueryDTO queryDTO) {
        return null;
    }
}
```

Service 是业务规则集中地。不要把这些逻辑放在 Controller：

- 用户名是否重复。
- 用户是否存在。
- 状态是否合法。
- 密码是否需要加密。
- 删除前是否要检查关联数据。

## 10.10 分页查询实现

```java
@Override
public PageResult<UserVO> page(UserQueryDTO queryDTO) {
    Page<User> page = new Page<>(queryDTO.getPageNum(), queryDTO.getPageSize());

    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(User::getDeleted, 0)
            .like(queryDTO.getUsername() != null, User::getUsername, queryDTO.getUsername())
            .like(queryDTO.getPhone() != null, User::getPhone, queryDTO.getPhone())
            .eq(queryDTO.getStatus() != null, User::getStatus, queryDTO.getStatus())
            .orderByDesc(User::getCreateTime);

    Page<User> result = userMapper.selectPage(page, wrapper);

    List<UserVO> records = result.getRecords().stream()
            .map(this::toVO)
            .toList();

    return new PageResult<>(result.getTotal(), records);
}
```

转换方法：

```java
private UserVO toVO(User user) {
    UserVO vo = new UserVO();
    vo.setId(user.getId());
    vo.setUsername(user.getUsername());
    vo.setNickname(user.getNickname());
    vo.setPhone(user.getPhone());
    vo.setEmail(user.getEmail());
    vo.setStatus(user.getStatus());
    vo.setCreateTime(user.getCreateTime());
    return vo;
}
```

前端接收：

```json
{
  "total": 20,
  "records": [
    {
      "id": 1,
      "username": "admin",
      "nickname": "管理员"
    }
  ]
}
```

## 10.11 新增用户实现

新增用户要做几件事：

```text
校验用户名是否重复。
设置默认状态。
加密密码。
保存用户。
```

示例：

```java
@Override
public void create(UserCreateDTO dto) {
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(User::getUsername, dto.getUsername());
    Long count = userMapper.selectCount(wrapper);

    if (count > 0) {
        throw new BusinessException("用户名已存在");
    }

    User user = new User();
    user.setUsername(dto.getUsername());
    user.setPassword(dto.getPassword());
    user.setNickname(dto.getNickname());
    user.setPhone(dto.getPhone());
    user.setEmail(dto.getEmail());
    user.setStatus(1);
    user.setDeleted(0);
    user.setCreateTime(LocalDateTime.now());
    user.setUpdateTime(LocalDateTime.now());

    userMapper.insert(user);
}
```

注意：这里的密码暂时未加密，是为了突出模块流程。真实项目应使用 BCrypt：

```java
user.setPassword(passwordEncoder.encode(dto.getPassword()));
```

登录认证章节会详细讲。

## 10.12 修改用户实现

```java
@Override
public void update(Long id, UserUpdateDTO dto) {
    User user = userMapper.selectById(id);
    if (user == null || Integer.valueOf(1).equals(user.getDeleted())) {
        throw new BusinessException("用户不存在");
    }

    user.setNickname(dto.getNickname());
    user.setPhone(dto.getPhone());
    user.setEmail(dto.getEmail());
    user.setStatus(dto.getStatus());
    user.setUpdateTime(LocalDateTime.now());

    userMapper.updateById(user);
}
```

修改接口通常不允许修改：

- id。
- username。
- password。
- createTime。
- deleted。

这些字段要么不应该由前端修改，要么需要独立接口处理。

## 10.13 查询详情实现

```java
@Override
public UserDetailVO getById(Long id) {
    User user = userMapper.selectById(id);
    if (user == null || Integer.valueOf(1).equals(user.getDeleted())) {
        throw new BusinessException("用户不存在");
    }

    UserDetailVO vo = new UserDetailVO();
    vo.setId(user.getId());
    vo.setUsername(user.getUsername());
    vo.setNickname(user.getNickname());
    vo.setPhone(user.getPhone());
    vo.setEmail(user.getEmail());
    vo.setStatus(user.getStatus());
    vo.setCreateTime(user.getCreateTime());
    vo.setRoles(List.of());
    return vo;
}
```

详情接口常用于编辑回显。

前端编辑用户时：

```text
打开编辑弹窗
调用 GET /users/{id}
后端返回详情
前端填充表单
用户修改后提交 PUT /users/{id}
```

## 10.14 删除用户实现

逻辑删除：

```java
@Override
public void delete(Long id) {
    User user = userMapper.selectById(id);
    if (user == null || Integer.valueOf(1).equals(user.getDeleted())) {
        throw new BusinessException("用户不存在");
    }

    user.setDeleted(1);
    user.setUpdateTime(LocalDateTime.now());
    userMapper.updateById(user);
}
```

删除前可以增加业务限制：

```java
if ("admin".equals(user.getUsername())) {
    throw new BusinessException("管理员账号不能删除");
}
```

真实系统中，删除用户前还可能检查：

- 是否绑定角色。
- 是否有关联订单。
- 是否是当前登录用户。
- 是否是系统内置账号。

## 10.15 状态切换实现

状态切换接口：

```text
PATCH /users/{id}/status
```

Service：

```java
@Override
public void updateStatus(Long id, UserStatusDTO dto) {
    if (dto.getStatus() != 0 && dto.getStatus() != 1) {
        throw new BusinessException("状态值不合法");
    }

    User user = userMapper.selectById(id);
    if (user == null || Integer.valueOf(1).equals(user.getDeleted())) {
        throw new BusinessException("用户不存在");
    }

    user.setStatus(dto.getStatus());
    user.setUpdateTime(LocalDateTime.now());
    userMapper.updateById(user);
}
```

前端开关组件：

```ts
function changeStatus(id, status) {
  return request.patch(`/users/${id}/status`, { status })
}
```

## 10.16 Controller 实现

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

    @GetMapping("/{id}")
    public Result<UserDetailVO> getById(@PathVariable Long id) {
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

    @PatchMapping("/{id}/status")
    public Result<Void> updateStatus(@PathVariable Long id, @Valid @RequestBody UserStatusDTO dto) {
        userService.updateStatus(id, dto);
        return Result.success();
    }
}
```

Controller 保持清爽，只做：

```text
接收请求
调用 Service
返回 Result
```

## 10.17 角色管理模块设计

角色表：

```sql
CREATE TABLE sys_role (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '角色ID',
    role_name VARCHAR(50) NOT NULL COMMENT '角色名称',
    role_code VARCHAR(50) NOT NULL COMMENT '角色编码',
    status TINYINT DEFAULT 1 COMMENT '状态：0禁用，1启用',
    deleted TINYINT DEFAULT 0 COMMENT '逻辑删除',
    create_time DATETIME COMMENT '创建时间',
    update_time DATETIME COMMENT '更新时间',
    UNIQUE KEY uk_role_code (role_code)
) COMMENT='角色表';
```

角色模块重点：

- 角色编码唯一。
- 系统内置角色不能随意删除。
- 删除角色前检查是否已分配给用户。
- 角色要能分配菜单权限。

接口：

```text
GET    /roles
GET    /roles/{id}
POST   /roles
PUT    /roles/{id}
DELETE /roles/{id}
```

角色模块和用户模块结构类似，适合用来练习标准 CRUD。

## 10.18 菜单管理模块设计

菜单通常是树形结构。

表设计：

```sql
CREATE TABLE sys_menu (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '菜单ID',
    parent_id BIGINT DEFAULT 0 COMMENT '父菜单ID',
    menu_name VARCHAR(50) NOT NULL COMMENT '菜单名称',
    path VARCHAR(200) COMMENT '路由路径',
    component VARCHAR(200) COMMENT '组件路径',
    permission VARCHAR(100) COMMENT '权限标识',
    menu_type TINYINT COMMENT '类型：1目录，2菜单，3按钮',
    sort INT DEFAULT 0 COMMENT '排序',
    status TINYINT DEFAULT 1 COMMENT '状态',
    create_time DATETIME COMMENT '创建时间',
    update_time DATETIME COMMENT '更新时间'
) COMMENT='菜单表';
```

菜单模块重点：

- `parent_id` 表示父子关系。
- 菜单需要组装成树。
- 按钮权限通常没有路由路径，但有权限标识。

树形结构 VO：

```java
@Data
public class MenuVO {
    private Long id;
    private Long parentId;
    private String menuName;
    private String path;
    private String permission;
    private List<MenuVO> children;
}
```

前端动态路由和权限菜单会依赖这个模块。

## 10.19 文件上传模块设计

文件上传是后台系统常见功能。

接口：

```text
POST /files/upload
```

Controller：

```java
@PostMapping("/upload")
public Result<FileVO> upload(@RequestParam("file") MultipartFile file) {
    return Result.success(fileService.upload(file));
}
```

Service 要处理：

- 文件是否为空。
- 文件大小限制。
- 文件类型校验。
- 文件名生成。
- 保存到本地或对象存储。
- 返回访问 URL。

VO：

```java
@Data
public class FileVO {
    private String filename;
    private String url;
    private Long size;
}
```

前端上传：

```ts
const formData = new FormData()
formData.append('file', file)
request.post('/files/upload', formData)
```

## 10.20 操作日志模块设计

操作日志用于记录用户做了什么。

表设计：

```sql
CREATE TABLE sys_operation_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT COMMENT '操作用户ID',
    username VARCHAR(50) COMMENT '操作用户名',
    operation VARCHAR(100) COMMENT '操作内容',
    request_uri VARCHAR(200) COMMENT '请求地址',
    request_method VARCHAR(20) COMMENT '请求方法',
    ip VARCHAR(50) COMMENT 'IP地址',
    create_time DATETIME COMMENT '创建时间'
) COMMENT='操作日志表';
```

记录内容：

```text
谁
在什么时候
访问了哪个接口
执行了什么操作
结果如何
```

后续可以通过 AOP 自动记录日志，而不是每个接口手写。

## 10.21 常见错误

### 错误一：Controller 写业务逻辑

不推荐：

```java
@PostMapping
public Result<Void> create(@RequestBody UserCreateDTO dto) {
    // 校验用户名
    // 查询数据库
    // 设置字段
    // 保存用户
}
```

推荐把业务放到 Service：

```java
userService.create(dto);
```

### 错误二：新增和修改共用一个 DTO

不推荐：

```java
public class UserDTO {
    private String username;
    private String password;
    private String nickname;
}
```

新增需要 password，修改可能不允许改 password。应拆分：

```text
UserCreateDTO
UserUpdateDTO
```

### 错误三：不做唯一性校验

只靠前端校验不够。后端要查重，数据库也要加唯一索引。

### 错误四：删除不考虑关联数据

删除角色前应该检查是否已有用户绑定该角色。

### 错误五：接口返回 Entity

容易泄露敏感字段，也会让接口和数据库结构绑定过深。

## 10.22 练习任务

### 任务一：完成商品管理模块

实现：

```text
GET    /products
GET    /products/{id}
POST   /products
PUT    /products/{id}
DELETE /products/{id}
PATCH  /products/{id}/status
```

### 任务二：完成角色管理模块

实现角色 CRUD，并保证 `roleCode` 唯一。

### 任务三：完成菜单树

从 `sys_menu` 查询所有菜单，组装成树形结构返回前端。

### 任务四：完成文件上传

实现本地文件上传，返回文件访问 URL。

### 任务五：增加操作日志表

设计操作日志表，先手动在新增用户时插入一条日志。

## 10.23 面试常问点

### 问题一：开发一个业务模块的标准流程是什么？

参考回答：

```text
先分析需求和数据结构，再设计表，然后创建 Entity、DTO、VO、Mapper、Service、Controller。
接着实现分页、详情、新增、修改、删除，并补充参数校验、异常处理和接口文档。
```

### 问题二：为什么新增 DTO 和修改 DTO 要分开？

参考回答：

```text
新增和修改允许提交的字段不同。
例如新增用户需要密码，修改用户资料时通常不允许直接修改密码。
分开 DTO 可以让接口语义更清晰，参数校验更准确。
```

### 问题三：为什么删除前要检查关联数据？

参考回答：

```text
如果数据已经被其他业务引用，直接删除可能导致数据不一致。
例如角色已分配给用户时，删除角色会影响用户权限。
```

### 问题四：菜单为什么要设计 parent_id？

参考回答：

```text
菜单通常是树形结构，parent_id 用来表示父子关系。
通过 parent_id 可以组装目录、菜单、按钮等层级结构。
```

## 10.24 本章总结

本章学习了标准业务模块开发：

```text
模块开发流程
用户管理模块
表设计
Entity
DTO
VO
Mapper
Service
Controller
分页查询
新增
修改
详情
删除
状态切换
角色管理
菜单管理
文件上传
操作日志
```

你现在应该具备开发普通后台 CRUD 模块的能力。

下一章将进入登录认证与 JWT。后台系统不是所有接口都能随便访问，必须先登录，再携带 Token 调用接口。你将学习账号密码登录、密码加密、JWT 生成、Token 校验和获取当前登录用户。

