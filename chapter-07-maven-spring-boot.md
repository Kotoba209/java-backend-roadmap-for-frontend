# 第七章：Maven 与 Spring Boot 项目入门

## 7.1 本章目标

前面已经学习了 Java 基础、面向对象、常用 API、MySQL 和 JDBC。现在开始进入真正的 Java 后端项目开发。

真实 Java 后端项目通常不是手动编译一个 `.java` 文件，而是使用 Maven 管理依赖和构建项目，使用 Spring Boot 快速搭建后端应用。

本章要解决这些问题：

```text
Maven 是什么？
pom.xml 和 package.json 有什么相似点？
Spring Boot 是什么？
Spring Boot 项目目录怎么组织？
application.yml 是什么？
启动类是什么？
Controller、Service 如何被 Spring 管理？
什么是 Bean？
什么是依赖注入？
```

学完本章后，你应该能够：

- 理解 Maven 的作用。
- 看懂基础 `pom.xml`。
- 理解 Spring Boot 的作用。
- 创建并运行 Spring Boot 项目。
- 理解 Spring Boot 标准目录结构。
- 编写第一个接口。
- 理解 `application.yml` 配置文件。
- 理解 Bean 和依赖注入。
- 能搭建后续项目实战的基础工程。

## 7.2 Maven 是什么

Maven 是 Java 项目的依赖管理和构建工具。

前端类比：

```text
npm / pnpm 管理前端依赖。
Maven 管理 Java 依赖。
```

前端项目中：

```json
{
  "dependencies": {
    "axios": "^1.6.0",
    "vue": "^3.4.0"
  }
}
```

Java Maven 项目中：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

对照理解：

| 前端 | Java 后端 |
|---|---|
| package.json | pom.xml |
| npm install | mvn install / Maven 自动下载 |
| node_modules | 本地 Maven 仓库 |
| npm run build | mvn package |
| 依赖包 | dependency |

Maven 主要解决：

- 下载依赖。
- 管理依赖版本。
- 编译代码。
- 运行测试。
- 打包项目。

## 7.3 pom.xml 基础结构

`pom.xml` 是 Maven 项目的核心配置文件。

一个简化版 `pom.xml`：

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>admin-system</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

核心字段：

- `groupId`：组织或公司标识。
- `artifactId`：项目名。
- `version`：项目版本。
- `dependencies`：项目依赖。

可以类比：

```json
{
  "name": "admin-system",
  "version": "1.0.0",
  "dependencies": {}
}
```

但 Maven 更强调 Java 包坐标：

```text
groupId + artifactId + version
```

这三者共同确定一个依赖。

## 7.4 常见 Maven 依赖

Spring Boot Web 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

作用：

```text
支持开发 Web 接口，包括 Controller、JSON、Tomcat 等。
```

MyBatis-Plus 依赖：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.7</version>
</dependency>
```

MySQL 驱动：

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

Lombok：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

参数校验：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

这些依赖会在后续项目中逐步使用。

## 7.5 Spring Boot 是什么

Spring Boot 是 Java 后端开发中非常主流的框架。

可以这样理解：

```text
Spring Boot 帮你快速创建和运行一个 Java 后端应用。
```

前端类比：

```text
Vue / React 解决前端应用组织问题。
Spring Boot 解决后端应用组织问题。
```

不用 Spring Boot 时，你可能需要手动配置大量内容：

- Web 服务器。
- JSON 转换。
- 依赖注入。
- 配置文件。
- 数据库连接。
- 日志。

Spring Boot 帮你做了大量自动配置，让你可以更快开始写业务接口。

一个简单接口：

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello Spring Boot";
    }
}
```

访问：

```text
GET http://localhost:8080/hello
```

返回：

```text
Hello Spring Boot
```

## 7.6 Spring Boot 项目目录结构

标准目录：

```text
admin-system
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── admin
    │   │               ├── AdminApplication.java
    │   │               ├── controller
    │   │               ├── service
    │   │               ├── mapper
    │   │               ├── entity
    │   │               ├── dto
    │   │               ├── vo
    │   │               ├── common
    │   │               ├── config
    │   │               └── exception
    │   └── resources
    │       └── application.yml
    └── test
        └── java
```

关键目录说明：

- `src/main/java`：Java 源代码。
- `src/main/resources`：配置文件、静态资源、Mapper XML。
- `src/test/java`：测试代码。
- `pom.xml`：Maven 配置。

业务包说明：

| 包 | 作用 |
|---|---|
| controller | 接口入口 |
| service | 业务逻辑 |
| mapper | 数据库访问 |
| entity | 数据库实体 |
| dto | 请求参数对象 |
| vo | 响应数据对象 |
| common | 通用类 |
| config | 配置类 |
| exception | 异常处理 |

前端类比：

```text
controller 类似 api 入口定义。
service 类似业务逻辑封装。
common 类似 utils。
config 类似项目配置。
dto / vo 类似 TypeScript 类型定义。
```

## 7.7 Spring Boot 启动类

启动类通常长这样：

```java
package com.example.admin;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(AdminApplication.class, args);
    }
}
```

`@SpringBootApplication` 是 Spring Boot 的核心注解。

它表示：

```text
这是一个 Spring Boot 应用。
请从这里启动并扫描相关组件。
```

运行 `main` 方法后，Spring Boot 会启动内置 Web 服务器，默认端口是 `8080`。

控制台通常会看到：

```text
Tomcat started on port 8080
Started AdminApplication
```

这表示后端服务已经启动。

## 7.8 application.yml 配置文件

`application.yml` 是 Spring Boot 的配置文件。

前端类比：

```text
.env.development
vite.config.ts
```

后端常见配置：

```yaml
server:
  port: 8080

spring:
  application:
    name: admin-system
  datasource:
    url: jdbc:mysql://localhost:3306/java_admin?useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

说明：

- `server.port`：后端服务端口。
- `spring.application.name`：应用名称。
- `spring.datasource.url`：数据库连接地址。
- `spring.datasource.username`：数据库用户名。
- `spring.datasource.password`：数据库密码。

如果端口改成：

```yaml
server:
  port: 9000
```

接口访问地址就变成：

```text
http://localhost:9000
```

## 7.9 YAML 语法注意点

YAML 使用缩进表示层级。

正确：

```yaml
server:
  port: 8080
```

错误：

```yaml
server:
port: 8080
```

缩进通常使用两个空格，不要使用 Tab。

列表写法：

```yaml
app:
  allowed-origins:
    - http://localhost:5173
    - http://localhost:3000
```

YAML 对空格敏感，初学时如果 Spring Boot 启动失败，要检查缩进。

## 7.10 第一个 Controller

创建：

```text
com.example.admin.controller.HelloController
```

代码：

```java
package com.example.admin.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello Spring Boot";
    }
}
```

启动项目后访问：

```text
GET http://localhost:8080/hello
```

返回：

```text
Hello Spring Boot
```

`@RestController` 表示这个类用于开发 REST 接口。

`@GetMapping("/hello")` 表示处理 GET `/hello` 请求。

前端请求：

```ts
const res = await fetch('http://localhost:8080/hello')
const text = await res.text()
console.log(text)
```

## 7.11 返回 JSON

后端更多时候返回 JSON，而不是普通字符串。

定义对象：

```java
public class UserVO {
    private Long id;
    private String username;

    public UserVO(Long id, String username) {
        this.id = id;
        this.username = username;
    }

    public Long getId() {
        return id;
    }

    public String getUsername() {
        return username;
    }
}
```

Controller：

```java
@GetMapping("/user")
public UserVO user() {
    return new UserVO(1L, "admin");
}
```

访问：

```text
GET http://localhost:8080/user
```

返回：

```json
{
  "id": 1,
  "username": "admin"
}
```

Spring Boot 会自动把 Java 对象转换成 JSON。这个过程通常由 Jackson 完成。

## 7.12 Bean 是什么

Bean 是由 Spring 容器管理的对象。

普通对象：

```java
UserService userService = new UserService();
```

Spring 管理的对象：

```java
@Service
public class UserService {
}
```

加上 `@Service` 后，Spring 会创建并管理这个对象。其他类需要它时，可以通过依赖注入获取。

常见组件注解：

| 注解 | 常用位置 |
|---|---|
| `@Component` | 通用组件 |
| `@Controller` | MVC 页面控制器 |
| `@RestController` | REST 接口控制器 |
| `@Service` | 业务逻辑类 |
| `@Repository` | 数据访问类 |
| `@Configuration` | 配置类 |

可以先简单理解：

```text
被 Spring 管理的对象就是 Bean。
```

## 7.13 依赖注入

依赖注入表示：一个类需要另一个类时，不自己 new，而是让 Spring 帮你提供。

不推荐：

```java
public class UserController {
    private UserService userService = new UserService();
}
```

推荐构造器注入：

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

Service：

```java
@Service
public class UserService {
}
```

Spring 会自动把 `UserService` 注入到 `UserController`。

前端类比不完全准确，但可以先理解为：

```text
组件需要某个服务，不自己创建，而是由框架统一装配。
```

依赖注入的好处：

- 降低类之间耦合。
- 方便测试。
- 方便替换实现。
- 由框架统一管理对象生命周期。

## 7.14 @Autowired 和构造器注入

早期项目中常见：

```java
@Autowired
private UserService userService;
```

这种方式叫字段注入。

更推荐构造器注入：

```java
private final UserService userService;

public UserController(UserService userService) {
    this.userService = userService;
}
```

原因：

- 依赖关系更清晰。
- 字段可以使用 `final`。
- 更利于测试。
- 避免隐藏依赖。

如果使用 Lombok，可以写：

```java
@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;
}
```

`@RequiredArgsConstructor` 会为 `final` 字段生成构造方法。

## 7.15 Service 接口和实现类

真实项目中常见写法：

```java
public interface UserService {
    UserVO getById(Long id);
}
```

实现类：

```java
@Service
public class UserServiceImpl implements UserService {

    @Override
    public UserVO getById(Long id) {
        return new UserVO(id, "admin");
    }
}
```

Controller 依赖接口：

```java
private final UserService userService;
```

为什么要接口？

```text
面向接口编程，方便替换实现。
便于测试和扩展。
符合 Java 后端常见分层规范。
```

小项目也可以先直接写 Service 类，但学习企业项目时，建议理解接口和实现类这种结构。

## 7.16 多环境配置

真实项目通常有多个环境：

```text
dev：开发环境
test：测试环境
prod：生产环境
```

配置文件：

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

主配置：

```yaml
spring:
  profiles:
    active: dev
```

开发环境：

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/java_admin_dev
    username: root
    password: 123456
```

生产环境可能是：

```yaml
spring:
  datasource:
    url: jdbc:mysql://mysql-prod:3306/java_admin
    username: admin_user
    password: ${MYSQL_PASSWORD}
```

前端类比：

```text
.env.development
.env.production
```

不同环境使用不同接口地址，后端也是类似，不同环境使用不同数据库、Redis、日志配置。

## 7.17 热部署和开发体验

开发阶段可以使用 Spring Boot DevTools：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

它可以在代码变化后自动重启应用。

不过真实团队中，是否使用 DevTools 看项目规范。对于初学者，IDEA 直接重启也可以。

后端开发常见流程：

```text
修改 Controller / Service
重启应用
用浏览器、Postman 或 Knife4j 测试接口
查看控制台日志
修复问题
```

## 7.18 常见错误

### 错误一：启动类位置放错

Spring Boot 默认扫描启动类所在包及其子包。

推荐：

```text
com.example.admin.AdminApplication
com.example.admin.controller.UserController
com.example.admin.service.UserService
```

如果 Controller 放在启动类包外，可能扫描不到。

### 错误二：忘记加 @RestController

错误：

```java
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

Spring 不会把它当接口类。

正确：

```java
@RestController
public class HelloController {
}
```

### 错误三：YAML 缩进错误

错误：

```yaml
server:
port: 8080
```

正确：

```yaml
server:
  port: 8080
```

### 错误四：Bean 没有交给 Spring 管理

Controller 依赖：

```java
private final UserService userService;
```

但 Service 忘记写：

```java
@Service
```

启动时可能报找不到 Bean。

## 7.19 练习任务

### 任务一：创建 Spring Boot 项目

创建一个项目：

```text
admin-system
```

依赖选择：

- Spring Web。
- Lombok。
- Validation。

### 任务二：编写 Hello 接口

实现：

```text
GET /hello
```

返回：

```text
Hello Spring Boot
```

### 任务三：编写用户详情接口

实现：

```text
GET /users/1
```

返回：

```json
{
  "id": 1,
  "username": "admin"
}
```

### 任务四：拆分 Controller 和 Service

要求：

- `UserController` 调用 `UserService`。
- `UserService` 返回用户数据。
- Controller 不直接创建用户数据。

### 任务五：修改端口

将服务端口改为：

```yaml
server:
  port: 9000
```

然后访问：

```text
http://localhost:9000/hello
```

## 7.20 面试常问点

### 问题一：Maven 有什么作用？

参考回答：

```text
Maven 用于管理 Java 项目的依赖、编译、测试和打包。
它通过 pom.xml 描述项目依赖和构建配置。
```

### 问题二：Spring Boot 有什么作用？

参考回答：

```text
Spring Boot 用于快速创建和运行 Spring 应用。
它提供自动配置、内置 Web 服务器和常用 starter，减少大量手动配置。
```

### 问题三：application.yml 是什么？

参考回答：

```text
application.yml 是 Spring Boot 的配置文件，可以配置端口、数据库、Redis、日志、多环境等内容。
```

### 问题四：什么是 Bean？

参考回答：

```text
Bean 是由 Spring 容器创建和管理的对象。
例如加了 @Service、@Component、@RestController 的类，通常会被 Spring 扫描并注册成 Bean。
```

### 问题五：什么是依赖注入？

参考回答：

```text
依赖注入是由 Spring 容器把一个对象需要的依赖自动提供给它，而不是在类中手动 new。
这样可以降低耦合，方便测试和扩展。
```

## 7.21 本章总结

本章学习了 Maven 与 Spring Boot 项目入门：

```text
Maven
pom.xml
dependency
Spring Boot
项目目录结构
启动类
application.yml
Controller
JSON 返回
Bean
依赖注入
构造器注入
Service 接口和实现类
多环境配置
```

前端人员可以这样类比：

```text
Maven 类似 npm / pnpm。
pom.xml 类似 package.json。
application.yml 类似环境配置和框架配置。
Spring Boot 类似后端应用框架。
Controller 是接口入口。
Service 是业务逻辑层。
Bean 是由框架管理的对象。
依赖注入是由框架帮你装配对象。
```

下一章将学习 Spring MVC 与 RESTful 接口开发。你会系统掌握 GET、POST、PUT、DELETE、路径参数、查询参数、JSON 请求体、统一返回、参数校验和跨域配置。

