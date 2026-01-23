# 专家标签体系设计文档

> 设计时间：2026-01-23  
> 核心理念：**专家在平台上的全流程都是在不断给专家身上打标签**

---

## 一、设计背景

### 1.1 现状分析

当前系统中存在以下实体关系：
- **岗位(Job)**：对外招聘入口，有推广奖励机制
- **项目(Project)**：通过 `tbl_project_jobs` 与岗位关联
- **专家(User)**：有 `tags` 字段存储标签ID
- **领域(Industry)**：通过 `tbl_expert_industries` 存储专家擅长领域

### 1.2 核心问题

| 问题 | 描述 |
|------|------|
| 耦合过紧 | 项目必须关联岗位，灵活性不足 |
| 标签利用率低 | 现有标签系统未充分利用，无法支撑精准筛选 |
| 标签来源单一 | 标签主要靠手动添加，缺乏自动化机制 |
| 缺乏分类体系 | 标签没有结构化分类，难以管理和筛选 |

### 1.3 设计目标

1. **解耦**：项目可以直接按标签筛选专家，不强依赖岗位
2. **自动化**：在关键节点自动为专家授予标签
3. **结构化**：建立清晰的标签分类体系
4. **可追溯**：记录标签的授予来源和时间

---

## 二、核心概念定义

### 2.1 四大实体的职责重新定义

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           实体职责清晰化                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  【岗位 Job】                    【项目 Project】                        │
│  ┌─────────────────────┐        ┌─────────────────────┐                │
│  │ 定位：招聘入口       │        │ 定位：工作载体       │                │
│  │ 职责：              │        │ 职责：              │                │
│  │  • 对外展示招聘信息  │        │  • 承载具体标注任务  │                │
│  │  • 配置招聘条件标签  │        │  • 按标签筛选专家    │                │
│  │  • 设置推广奖励     │        │  • 配置工作流程     │                │
│  │  • 审核通过授予标签  │   →→→  │  • 与岗位可选关联    │                │
│  └─────────────────────┘        └─────────────────────┘                │
│           ↓                              ↑                              │
│      授予入口标签                    按标签匹配                          │
│           ↓                              ↑                              │
│  ┌─────────────────────┐        ┌─────────────────────┐                │
│  │ 【标签 Tag】         │←←←←←←←│ 【专家 Expert】      │                │
│  │ 定位：能力画像       │        │ 定位：劳动力        │                │
│  │ 职责：              │        │ 职责：              │                │
│  │  • 描述专家特征     │        │  • 完成标注任务     │                │
│  │  • 支撑精准匹配     │        │  • 积累能力标签     │                │
│  │  • 分类分层管理     │        │  • 获取收入        │                │
│  └─────────────────────┘        └─────────────────────┘                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 四大实体关系矩阵

| 关系 | 类型 | 说明 | 是否必须 |
|------|------|------|----------|
| 岗位 → 标签 | 配置关系 | 岗位配置准入标签要求 + 审核通过授予标签 | 可选 |
| 项目 → 标签 | 筛选关系 | 项目按标签组合筛选专家 | 推荐 |
| 项目 → 岗位 | 关联关系 | 项目可关联岗位用于推广引流 | **可选** |
| 专家 → 标签 | 拥有关系 | 专家在全流程中积累标签 | 核心 |

---

## 三、标签分类体系

### 3.1 标签分类总览

```
专家标签体系
├── 1. 身份认证类 (identity)
│   ├── 实名认证
│   ├── 学历认证
│   └── 资质认证
│
├── 2. 能力资质类 (qualification)
│   ├── 领域专业
│   ├── 技能证书
│   └── 平台考核
│
├── 3. 行为表现类 (behavior)
│   ├── 活跃度
│   ├── 质量评级
│   └── 信用等级
│
├── 4. 项目经验类 (experience)
│   ├── 参与项目
│   ├── 任务类型
│   └── 累计产出
│
├── 5. 来源渠道类 (source)
│   ├── 招聘渠道
│   ├── 邀请来源
│   └── 岗位入口
│
└── 6. 运营管理类 (operation)
    ├── VIP等级
    ├── 特殊标记
    └── 黑白名单
```

### 3.2 标签详细定义

#### 3.2.1 身份认证类 (identity)

| 标签组 | 标签值 | 说明 | 授予条件 |
|--------|--------|------|----------|
| 实名认证 | `已实名` | 完成三要素认证 | 实名认证通过 |
| 学历认证 | `本科` `硕士` `博士` `博士后` | 最高学历 | 资料审核通过 |
| 学校类型 | `985` `211` `双一流` `海外名校` | 院校等级 | 教育经历审核 |
| 资质认证 | `CPA` `CFA` `医师资格` `律师资格` | 专业资质 | 证明材料审核 |

#### 3.2.2 能力资质类 (qualification)

| 标签组 | 标签值 | 说明 | 授予条件 |
|--------|--------|------|----------|
| 主领域 | `经济金融` `医疗健康` `法律` `代码` ... | 一级领域 | 资料填写+审核 |
| 子领域 | `投资` `临床医学` `民法` `Python` ... | 二/三级领域 | 资料填写+审核 |
| 语言能力 | `英语` `日语` `法语` `德语` ... | 外语技能 | 资料审核/考试 |
| 平台认证 | `初级标注师` `中级标注师` `高级标注师` | 平台考核等级 | 准入考试通过 |
| 项目认证 | `项目A认证` `项目B认证` | 项目专项认证 | 项目考试通过 |

#### 3.2.3 行为表现类 (behavior)

| 标签组 | 标签值 | 说明 | 授予条件 |
|--------|--------|------|----------|
| 活跃度 | `高活跃` `中活跃` `低活跃` `沉默` | 近期活跃情况 | 系统自动计算 |
| 质量评级 | `S级` `A级` `B级` `C级` | 标注质量等级 | 质检数据统计 |
| 效率评级 | `高效` `正常` `偏慢` | 工作效率等级 | 任务耗时统计 |
| 信用等级 | `优秀` `良好` `一般` `较差` | 综合信用 | 多维度计算 |

#### 3.2.4 项目经验类 (experience)

| 标签组 | 标签值 | 说明 | 授予条件 |
|--------|--------|------|----------|
| 参与项目 | `[项目名]参与者` | 项目参与记录 | 加入项目 |
| 项目角色 | `标注员` `质检员` `审核员` | 担任角色 | 分配角色 |
| 任务类型 | `对话标注` `图像标注` `代码标注` ... | 经验任务类型 | 完成任务 |
| 产出等级 | `千题` `万题` `十万题` | 累计产出量 | 任务完成统计 |

#### 3.2.5 来源渠道类 (source)

| 标签组 | 标签值 | 说明 | 授予条件 |
|--------|--------|------|----------|
| 注册渠道 | `官网` `微信` `APP` `H5` | 注册来源 | 注册时记录 |
| 推广渠道 | `校园大使` `供应商` `自然流量` | 推广来源 | 注册时记录 |
| 邀请来源 | `专家邀请` `管理员邀请` | 被邀请方式 | 邀请关系记录 |
| 岗位入口 | `[岗位名]入口` | 通过哪个岗位进入 | 岗位申请通过 |

#### 3.2.6 运营管理类 (operation)

| 标签组 | 标签值 | 说明 | 授予条件 |
|--------|--------|------|----------|
| VIP等级 | `普通` `银牌` `金牌` `钻石` | 会员等级 | 积分/贡献计算 |
| 特殊标记 | `重点关注` `潜力专家` `核心骨干` | 运营标记 | 管理员手动 |
| 风险标记 | `疑似作弊` `多次返工` `投诉记录` | 风控标记 | 系统/人工判定 |

---

## 四、标签授予节点

### 4.1 专家全流程标签授予图

```
专家生命周期流程
═══════════════════════════════════════════════════════════════════════════

[注册] ──→ [填写资料] ──→ [实名认证] ──→ [准入考试] ──→ [岗位申请] ──→ [项目参与] ──→ [任务执行] ──→ [持续运营]
  │           │             │             │             │             │             │             │
  ▼           ▼             ▼             ▼             ▼             ▼             ▼             ▼
┌───┐     ┌─────┐       ┌─────┐       ┌─────┐       ┌─────┐       ┌─────┐       ┌─────┐       ┌─────┐
│来源│     │学历 │       │实名 │       │平台 │       │岗位 │       │项目 │       │行为 │       │运营 │
│标签│     │领域 │       │认证 │       │认证 │       │入口 │       │经验 │       │表现 │       │标签 │
│   │     │资质 │       │标签 │       │标签 │       │标签 │       │标签 │       │标签 │       │   │
└───┘     └─────┘       └─────┘       └─────┘       └─────┘       └─────┘       └─────┘       └─────┘

```

### 4.2 各节点详细授予规则

#### 节点1：注册

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 用户完成注册 | 注册渠道标签（官网/微信/APP） | source |
| 有邀请人 | 邀请来源标签 | source |
| 有邀请岗位 | 岗位入口标签（待审核） | source |
| 有推广码 | 推广渠道标签 | source |

#### 节点2：资料填写与审核

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 填写学历 + 审核通过 | 学历标签（本科/硕士/博士） | identity |
| 教育经历 + 审核通过 | 学校类型标签（985/211/双一流） | identity |
| 选择领域 + 审核通过 | 领域标签（主领域/子领域） | qualification |
| 上传证书 + 审核通过 | 资质认证标签 | qualification |

#### 节点3：实名认证

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 三要素认证通过 | `已实名` | identity |

#### 节点4：准入考试

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 考试通过（60-70分） | `初级标注师` | qualification |
| 考试通过（70-85分） | `中级标注师` | qualification |
| 考试通过（85分以上） | `高级标注师` | qualification |

#### 节点5：岗位申请审核

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| AI审核通过 | 岗位配置的授予标签 | 岗位配置 |
| 人工审核通过 | 岗位配置的授予标签 | 岗位配置 |
| 审核通过 | `[岗位名]入口` | source |

#### 节点6：项目参与

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 加入项目 | `[项目名]参与者` | experience |
| 试标通过 | `[项目名]试标通过` | experience |
| 分配质检角色 | `质检员` | experience |
| 完成项目考试 | `[项目名]认证` | qualification |

#### 节点7：任务执行

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 完成首题 | `任务类型标签`（如：对话标注） | experience |
| 累计100题 | `百题标注` | experience |
| 累计1000题 | `千题标注` | experience |
| 累计10000题 | `万题标注` | experience |
| 质量评分 ≥ 4.5 | `S级质量` | behavior |
| 质量评分 ≥ 4.0 | `A级质量` | behavior |

#### 节点8：持续运营（系统定时任务）

| 触发条件 | 授予/更新标签 | 标签分类 |
|----------|--------------|----------|
| 7天内有活动 | `高活跃` | behavior |
| 30天内有活动 | `中活跃` | behavior |
| 30天无活动 | `低活跃` | behavior |
| 90天无活动 | `沉默用户` | behavior |
| 综合评估 | VIP等级更新 | operation |
| 管理员标记 | 特殊标记 | operation |

---

## 五、岗位与标签的关系

### 5.1 岗位的双向标签配置

```
┌─────────────────────────────────────────────────────────────────┐
│                         岗位标签配置                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  【准入标签要求】                    【审核通过授予标签】          │
│  ┌─────────────────────┐          ┌─────────────────────┐      │
│  │ • 筛选条件          │          │ • 入口标记          │      │
│  │ • 硬性门槛          │    →→→   │ • 能力认证          │      │
│  │ • 匹配度计算        │  审核通过  │ • 项目准入资格      │      │
│  └─────────────────────┘          └─────────────────────┘      │
│                                                                 │
│  示例：                                                         │
│  ┌─────────────────────┐          ┌─────────────────────┐      │
│  │ 准入要求:           │          │ 授予标签:           │      │
│  │  • 学历: 硕士及以上  │    →→→   │  • [岗位名]入口     │      │
│  │  • 领域: 经济金融   │          │  • 金融专家         │      │
│  │  • 认证: 已实名     │          │  • 项目A准入资格    │      │
│  └─────────────────────┘          └─────────────────────┘      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 岗位标签配置数据结构

```json
{
  "job_id": 1,
  "job_name": "金融领域数据标注专家",
  
  "tag_requirements": {
    "required": [
      {"category": "identity", "group": "学历认证", "values": ["硕士", "博士"]},
      {"category": "identity", "group": "实名认证", "values": ["已实名"]}
    ],
    "preferred": [
      {"category": "qualification", "group": "主领域", "values": ["经济金融"]},
      {"category": "qualification", "group": "资质认证", "values": ["CFA", "CPA"]}
    ],
    "excluded": [
      {"category": "operation", "group": "风险标记", "values": ["疑似作弊"]}
    ]
  },
  
  "grant_tags_on_approval": [
    {"category": "source", "group": "岗位入口", "value": "金融标注岗入口"},
    {"category": "qualification", "group": "项目认证", "value": "金融项目准入资格"}
  ]
}
```

---

## 六、项目与标签的关系

### 6.1 项目按标签筛选专家

项目不再强依赖岗位，可直接配置标签筛选条件：

```
┌─────────────────────────────────────────────────────────────────┐
│                       项目专家筛选配置                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  方式一：按标签筛选（推荐）                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 必须满足:                                                │   │
│  │   • 学历 IN (硕士, 博士)                                 │   │
│  │   • 领域 = 经济金融                                      │   │
│  │   • 平台认证 IN (中级标注师, 高级标注师)                   │   │
│  │                                                         │   │
│  │ 优先满足:                                                │   │
│  │   • 质量评级 IN (S级, A级)                               │   │
│  │   • 活跃度 = 高活跃                                      │   │
│  │                                                         │   │
│  │ 排除条件:                                                │   │
│  │   • 风险标记 = 疑似作弊                                  │   │
│  │   • 信用等级 = 较差                                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  方式二：按岗位筛选（可选）                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 关联岗位: 金融领域数据标注专家                             │   │
│  │ 作用:                                                    │   │
│  │   • 从该岗位通过的专家中筛选                              │   │
│  │   • 用于推广引流                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 项目筛选配置数据结构

```json
{
  "project_id": 1,
  "project_name": "金融数据标注项目",
  
  "expert_filter": {
    "mode": "tag",  // tag: 按标签筛选, job: 按岗位筛选, mixed: 混合
    
    "tag_conditions": {
      "must": [
        {"category": "identity", "group": "学历认证", "operator": "in", "values": ["硕士", "博士"]},
        {"category": "qualification", "group": "主领域", "operator": "eq", "value": "经济金融"}
      ],
      "should": [
        {"category": "behavior", "group": "质量评级", "operator": "in", "values": ["S级", "A级"]}
      ],
      "must_not": [
        {"category": "operation", "group": "风险标记", "operator": "eq", "value": "疑似作弊"}
      ]
    },
    
    "job_ids": [1, 2],  // 可选关联岗位
    "job_filter_mode": "union"  // union: 并集, intersection: 交集
  }
}
```

---

## 七、数据库设计

### 7.1 标签表重构

```sql
-- 标签定义表（重构）
CREATE TABLE tbl_tags (
    tag_id SERIAL PRIMARY KEY,
    
    -- 标签分类
    category VARCHAR(50) NOT NULL,       -- 标签大类: identity/qualification/behavior/experience/source/operation
    tag_group VARCHAR(100) NOT NULL,     -- 标签组: 学历认证/主领域/活跃度 等
    tag_value VARCHAR(200) NOT NULL,     -- 标签值: 硕士/博士/高活跃 等
    
    -- 标签属性
    display_name VARCHAR(200),           -- 显示名称
    description TEXT,                    -- 标签描述
    sort_order INT DEFAULT 0,            -- 排序
    color VARCHAR(20),                   -- 标签颜色（用于前端展示）
    icon VARCHAR(100),                   -- 标签图标
    
    -- 标签规则
    is_system BOOLEAN DEFAULT false,     -- 是否系统标签（不可删除）
    is_auto_grant BOOLEAN DEFAULT false, -- 是否自动授予
    auto_grant_rule JSONB,               -- 自动授予规则（JSON）
    is_exclusive BOOLEAN DEFAULT false,  -- 是否互斥（同组只能有一个）
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    
    -- 唯一约束
    CONSTRAINT uq_tag_category_group_value UNIQUE (category, tag_group, tag_value)
);

COMMENT ON TABLE tbl_tags IS '标签定义表';
COMMENT ON COLUMN tbl_tags.category IS '标签大类: identity-身份认证, qualification-能力资质, behavior-行为表现, experience-项目经验, source-来源渠道, operation-运营管理';
COMMENT ON COLUMN tbl_tags.is_exclusive IS '是否互斥：同一tag_group下，专家只能拥有一个标签';
```

### 7.2 专家标签关联表

```sql
-- 专家标签关联表（新增）
CREATE TABLE tbl_expert_tags (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,                -- 专家用户ID
    tag_id INT NOT NULL,                 -- 标签ID
    
    -- 授予信息
    grant_source VARCHAR(50) NOT NULL,   -- 授予来源: system/manual/job/project/exam
    grant_source_id INT,                 -- 来源ID（岗位ID/项目ID/考试ID等）
    grant_source_name VARCHAR(200),      -- 来源名称
    granted_by INT,                      -- 授予人（人工授予时）
    granted_at TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    
    -- 有效期
    valid_from TIMESTAMP,                -- 生效时间
    valid_until TIMESTAMP,               -- 失效时间（NULL表示永久有效）
    
    -- 状态
    status VARCHAR(20) DEFAULT 'active', -- active/expired/revoked
    revoke_reason TEXT,                  -- 撤销原因
    revoked_by INT,                      -- 撤销人
    revoked_at TIMESTAMP,                -- 撤销时间
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    
    -- 唯一约束（同一用户同一标签只能有一条有效记录）
    CONSTRAINT uq_expert_tag_active UNIQUE (user_id, tag_id) WHERE status = 'active' AND deleted = false
);

CREATE INDEX idx_expert_tags_user_id ON tbl_expert_tags(user_id) WHERE deleted = false;
CREATE INDEX idx_expert_tags_tag_id ON tbl_expert_tags(tag_id) WHERE deleted = false;
CREATE INDEX idx_expert_tags_status ON tbl_expert_tags(user_id, status) WHERE deleted = false;

COMMENT ON TABLE tbl_expert_tags IS '专家标签关联表';
COMMENT ON COLUMN tbl_expert_tags.grant_source IS '授予来源: system-系统自动, manual-人工授予, job-岗位审核, project-项目授予, exam-考试通过';
```

### 7.3 岗位标签配置表

```sql
-- 岗位标签配置表（新增）
CREATE TABLE tbl_job_tag_configs (
    id SERIAL PRIMARY KEY,
    job_id INT NOT NULL,                 -- 岗位ID
    
    -- 配置类型
    config_type VARCHAR(20) NOT NULL,    -- requirement: 准入要求, grant: 授予标签
    requirement_type VARCHAR(20),        -- required/preferred/excluded（仅requirement类型使用）
    
    -- 标签信息
    tag_id INT,                          -- 标签ID（直接关联）
    tag_category VARCHAR(50),            -- 或：标签大类
    tag_group VARCHAR(100),              -- 或：标签组
    tag_values JSONB,                    -- 或：标签值数组
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

CREATE INDEX idx_job_tag_configs_job_id ON tbl_job_tag_configs(job_id) WHERE deleted = false;

COMMENT ON TABLE tbl_job_tag_configs IS '岗位标签配置表';
COMMENT ON COLUMN tbl_job_tag_configs.config_type IS '配置类型: requirement-准入要求, grant-审核通过授予';
COMMENT ON COLUMN tbl_job_tag_configs.requirement_type IS '要求类型: required-必须, preferred-优先, excluded-排除';
```

### 7.4 项目标签筛选配置表

```sql
-- 项目专家筛选配置表（新增）
CREATE TABLE tbl_project_expert_filter (
    id SERIAL PRIMARY KEY,
    project_id INT NOT NULL,             -- 项目ID
    
    -- 筛选模式
    filter_mode VARCHAR(20) NOT NULL DEFAULT 'tag',  -- tag/job/mixed
    
    -- 标签筛选条件（JSON）
    tag_conditions JSONB,                -- 标签筛选条件
    
    -- 岗位关联（可选）
    job_ids JSONB,                       -- 关联岗位ID数组
    job_filter_mode VARCHAR(20),         -- union/intersection
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

CREATE UNIQUE INDEX idx_project_expert_filter_project ON tbl_project_expert_filter(project_id) WHERE deleted = false;

COMMENT ON TABLE tbl_project_expert_filter IS '项目专家筛选配置表';
COMMENT ON COLUMN tbl_project_expert_filter.filter_mode IS '筛选模式: tag-按标签, job-按岗位, mixed-混合';
```

### 7.5 标签授予日志表

```sql
-- 标签授予日志表（新增）
CREATE TABLE tbl_tag_grant_logs (
    log_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,                -- 专家用户ID
    tag_id INT NOT NULL,                 -- 标签ID
    
    -- 操作信息
    action VARCHAR(20) NOT NULL,         -- grant/revoke/update
    
    -- 授予来源
    grant_source VARCHAR(50),            -- system/manual/job/project/exam
    grant_source_id INT,
    grant_source_name VARCHAR(200),
    
    -- 操作人
    operator_id INT,                     -- 操作人ID
    operator_type VARCHAR(20),           -- system/admin/expert
    
    -- 备注
    remark TEXT,
    
    -- 时间
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

CREATE INDEX idx_tag_grant_logs_user_id ON tbl_tag_grant_logs(user_id);
CREATE INDEX idx_tag_grant_logs_tag_id ON tbl_tag_grant_logs(tag_id);
CREATE INDEX idx_tag_grant_logs_create_time ON tbl_tag_grant_logs(create_time DESC);

COMMENT ON TABLE tbl_tag_grant_logs IS '标签授予日志表';
```

---

## 八、推广奖励机制说明

### 8.1 当前机制保持

推广奖励机制与岗位绑定，保持不变：

```
推广奖励流程
═══════════════════════════════════════════════════

专家A                    专家B
  │                        │
  │  ① 生成邀请链接        │
  │  (携带岗位ID)          │
  │        ────────────►   │
  │                        │
  │                    ② 注册
  │                    (记录邀请人+岗位)
  │                        │
  │                    ③ 申请岗位
  │                        │
  │                    ④ 审核通过
  │                        │
  │                    ⑤ 完成2道题
  │                        │
  │  ⑥ 发放推广奖金         │
  │◄────────────────────   │
  │                        │

```

### 8.2 推广奖励与标签的关系

- 推广奖励仍然与岗位绑定
- 被邀请人通过岗位审核时，同时：
  - 触发推广奖励流程
  - 授予岗位入口标签
  - 授予岗位配置的其他标签

---

## 九、实施建议

### 9.1 分阶段实施

| 阶段 | 内容 | 优先级 |
|------|------|--------|
| 阶段1 | 标签分类体系搭建、基础数据表创建 | P0 |
| 阶段2 | 岗位标签配置功能、审核授予标签 | P0 |
| 阶段3 | 自动标签授予机制（注册/考试/任务） | P1 |
| 阶段4 | 项目按标签筛选专家 | P1 |
| 阶段5 | 行为标签自动计算（活跃度/质量） | P2 |
| 阶段6 | 标签统计分析、运营看板 | P2 |

### 9.2 兼容性考虑

- 现有 `tbl_users.tags` 字段保留，逐步迁移到新的 `tbl_expert_tags` 表
- 现有 `tbl_project_jobs` 关系保留，新增按标签筛选方式
- 现有岗位申请流程保留，增加标签授予逻辑

### 9.3 初始化标签数据

建议预置的系统标签：

```sql
-- 身份认证类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system, is_exclusive) VALUES
('identity', '实名认证', '已实名', true, true),
('identity', '学历认证', '本科', true, true),
('identity', '学历认证', '硕士', true, true),
('identity', '学历认证', '博士', true, true),
('identity', '学历认证', '博士后', true, true),
('identity', '学校类型', '985', true, false),
('identity', '学校类型', '211', true, false),
('identity', '学校类型', '双一流', true, false),
('identity', '学校类型', '海外名校', true, false);

-- 能力资质类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system, is_exclusive) VALUES
('qualification', '平台认证', '初级标注师', true, true),
('qualification', '平台认证', '中级标注师', true, true),
('qualification', '平台认证', '高级标注师', true, true);

-- 行为表现类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system, is_exclusive) VALUES
('behavior', '活跃度', '高活跃', true, true),
('behavior', '活跃度', '中活跃', true, true),
('behavior', '活跃度', '低活跃', true, true),
('behavior', '活跃度', '沉默用户', true, true),
('behavior', '质量评级', 'S级', true, true),
('behavior', '质量评级', 'A级', true, true),
('behavior', '质量评级', 'B级', true, true),
('behavior', '质量评级', 'C级', true, true);

-- 运营管理类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system) VALUES
('operation', 'VIP等级', '普通', true),
('operation', 'VIP等级', '银牌', true),
('operation', 'VIP等级', '金牌', true),
('operation', 'VIP等级', '钻石', true),
('operation', '风险标记', '疑似作弊', true),
('operation', '风险标记', '多次返工', true);
```

---

## 十、总结

### 10.1 核心变化

| 维度 | 原设计 | 新设计 |
|------|--------|--------|
| 项目-岗位关系 | 强关联 | 可选关联 |
| 专家筛选方式 | 通过岗位 | 通过标签（主）+ 岗位（辅） |
| 标签管理 | 简单列表 | 分类分层体系 |
| 标签来源 | 手动为主 | 自动+手动 |
| 岗位作用 | 项目入口 | 招聘入口 + 标签授予 |

### 10.2 收益分析

1. **精准匹配**：项目可以更精准地找到符合要求的专家
2. **灵活运营**：减少岗位与项目的耦合，运营更灵活
3. **专家画像**：建立完整的专家能力画像，支撑智能推荐
4. **可追溯性**：标签授予有迹可循，便于审计
5. **扩展性强**：标签体系可灵活扩展，支持业务发展

---

> 文档版本：v1.0  
> 作者：AI 助手  
> 日期：2026-01-23
