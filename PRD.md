# 客服管理系统 PRD 文档

## 1. 产品概述

### 1.1 产品简介
客服管理系统是一个面向企业客服团队的综合性管理平台，旨在提升客服工作效率、优化服务质量、提供数据化运营支持。系统包含客服工作台、管理后台、数据分析等核心功能模块。

### 1.2 产品目标
- 提升客服工作效率和服务质量
- 实现客服团队数字化管理和绩效分析
- 提供实时监控和预警机制
- 支持多角色权限管理

### 1.3 目标用户
- **普通客服**：负责日常客户咨询和问题解答
- **管理员**：负责团队管理、数据分析和决策支持
- **超级管理员**：拥有系统最高权限，可进行全局配置和管理

## 2. 功能模块详细说明

### 2.1 客服列表模块

#### 2.1.1 基础信息展示
| 字段名称 | 数据类型 | 说明 | 计算公式/逻辑 |
|---------|---------|------|---------------|
| 客服头像 | 图片URL | 客服个人头像 | 上传路径 + 文件名 |
| 客服姓名 | 字符串 | 客服真实姓名 | 用户输入 |
| 工号 | 字符串 | 唯一标识 | CS + 三位数字 |
| 所属分组 | 字符串 | 部门分组 | 下拉选择（售前咨询组、售后服务组、VIP客服组、管理组） |
| 在线状态 | 枚举 | 当前状态 | online/busy/offline |
| 当前接待量 | 整数 | 正在处理的会话数 | COUNT(会话ID WHERE 客服ID=当前客服 AND 状态='进行中') |
| 最大接待量 | 整数 | 可同时处理的最大会话数 | 根据客服等级设置 |
| 最后登录时间 | 时间戳 | 最近登录时间 | MAX(登录时间 WHERE 客服ID=当前客服) |

#### 2.1.2 高级筛选功能
| 筛选条件 | 字段类型 | 说明 | 计算逻辑 |
|---------|---------|------|---------|
| 状态筛选 | 多选 | 在线/忙碌/离线 | WHERE 状态 IN (选中状态) |
| 分组筛选 | 下拉 | 所属部门分组 | WHERE 分组 = 选中分组 |
| 技能标签 | 多选 | 客服专长标签 | WHERE 标签 CONTAINS 选中标签 |

#### 2.1.3 客服详情数据
| 指标名称 | 数据类型 | 说明 | 计算公式 |
|---------|---------|------|---------|
| 累计接待量 | 整数 | 历史总接待客户数 | COUNT(DISTINCT 用户ID WHERE 客服ID=当前客服) |
| 满意度 | 小数 | 客户评价平均分 | AVG(评分 WHERE 客服ID=当前客服 AND 评分>0) |
| 平均响应时间 | 时间 | 首次回复平均耗时 | AVG(首次回复时间 - 消息时间) |
| 今日接待量 | 整数 | 当日接待客户数 | COUNT(DISTINCT 用户ID WHERE 客服ID=当前客服 AND 日期=今天) |
| 入职时间 | 日期 | 参加工作时间 | 用户录入时间 |

### 2.2 数据概览模块

#### 2.2.1 核心统计指标
| 指标名称 | 数据类型 | 说明 | 计算公式 |
|---------|---------|------|---------|
| 排队用户数 | 整数 | 等待分配客服的用户数 | COUNT(用户ID WHERE 状态='排队') |
| 进行中对话数 | 整数 | 正在进行的客服会话数 | COUNT(会话ID WHERE 状态='进行中') |
| 平均满意度 | 小数 | 所有客服的平均满意度 | AVG(客服满意度) |
| 在线客服数 | 整数 | 当前在线的客服人数 | COUNT(客服ID WHERE 状态='online') |
| 今日接待量 | 整数 | 当日总接待客户数 | COUNT(DISTINCT 会话ID WHERE 日期=今天) |
| 平均响应时间 | 时间 | 平均首次回复耗时 | AVG(首次回复时间 - 用户消息时间) |
| 转接率 | 百分比 | 会话转接比例 | (转接会话数 / 总会话数) × 100% |
| 解决率 | 百分比 | 问题成功解决比例 | (标记'已解决'的会话数 / 总会话数) × 100% |

#### 2.2.2 绩效分析表
| 字段名称 | 数据类型 | 说明 | 计算公式 |
|---------|---------|------|---------|
| 接待量 | 整数 | 当日处理的会话数 | COUNT(会话ID WHERE 客服ID=当前客服 AND 日期=今天) |
| 响应时长 | 时间 | 平均响应时间 | AVG(回复时间 - 用户消息时间) |
| 满意度 | 小数 | 客户评价分数 | AVG(评分 WHERE 客服ID=当前客服) |
| 转接数 | 整数 | 转移给其他客服的次数 | COUNT(转接记录 WHERE 源客服=当前客服) |
| 解决率 | 百分比 | 问题解决比例 | ('已解决'标记数 / 总会话数) × 100% |
| 工作时长 | 时间 | 当日在线总时长 | SUM(在线时间段) |

### 2.3 管理员工作台模块

#### 2.3.1 会话管理功能
| 功能点 | 说明 | 数据逻辑 |
|-------|------|---------|
| 全局会话查看 | 查看所有客服的当前会话 | SELECT * FROM 会话表 WHERE 状态='进行中' |
| 客服筛选 | 按客服筛选显示对应会话 | WHERE 客服ID = 选中的客服ID |
| 会话转移 | 将用户会话转移给其他客服 | UPDATE 会话表 SET 客服ID=目标客服 WHERE 会话ID=当前会话 |

#### 2.3.2 敏感词预警
| 字段名称 | 说明 | 触发条件 |
|---------|------|---------|
| 敏感词 | 预设的关键词列表 | "投诉"、"退款"、"报警"等 |
| 预警时间 | 检测到敏感词的时间戳 | 当前时间 |
| 对话双方 | 客服和用户信息 | 会话表中的客服ID和用户ID |
| 上下文 | 包含敏感词的对话内容 | 消息内容 LIKE '%敏感词%' |

### 2.4 普通客服工作台模块

#### 2.4.1 会话列表
| 字段名称 | 数据类型 | 说明 | 计算逻辑 |
|---------|---------|------|---------|
| 用户姓名 | 字符串 | 客户姓名 | 用户表姓名字段 |
| VIP标识 | 布尔 | 是否VIP用户 | 用户表VIP等级 > 0 |
| 最后消息 | 字符串 | 最新消息内容 | 消息表最新记录 |
| 未读数 | 整数 | 未读消息数量 | COUNT(消息ID WHERE 已读=false AND 方向='用户') |

#### 2.4.2 消息发送功能
| 字段名称 | 数据类型 | 说明 | 计算逻辑 |
|---------|---------|------|---------|
| 消息内容 | 文本 | 客服回复内容 | 用户输入 |
| 消息类型 | 枚举 | 消息格式 | text/image/file/emoji |
| 发送时间 | 时间戳 | 消息发送时间 | 当前时间 |
| 消息方向 | 枚举 | 发送方 | '客服' |

## 3. 数据结构和计算逻辑

### 3.1 核心数据表设计

#### 3.1.1 客服信息表 (cs_staff)
```sql
CREATE TABLE cs_staff (
    staff_id VARCHAR(10) PRIMARY KEY,  -- 工号
    staff_name VARCHAR(50) NOT NULL,   -- 姓名
    avatar_url VARCHAR(200),           -- 头像URL
    department VARCHAR(50),            -- 部门
    status ENUM('online', 'busy', 'offline'), -- 在线状态
    max_concurrent INT DEFAULT 10,     -- 最大并发数
    join_date DATE,                    -- 入职日期
    is_active BOOLEAN DEFAULT TRUE,    -- 是否激活
    is_admin BOOLEAN DEFAULT FALSE      -- 是否管理员
);
```

#### 3.1.2 会话信息表 (cs_conversation)
```sql
CREATE TABLE cs_conversation (
    conversation_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id VARCHAR(50) NOT NULL,      -- 用户ID
    staff_id VARCHAR(10),              -- 客服ID
    status ENUM('waiting', 'active', 'transferred', 'ended'), -- 会话状态
    start_time DATETIME,               -- 开始时间
    end_time DATETIME,                 -- 结束时间
    resolution_status ENUM('solved', 'pending', 'escalated'), -- 解决状态
    satisfaction_score INT,            -- 满意度评分 1-5
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

#### 3.1.3 消息记录表 (cs_message)
```sql
CREATE TABLE cs_message (
    message_id INT AUTO_INCREMENT PRIMARY KEY,
    conversation_id INT NOT NULL,       -- 会话ID
    sender_type ENUM('user', 'staff'),  -- 发送方类型
    sender_id VARCHAR(50),             -- 发送者ID
    content TEXT,                      -- 消息内容
    message_type ENUM('text', 'image', 'file', 'emoji'), -- 消息类型
    sent_at DATETIME DEFAULT CURRENT_TIMESTAMP, -- 发送时间
    is_read BOOLEAN DEFAULT FALSE,     -- 是否已读
    FOREIGN KEY (conversation_id) REFERENCES cs_conversation(conversation_id)
);
```

### 3.2 关键指标计算公式

#### 3.2.1 效率指标
1. **平均响应时间**
```sql
SELECT AVG(
    TIMESTAMPDIFF(SECOND, 
        (SELECT sent_at FROM cs_message m1 
         WHERE m1.conversation_id = m.conversation_id 
         AND m1.sender_type = 'user' 
         ORDER BY sent_at DESC LIMIT 1),
        m.sent_at
    )
) / 60 as avg_response_minutes
FROM cs_message m
WHERE m.sender_type = 'staff'
AND DATE(m.sent_at) = CURDATE();
```

2. **解决率**
```sql
SELECT 
    COUNT(CASE WHEN resolution_status = 'solved' THEN 1 END) * 100.0 / 
    COUNT(*) as resolution_rate
FROM cs_conversation
WHERE DATE(created_at) = CURDATE();
```

3. **转接率**
```sql
SELECT 
    COUNT(CASE WHEN status = 'transferred' THEN 1 END) * 100.0 / 
    COUNT(*) as transfer_rate
FROM cs_conversation
WHERE DATE(created_at) = CURDATE();
```

#### 3.2.2 服务质量指标
1. **客户满意度**
```sql
SELECT AVG(satisfaction_score) as avg_satisfaction
FROM cs_conversation
WHERE satisfaction_score IS NOT NULL
AND DATE(created_at) = CURDATE();
```

2. **客服绩效分数**
```sql
-- 综合评分计算公式
绩效分数 = (
    满意度权重 × 满意度评分 +
    响应速度权重 × (1 - 标准化响应时间) +
    解决率权重 × 解决率 +
    接待量权重 × 标准化接待量
) × 100
```

#### 3.2.3 工作负载指标
1. **客服负载率**
```sql
SELECT 
    staff_id,
    COUNT(*) * 100.0 / max_concurrent as load_rate
FROM cs_conversation c
JOIN cs_staff s ON c.staff_id = s.staff_id
WHERE c.status = 'active'
GROUP BY staff_id, max_concurrent;
```

2. **平均排队时间**
```sql
SELECT AVG(
    TIMESTAMPDIFF(MINUTE, 
        created_at, 
        (SELECT sent_at FROM cs_message 
         WHERE conversation_id = c.conversation_id 
         AND sender_type = 'staff' 
         ORDER BY sent_at ASC LIMIT 1)
    )
) as avg_wait_time
FROM cs_conversation c
WHERE DATE(c.created_at) = CURDATE();
```

### 3.3 实时监控逻辑

#### 3.3.1 队列监控
```sql
-- 排队用户数
SELECT COUNT(*) as queue_count
FROM cs_conversation
WHERE status = 'waiting';

-- 超时告警（等待超过5分钟）
SELECT conversation_id, user_id, 
       TIMESTAMPDIFF(MINUTE, created_at, NOW()) as wait_minutes
FROM cs_conversation
WHERE status = 'waiting'
AND TIMESTAMPDIFF(MINUTE, created_at, NOW()) > 5;
```

#### 3.3.2 敏感词检测
```sql
-- 敏感词预警查询
SELECT DISTINCT 
    c.conversation_id,
    c.user_id,
    s.staff_id,
    m.content,
    m.sent_at
FROM cs_message m
JOIN cs_conversation c ON m.conversation_id = c.conversation_id
JOIN cs_staff s ON c.staff_id = s.staff_id
WHERE m.content REGEXP '(投诉|退款|报警|投诉到底)'
AND m.sent_at > DATE_SUB(NOW(), INTERVAL 1 HOUR);
```

## 4. 系统架构和权限管理

### 4.1 角色权限矩阵

| 功能模块 | 普通客服 | 管理员 | 超级管理员 |
|---------|---------|-------|-----------|
| 查看自己会话 | ✓ | ✓ | ✓ |
| 查看所有会话 | ✗ | ✓ | ✓ |
| 转接会话 | ✓ | ✓ | ✓ |
| 查看统计数据 | ✗ | ✓ | ✓ |
| 管理客服状态 | ✗ | ✓ | ✓ |
| 导出报表 | ✗ | ✗ | ✓ |
| 系统配置 | ✗ | ✗ | ✓ |

### 4.2 数据访问控制
- **普通客服**：只能查看和处理分配给自己的会话
- **管理员**：可以查看所有客服的会话，进行转移分配
- **超级管理员**：拥有系统最高权限，可以进行全局配置和管理

## 5. 技术规范

### 5.1 前端技术栈
- HTML5 + CSS3 + JavaScript
- 响应式设计，支持PC端访问
- 实时通信：WebSocket或Server-Sent Events

### 5.2 后端技术要求
- RESTful API设计
- 实时消息推送
- 数据加密存储
- 操作日志记录

### 5.3 数据库设计
- 支持MySQL/PostgreSQL
- 索引优化，支持高并发查询
- 数据备份和恢复机制

## 6. 性能指标要求

### 6.1 响应时间要求
- 页面加载时间：< 2秒
- 消息发送延迟：< 500ms
- 统计数据查询：< 3秒

### 6.2 并发处理能力
- 支持同时在线客服：100+
- 支持同时会话数：1000+
- 消息处理吞吐量：10000条/分钟

### 6.3 数据存储
- 消息记录保存：6个月
- 操作日志保存：1年
- 统计数据备份：永久

## 7. 安全要求

### 7.1 数据安全
- 用户敏感信息AES加密存储
- 客服端脱敏显示（如手机号中间4位隐藏）
- HTTPS传输加密

### 7.2 操作安全
- 登录验证：工号+密码+可选短信验证
- 操作日志：记录所有关键操作
- 权限验证：每次API调用验证用户权限

## 8. 部署和维护

### 8.1 部署环境
- 支持Docker容器化部署
- 支持云平台部署
- 数据库主从分离配置

### 8.2 监控告警
- 系统性能监控
- 业务指标监控
- 异常情况自动告警

### 8.3 备份恢复
- 数据库定时备份
- 文件存储备份
- 灾难恢复方案

---

## 附录

### 附录A：字段说明表

#### 客服状态枚举值
- `online`：在线，可接收新会话
- `busy`：忙碌，暂不接收新会话  
- `offline`：离线，不在线

#### 会话状态枚举值
- `waiting`：等待分配客服
- `active`：进行中
- `transferred`：已转接
- `ended`：已结束

#### 消息类型枚举值
- `text`：文本消息
- `image`：图片消息
- `file`：文件消息
- `emoji`：表情符号

### 附录B：数据字典

#### 客服分组
- 售前咨询组：负责产品咨询和售前服务
- 售后服务组：负责售后问题和投诉处理
- VIP客服组：负责VIP客户专属服务
- 管理组：管理员团队

#### 敏感词列表（可配置）
- 投诉、投诉到底
- 退款、全额退款
- 报警、举报
- 欺诈、骗局
- 工商、315

### 附录C：计算示例

#### 绩效评分计算示例
假设客服张明明的当日数据：
- 满意度：4.8分（满分5分）
- 平均响应时间：1.2分钟
- 解决率：95%
- 接待量：25人

计算公式：
```
标准化满意度 = 4.8 / 5 = 0.96
标准化响应时间 = 1 - (1.2 / 5) = 0.76  
标准化解决率 = 0.95
标准化接待量 = 25 / 30 = 0.83 (假设当日最多接待30人)

绩效分数 = (0.4 × 0.96) + (0.3 × 0.76) + (0.2 × 0.95) + (0.1 × 0.83) = 0.89
绩效得分 = 0.89 × 100 = 89分
```

其中权重设置为：满意度40%、响应速度30%、解决率20%、接待量10%。

---

*本文档版本：v1.0*  
*最后更新：2024年11月26日*  
*文档编写：产品部*