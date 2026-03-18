# 疫苗预约与接种管理系统

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-17-orange.svg)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-blue.svg)](https://www.mysql.com/)

## 项目简介

本项目为儿童疫苗预约与接种管理系统，采用前后端分离架构，实现家长端、医生端、管理员端三端业务闭环。

系统围绕"预约 → 审核 → 接种 → 记录 → 统计"完整业务流程设计。

## 核心特性

- **前后端分离架构**：RESTful API 设计，支持多端适配与独立部署
- **三端权限模型**：基于 RBAC 的权限控制，区分管理员、医生、居民角色
- **库存联动设计**：FEFO（先过期先出）批次管理，预约锁定与核销扣减联动
- **业务流程闭环**：从预约、签到、预检、接种到留观的完整状态流转
- **可视化决策支持**：多维统计大屏、接种趋势分析、库存预警监控
- **数据一致性保障**：乐观锁并发控制、数据库事务、逻辑删除机制

## 技术栈

### 后端技术栈

| 技术 | 版本 | 说明 |
|------|------|------|
| Java | 17 | 开发语言 |
| Spring Boot | 3.2 | 应用框架 |
| MyBatis-Plus | 3.5+ | ORM 框架 |
| Spring Security | 6.x | 安全认证 |
| MySQL | 8.0 | 关系型数据库 |
| Druid | 1.2+ | 数据库连接池 |
| Redis | 5.x | 缓存层（Spring Cache 集成） |
| Knife4j | 4.x | API 文档（OpenAPI 3） |

### 后端增强特性

- **密码安全**：BCrypt 加密存储，登录时验证
- **服务层日志**：AOP 切面自动记录方法调用与耗时
- **单元测试**：H2 内存数据库，Service 层单元测试覆盖率

### 前端技术栈

| 技术 | 说明 |
|------|------|
| UniApp | 跨端应用框架 |
| Vue 2/3 | 前端组件化开发 |
| uni-ui | 组件库 |
| Vuex | 状态管理 |
| u-charts | 数据可视化图表 |

## 系统架构

```
┌─────────────────────────────────────────────────────────────┐
│                        客户端层                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │ 管理后台  │  │ 医生端    │  │ 居民端    │  （UniApp）     │
│  │ (H5/PC)  │  │  (小程序) │  │  (小程序) │                  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                  │
└───────┼─────────────┼─────────────┼──────────────────────────┘
        │             │             │
        └─────────────┴─────────────┘
                      │ HTTP/RESTful API
┌─────────────────────┼───────────────────────────────────────┐
│                     ▼                                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   控制层 (Controller)                 │   │
│  │  /admin/**  │  /doctor/**  │  /user/**  │  /common/** │   │
│  └─────────────────────────────────────────────────────┘   │
│                     │                                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   业务层 (Service)                    │   │
│  │  用户管理 │ 疫苗管理 │ 预约管理 │ 库存管理 │ 统计报表  │   │
│  │          │       AOP 服务层日志                        │   │
│  └─────────────────────────────────────────────────────┘   │
│                     │                                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   数据层 (Mapper/DAO)                 │   │
│  │           MyBatis-Plus 数据访问与业务逻辑             │   │
│  └─────────────────────────────────────────────────────┘   │
│                     │                                       │
│         ┌───────────┴───────────┐                          │
│         ▼                       ▼                          │
│  ┌──────────────┐      ┌──────────────┐                   │
│  │   MySQL 8.0  │      │    Redis     │                   │
│  │  26 张业务表  │      │   缓存层     │                   │
│  │ 逻辑删除/乐观锁│      │  Spring Cache│                   │
│  └──────────────┘      └──────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

## 功能模块

### 管理员端

| 模块 | 功能说明 |
|------|----------|
| 用户管理 | 用户注册审核、角色分配、状态管理（启用/禁用/注销）、密码重置 |
| 疫苗管理 | 疫苗信息维护、剂次配置、上下架控制、接种说明管理 |
| 接种点管理 | 接种点基础信息管理、驻场医生指派、启用/禁用控制 |
| 库存管理 | 总仓批次入库、接种点调拨、FEFO 批次分配、库存预警 |
| 预约排期 | 预约列表查询、排班管理、医生调度 |
| 公告管理 | 公告发布与审核、医生提交公告审批、置顶与下架控制 |
| 统计报表 | 今日接种统计、近 7 天趋势、月度分析、多维热力图、库存预警看板 |
| 智能提醒 | 接种前 72 小时自动提醒、宣传推送管理 |

### 医生端

| 模块 | 功能说明 |
|------|----------|
| 个人档案 | 医生基础信息、历史接种记录统计 |
| 预约处理 | 今日待签到列表、预约确认签到、预检评估（通过/未通过） |
| 接种核销 | 接种完成确认、批次号自动关联、接种记录生成 |
| 留观管理 | 留观中儿童列表、剩余时长监控、延长/终止留观 |
| 公告提交 | 公告拟稿与提交、审核状态跟踪、撤回与重新提交 |
| 调遣通知 | 驻场调遣通知接收、已读状态管理 |

### 居民端

| 模块 | 功能说明 |
|------|----------|
| 接种点查询 | 附近接种点列表、疫苗库存查询、驻场医生信息 |
| 预约接种 | 疫苗选择、医生排班查看、时段预约、智能冲突检查 |
| 儿童档案 | 儿童基础信息管理、禁忌症记录、接种卡号绑定 |
| 接种记录 | 历史接种记录查询、下次接种提醒、不良反应上报 |
| 公告浏览 | 最新公告列表、置顶公告展示 |

## 核心业务流程

### 预约接种流程

```
居民选择疫苗 → 选择接种点 → 查看医生排班 → 选择时段预约
                                              ↓
                              接种完成 ← 留观结束确认 ← 医生核销接种 ← 签到确认 ← 预约成功
```

### 库存管理流程

```
总仓批次入库 → 管理员调拨至接种点 → 预约时 FEFO 锁定批次 → 核销时扣减库存
                                      ↓
                             库存预警监控（阈值/效期）
```

## 数据库设计

系统包含 22 张核心数据表，采用逻辑删除与乐观锁机制保障数据完整性。

### 核心实体关系

```
sys_user（系统用户）
  ├── child_profile（儿童档案）1:N
  ├── appointment（预约单）1:N
  ├── vaccination_record（接种记录）1:N
  └── notice（公告）1:N

vaccination_site（接种点）
  ├── site_vaccine_stock（接种点批次库存）1:N
  ├── doctor_schedule（医生排班）1:N
  └── appointment（预约）1:N

vaccine（疫苗信息）
  ├── vaccine_site_stock（按点库存）1:N
  ├── appointment（预约）1:N
  └── vaccination_record（接种记录）1:N

appointment（预约单）
  └── vaccination_record（接种记录）0:1
```

### 关键表结构

| 表名 | 说明 | 核心字段 |
|------|------|----------|
| `sys_user` | 系统用户表 | id, username, role, status, last_login_time |
| `child_profile` | 儿童档案表 | id, parent_id, name, birth_date, contraindication_allergy |
| `vaccination_site` | 接种点表 | id, site_name, status, current_doctor_id |
| `vaccine` | 疫苗信息表 | id, vaccine_name, total_doses, interval_days, status |
| `site_vaccine_stock` | 接种点批次库存 | id, site_id, batch_id, available_stock, locked_stock |
| `appointment` | 预约单表 | id, user_id, child_id, vaccine_id, doctor_schedule_id |
| `vaccination_record` | 接种记录表 | id, appointment_id, batch_id, vaccine_code, observation_ok |
| `doctor_schedule` | 医生排班表 | id, doctor_id, site_id, schedule_date, time_slot, max_capacity |

## 项目结构

```
Pinenuts/
├── vaccine.app/                    # 前端 UniApp 项目
│   ├── pages/                      # 页面目录
│   │   ├── admin/                  # 管理员端页面
│   │   ├── doctor/                 # 医生端页面
│   │   └── user/                   # 居民端页面
│   ├── components/                 # 公共组件
│   ├── static/                     # 静态资源
│   ├── store/                      # Vuex 状态管理
│   └── utils/                      # 工具函数
│
└── vaccine_system/                 # 后端 Spring Boot 项目
    ├── src/main/java/com/vaccine/
    │   ├── controller/             # 控制层
    │   │   ├── admin/              # 管理员接口
    │   │   ├── doctor/             # 医生接口
    │   │   └── user/               # 用户接口
    │   ├── service/                # 业务层
    │   ├── mapper/                 # 数据访问层
    │   ├── entity/                 # 实体类
    │   ├── dto/                    # 数据传输对象
    │   ├── vo/                     # 视图对象
    │   ├── config/                 # 配置类
    │   └── utils/                  # 工具类
    ├── src/main/resources/
    │   ├── sql/                    # 数据库脚本
    │   └── application.properties  # 应用配置
    └── docs/                       # 项目文档
        ├── API.md                  # 接口文档
        └── database.md             # 数据库设计文档
```

## 快速开始

### 环境要求

- JDK 17+
- MySQL 8.0+
- Maven 3.8+
- HBuilderX / VS Code（前端）

### 后端部署

1. 创建数据库并导入初始化脚本

```bash
mysql -u root -p < vaccine_system/src/main/resources/sql/vaccine_system.sql
```

2. 修改数据库配置

```properties
# vaccine_system/src/main/resources/application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/vaccine_system?useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=your_password
```

3. 编译并启动应用

```bash
cd vaccine_system
mvn clean package
java -jar target/vaccine-system-*.jar
```

应用默认启动于 `http://localhost:8080`，API 文档地址：`http://localhost:8080/doc.html`

### 前端部署

1. 使用 HBuilderX 打开 `vaccine.app` 目录
2. 配置后端 API 地址（`manifest.json` → `h5` → `router` → `base`）
3. 选择运行目标（H5 / 微信小程序 / App）

## 接口文档

系统提供完整的 RESTful API 接口，按角色划分为以下模块：

| 模块前缀 | 说明 |
|----------|------|
| `/common/**` | 公共接口（登录、注册等） |
| `/admin/**` | 管理员接口 |
| `/doctor/**` | 医生接口 |
| `/user/**` | 居民接口 |

详细接口定义参见：[API 文档](./vaccine_system/docs/API.md)

## 系统亮点

### 1. 技术架构增强

- **Redis 缓存层**：热点数据（用户信息、疫苗列表、接种点）缓存，加速查询响应
- **BCrypt 密码加密**：用户密码加密存储，安全性符合生产标准
- **AOP 服务日志**：自动记录方法调用、参数、耗时，便于问题追踪
- **单元测试覆盖**：Service 层核心业务单元测试，保证代码质量

### 2. 精细化库存管理

- **双轨库存设计**：总仓批次库存与接种点可用/锁定库存分离，支持精细化追溯
- **FEFO 批次分配**：预约时自动按先过期先出原则分配批次，避免疫苗浪费
- **库存预警机制**：支持低库存预警、效期预警，自动同步供应商

### 3. 排班先行预约模式

- 居民直接选择医生排班进行预约，一个排班时段仅支持一个预约（capacity=1）
- 避免传统"先约时间后分医生"模式下的资源冲突问题

### 4. 完整接种闭环

```
已预约(1) → 已签到(6) → 预检通过(9) → 留观中(10) → 已完成(2)
                ↓           ↓
            预检未通过(9)  异常终止
```

### 5. 多维度统计分析

- 今日接种实时统计
- 近 7 天接种趋势折线图
- 月度接种趋势对比
- 多维数据热力图（时段 × 疫苗 × 接种点）
- 库存预警可视化看板

## 项目统计

| 维度 | 数据 |
|------|------|
| 前端代码量 | 约 6,852 行 |
| 前端页面数 | 119 个 Vue 文件 |
| 后端代码量 | 约 10,097 行 |
| 数据库表数 | 26 张（含扩展表） |
| API 接口数 | 100+ |
| 单元测试 | Service 层覆盖率 |

## 贡献指南

欢迎提交 Issue 和 Pull Request。

## 许可证

本项目基于 [MIT License](LICENSE) 开源。

## 文档索引

- [API 接口文档](./vaccine_system/docs/API.md)
- [数据库设计文档](./vaccine_system/docs/database.md)
- [项目使用手册](./vaccine_system/docs/uniapp项目使用文档.md)