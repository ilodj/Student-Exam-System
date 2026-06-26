# 在线学生考试管理系统 — 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个功能完整的在线考试管理系统，支持题库管理、自动组卷、在线考试、自动批改、成绩分析，并在此过程中学会前后端全栈交互的所有核心知识点。

**Architecture:** 经典前后端分离架构。前端 Vue2 + Element UI 通过 Axios 发 HTTP 请求到后端 SpringBoot REST API，后端通过 MyBatis-Plus 操作 MySQL。认证使用 JWT 无状态方案，前后端通过请求头 `Authorization: Bearer <token>` 传递身份。

**Tech Stack:** Vue2, Element UI, Vuex, Vue Router, Axios, SpringBoot 2.x, MyBatis-Plus, Spring Security, JWT, MySQL 8.0

## Global Constraints

- Java 17+, Node v20+, npm 10+
- Maven Wrapper（`mvnw`）构建后端，无需系统安装 Maven
- 后端端口 8080，前端端口 8081（开发服务器）
- 数据库名 `exam_system`，字符集 utf8mb4
- 所有 API 统一返回格式 `{ code: 200, message: "success", data: ... }`
- JWT Token 有效期 24 小时，存储在浏览器 localStorage
- 项目根目录 `F:/exam-system/`，前后端分为 `backend/` 和 `frontend/` 两个子目录

---

## 项目文件结构总览

```
F:/exam-system/
├── backend/                          # SpringBoot 后端项目
│   ├── pom.xml
│   ├── mvnw / mvnw.cmd               # Maven Wrapper（无需安装 Maven）
│   ├── .mvn/wrapper/
│   └── src/main/
│       ├── java/com/exam/
│       │   ├── ExamApplication.java   # SpringBoot 启动类
│       │   ├── config/                # 配置类
│       │   │   ├── SecurityConfig.java
│       │   │   ├── CorsConfig.java
│       │   │   └── MyBatisPlusConfig.java
│       │   ├── common/                # 公共工具
│       │   │   ├── Result.java        # 统一响应格式
│       │   │   ├── GlobalExceptionHandler.java
│       │   │   └── JwtUtil.java
│       │   ├── security/              # Spring Security 相关
│       │   │   ├── JwtAuthenticationFilter.java
│       │   │   └── UserDetailsServiceImpl.java
│       │   ├── entity/                # 数据库实体
│       │   ├── dto/                   # 前后端传输对象
│       │   ├── mapper/                # MyBatis Mapper 接口
│       │   ├── service/               # 业务逻辑层
│       │   │   └── impl/
│       │   └── controller/            # REST 控制器
│       └── resources/
│           ├── application.yml
│           └── db/schema.sql          # 数据库建表脚本
├── frontend/                          # Vue2 前端项目
│   ├── package.json
│   ├── vue.config.js
│   └── src/
│       ├── main.js
│       ├── App.vue
│       ├── router/index.js
│       ├── store/index.js
│       ├── utils/request.js           # Axios 封装
│       ├── api/                        # 各模块 API 调用
│       ├── views/                      # 页面组件
│       └── components/                 # 公共组件
└── docs/
    └── superpowers/
        ├── specs/2026-06-26-exam-system-design.md
        └── plans/2026-06-26-exam-system-plan.md
```

---

## Phase 1: 基础设施搭建

### Task 1.1: 初始化 Git 仓库与 GitHub 连接

**Files:**
- Create: `F:/exam-system/.gitignore`
- Create: `F:/exam-system/README.md`

**Steps:**

- [ ] **Step 1: 初始化 Git 仓库**
```bash
cd /f/exam-system
git init
```

- [ ] **Step 2: 创建 .gitignore**
```gitignore
# Java
*.class
*.jar
*.war
target/
!.mvn/wrapper/maven-wrapper.jar

# Node
node_modules/
dist/

# IDE
.idea/
.vscode/
*.iml

# OS
.DS_Store
Thumbs.db

# Env
.env
.env.local
```

- [ ] **Step 3: 创建 README.md**
```markdown
# 在线学生考试管理系统

基于 Vue2 + SpringBoot + MySQL 的全栈在线考试平台。

## 技术栈
- 前端: Vue2 + Element UI + Vuex + Axios
- 后端: SpringBoot 2.x + MyBatis-Plus + Spring Security + JWT
- 数据库: MySQL 8.0

## 功能
- 三种角色: 学生 / 老师 / 管理员
- 题库管理: 课程-章节-知识点三级体系
- 考试管理: 手动组卷 + 规则自动组卷
- 在线考试: 倒计时 / 切屏检测 / 题目乱序
- 成绩分析: 自动批改 + 主观题阅卷 + 统计分析

## 运行

### 后端
```bash
cd backend
./mvnw spring-boot:run   # Windows 用 mvnw.cmd spring-boot:run
```

### 前端
```bash
cd frontend
npm install
npm run serve
```
```

- [ ] **Step 4: 创建 GitHub 仓库并推送**

在浏览器打开 https://github.com/new 创建新仓库，命名为 `exam-system`，不要勾选 "Initialize this repository with a README"。

创建后执行：
```bash
git add .
git commit -m "chore: init project"
git remote add origin https://github.com/YOUR_USERNAME/exam-system.git
git branch -M main
git push -u origin main
```

### Task 1.2: 创建后端 SpringBoot 项目

**Files:**
- Create: `F:/exam-system/backend/pom.xml`
- Create: `F:/exam-system/backend/src/main/java/com/exam/ExamApplication.java`
- Create: `F:/exam-system/backend/src/main/resources/application.yml`

**知识点：Maven 项目结构、SpringBoot 起步依赖、application.yml 配置**

- [ ] **Step 1: 创建后端目录结构**
```bash
cd /f/exam-system
mkdir -p backend/src/main/java/com/exam/{config,common,security,entity,dto,mapper,service/impl,controller}
mkdir -p backend/src/main/resources/db
mkdir -p backend/src/test/java/com/exam
```

- [ ] **Step 2: 编写 pom.xml**

创建 `F:/exam-system/backend/pom.xml`：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
    </parent>

    <groupId>com.exam</groupId>
    <artifactId>exam-system</artifactId>
    <version>1.0.0</version>
    <name>exam-system</name>

    <properties>
        <java.version>17</java.version>
        <mybatis-plus.version>3.5.5</mybatis-plus.version>
        <jjwt.version>0.9.1</jjwt.version>
    </properties>

    <dependencies>
        <!-- SpringBoot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Security -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!-- MyBatis-Plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>

        <!-- MySQL 驱动 -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>${jjwt.version}</version>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.43</version>
        </dependency>

        <!-- Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**知识讲解**：
- `spring-boot-starter-web`：嵌入 Tomcat + SpringMVC，让你能写 REST 接口
- `spring-boot-starter-security`：提供认证和授权框架
- `mybatis-plus-boot-starter`：国产 ORM 增强工具，继承自 MyBatis，单表 CRUD 不用写 SQL
- `mysql-connector-j`：Java 连接 MySQL 的驱动程序
- `jjwt`：生成和解析 JWT Token 的库
- `lombok`：用注解自动生成 getter/setter/构造器，减少样板代码
- `fastjson`：阿里巴巴的 JSON 处理库，用于 JSON 字段的序列化

- [ ] **Step 3: 安装 Maven Wrapper**

在 `F:/exam-system/backend/` 目录执行：
```bash
cd /f/exam-system/backend
mvn wrapper:wrapper 2>&1 || echo "需要先下载 Maven Wrapper"
```

如果上面命令失败（因为没有 mvn），手动创建 `mvnw`：

创建 `F:/exam-system/backend/.mvn/wrapper/maven-wrapper.properties`：
```properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.6/apache-maven-3.9.6-bin.zip
wrapperUrl=https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper/3.2.0/maven-wrapper-3.2.0.jar
```

从 Spring Initializr 项目或其他方式获取 `mvnw.cmd` 和 `mvnw` 文件。

- [ ] **Step 4: 编写 SpringBoot 启动类**

创建 `F:/exam-system/backend/src/main/java/com/exam/ExamApplication.java`：
```java
package com.exam;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ExamApplication {
    public static void main(String[] args) {
        SpringApplication.run(ExamApplication.class, args);
    }
}
```

**知识讲解**：
- `@SpringBootApplication` 是一个组合注解，等于 `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
- `SpringApplication.run()` 启动内嵌 Tomcat 并扫描当前包及子包下所有 Spring 组件

- [ ] **Step 5: 编写 application.yml**

创建 `F:/exam-system/backend/src/main/resources/application.yml`：
```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/exam_system?useUnicode=true&characterEncoding=utf8mb4&serverTimezone=Asia/Shanghai&useSSL=false
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  # 打印 SQL 到控制台
    map-underscore-to-camel-case: true  # 数据库下划线 → Java 驼峰自动转换
  global-config:
    db-config:
      id-type: auto  # 主键自增
```

**知识讲解**：
- `server.port`：应用启动监听的端口号
- `spring.datasource`：数据库连接信息，`url` 中的参数指定了字符集和时区
- `map-underscore-to-camel-case: true`：数据库字段 `real_name` 自动映射到 Java 属性 `realName`，无需手动转换

- [ ] **Step 6: 验证后端能启动**

```bash
cd /f/exam-system/backend
./mvnw spring-boot:run
```

预期看到 `Started ExamApplication in X seconds`。数据库没创建没关系，先确认 Spring 能加载即可，然后 Ctrl+C 停掉。

### Task 1.3: 创建数据库和表

**知识点：MySQL 建库建表、外键约束、字符集、JSON 类型**

- [ ] **Step 1: 连接 MySQL 并创建数据库**

打开 MySQL 客户端（Navicat 或命令行）：
```sql
CREATE DATABASE IF NOT EXISTS exam_system
  DEFAULT CHARACTER SET utf8mb4
  DEFAULT COLLATE utf8mb4_general_ci;
```

- [ ] **Step 2: 创建所有表**

在 MySQL 中执行以下完整建表脚本：

```sql
USE exam_system;

-- 用户表
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE COMMENT '用户名',
    password VARCHAR(255) NOT NULL COMMENT '密码(BCrypt加密)',
    real_name VARCHAR(50) COMMENT '真实姓名',
    role VARCHAR(20) NOT NULL COMMENT '角色: STUDENT/TEACHER/ADMIN',
    phone VARCHAR(20) COMMENT '手机号',
    email VARCHAR(100) COMMENT '邮箱',
    avatar VARCHAR(255) COMMENT '头像URL',
    status TINYINT DEFAULT 0 COMMENT '状态: 0-正常 1-禁用',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'
) COMMENT '用户表';

-- 课程表
CREATE TABLE course (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    course_name VARCHAR(100) NOT NULL COMMENT '课程名称',
    description TEXT COMMENT '课程描述',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'
) COMMENT '课程表';

-- 章节表
CREATE TABLE chapter (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    course_id BIGINT NOT NULL COMMENT '所属课程',
    chapter_name VARCHAR(100) NOT NULL COMMENT '章节名称',
    sort_order INT DEFAULT 0 COMMENT '排序序号',
    CONSTRAINT fk_chapter_course FOREIGN KEY (course_id) REFERENCES course(id)
) COMMENT '章节表';

-- 知识点标签表
CREATE TABLE knowledge_point (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    course_id BIGINT NOT NULL COMMENT '所属课程',
    point_name VARCHAR(100) NOT NULL COMMENT '知识点名称',
    CONSTRAINT fk_kp_course FOREIGN KEY (course_id) REFERENCES course(id)
) COMMENT '知识点标签表';

-- 题库表
CREATE TABLE question (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    course_id BIGINT NOT NULL COMMENT '所属课程',
    chapter_id BIGINT NOT NULL COMMENT '所属章节',
    type VARCHAR(20) NOT NULL COMMENT '题型: SINGLE/MULTI/JUDGE/SUBJECTIVE',
    title TEXT NOT NULL COMMENT '题面',
    options JSON COMMENT '选项列表',
    answer TEXT COMMENT '正确答案',
    analysis TEXT COMMENT '题目解析',
    difficulty TINYINT DEFAULT 1 COMMENT '难度 1-5',
    tags JSON COMMENT '关联知识点ID列表',
    creator_id BIGINT COMMENT '创建者',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    CONSTRAINT fk_q_course FOREIGN KEY (course_id) REFERENCES course(id),
    CONSTRAINT fk_q_chapter FOREIGN KEY (chapter_id) REFERENCES chapter(id)
) COMMENT '题库表';

-- 考试表
CREATE TABLE exam (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    exam_name VARCHAR(200) NOT NULL COMMENT '考试名称',
    course_id BIGINT NOT NULL COMMENT '所属课程',
    teacher_id BIGINT NOT NULL COMMENT '创建老师',
    start_time DATETIME COMMENT '考试开始时间',
    end_time DATETIME COMMENT '考试结束时间',
    duration INT COMMENT '考试时长(分钟)',
    status VARCHAR(20) DEFAULT 'NOT_STARTED' COMMENT '状态',
    shuffle_question TINYINT DEFAULT 0 COMMENT '题目乱序: 0-否 1-是',
    shuffle_option TINYINT DEFAULT 0 COMMENT '选项乱序: 0-否 1-是',
    tab_switch_limit TINYINT DEFAULT 3 COMMENT '切屏次数限制',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    CONSTRAINT fk_exam_course FOREIGN KEY (course_id) REFERENCES course(id),
    CONSTRAINT fk_exam_teacher FOREIGN KEY (teacher_id) REFERENCES sys_user(id)
) COMMENT '考试表';

-- 考试-题目关联表
CREATE TABLE exam_question (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    exam_id BIGINT NOT NULL COMMENT '考试ID',
    question_id BIGINT NOT NULL COMMENT '题目ID',
    score INT NOT NULL COMMENT '该题分值',
    CONSTRAINT fk_eq_exam FOREIGN KEY (exam_id) REFERENCES exam(id),
    CONSTRAINT fk_eq_question FOREIGN KEY (question_id) REFERENCES question(id)
) COMMENT '考试题目关联表';

-- 学生考试记录表
CREATE TABLE exam_record (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    student_id BIGINT NOT NULL COMMENT '学生ID',
    exam_id BIGINT NOT NULL COMMENT '考试ID',
    start_time DATETIME COMMENT '开始时间',
    submit_time DATETIME COMMENT '提交时间',
    total_score INT COMMENT '总分',
    status VARCHAR(20) DEFAULT 'IN_PROGRESS' COMMENT '状态: IN_PROGRESS/FINISHED',
    tab_switch_count INT DEFAULT 0 COMMENT '切屏次数',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    CONSTRAINT fk_er_student FOREIGN KEY (student_id) REFERENCES sys_user(id),
    CONSTRAINT fk_er_exam FOREIGN KEY (exam_id) REFERENCES exam(id)
) COMMENT '考试记录表';

-- 学生答题明细表
CREATE TABLE exam_answer (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    record_id BIGINT NOT NULL COMMENT '考试记录ID',
    question_id BIGINT NOT NULL COMMENT '题目ID',
    student_answer TEXT COMMENT '学生答案',
    score INT COMMENT '该题得分',
    is_correct TINYINT COMMENT '客观题: 0-错 1-对 NULL-待批改',
    CONSTRAINT fk_ea_record FOREIGN KEY (record_id) REFERENCES exam_record(id),
    CONSTRAINT fk_ea_question FOREIGN KEY (question_id) REFERENCES question(id)
) COMMENT '答题明细表';

-- 插入默认管理员账号（密码: admin123，后面启动后会由 BCrypt 加密）
-- 先插一条明文，后续通过注册接口创建正式账号
INSERT INTO sys_user (username, password, real_name, role, status)
VALUES ('admin', '$2a$10$N.zmdr9k7uOCQb376NoUnuTJ8iAt6Z5EHsM8lE9lBOsl7iAt6Z5Eh', '管理员', 'ADMIN', 0);
```

- [ ] **Step 3: 验证表创建成功**
```sql
SHOW TABLES;
-- 预期看到 9 张表
```

---

## Phase 2: 后端公共层

### Task 2.1: 统一响应格式 Result

**知识点：统一 API 响应、链式调用（fluent API）**

**Files:**
- Create: `F:/exam-system/backend/src/main/java/com/exam/common/Result.java`

```java
package com.exam.common;

public class Result<T> {
    private int code;
    private String message;
    private T data;

    private Result() {}

    public static <T> Result<T> success(T data) {
        Result<T> r = new Result<>();
        r.code = 200;
        r.message = "success";
        r.data = data;
        return r;
    }

    public static <T> Result<T> success() {
        return success(null);
    }

    public static <T> Result<T> error(int code, String message) {
        Result<T> r = new Result<>();
        r.code = code;
        r.message = message;
        return r;
    }

    public static <T> Result<T> error(String message) {
        return error(500, message);
    }

    // getter / setter（或用 Lombok @Data）
    public int getCode() { return code; }
    public void setCode(int code) { this.code = code; }
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    public T getData() { return data; }
    public void setData(T data) { this.data = data; }
}
```

**前端收到的 JSON 示例**：
```json
{ "code": 200, "message": "success", "data": { "id": 1, "username": "admin" } }
```

### Task 2.2: 全局异常处理

**知识点：`@RestControllerAdvice`、`@ExceptionHandler`、前后端错误传递**

**Files:**
- Create: `F:/exam-system/backend/src/main/java/com/exam/common/GlobalExceptionHandler.java`

```java
package com.exam.common;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RuntimeException.class)
    public Result<?> handleRuntimeException(RuntimeException e) {
        log.error("运行时异常: {}", e.getMessage(), e);
        return Result.error(e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public Result<?> handleException(Exception e) {
        log.error("系统异常: {}", e.getMessage(), e);
        return Result.error("服务器内部错误");
    }
}
```

**知识讲解**：
- `@RestControllerAdvice`：全局拦截所有 Controller 抛出的异常
- `@ExceptionHandler(RuntimeException.class)`：只拦截 RuntimeException
- 如果不加这个，后端出错会返回 HTML 错误页面给前端，前端解析不了；加了之后始终返回 JSON

### Task 2.3: JWT 工具类

**知识点：JWT 结构（Header.Payload.Signature）、生成与解析、Token 在前后端间如何传递**

**Files:**
- Create: `F:/exam-system/backend/src/main/java/com/exam/common/JwtUtil.java`

```java
package com.exam.common;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class JwtUtil {
    private static final String SECRET = "exam-system-secret-key-2026-this-is-a-long-enough-key";
    private static final long EXPIRATION = 24 * 60 * 60 * 1000; // 24小时

    // 生成 Token
    public static String generateToken(Long userId, String username, String role) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("userId", userId);
        claims.put("username", username);
        claims.put("role", role);
        return Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION))
                .signWith(SignatureAlgorithm.HS256, SECRET)
                .compact();
    }

    // 从 Token 中解析 Claims
    public static Claims parseToken(String token) {
        return Jwts.parser()
                .setSigningKey(SECRET)
                .parseClaimsJws(token)
                .getBody();
    }

    // 判断 Token 是否过期
    public static boolean isExpired(String token) {
        try {
            return parseToken(token).getExpiration().before(new Date());
        } catch (Exception e) {
            return true;
        }
    }
}
```

**知识讲解——前后端 JWT 交互全链路**：
```
1. 用户在前端登录页面输入用户名密码
2. 前端 axios POST /api/auth/login { username, password }
3. 后端验证密码正确后，调用 JwtUtil.generateToken() 生成 Token
4. 后端返回 { code: 200, data: { token: "eyJhbGciOi..." } }
5. 前端收到 Token，存入 localStorage.setItem("token", token)
6. 之后前端每次发请求，都在请求头加 Authorization: Bearer <token>
   → axios 拦截器自动做这件事（后面 Task 10.2 实现）
7. 后端 JwtAuthenticationFilter 从请求头取出 Token，解析并验证
8. 后端拿到 Token 里的 userId 和 role，判断这个用户是谁、有什么权限
```

### Task 2.4: CORS 跨域配置

**知识点：什么是跨域（CORS）、为什么前后端分离会有跨域问题、如何解决**

**Files:**
- Create: `F:/exam-system/backend/src/main/java/com/exam/config/CorsConfig.java`

```java
package com.exam.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedOriginPattern("*");   // 允许所有来源
        config.addAllowedMethod("*");          // 允许所有 HTTP 方法
        config.addAllowedHeader("*");          // 允许所有请求头
        config.setAllowCredentials(true);      // 允许携带 Cookie

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);  // 对所有路径生效
        return new CorsFilter(source);
    }
}
```

**知识讲解——跨域问题**：
```
浏览器有一个"同源策略"：前端在 localhost:8081，后端在 localhost:8080
端口不同 = 不同源 = 浏览器默认阻止请求

后端加了这个 CORS 配置后，响应头会带：
  Access-Control-Allow-Origin: http://localhost:8081
浏览器看到这个头，就知道"后端允许 8081 访问"，放行请求
```

---

## Phase 3: 登录认证

### Task 3.1: 用户实体类

**知识点：`@TableName` 映射数据库表、`@TableId` 标记主键、Lombok `@Data` 自动生成 getter/setter**

**Files:**
- Create: `F:/exam-system/backend/src/main/java/com/exam/entity/User.java`

```java
package com.exam.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@TableName("sys_user")  // 指定对应的数据库表名
public class User {
    @TableId(type = IdType.AUTO)  // 主键自增
    private Long id;
    private String username;
    private String password;
    private String realName;      // 数据库 real_name → Java realName（自动驼峰转换）
    private String role;
    private String phone;
    private String email;
    private String avatar;
    private Integer status;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
}
```

### Task 3.2: UserMapper 接口

**知识点：MyBatis-Plus BaseMapper —— 继承后自带 CRUD 方法，无需写 SQL**

```java
package com.exam.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.exam.entity.User;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserMapper extends BaseMapper<User> {
    // 继承 BaseMapper 后自带: insert, deleteById, updateById, selectById, selectList 等
    // 如果需要复杂查询，在这里定义方法 + XML SQL
}
```

### Task 3.3: Spring Security 核心配置

**知识点：SecurityFilterChain、密码加密器 BCryptPasswordEncoder、认证管理器**

**Files:**
- Create: `F:/exam-system/backend/src/main/java/com/exam/config/SecurityConfig.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/security/UserDetailsServiceImpl.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/security/JwtAuthenticationFilter.java`

**`SecurityConfig.java`**：
```java
package com.exam.config;

import com.exam.security.JwtAuthenticationFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();  // 密码加密器
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http,
                                           JwtAuthenticationFilter jwtFilter) throws Exception {
        http
            .csrf().disable()                    // 前后端分离，关闭 CSRF
            .cors()                               // 启用 CORS
            .and()
            .sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)  // 不创建 session
            .and()
            .authorizeHttpRequests()
            .antMatchers("/api/auth/login").permitAll()     // 登录接口无需认证
            .antMatchers("/api/auth/register").permitAll()  // 注册接口无需认证
            .anyRequest().authenticated()                    // 其他接口需要登录
            .and()
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

**知识讲解——Spring Security 做了什么**：
```
1. 密码加密：BCryptPasswordEncoder 是单向哈希，相同密码每次加密结果不同
   存储时：password = encoder.encode("123456")
   验证时：encoder.matches("123456", storedPassword)

2. SessionCreationPolicy.STATELESS：不使用服务端 Session
   → 每次请求都是独立的，靠 JWT Token 识别用户

3. .antMatchers("/api/auth/login").permitAll()
   → 登录接口白名单，不需要带 Token 就能访问

4. .anyRequest().authenticated()
   → 除了白名单，所有接口都必须带有效 Token 才能访问
```

**`UserDetailsServiceImpl.java`**：
```java
package com.exam.security;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.exam.entity.User;
import com.exam.mapper.UserMapper;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UserMapper userMapper;

    public UserDetailsServiceImpl(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 从数据库查用户
        User user = userMapper.selectOne(
            new LambdaQueryWrapper<User>().eq(User::getUsername, username)
        );
        if (user == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        if (user.getStatus() == 1) {
            throw new RuntimeException("用户已被禁用");
        }
        // 转成 Spring Security 的 UserDetails
        return org.springframework.security.core.userdetails.User
                .withUsername(user.getUsername())
                .password(user.getPassword())
                .roles(user.getRole())
                .build();
    }
}
```

**`JwtAuthenticationFilter.java`**：
```java
package com.exam.security;

import com.exam.common.JwtUtil;
import io.jsonwebtoken.Claims;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Collections;

@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {
        // 1. 从请求头获取 Token
        String token = request.getHeader("Authorization");

        if (token != null && token.startsWith("Bearer ")) {
            token = token.substring(7);  // 去掉 "Bearer " 前缀
            try {
                if (!JwtUtil.isExpired(token)) {
                    // 2. 解析 Token
                    Claims claims = JwtUtil.parseToken(token);
                    String username = claims.get("username", String.class);
                    String role = claims.get("role", String.class);

                    // 3. 构建认证对象，放入 SecurityContext
                    UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                            username, null,
                            Collections.singletonList(new SimpleGrantedAuthority("ROLE_" + role))
                        );
                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            } catch (Exception e) {
                // Token 无效，不做认证
            }
        }
        chain.doFilter(request, response);  // 放行
    }
}
```

**知识讲解——JWT 过滤器执行流程**：
```
请求进来 → JwtAuthenticationFilter.doFilterInternal()
  → 从请求头取 Authorization
  → 去掉 "Bearer " 前缀
  → 解析 Token，拿到 userId、username、role
  → 创建认证对象放入 SecurityContext
  → SecurityContext 里有认证信息 → 请求被认为是"已登录"的
  → Controller 里可以通过 @PreAuthorize 或获取 SecurityContext 判断角色
```

### Task 3.4: 登录与注册接口

**知识点：POST 请求接收 JSON、`@RequestBody` 绑定参数、Service 层业务逻辑**

**Files:**
- Create: `F:/exam-system/backend/src/main/java/com/exam/dto/LoginDTO.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/dto/RegisterDTO.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/service/UserService.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/service/impl/UserServiceImpl.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/controller/AuthController.java`

**`LoginDTO.java`**：
```java
package com.exam.dto;

public class LoginDTO {
    private String username;
    private String password;
    // getter / setter
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
}
```

**`AuthController.java`**：
```java
package com.exam.controller;

import com.exam.common.JwtUtil;
import com.exam.common.Result;
import com.exam.dto.LoginDTO;
import com.exam.dto.RegisterDTO;
import com.exam.entity.User;
import com.exam.service.UserService;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final UserService userService;
    private final PasswordEncoder passwordEncoder;

    public AuthController(UserService userService, PasswordEncoder passwordEncoder) {
        this.userService = userService;
        this.passwordEncoder = passwordEncoder;
    }

    @PostMapping("/login")
    public Result<Map<String, Object>> login(@RequestBody LoginDTO loginDTO) {
        // 1. 查用户
        User user = userService.getByUsername(loginDTO.getUsername());
        if (user == null) {
            return Result.error("用户名或密码错误");
        }
        // 2. 验密码
        if (!passwordEncoder.matches(loginDTO.getPassword(), user.getPassword())) {
            return Result.error("用户名或密码错误");
        }
        // 3. 生成 Token
        String token = JwtUtil.generateToken(user.getId(), user.getUsername(), user.getRole());
        Map<String, Object> data = new HashMap<>();
        data.put("token", token);
        data.put("userId", user.getId());
        data.put("username", user.getUsername());
        data.put("realName", user.getRealName());
        data.put("role", user.getRole());
        return Result.success(data);
    }

    @PostMapping("/register")
    public Result<?> register(@RequestBody RegisterDTO dto) {
        User user = new User();
        user.setUsername(dto.getUsername());
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user.setRealName(dto.getRealName());
        user.setRole("STUDENT");  // 注册默认学生角色
        user.setStatus(0);
        userService.save(user);
        return Result.success("注册成功");
    }

    @GetMapping("/info")
    public Result<User> info() {
        // 从 SecurityContext 获取当前登录用户名
        String username = (String) org.springframework.security.core.context
                .SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        User user = userService.getByUsername(username);
        user.setPassword(null);  // 不返回密码
        return Result.success(user);
    }
}
```

**知识讲解——前后端登录传参全过程**：
```
前端：
  axios.post('/api/auth/login', { username: 'admin', password: '123456' })
  → 浏览器发送 HTTP 请求：
    POST /api/auth/login HTTP/1.1
    Content-Type: application/json
    Body: {"username":"admin","password":"123456"}

后端：
  @RequestBody LoginDTO loginDTO
  → Spring 自动把 JSON body 转成 LoginDTO 对象
  → loginDTO.getUsername() = "admin"
  → loginDTO.getPassword() = "123456"
  → 返回 Result<Map> → Spring 自动转 JSON 响应

前端收到：
  { code: 200, message: "success", data: { token: "eyJ...", role: "ADMIN" } }
  → localStorage.setItem("token", token)
  → router.push("/")  跳转到主页
```

---

## Phase 4: 用户管理 CRUD

### Task 4.1: 用户 CRUD 全栈（Controller → Service → Mapper）

**知识点：RESTful API 设计、分页查询、`@PathVariable` 取路径参数、`@RequestParam` 取查询参数**

**Files:**
- Create: `F:/exam-system/backend/src/main/java/com/exam/service/UserService.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/service/impl/UserServiceImpl.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/controller/UserController.java`

**`UserService.java`**：
```java
package com.exam.service;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.IService;
import com.exam.entity.User;

public interface UserService extends IService<User> {
    User getByUsername(String username);
    Page<User> pageUsers(int page, int size, String keyword, String role);
}
```

**知识讲解——MyBatis-Plus Service 层**：
```
IService<User> 是 MyBatis-Plus 提供的通用 Service 接口
继承后自带 save(), removeById(), updateById(), getById(), list() 等方法
ServiceImpl<UserMapper, User> 是通用实现类
```

**`UserController.java`**：
```java
package com.exam.controller;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.exam.common.Result;
import com.exam.entity.User;
import com.exam.service.UserService;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;
    private final PasswordEncoder passwordEncoder;

    public UserController(UserService userService, PasswordEncoder passwordEncoder) {
        this.userService = userService;
        this.passwordEncoder = passwordEncoder;
    }

    // GET /api/users?page=1&size=10&keyword=&role=
    @GetMapping
    @PreAuthorize("hasRole('ADMIN')")
    public Result<Page<User>> list(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(required = false) String keyword,
            @RequestParam(required = false) String role) {
        return Result.success(userService.pageUsers(page, size, keyword, role));
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public Result<?> create(@RequestBody User user) {
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        userService.save(user);
        return Result.success("创建成功");
    }

    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public Result<?> update(@PathVariable Long id, @RequestBody User user) {
        user.setId(id);
        if (user.getPassword() != null && !user.getPassword().isEmpty()) {
            user.setPassword(passwordEncoder.encode(user.getPassword()));
        } else {
            user.setPassword(null);  // 不修改密码
        }
        userService.updateById(user);
        return Result.success("更新成功");
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public Result<?> delete(@PathVariable Long id) {
        userService.removeById(id);
        return Result.success("删除成功");
    }
}
```

**知识讲解——前后端传参的四种方式**：

| 方式 | 后端 | 前端写法 | 适用场景 |
|------|------|----------|----------|
| **路径参数** | `@PathVariable` | `GET /api/users/5` | 查单个资源 |
| **查询参数** | `@RequestParam` | `GET /api/users?page=1&size=10` | 列表筛选/分页 |
| **JSON Body** | `@RequestBody` | `POST /api/users { name: "张三" }` | 新增/修改 |
| **路径+Body** | `@PathVariable` + `@RequestBody` | `PUT /api/users/5 { name: "李四" }` | 修改指定资源 |

---

## Phase 5: 课程/章节/知识点 CRUD

### Task 5.1: 课程管理（完整 CRUD）

**Files:**
- Create: `F:/exam-system/backend/src/main/java/com/exam/entity/Course.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/mapper/CourseMapper.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/service/CourseService.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/service/impl/CourseServiceImpl.java`
- Create: `F:/exam-system/backend/src/main/java/com/exam/controller/CourseController.java`

结构与用户模块类似（entity → mapper → service → controller），这里展示 Controller 体现 RESTful 风格：

```java
@RestController
@RequestMapping("/api/courses")
public class CourseController {
    private final CourseService courseService;

    @GetMapping
    public Result<List<Course>> list() {
        return Result.success(courseService.list());
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public Result<?> create(@RequestBody Course course) { ... }

    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public Result<?> update(@PathVariable Long id, @RequestBody Course course) { ... }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public Result<?> delete(@PathVariable Long id) { ... }
}
```

### Task 5.2: 章节管理（带课程关联）

**关键差异**：章节需要按课程查询，所以 list 方法接受 `courseId` 查询参数。

```java
@RestController
@RequestMapping("/api/chapters")
public class ChapterController {

    @GetMapping
    public Result<List<Chapter>> list(@RequestParam Long courseId) {
        List<Chapter> list = chapterService.list(
            new LambdaQueryWrapper<Chapter>()
                .eq(Chapter::getCourseId, courseId)
                .orderByAsc(Chapter::getSortOrder)
        );
        return Result.success(list);
    }
}
```

### Task 5.3: 知识点管理

与章节类似，按 `courseId` 筛选。

---

## Phase 6: 题库管理

### Task 6.1: 题目 CRUD

**关键点**：题目中的 `options` 和 `tags` 是 JSON 字段，需要 MyBatis-Plus 的类型处理器。

**`Question.java`** 关键部分：
```java
@Data
@TableName(value = "question", autoResultMap = true)  // autoResultMap 用于 JSON 映射
public class Question {
    @TableId(type = IdType.AUTO)
    private Long id;
    private Long courseId;
    private Long chapterId;
    private String type;
    private String title;

    @TableField(typeHandler = JacksonTypeHandler.class)
    private List<String> options;  // ["A.选项1", "B.选项2", "C.选项3", "D.选项4"]

    private String answer;
    private String analysis;
    private Integer difficulty;

    @TableField(typeHandler = JacksonTypeHandler.class)
    private List<Long> tags;  // [1, 3, 5] 知识点 ID 列表

    private Long creatorId;
    private LocalDateTime createTime;
}
```

**知识讲解——JSON 字段处理**：
```
MySQL 的 JSON 类型列可以存储复杂结构
Java 侧用 List<String> 配合 JacksonTypeHandler
MyBatis-Plus 自动完成：List ↔ JSON 字符串 的转换

存入时：["A.选项1", "B.选项2"] → MySQL JSON 列
读取时：MySQL JSON 列 → List<String> Java 对象
```

**Controller 多条件查询**：
```java
@GetMapping
public Result<Page<Question>> list(
        @RequestParam(defaultValue = "1") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(required = false) Long courseId,
        @RequestParam(required = false) Long chapterId,
        @RequestParam(required = false) String type,
        @RequestParam(required = false) Integer difficulty) {
    LambdaQueryWrapper<Question> wrapper = new LambdaQueryWrapper<>();
    if (courseId != null) wrapper.eq(Question::getCourseId, courseId);
    if (chapterId != null) wrapper.eq(Question::getChapterId, chapterId);
    if (type != null) wrapper.eq(Question::getType, type);
    if (difficulty != null) wrapper.eq(Question::getDifficulty, difficulty);
    wrapper.orderByDesc(Question::getCreateTime);
    return Result.success(questionService.page(new Page<>(page, size), wrapper));
}
```

---

## Phase 7: 考试管理

### Task 7.1: 考试 CRUD

标准 CRUD，考点是创建考试时 `teacher_id` 从 JWT 中自动获取。

### Task 7.2: 组卷（手动选题 + 规则组卷）

**这个模块是项目核心技术点之一。**

**请求 DTO**：
```java
// 手动组卷请求
public class ManualSelectDTO {
    private Long examId;
    private List<QuestionScore> questions;
    // QuestionScore: { questionId: 1, score: 5 }
}

// 自动规则组卷请求
public class RuleSelectDTO {
    private Long examId;
    private List<Rule> rules;
    // Rule: { chapterId: 2, type: "SINGLE", difficulty: 3, count: 10, score: 3 }
}
```

**规则组卷算法**（Service 实现）：
```java
public void selectByRules(RuleSelectDTO dto) {
    for (Rule rule : dto.getRules()) {
        // 按条件随机查询题目
        List<Question> pool = questionService.list(new LambdaQueryWrapper<Question>()
            .eq(rule.getChapterId() != null, Question::getChapterId, rule.getChapterId())
            .eq(rule.getType() != null, Question::getType, rule.getType())
            .eq(rule.getDifficulty() != null, Question::getDifficulty, rule.getDifficulty())
        );
        // 随机抽取指定数量
        Collections.shuffle(pool);
        List<Question> selected = pool.subList(0, Math.min(rule.getCount(), pool.size()));
        // 插入 exam_question 表
        for (Question q : selected) {
            ExamQuestion eq = new ExamQuestion();
            eq.setExamId(dto.getExamId());
            eq.setQuestionId(q.getId());
            eq.setScore(rule.getScore());
            examQuestionService.save(eq);
        }
    }
}
```

---

## Phase 8: 学生考试

### Task 8.1: 开始考试（返回乱序题目 + 创建答题记录）

```
POST /api/exams/{id}/start
→ 后端：
  1. 验证考试状态和时间窗口
  2. 查 exam_question 获取本次考试所有题目
  3. 如果 exam.shuffle_question=true → Collections.shuffle(questions)
  4. 如果 exam.shuffle_option=true → 每题内部随机排列 options
  5. 创建 exam_record（开始计时）
  6. 创建 exam_answer 明细（每题一条，答案为空）
  7. 返回题目列表（但不含正确答案！）
```

### Task 8.2: 提交试卷 + 自动批改

```
POST /api/exams/{id}/submit
→ 后端：
  1. 更新 exam_record.submit_time、status=FINISHED
  2. 遍历学生答案：
     - 客观题（SINGLE/MULTI/JUDGE）：对比正确答案，自动判对错
     - 简答题（SUBJECTIVE）：标记 is_correct=NULL 等待批改
  3. 计算客观题总分，更新 exam_record.total_score
```

### Task 8.3: 切屏上报

```
POST /api/exams/{id}/tab-switch
→ 后端：exam_record.tab_switch_count += 1
  如果超过 tab_switch_limit → 返回警告或自动交卷
```

---

## Phase 9: 阅卷、成绩统计、错题本

### Task 9.1: 主观题批改

老师查看学生的 exam_answer 中 is_correct=NULL 的记录，逐题打分。

### Task 9.2: 成绩统计

```java
@GetMapping("/api/exams/{id}/stats")
public Result<Map<String, Object>> stats(@PathVariable Long id) {
    List<ExamRecord> records = examRecordService.list(
        new LambdaQueryWrapper<ExamRecord>()
            .eq(ExamRecord::getExamId, id)
            .eq(ExamRecord::getStatus, "FINISHED")
    );
    // 计算：最高分、最低分、平均分、及格率、分数段分布
    // 返回统计数据
}
```

### Task 9.3: 错题本

```java
@GetMapping("/api/error-book")
public Result<List<Question>> errorBook() {
    // 获取当前学生 ID
    Long studentId = getCurrentUserId();
    // 查询所有 exam_answer 中 is_correct=0 的记录
    // JOIN question 表返回题目详情
}
```

---

## Phase 10: 前端项目

### Task 10.1: Vue 项目初始化

**知识点：Vue CLI 创建项目、Element UI 安装、项目结构**

```bash
cd /f/exam-system
npx @vue/cli create frontend
# 选择: Vue 2, Babel, Router, Vuex, CSS Pre-processors
# 或手动创建（更适合学习）:
```

如果不用 Vue CLI，手动创建 `frontend/package.json`：
```json
{
  "name": "exam-system-frontend",
  "version": "1.0.0",
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build"
  },
  "dependencies": {
    "vue": "^2.7.16",
    "vue-router": "^3.6.5",
    "vuex": "^3.6.2",
    "axios": "^1.7.2",
    "element-ui": "^2.15.14"
  },
  "devDependencies": {
    "@vue/cli-service": "^5.0.8",
    "vue-template-compiler": "^2.7.16"
  }
}
```

### Task 10.2: Axios 封装 + 请求拦截器

**这个模块极其重要——涉及前后端交互的核心：Token 怎么带、401 怎么处理、跨域怎么配**

**`src/utils/request.js`**：
```javascript
import axios from 'axios'
import { Message } from 'element-ui'
import router from '../router'

const request = axios.create({
  baseURL: 'http://localhost:8080',  // 后端地址
  timeout: 10000
})

// 请求拦截器：每次请求自动带 Token
request.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers['Authorization'] = 'Bearer ' + token
    }
    return config
  },
  error => Promise.reject(error)
)

// 响应拦截器：统一处理错误
request.interceptors.response.use(
  response => {
    const res = response.data
    if (res.code !== 200) {
      Message.error(res.message || '请求失败')
      return Promise.reject(new Error(res.message))
    }
    return res  // 直接返回 data，调用方用 res.data 取数据
  },
  error => {
    if (error.response && error.response.status === 401) {
      localStorage.removeItem('token')
      router.push('/login')
      Message.error('登录已过期，请重新登录')
    } else {
      Message.error('网络错误')
    }
    return Promise.reject(error)
  }
)

export default request
```

**知识讲解——Axios 拦截器做了什么**：
```
请求拦截器（发请求前）：
  → 从 localStorage 取 token
  → 加到请求头 Authorization: Bearer <token>
  → 后端 JwtAuthenticationFilter 拿到这个头，解析出用户身份

响应拦截器（收到响应后）：
  → 如果 code !== 200 → 弹出错误提示
  → 如果 HTTP 状态码 401 → token 过期 → 清空本地 token → 跳转登录页
```

### Task 10.3: Vue 路由（含角色权限守卫）

**`src/router/index.js`**：
```javascript
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

const router = new Router({
  mode: 'history',  // 不带 # 的 URL（需要后端配合，否则刷新404）
  routes: [
    {
      path: '/login',
      name: 'Login',
      component: () => import('@/views/Login.vue')
    },
    {
      path: '/admin',
      component: () => import('@/views/Layout.vue'),
      meta: { role: 'ADMIN' },
      children: [
        { path: 'users', name: 'Users', component: () => import('@/views/admin/Users.vue') }
      ]
    },
    {
      path: '/teacher',
      component: () => import('@/views/Layout.vue'),
      meta: { role: 'TEACHER' },
      children: [
        { path: 'exams', name: 'ExamList', component: () => import('@/views/teacher/ExamList.vue') },
        { path: 'exams/create', name: 'ExamCreate', component: () => import('@/views/teacher/ExamCreate.vue') },
        { path: 'exams/:id/questions', name: 'ExamQuestions', component: () => import('@/views/teacher/ExamQuestions.vue') },
        { path: 'exams/:id/grading', name: 'ExamGrading', component: () => import('@/views/teacher/Grading.vue') },
        { path: 'exams/:id/stats', name: 'ExamStats', component: () => import('@/views/teacher/Stats.vue') },
        { path: 'questions', name: 'Questions', component: () => import('@/views/teacher/QuestionManage.vue') }
      ]
    },
    {
      path: '/student',
      component: () => import('@/views/Layout.vue'),
      meta: { role: 'STUDENT' },
      children: [
        { path: 'exams', name: 'StudentExams', component: () => import('@/views/student/ExamList.vue') },
        { path: 'exam/:id', name: 'ExamTake', component: () => import('@/views/student/ExamTake.vue') },
        { path: 'scores', name: 'Scores', component: () => import('@/views/student/Scores.vue') },
        { path: 'error-book', name: 'ErrorBook', component: () => import('@/views/student/ErrorBook.vue') }
      ]
    },
    { path: '/', redirect: '/login' }
  ]
})

// 路由守卫：检查角色权限
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  const role = localStorage.getItem('role')
  if (to.path === '/login') {
    next()
  } else if (!token) {
    next('/login')
  } else if (to.meta.role && to.meta.role !== role) {
    next('/login')  // 角色不匹配，跳登录
    // 实际应该提示"无权限访问"
  } else {
    next()
  }
})

export default router
```

### Task 10.4: Vuex 状态管理

**`src/store/index.js`**：
```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    user: null  // 当前登录用户信息
  },
  mutations: {
    SET_USER(state, user) {
      state.user = user
    },
    CLEAR_USER(state) {
      state.user = null
    }
  },
  actions: {
    login({ commit }, userInfo) {
      commit('SET_USER', userInfo)
      localStorage.setItem('role', userInfo.role)
    },
    logout({ commit }) {
      commit('CLEAR_USER')
      localStorage.removeItem('token')
      localStorage.removeItem('role')
    }
  }
})
```

### Task 10.5: 登录页面

```vue
<template>
  <div class="login-container">
    <el-card class="login-card">
      <h2>考试管理系统</h2>
      <el-form :model="form" :rules="rules" ref="form">
        <el-form-item prop="username">
          <el-input v-model="form.username" placeholder="用户名" />
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="form.password" type="password" placeholder="密码" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="login" :loading="loading">登录</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import { login } from '@/api/auth'

export default {
  data() {
    return {
      form: { username: '', password: '' },
      loading: false,
      rules: {
        username: [{ required: true, message: '请输入用户名' }],
        password: [{ required: true, message: '请输入密码' }]
      }
    }
  },
  methods: {
    async login() {
      this.$refs.form.validate(async valid => {
        if (!valid) return
        this.loading = true
        try {
          const res = await login(this.form)
          localStorage.setItem('token', res.data.token)
          localStorage.setItem('role', res.data.role)
          // 按角色跳转
          if (res.data.role === 'ADMIN') this.$router.push('/admin/users')
          else if (res.data.role === 'TEACHER') this.$router.push('/teacher/exams')
          else this.$router.push('/student/exams')
        } finally {
          this.loading = false
        }
      })
    }
  }
}
</script>
```

### Task 10.6 - 10.15: 各业务页面

按顺序开发以下页面，每页遵循相同模式：API 调用 → 数据渲染 → 表单操作：

1. **管理员**：用户管理页、课程管理页
2. **老师**：题库管理页、考试列表页、创建考试页、组卷页、阅卷页、成绩统计页
3. **学生**：考试列表页、考试答题页（核心：倒计时+切屏检测+题目乱序）、成绩列表页、错题本页

每个页面都遵循 Vue2 标准模式：
```vue
<template>
  <!-- Element UI 组件：el-table, el-form, el-dialog, el-pagination 等 -->
</template>
<script>
import { xxxApi } from '@/api/xxx'
export default {
  data() { return { ... } },
  created() { this.fetchData() },
  methods: {
    async fetchData() { const res = await xxxApi.list(params); this.data = res.data }
  }
}
</script>
```

---

## Phase 11: 联调与收尾

### Task 11.1: 前后端联调

1. 启动后端：`cd backend && ./mvnw spring-boot:run`
2. 启动前端：`cd frontend && npm run serve`
3. 打开浏览器 `http://localhost:8081`
4. 用 admin / 123456 登录（需要先在数据库插入初始数据）
5. 逐模块走通完整流程

### Task 11.2: 初始化数据脚本

创建 SQL 脚本插入测试数据：课程、章节、题库、考试等，方便联调和演示。

---

以上是完整实施计划。每个 Task 都包含具体代码。
