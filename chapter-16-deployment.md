# 第十六章：Docker、Nginx 与部署上线

## 16.1 本章目标

一个后端项目不能只在本地 IDE 中运行。真正完整的后端能力，必须包括部署上线和问题排查。

本章要解决：

```text
Spring Boot 项目如何打包？
jar 包如何运行？
开发、测试、生产环境配置如何区分？
MySQL 和 Redis 如何部署？
Java 后端如何做 Docker 镜像？
Nginx 如何代理后端接口？
线上日志如何查看？
接口访问失败如何排查？
```

学完本章后，你应该能够：

- 使用 Maven 打包 Spring Boot 项目。
- 使用 `java -jar` 运行后端服务。
- 配置多环境。
- 编写 Dockerfile。
- 使用 Docker 部署 MySQL、Redis 和 Java 后端。
- 使用 Nginx 反向代理接口。
- 查看日志并排查常见部署问题。

## 16.2 Maven 打包

在项目根目录执行：

```bash
mvn clean package
```

打包成功后，通常会在：

```text
target/
```

目录下生成：

```text
admin-system-1.0.0.jar
```

跳过测试：

```bash
mvn clean package -DskipTests
```

不建议长期跳过测试，但学习阶段或临时部署可以使用。

打包前要确认：

```text
项目能正常启动。
配置文件正确。
依赖能下载。
没有编译错误。
```

## 16.3 jar 包运行

运行：

```bash
java -jar admin-system-1.0.0.jar
```

指定环境：

```bash
java -jar admin-system-1.0.0.jar --spring.profiles.active=prod
```

指定端口：

```bash
java -jar admin-system-1.0.0.jar --server.port=8080
```

后台运行 Linux 示例：

```bash
nohup java -jar admin-system-1.0.0.jar --spring.profiles.active=prod > app.log 2>&1 &
```

查看日志：

```bash
tail -f app.log
```

## 16.4 多环境配置

配置文件：

```text
application.yml
application-dev.yml
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
  data:
    redis:
      host: localhost
      port: 6379
```

生产环境：

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://mysql:3306/java_admin
    username: root
    password: ${MYSQL_PASSWORD}
  data:
    redis:
      host: redis
      port: 6379
```

生产密码不要硬编码到仓库中。可以通过环境变量传入。

## 16.5 Linux 常用命令

查看当前目录：

```bash
pwd
```

查看文件：

```bash
ls -la
```

查看 Java 版本：

```bash
java -version
```

查看端口：

```bash
netstat -tunlp | grep 8080
```

查看进程：

```bash
ps -ef | grep java
```

停止进程：

```bash
kill -15 进程ID
```

查看日志：

```bash
tail -f app.log
```

搜索日志：

```bash
grep "ERROR" app.log
```

这些命令是部署排查的基础。

## 16.6 Docker 是什么

Docker 可以把应用和运行环境打包成容器。

可以这样理解：

```text
以前部署：手动安装 JDK、MySQL、Redis、配置环境。
Docker 部署：用镜像启动容器，环境更一致。
```

常见概念：

| 概念 | 说明 |
|---|---|
| 镜像 image | 应用运行模板 |
| 容器 container | 镜像运行后的实例 |
| Dockerfile | 构建镜像的说明文件 |
| docker-compose | 编排多个容器 |

后端项目通常需要：

```text
MySQL 容器
Redis 容器
Java 后端容器
Nginx 容器
```

## 16.7 Docker 部署 MySQL

启动 MySQL：

```bash
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -e MYSQL_DATABASE=java_admin \
  mysql:8.0
```

说明：

- `-d`：后台运行。
- `--name mysql`：容器名称。
- `-p 3306:3306`：端口映射。
- `MYSQL_ROOT_PASSWORD`：root 密码。
- `MYSQL_DATABASE`：默认创建数据库。

查看容器：

```bash
docker ps
```

查看日志：

```bash
docker logs mysql
```

## 16.8 Docker 部署 Redis

启动 Redis：

```bash
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:7
```

查看：

```bash
docker ps
```

进入 Redis：

```bash
docker exec -it redis redis-cli
```

测试：

```bash
SET name admin
GET name
```

生产环境建议配置密码、持久化和网络隔离。

## 16.9 Java 后端 Dockerfile

Dockerfile：

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY target/admin-system-1.0.0.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar", "--spring.profiles.active=prod"]
```

构建镜像：

```bash
docker build -t admin-system:1.0.0 .
```

运行：

```bash
docker run -d \
  --name admin-system \
  -p 8080:8080 \
  -e MYSQL_PASSWORD=123456 \
  admin-system:1.0.0
```

查看日志：

```bash
docker logs -f admin-system
```

## 16.10 docker-compose 编排

`docker-compose.yml`：

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: java_admin
    ports:
      - "3306:3306"

  redis:
    image: redis:7
    container_name: redis
    ports:
      - "6379:6379"

  admin-system:
    image: admin-system:1.0.0
    container_name: admin-system
    depends_on:
      - mysql
      - redis
    environment:
      MYSQL_PASSWORD: 123456
    ports:
      - "8080:8080"
```

启动：

```bash
docker compose up -d
```

停止：

```bash
docker compose down
```

查看日志：

```bash
docker compose logs -f admin-system
```

## 16.11 Nginx 反向代理

前端通常部署到 Nginx，后端也可以通过 Nginx 代理。

示例：

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://admin-system:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

前端请求：

```ts
request.get('/api/users')
```

Nginx 转发到：

```text
http://admin-system:8080/users
```

这样前端不需要直接暴露后端端口。

## 16.12 跨域和反向代理

开发环境：

```text
前端 localhost:5173
后端 localhost:8080
```

可能需要 CORS 或 Vite 代理。

生产环境更推荐：

```text
同一个域名下：
/ 前端页面
/api 后端接口
```

例如：

```text
https://example.com
https://example.com/api/users
```

这样浏览器看起来是同源，跨域问题更少。

## 16.13 日志配置

建议配置日志文件：

```yaml
logging:
  file:
    name: logs/admin-system.log
  level:
    com.example.admin: info
```

查看日志：

```bash
tail -f logs/admin-system.log
```

生产环境排查重点：

```text
启动是否成功。
端口是否监听。
数据库是否连接成功。
Redis 是否连接成功。
接口是否报 401、403、500。
SQL 是否执行异常。
文件上传路径是否存在。
```

## 16.14 常见部署问题

### 问题一：端口被占用

现象：

```text
Port 8080 was already in use
```

处理：

```bash
netstat -tunlp | grep 8080
```

更换端口或停止占用进程。

### 问题二：数据库连接失败

检查：

```text
数据库地址是否正确。
端口是否开放。
用户名密码是否正确。
数据库是否存在。
容器网络是否能互通。
```

### 问题三：Redis 连接失败

检查：

```text
Redis 容器是否启动。
host 和 port 是否正确。
是否配置了密码。
```

### 问题四：接口 401

检查：

```text
前端是否携带 Token。
Header 名称是否正确。
Token 是否过期。
后端是否正确解析 Bearer。
```

### 问题五：Nginx 代理 502

检查：

```text
后端服务是否启动。
proxy_pass 地址是否正确。
容器名是否能解析。
后端端口是否正确。
```

## 16.15 上线检查清单

部署前检查：

```text
生产配置文件已准备。
数据库初始化 SQL 已执行。
MySQL 可连接。
Redis 可连接。
后端 jar 可运行。
Docker 镜像可构建。
接口文档可访问。
登录接口可用。
Token 拦截生效。
Nginx 代理正确。
日志文件可查看。
文件上传目录存在。
```

上线后验证：

```text
访问首页。
登录系统。
查看菜单。
查询用户列表。
新增一条测试数据。
查看日志。
重启服务后再次访问。
```

## 16.16 练习任务

### 任务一：Maven 打包

执行：

```bash
mvn clean package
```

确认 `target` 下生成 jar。

### 任务二：jar 运行

使用：

```bash
java -jar xxx.jar --spring.profiles.active=prod
```

启动后访问登录接口。

### 任务三：编写 Dockerfile

为后端项目构建 Docker 镜像。

### 任务四：docker-compose 启动 MySQL、Redis、后端

编写 `docker-compose.yml` 并启动。

### 任务五：配置 Nginx 代理

实现：

```text
/api -> 后端服务
/ -> 前端静态页面
```

## 16.17 面试常问点

### 问题一：Spring Boot 项目如何部署？

参考回答：

```text
可以使用 Maven 打包成 jar，通过 java -jar 运行。
也可以编写 Dockerfile 构建镜像，再用 Docker 或 docker-compose 部署。
```

### 问题二：Nginx 在前后端分离项目中有什么作用？

参考回答：

```text
Nginx 可以部署前端静态资源，也可以作为反向代理，把 /api 请求转发到后端服务。
```

### 问题三：线上接口访问失败如何排查？

参考回答：

```text
先确认服务是否启动、端口是否监听、Nginx 代理是否正确。
再查看后端日志，检查数据库、Redis、Token、权限和异常堆栈。
```

## 16.18 本章总结

本章学习了项目部署上线：

```text
Maven 打包
jar 运行
多环境配置
Linux 常用命令
Docker
MySQL 容器
Redis 容器
Dockerfile
docker-compose
Nginx 反向代理
跨域处理
日志配置
部署问题排查
上线检查清单
```

到这里，你已经完成从前端开发人员转型 Java 后端开发的完整学习闭环：

```text
Java 基础
面向对象
常用 API
MySQL
JDBC
Spring Boot
Spring MVC
MyBatis-Plus
业务模块
登录认证
权限控制
Redis
工程规范
项目实战
部署上线
```

真正掌握后端开发，不是背完知识点，而是能把这些能力组合起来，独立完成一个可运行、可维护、可部署的后端系统。

