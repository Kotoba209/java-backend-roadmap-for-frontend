# 第十二章：权限控制与 RBAC

## 12.1 本章目标

上一章解决了“用户是谁”的问题，也就是登录认证。本章解决“用户能做什么”的问题，也就是权限控制。

后台管理系统中常见需求：

```text
普通用户不能进入用户管理页面。
财务人员只能查看订单，不能删除订单。
管理员可以分配角色。
不同角色看到不同菜单。
没有 user:add 权限时，不显示新增按钮。
即使前端隐藏按钮，后端接口也必须校验权限。
```

学完本章后，你应该能够：

- 理解认证和授权的区别。
- 理解 RBAC 权限模型。
- 设计用户、角色、菜单、权限相关表。
- 实现用户角色分配。
- 实现角色菜单分配。
- 返回当前用户菜单树。
- 返回当前用户权限标识。
- 理解前端菜单权限和按钮权限。
- 理解后端接口权限校验。
- 掌握 Sa-Token 和 Spring Security 的选型思路。

## 12.2 认证和授权再次区分

认证：

```text
确认你是谁。
```

例如：

```text
admin 登录成功。
```

授权：

```text
确认你能访问什么。
```

例如：

```text
admin 可以访问用户管理。
test 不能删除用户。
editor 可以编辑文章，但不能管理角色。
```

前端路由守卫只能控制页面跳转，不能保证接口安全。  
真正的权限校验必须在后端接口层完成。

## 12.3 RBAC 是什么

RBAC 全称 Role-Based Access Control，基于角色的访问控制。

核心关系：

```text
用户 -> 角色 -> 权限
```

例如：

```text
张三 拥有 管理员角色
管理员角色 拥有 用户新增、用户删除、角色分配权限
所以张三可以新增用户、删除用户、分配角色
```

为什么不直接给用户分配权限？

```text
如果每个用户单独分配权限，维护成本很高。
通过角色分组，可以批量管理权限。
```

常见角色：

```text
超级管理员
系统管理员
运营人员
财务人员
普通用户
访客
```

## 12.4 RBAC 表设计

最小 RBAC 通常需要五张表：

```text
sys_user：用户表
sys_role：角色表
sys_menu：菜单权限表
sys_user_role：用户角色关联表
sys_role_menu：角色菜单关联表
```

用户表前面已经设计过。

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

用户角色表：

```sql
CREATE TABLE sys_user_role (
    user_id BIGINT NOT NULL COMMENT '用户ID',
    role_id BIGINT NOT NULL COMMENT '角色ID',
    PRIMARY KEY (user_id, role_id)
) COMMENT='用户角色关联表';
```

菜单权限表：

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

## 12.5 菜单、按钮、接口权限

权限不只是菜单。

常见权限类型：

```text
目录：例如 系统管理。
菜单：例如 用户管理。
按钮：例如 新增用户、删除用户。
接口：例如 POST /users、DELETE /users/{id}。
```

菜单表示页面入口：

```text
用户管理页面
角色管理页面
菜单管理页面
```

按钮表示页面上的操作：

```text
user:add
user:update
user:delete
role:assign
```

接口权限表示后端真正的访问控制：

```java
@RequiresPermission("user:add")
@PostMapping("/users")
public Result<Void> create(@RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

前端隐藏按钮只是体验优化，后端接口权限才是安全边界。

## 12.6 分配用户角色

接口设计：

```text
PUT /users/{id}/roles
```

请求：

```json
{
  "roleIds": [1, 2, 3]
}
```

DTO：

```java
@Data
public class AssignUserRoleDTO {

    @NotEmpty(message = "角色不能为空")
    private List<Long> roleIds;
}
```

实现思路：

```text
1. 校验用户是否存在。
2. 校验角色是否存在且启用。
3. 删除用户原有角色。
4. 插入新的用户角色关系。
```

伪代码：

```java
@Transactional
public void assignRoles(Long userId, AssignUserRoleDTO dto) {
    User user = userMapper.selectById(userId);
    if (user == null) {
        throw new BusinessException("用户不存在");
    }

    userRoleMapper.deleteByUserId(userId);

    for (Long roleId : dto.getRoleIds()) {
        userRoleMapper.insert(userId, roleId);
    }
}
```

这里要使用事务。否则删除旧角色成功，插入新角色失败，会导致用户角色丢失。

## 12.7 分配角色菜单

接口：

```text
PUT /roles/{id}/menus
```

请求：

```json
{
  "menuIds": [1, 2, 3, 10, 11]
}
```

实现思路：

```text
1. 校验角色是否存在。
2. 校验菜单是否存在。
3. 删除角色原有菜单。
4. 插入新的角色菜单关系。
```

Service：

```java
@Transactional
public void assignMenus(Long roleId, AssignRoleMenuDTO dto) {
    Role role = roleMapper.selectById(roleId);
    if (role == null) {
        throw new BusinessException("角色不存在");
    }

    roleMenuMapper.deleteByRoleId(roleId);

    for (Long menuId : dto.getMenuIds()) {
        roleMenuMapper.insert(roleId, menuId);
    }
}
```

角色菜单分配通常是权限管理页面的核心功能。

## 12.8 查询当前用户菜单

前端登录后需要知道当前用户可以看到哪些菜单。

接口：

```text
GET /auth/menus
```

查询逻辑：

```text
当前用户 ID
-> 查询用户角色
-> 查询角色菜单
-> 过滤目录和菜单
-> 组装树形结构
-> 返回前端
```

SQL 思路：

```sql
SELECT DISTINCT m.*
FROM sys_menu m
JOIN sys_role_menu rm ON m.id = rm.menu_id
JOIN sys_user_role ur ON rm.role_id = ur.role_id
WHERE ur.user_id = #{userId}
  AND m.status = 1
ORDER BY m.sort ASC;
```

VO：

```java
@Data
public class MenuVO {
    private Long id;
    private Long parentId;
    private String menuName;
    private String path;
    private String component;
    private List<MenuVO> children;
}
```

组装树：

```java
private List<MenuVO> buildTree(List<MenuVO> menus, Long parentId) {
    return menus.stream()
            .filter(menu -> parentId.equals(menu.getParentId()))
            .peek(menu -> menu.setChildren(buildTree(menus, menu.getId())))
            .toList();
}
```

前端拿到菜单树后，可以生成动态路由或侧边栏菜单。

## 12.9 查询当前用户权限标识

按钮权限通常依赖权限标识。

接口：

```text
GET /auth/permissions
```

返回：

```json
[
  "user:list",
  "user:add",
  "user:update",
  "user:delete"
]
```

前端使用：

```vue
<button v-if="hasPermission('user:add')">新增用户</button>
```

后端查询：

```sql
SELECT DISTINCT m.permission
FROM sys_menu m
JOIN sys_role_menu rm ON m.id = rm.menu_id
JOIN sys_user_role ur ON rm.role_id = ur.role_id
WHERE ur.user_id = #{userId}
  AND m.permission IS NOT NULL
  AND m.permission != '';
```

权限标识命名建议：

```text
模块:操作
```

例如：

```text
user:list
user:add
user:update
user:delete
role:assign
menu:list
```

## 12.10 后端接口权限校验

前端权限不能替代后端权限。

危险场景：

```text
前端隐藏了删除按钮。
用户仍然可以通过 Postman 调用 DELETE /users/1。
```

所以后端接口必须校验权限。

可以自定义注解：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequiresPermission {
    String value();
}
```

使用：

```java
@RequiresPermission("user:delete")
@DeleteMapping("/users/{id}")
public Result<Void> delete(@PathVariable Long id) {
    userService.delete(id);
    return Result.success();
}
```

再通过 AOP 或拦截器读取注解，判断当前用户是否有权限。

学习阶段也可以使用成熟框架：

- Sa-Token。
- Spring Security。

## 12.11 Sa-Token 方案

Sa-Token 是一个较轻量的 Java 权限认证框架，上手比 Spring Security 简单。

接口权限示例：

```java
@SaCheckPermission("user:add")
@PostMapping("/users")
public Result<Void> create(@RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

登录：

```java
StpUtil.login(user.getId());
```

获取当前用户：

```java
Long userId = StpUtil.getLoginIdAsLong();
```

判断权限：

```java
StpUtil.checkPermission("user:add");
```

对于学习者来说，Sa-Token 比较适合快速实现登录和权限控制。

## 12.12 Spring Security 方案

Spring Security 是 Spring 官方安全框架，功能非常强大。

特点：

```text
能力完整。
生态成熟。
适合复杂企业项目。
学习曲线较陡。
配置相对复杂。
```

权限注解：

```java
@PreAuthorize("hasAuthority('user:add')")
@PostMapping("/users")
public Result<Void> create(@RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

如果项目需要深度安全能力，例如 OAuth2、复杂认证链、多种登录方式，Spring Security 更合适。

如果目标是先掌握后台管理系统权限模型，Sa-Token 更容易落地。

## 12.13 常见错误

### 错误一：只做前端权限

前端隐藏按钮不等于接口安全。后端必须校验接口权限。

### 错误二：直接给用户分配权限

用户多时维护成本高。更推荐：

```text
用户 -> 角色 -> 权限
```

### 错误三：删除角色不检查用户绑定

如果角色已分配给用户，直接删除会影响用户权限。删除前应检查关联关系。

### 错误四：权限标识命名混乱

不推荐：

```text
addUser
userAdd
USER_ADD
```

推荐：

```text
user:add
user:update
user:delete
```

## 12.14 练习任务

### 任务一：设计 RBAC 表

创建：

```text
sys_user
sys_role
sys_menu
sys_user_role
sys_role_menu
```

### 任务二：实现分配用户角色

接口：

```text
PUT /users/{id}/roles
```

### 任务三：实现分配角色菜单

接口：

```text
PUT /roles/{id}/menus
```

### 任务四：返回当前用户菜单树

接口：

```text
GET /auth/menus
```

### 任务五：返回当前用户权限标识

接口：

```text
GET /auth/permissions
```

## 12.15 面试常问点

### 问题一：RBAC 是什么？

参考回答：

```text
RBAC 是基于角色的访问控制模型。
用户关联角色，角色关联权限，通过角色统一管理用户权限。
```

### 问题二：为什么后端必须做权限校验？

参考回答：

```text
前端权限可以被绕过，用户可以直接调用接口。
后端接口才是真正的安全边界，因此必须在后端做权限校验。
```

### 问题三：菜单权限和按钮权限有什么区别？

参考回答：

```text
菜单权限控制用户能看到哪些页面。
按钮权限控制用户能执行哪些操作，例如新增、修改、删除。
按钮权限通常通过 permission 标识控制。
```

## 12.16 本章总结

本章学习了权限控制与 RBAC：

```text
认证和授权
RBAC 模型
用户表
角色表
菜单表
用户角色表
角色菜单表
用户角色分配
角色菜单分配
菜单树
权限标识
前端按钮权限
后端接口权限
Sa-Token
Spring Security
```

现在你已经能理解一个后台系统的完整安全模型：

```text
登录认证解决用户是谁。
权限控制解决用户能做什么。
前端控制显示。
后端控制安全。
```

下一章将学习 Redis。Redis 会用于缓存登录用户、验证码、防重复提交、热点数据缓存等后端常见场景。

