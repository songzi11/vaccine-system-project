# 数据库与后端实体不一致修复计划

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 修复数据库设计文档与后端代码的不一致问题，实现缺失的 user_notice_read 功能，并同步文档

**Architecture:**
- 新增 UserNoticeRead 实体类、Mapper、Service、Controller，实现公告已读状态跟踪
- 更新 database.md 文档以反映 DoctorDispatch 简化后的状态枚举
- 参考现有 NoticeFeedback 的实现模式（结构最相似）

**Tech Stack:** Spring Boot 3.2, MyBatis-Plus, Java 17

---

## 背景分析

### 问题1: user_notice_read 表无后端实现
- 数据库表存在且有数据
- 缺少实体类、Mapper、Service、Controller
- 影响：无法实现公告已读/未读状态跟踪

### 问题2: DoctorDispatch 状态枚举与文档不一致
- 文档定义：0-待审批，1-同意，2-拒绝
- 实际实现：0-已推送，1-已读（已简化审批流程）
- 需要同步更新文档

---

## 任务清单

### Task 1: 创建 UserNoticeRead 实体类

**Files:**
- Create: `vaccine_system/src/main/java/com/tjut/edu/vaccine_system/model/entity/UserNoticeRead.java`

- [ ] **Step 1: 创建实体类**

```java
package com.tjut.edu.vaccine_system.model.entity;

import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.time.LocalDateTime;

/**
 * 用户公告已读表
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@TableName("user_notice_read")
public class UserNoticeRead implements Serializable {

    private static final long serialVersionUID = 1L;

    @TableId(type = IdType.AUTO)
    private Long id;
    /** 用户ID */
    private Long userId;
    /** 公告ID */
    private Long noticeId;
    /** 阅读时间 */
    private LocalDateTime readTime;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
}
```

- [ ] **Step 2: 提交**

```bash
git add vaccine_system/src/main/java/com/tjut/edu/vaccine_system/model/entity/UserNoticeRead.java
git commit -m "feat: add UserNoticeRead entity for notice read tracking"
```

---

### Task 2: 创建 UserNoticeRead Mapper

**Files:**
- Create: `vaccine_system/src/main/java/com/tjut/edu/vaccine_system/mapper/UserNoticeReadMapper.java`

- [ ] **Step 1: 创建 Mapper 接口**

```java
package com.tjut.edu.vaccine_system.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.tjut.edu.vaccine_system.model.entity.UserNoticeRead;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserNoticeReadMapper extends BaseMapper<UserNoticeRead> {
}
```

- [ ] **Step 2: 提交**

```bash
git add vaccine_system/src/main/java/com/tjut/edu/vaccine_system/mapper/UserNoticeReadMapper.java
git commit -m "feat: add UserNoticeReadMapper"
```

---

### Task 3: 创建 UserNoticeRead Service

**Files:**
- Create: `vaccine_system/src/main/java/com/tjut/edu/vaccine_system/service/UserNoticeReadService.java`
- Create: `vaccine_system/src/main/java/com/tjut/edu/vaccine_system/service/impl/UserNoticeReadServiceImpl.java`

- [ ] **Step 1: 创建 Service 接口**

```java
package com.tjut.edu.vaccine_system.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.tjut.edu.vaccine_system.model.entity.UserNoticeRead;

/**
 * 用户公告已读 Service
 */
public interface UserNoticeReadService extends IService<UserNoticeRead> {

    /**
     * 标记公告为已读（幂等：已读则更新时间，未读则新增）
     */
    void markAsRead(Long userId, Long noticeId);

    /**
     * 检查公告是否已读
     */
    boolean isRead(Long userId, Long noticeId);

    /**
     * 获取用户已读公告ID列表
     */
    java.util.List<Long> listReadNoticeIds(Long userId);
}
```

- [ ] **Step 2: 创建 Service 实现类**

```java
package com.tjut.edu.vaccine_system.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.tjut.edu.vaccine_system.mapper.UserNoticeReadMapper;
import com.tjut.edu.vaccine_system.model.entity.UserNoticeRead;
import com.tjut.edu.vaccine_system.service.UserNoticeReadService;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.List;
import java.util.stream.Collectors;

@Service
public class UserNoticeReadServiceImpl extends ServiceImpl<UserNoticeReadMapper, UserNoticeRead> implements UserNoticeReadService {

    @Override
    public void markAsRead(Long userId, Long noticeId) {
        // 检查是否已读
        LambdaQueryWrapper<UserNoticeRead> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(UserNoticeRead::getUserId, userId)
               .eq(UserNoticeRead::getNoticeId, noticeId);
        UserNoticeRead existing = this.getOne(wrapper);

        if (existing != null) {
            // 已读则更新时间
            existing.setReadTime(LocalDateTime.now());
            this.updateById(existing);
        } else {
            // 未读则新增
            UserNoticeRead record = UserNoticeRead.builder()
                    .userId(userId)
                    .noticeId(noticeId)
                    .readTime(LocalDateTime.now())
                    .build();
            this.save(record);
        }
    }

    @Override
    public boolean isRead(Long userId, Long noticeId) {
        LambdaQueryWrapper<UserNoticeRead> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(UserNoticeRead::getUserId, userId)
               .eq(UserNoticeRead::getNoticeId, noticeId);
        return this.count(wrapper) > 0;
    }

    @Override
    public List<Long> listReadNoticeIds(Long userId) {
        LambdaQueryWrapper<UserNoticeRead> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(UserNoticeRead::getUserId, userId)
               .select(UserNoticeRead::getNoticeId);
        return this.list(wrapper).stream()
                .map(UserNoticeRead::getNoticeId)
                .collect(Collectors.toList());
    }
}
```

- [ ] **Step 3: 提交**

```bash
git add vaccine_system/src/main/java/com/tjut/edu/vaccine_system/service/UserNoticeReadService.java
git add vaccine_system/src/main/java/com/tjut/edu/vaccine_system/service/impl/UserNoticeReadServiceImpl.java
git commit -m "feat: add UserNoticeReadService for notice read tracking"
```

---

### Task 4: 添加公告已读标记接口

**Files:**
- Modify: `vaccine_system/src/main/java/com/tjut/edu/vaccine_system/controller/user/UserNoticeController.java`

- [ ] **Step 1: 添加已读标记接口**

在 UserNoticeController 中添加：

```java
@Operation(summary = "标记公告为已读")
@PostMapping("/{id}/read")
public Result<Void> markAsRead(@PathVariable Long id, @RequestParam Long userId) {
    userNoticeReadService.markAsRead(userId, id);
    return Result.ok();
}

@Operation(summary = "获取用户已读公告ID列表")
@GetMapping("/read-ids")
public Result<List<Long>> getReadNoticeIds(@RequestParam Long userId) {
    return Result.ok(userNoticeReadService.listReadNoticeIds(userId));
}

@Operation(summary = "检查公告是否已读")
@GetMapping("/{id}/check-read")
public Result<Boolean> checkRead(@PathVariable Long id, @RequestParam Long userId) {
    return Result.ok(userNoticeReadService.isRead(userId, id));
}
```

并在类中添加依赖（注意：NoticeService 已存在，只需添加新字段）：

```java
private final UserNoticeReadService userNoticeReadService;
```

同时添加 imports：

```java
import com.tjut.edu.vaccine_system.service.UserNoticeReadService;
import java.util.List;
```

- [ ] **Step 2: 提交**

```bash
git add vaccine_system/src/main/java/com/tjut/edu/vaccine_system/controller/user/UserNoticeController.java
git commit -m "feat: add notice read marking endpoints to UserNoticeController"
```

---

### Task 5: 更新数据库设计文档

**Files:**
- Modify: `vaccine_system/docs/database.md`

- [ ] **Step 1: 更新 DoctorDispatch 状态说明**

在 database.md 的 "3.1 doctor_dispatch（医生调遣申请表）" 部分，将 status 说明从：
```
| status | tinyint | 0-待审批，1-同意，2-拒绝（DoctorDispatchStatusEnum）。 |
```

修改为：
```
| status | tinyint | 0-已推送，1-已读（DoctorDispatchStatusEnum，已简化审批流程）。 |
```

- [ ] **Step 2: 提交**

```bash
git add vaccine_system/docs/database.md
git commit -m "docs: update DoctorDispatch status in database.md"
```

---

### Task 6: 验证编译

- [ ] **Step 1: 编译项目**

```bash
cd vaccine_system
mvn compile -q
```

Expected: BUILD SUCCESS

- [ ] **Step 2: 提交**

```bash
git commit --allow-empty -m "chore: verify compilation"
```

---

## 执行顺序

1. Task 1: 创建 UserNoticeRead 实体类
2. Task 2: 创建 UserNoticeRead Mapper
3. Task 3: 创建 UserNoticeRead Service
4. Task 4: 添加公告已读标记接口
5. Task 5: 更新数据库设计文档
6. Task 6: 验证编译

---

## 预期结果

- 缺失的 user_notice_read 功能完整实现
- 数据库文档与实际代码一致
- 项目编译通过