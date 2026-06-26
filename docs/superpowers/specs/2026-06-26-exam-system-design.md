# 在线学生考试管理系统 — 设计文档

> 版本: v1.0 | 日期: 2026-06-26

## 1. 技术栈

| 层 | 技术 |
|-----|------|
| 前端 | Vue2 + Vue Router + Vuex + Element UI + Axios |
| 后端 | SpringBoot 2.x + MyBatis-Plus + Spring Security + JWT |
| 数据库 | MySQL 8.0 |

## 2. 用户角色

| 角色 | 职责 |
|------|------|
| 管理员 | 管理用户（增删改查）、管理课程 |
| 老师 | 管理题库、创建考试、组卷、阅卷、查看成绩统计 |
| 学生 | 参加考试、查看成绩、错题本 |

## 3. 数据库设计

### sys_user — 用户表
id, username, password(BCrypt), real_name, role(STUDENT/TEACHER/ADMIN), phone, email, avatar, status, create_time

### course — 课程表
id, course_name, description, create_time

### chapter — 章节表
id, course_id(FK), chapter_name, sort_order

### knowledge_point — 知识点标签表
id, course_id(FK), point_name

### question — 题库表
id, course_id(FK), chapter_id(FK), type(SINGLE/MULTI/JUDGE/SUBJECTIVE), title, options(JSON), answer, analysis, difficulty, tags(JSON), creator_id(FK), create_time

### exam — 考试表
id, exam_name, course_id(FK), teacher_id(FK), start_time, end_time, duration, status, shuffle_question, shuffle_option, tab_switch_limit, create_time

### exam_question — 考试-题目关联表
id, exam_id(FK), question_id(FK), score

### exam_record — 学生考试记录表
id, student_id(FK), exam_id(FK), start_time, submit_time, total_score, status(IN_PROGRESS/FINISHED), tab_switch_count, create_time

### exam_answer — 学生答题明细表
id, record_id(FK), question_id(FK), student_answer, score, is_correct

## 4. API 设计

### 认证
- `POST /api/auth/login` — 登录，返回 JWT
- `GET /api/auth/info` — 获取当前用户信息

### 用户管理（管理员）
- `GET/POST/PUT/DELETE /api/users`

### 课程（管理员）
- `GET/POST/PUT/DELETE /api/courses`

### 章节
- `GET/POST/PUT/DELETE /api/chapters?courseId={id}`

### 题库（老师）
- `GET/POST/PUT/DELETE /api/questions` — 支持按课程、章节、知识点、题型筛选

### 考试管理（老师）
- `GET/POST/PUT/DELETE /api/exams`
- `POST /api/exams/{id}/questions` — 组卷（手动选题 + 规则组卷）
- `GET /api/exams/{id}/records` — 查看学生成绩列表
- `GET /api/exams/{id}/stats` — 成绩统计
- `POST /api/exams/{id}/grade` — 批改简答题

### 学生考试
- `GET /api/exams/mine` — 我的考试列表
- `POST /api/exams/{id}/start` — 开始考试
- `POST /api/exams/{id}/submit` — 提交试卷
- `POST /api/exams/{id}/tab-switch` — 上报切屏
- `GET /api/scores/mine` — 我的成绩列表
- `GET /api/scores/{id}` — 查看单次考试详情
- `GET /api/error-book` — 错题本

## 5. 功能要点

- 题型：单选 + 多选 + 判断 + 简答
- 组卷：手动选题 + 规则自动组卷（按章节/知识点/难度随机抽题）
- 防作弊：倒计时自动交卷 + 切屏检测 + 题目乱序 + 选项随机
- 批改：客观题自动批改，主观题人工批改
- 成绩：统计分析（平均分、分布、及格率）+ 错题本

## 6. 开发顺序

1. 后端项目初始化 + 数据库建表
2. 登录认证（Spring Security + JWT）
3. 用户管理 CRUD
4. 课程/章节/知识点 CRUD
5. 题库 CRUD
6. 考试管理（创建 + 手动组卷 + 规则组卷）
7. 学生考试（开始 + 防作弊 + 提交 + 自动批改）
8. 阅卷（简答题人工批改）
9. 成绩统计分析
10. 错题本
11. 前端项目初始化 + 登录页 + 路由
12. 前端各页面逐模块对接
13. 前后端联调
