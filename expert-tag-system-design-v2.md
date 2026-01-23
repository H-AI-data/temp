# 专家职级与标签体系设计文档 V2

> 设计时间：2026-01-23  
> 版本：v2.0（职级体系 + 完整标签体系）  
> 核心理念：**用职级体系管理核心能力认证，用标签体系构建完整画像，把专家困在算法里**

---

## 一、设计概述

### 1.1 双轨制体系架构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          专家能力管理双轨制                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────┐    ┌─────────────────────────────┐       │
│   │      职级体系（主轨）         │    │      标签体系（辅轨）         │       │
│   │      Level System           │    │      Tag System            │       │
│   ├─────────────────────────────┤    ├─────────────────────────────┤       │
│   │                             │    │                             │       │
│   │  • 按领域划分               │    │  • 6大类完整标签体系         │       │
│   │  • L1 → L2 → L3 → 专委会   │    │  • 全流程节点自动授予        │       │
│   │  • 结构化晋升路径           │    │  • 身份/能力/行为/经验/      │       │
│   │  • AI + 人工评估            │    │    来源/运营 全覆盖          │       │
│   │  • 项目准入的核心依据       │    │  • 项目准入的补充条件        │       │
│   │  • 专家可见，有荣誉感       │    │  • 精细化专家画像            │       │
│   │                             │    │                             │       │
│   └──────────────┬──────────────┘    └──────────────┬──────────────┘       │
│                  │                                  │                       │
│                  └────────────┬─────────────────────┘                       │
│                               │                                             │
│                               ▼                                             │
│                  ┌─────────────────────────────┐                            │
│                  │       项目准入判断           │                            │
│                  │  职级要求 AND/OR 标签要求    │                            │
│                  └─────────────────────────────┘                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 双轨制核心区别

| 维度 | 职级体系 | 标签体系 |
|------|----------|----------|
| **定位** | 核心能力认证，决定准入资格 | 完整画像构建，精细化运营 |
| **结构** | 领域 × 等级（严格层级） | 6大类 × 分组 × 值（全覆盖） |
| **授予方式** | 先验 + 后验，有晋升降级路径 | 全流程节点自动授予 |
| **主要用途** | 项目准入的主要依据 | 专家筛选、运营分析、特殊处理 |
| **可见性** | 专家可见，荣誉墙展示 | 部分可见，部分内部使用 |
| **数量** | 每领域一个职级 | 多标签叠加 |

### 1.3 四大实体关系

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           实体关系图                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  【岗位 Job】                           【项目 Project】                 │
│  ┌─────────────────────┐               ┌─────────────────────┐         │
│  │ 定位：招聘入口       │               │ 定位：工作载体       │         │
│  │ 职责：              │               │ 职责：              │         │
│  │  • 对外展示招聘信息  │               │  • 承载具体标注任务  │         │
│  │  • 配置招聘条件      │               │  • 按职级+标签筛选   │         │
│  │  • 设置推广奖励     │      →→→      │  • 配置工作流程     │         │
│  │  • 审核通过授予职级  │   关联项目     │  • 与岗位可选关联    │         │
│  │  • 审核通过授予标签  │               │                     │         │
│  └─────────────────────┘               └─────────────────────┘         │
│           ↓                                     ↑                       │
│      授予职级+标签                          按职级+标签匹配              │
│           ↓                                     ↑                       │
│  ┌─────────────────────┐               ┌─────────────────────┐         │
│  │ 【职级 Level】       │               │ 【标签 Tag】         │         │
│  │ 定位：能力等级       │               │ 定位：能力画像       │         │
│  │ 职责：              │               │ 职责：              │         │
│  │  • 领域×等级认证    │               │  • 描述专家特征     │         │
│  │  • 项目准入主依据   │               │  • 6大类全覆盖      │         │
│  │  • 晋升降级体系     │               │  • 支撑精准匹配     │         │
│  └──────────┬──────────┘               └──────────┬──────────┘         │
│             │                                     │                     │
│             └─────────────┬───────────────────────┘                     │
│                           │                                             │
│                           ▼                                             │
│                  ┌─────────────────────┐                                │
│                  │   【专家 Expert】    │                                │
│                  │   定位：劳动力       │                                │
│                  │   • 完成标注任务     │                                │
│                  │   • 积累职级+标签    │                                │
│                  │   • 获取收入        │                                │
│                  └─────────────────────┘                                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.4 实体关系矩阵

| 关系 | 类型 | 说明 | 是否必须 |
|------|------|------|----------|
| 岗位 → 职级 | 授予关系 | 岗位审核通过授予领域职级 | 推荐 |
| 岗位 → 标签 | 授予关系 | 岗位审核通过授予入口标签等 | 可选 |
| 岗位 → 项目 | 绑定关系 | 岗位绑定需求项目（推广奖励用） | 推荐 |
| 项目 → 职级 | 准入关系 | 项目各环节要求最低职级 | 核心 |
| 项目 → 标签 | 筛选关系 | 项目按标签组合补充筛选 | 可选 |
| 专家 → 职级 | 拥有关系 | 专家在各领域的职级 | 核心 |
| 专家 → 标签 | 拥有关系 | 专家在全流程中积累的标签 | 核心 |

---

## 二、职级体系设计

### 2.1 职级结构

```
                           专家职级体系
═══════════════════════════════════════════════════════════════════════════

                    ┌─────────────────┐
                    │   专家委员会     │  ← 顶级荣誉，参与平台决策
                    │   (Committee)   │     后验授予，极少数人
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │       L3        │  ← 资深专家，可带队质检
                    │   (Senior)      │     后验授予，需长期优秀表现
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │       L2        │  ← 熟练专家，可独立质检
                    │   (Advanced)    │     先验/后验授予
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │       L1        │  ← 初级专家，基础标注
                    │   (Junior)      │     先验授予，准入门槛
                    └─────────────────┘

每个职级都关联到具体的三级领域，例如：
  • 化学分子领域 - L2
  • 代码/Python - L1  
  • 经济金融/投资 - L3
```

### 2.2 职级定义（自然语言描述）

#### L1 - 初级专家 (Junior)

```yaml
名称: 初级专家
代码: L1
荣誉标识: 🌱 新锐
授予方式: 先验授予（岗位申请审核时）

授予条件:
  - 相关领域学历背景（本科及以上）
  - 或相关领域工作经验（1年以上）
  - 通过岗位申请的 AI/人工审核

能力要求:
  - 能理解领域基础概念
  - 能完成标准化标注任务
  - 需要指导和审核

可参与工作:
  - 标注环节的基础标注
  - 需经过试标考核
```

#### L2 - 进阶专家 (Advanced)

```yaml
名称: 进阶专家
代码: L2
荣誉标识: ⭐ 精英
授予方式: 先验授予 或 后验晋升

先验授予条件:
  - 相关领域硕士及以上学历
  - 或相关领域工作经验（3年以上）
  - 或持有相关专业资格证书
  - 通过岗位申请的 AI/人工审核

后验晋升条件（从L1晋升）:
  - 累计完成该领域 500+ 题目
  - 题目通过率 ≥ 90%
  - 近30天无严重 Issue
  - 无违规违纪记录

能力要求:
  - 深入理解领域专业知识
  - 能独立完成复杂标注
  - 能发现并报告问题

可参与工作:
  - 标注环节的复杂标注
  - 质检环节的一级质检（质1）
```

#### L3 - 资深专家 (Senior)

```yaml
名称: 资深专家
代码: L3
荣誉标识: 🏆 大师
授予方式: 仅后验晋升（不可先验授予）

晋升条件（从L2晋升）:
  - 累计完成该领域 2000+ 题目
  - 题目通过率 ≥ 95%
  - 质检准确率 ≥ 92%
  - 近90天无严重 Issue
  - 无违规违纪记录
  - 人工审核确认

能力要求:
  - 该领域专家级知识储备
  - 能把控质量标准
  - 能指导新人

可参与工作:
  - 所有标注工作
  - 质检环节的二级质检（质2）
  - 可担任小组长
```

#### 专家委员会 (Committee)

```yaml
名称: 专家委员会
代码: COMMITTEE
荣誉标识: 👑 专家委员
授予方式: 仅后验授予（需邀请+审核）

授予条件:
  - L3职级满6个月
  - 在该领域有突出贡献
  - 平台邀请 + 本人同意
  - 委员会投票通过

能力要求:
  - 行业顶级专家
  - 能制定质量标准
  - 能参与规则制定

可参与工作:
  - 参与平台规则制定
  - 终审疑难案例
  - 培训指导
  - 荣誉顾问
```

### 2.3 职级数据库设计

```sql
-- 领域职级定义表
CREATE TABLE tbl_domain_levels (
    id SERIAL PRIMARY KEY,
    
    -- 领域信息（三级领域）
    domain_l1_id INT,                    -- 一级领域ID（NULL表示通用模板）
    domain_l2_id INT,                    -- 二级领域ID（可选）
    domain_l3_id INT,                    -- 三级领域ID（可选）
    domain_name VARCHAR(200),            -- 领域名称（冗余，便于展示）
    
    -- 职级信息
    level_code VARCHAR(20) NOT NULL,     -- 职级代码: L1/L2/L3/COMMITTEE
    level_name VARCHAR(50) NOT NULL,     -- 职级名称
    level_order INT NOT NULL,            -- 职级顺序（用于比较）
    
    -- 职级定义（自然语言）
    description TEXT,                    -- 职级描述
    grant_conditions TEXT,               -- 授予条件（自然语言）
    promotion_conditions TEXT,           -- 晋升条件（自然语言）
    ability_requirements TEXT,           -- 能力要求
    work_scope TEXT,                     -- 可参与工作范围
    
    -- 量化指标（用于AI评估）
    min_task_count INT,                  -- 最低题目数
    min_pass_rate DECIMAL(5,2),          -- 最低通过率
    min_qa_accuracy DECIMAL(5,2),        -- 最低质检准确率
    max_issue_count INT,                 -- 最大Issue数（周期内）
    require_manual_review BOOLEAN,       -- 是否需要人工审核
    
    -- 荣誉展示
    honor_icon VARCHAR(50),              -- 荣誉图标
    honor_title VARCHAR(100),            -- 荣誉称号
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

COMMENT ON TABLE tbl_domain_levels IS '领域职级定义表';
COMMENT ON COLUMN tbl_domain_levels.level_order IS '职级顺序: L1=1, L2=2, L3=3, COMMITTEE=4';

-- 触发器
CREATE TRIGGER trigger_update_time BEFORE UPDATE ON tbl_domain_levels
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_domain_levels TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_domain_levels_id_seq TO dlp;
```

```sql
-- 专家职级表
CREATE TABLE tbl_expert_levels (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,                -- 专家用户ID
    
    -- 领域职级
    domain_l1_id INT NOT NULL,           -- 一级领域ID
    domain_l2_id INT,                    -- 二级领域ID
    domain_l3_id INT,                    -- 三级领域ID
    level_code VARCHAR(20) NOT NULL,     -- 当前职级代码
    
    -- 授予信息
    grant_type VARCHAR(20) NOT NULL,     -- 授予类型: priori(先验)/posteriori(后验)
    grant_source VARCHAR(50) NOT NULL,   -- 授予来源: job/promotion/manual
    grant_source_id INT,                 -- 来源ID
    granted_by INT,                      -- 授予人
    granted_at TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    grant_reason TEXT,                   -- 授予原因
    
    -- AI评估数据快照
    evaluation_snapshot JSONB,           -- 评估时的数据快照
    
    -- 状态
    status VARCHAR(20) DEFAULT 'active', -- active/suspended/revoked
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

-- 索引
CREATE INDEX idx_expert_levels_user ON tbl_expert_levels(user_id) WHERE deleted = false;
CREATE INDEX idx_expert_levels_domain ON tbl_expert_levels(domain_l1_id, domain_l2_id, domain_l3_id) WHERE deleted = false;
CREATE INDEX idx_expert_levels_level ON tbl_expert_levels(level_code) WHERE deleted = false AND status = 'active';

-- 唯一约束：同一用户同一领域只能有一个有效职级
CREATE UNIQUE INDEX idx_expert_levels_unique ON tbl_expert_levels(user_id, domain_l1_id, COALESCE(domain_l2_id, 0), COALESCE(domain_l3_id, 0)) 
WHERE status = 'active' AND deleted = false;

COMMENT ON TABLE tbl_expert_levels IS '专家职级表';

-- 触发器
CREATE TRIGGER trigger_update_time BEFORE UPDATE ON tbl_expert_levels
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_expert_levels TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_expert_levels_id_seq TO dlp;
```

```sql
-- 职级变更日志表
CREATE TABLE tbl_level_change_logs (
    log_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    
    -- 领域
    domain_l1_id INT NOT NULL,
    domain_l2_id INT,
    domain_l3_id INT,
    
    -- 变更信息
    action VARCHAR(20) NOT NULL,         -- grant/promote/demote/suspend/revoke
    from_level VARCHAR(20),              -- 变更前职级
    to_level VARCHAR(20),                -- 变更后职级
    
    -- 评估数据
    evaluation_data JSONB,               -- AI评估的详细数据
    
    -- 操作信息
    operator_type VARCHAR(20),           -- system/admin/ai
    operator_id INT,
    reason TEXT,
    
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

CREATE INDEX idx_level_change_logs_user ON tbl_level_change_logs(user_id);
CREATE INDEX idx_level_change_logs_time ON tbl_level_change_logs(create_time DESC);

COMMENT ON TABLE tbl_level_change_logs IS '职级变更日志表';

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_level_change_logs TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_level_change_logs_log_id_seq TO dlp;
```

---

## 三、标签体系设计（完整版）

### 3.1 标签分类总览

```
专家标签体系（6大类）
├── 1. 身份认证类 (identity)
│   ├── 实名认证
│   ├── 学历认证
│   ├── 学校类型
│   └── 资质认证
│
├── 2. 能力资质类 (qualification)
│   ├── 主领域
│   ├── 子领域
│   ├── 语言能力
│   ├── 平台认证
│   └── 项目认证
│
├── 3. 行为表现类 (behavior)
│   ├── 活跃度
│   ├── 质量评级
│   ├── 效率评级
│   └── 信用等级
│
├── 4. 项目经验类 (experience)
│   ├── 参与项目
│   ├── 项目角色
│   ├── 任务类型
│   └── 产出等级
│
├── 5. 来源渠道类 (source)
│   ├── 注册渠道
│   ├── 推广渠道
│   ├── 邀请来源
│   └── 岗位入口
│
└── 6. 运营管理类 (operation)
    ├── VIP等级
    ├── 特殊标记
    ├── 风险标记
    ├── 临时权限
    └── 内部标记
```

### 3.2 标签详细定义

#### 3.2.1 身份认证类 (identity)

| 标签组 | 标签值 | 说明 | 授予条件 | 是否互斥 |
|--------|--------|------|----------|----------|
| 实名认证 | `已实名` | 完成三要素认证 | 实名认证通过 | 是 |
| 学历认证 | `高中及以下` `大专` `本科` `硕士` `博士` `博士后` | 最高学历 | 资料审核通过 | 是 |
| 学校类型 | `985` `211` `双一流` `海外QS100` `海外名校` `普通本科` | 院校等级 | 教育经历审核 | 否（可多个） |
| 资质认证 | `CPA` `CFA` `医师资格` `律师资格` `教师资格` 等 | 专业资质 | 证明材料审核 | 否（可多个） |

#### 3.2.2 能力资质类 (qualification)

| 标签组 | 标签值 | 说明 | 授予条件 | 是否互斥 |
|--------|--------|------|----------|----------|
| 主领域 | `经济金融` `医疗健康` `法律` `代码` `教育` 等 | 一级领域 | 资料填写+审核 | 否 |
| 子领域 | `投资` `临床医学` `民法` `Python` `K12` 等 | 二/三级领域 | 资料填写+审核 | 否 |
| 语言能力 | `英语` `日语` `法语` `德语` `韩语` `西班牙语` 等 | 外语技能 | 资料审核/考试 | 否 |
| 平台认证 | `初级标注师` `中级标注师` `高级标注师` | 平台考核等级 | 准入考试通过 | 是 |
| 项目认证 | `[项目名]认证` | 项目专项认证 | 项目考试通过 | 否 |

#### 3.2.3 行为表现类 (behavior)

| 标签组 | 标签值 | 说明 | 授予条件 | 是否互斥 |
|--------|--------|------|----------|----------|
| 活跃度 | `高活跃` `中活跃` `低活跃` `沉默用户` | 近期活跃情况 | 系统自动计算 | 是 |
| 质量评级 | `S级` `A级` `B级` `C级` `D级` | 标注质量等级 | 质检数据统计 | 是 |
| 效率评级 | `高效` `正常` `偏慢` | 工作效率等级 | 任务耗时统计 | 是 |
| 信用等级 | `优秀` `良好` `一般` `较差` | 综合信用 | 多维度计算 | 是 |

#### 3.2.4 项目经验类 (experience)

| 标签组 | 标签值 | 说明 | 授予条件 | 是否互斥 |
|--------|--------|------|----------|----------|
| 参与项目 | `[项目名]参与者` | 项目参与记录 | 加入项目 | 否 |
| 项目角色 | `标注员` `质检员` `审核员` `小组长` | 担任角色 | 分配角色 | 否 |
| 任务类型 | `对话标注` `图像标注` `代码标注` `文本分类` 等 | 经验任务类型 | 完成任务 | 否 |
| 产出等级 | `百题` `千题` `万题` `十万题` | 累计产出量 | 任务完成统计 | 是 |

#### 3.2.5 来源渠道类 (source)

| 标签组 | 标签值 | 说明 | 授予条件 | 是否互斥 |
|--------|--------|------|----------|----------|
| 注册渠道 | `官网` `微信` `APP` `H5` `小程序` | 注册来源 | 注册时记录 | 是 |
| 推广渠道 | `校园大使` `供应商` `自然流量` `广告投放` | 推广来源 | 注册时记录 | 是 |
| 邀请来源 | `专家邀请` `管理员邀请` `无邀请` | 被邀请方式 | 邀请关系记录 | 是 |
| 岗位入口 | `[岗位名]入口` | 通过哪个岗位进入 | 岗位申请通过 | 否 |

#### 3.2.6 运营管理类 (operation)

| 标签组 | 标签值 | 说明 | 授予条件 | 是否互斥 |
|--------|--------|------|----------|----------|
| VIP等级 | `普通` `银牌` `金牌` `钻石` | 会员等级 | 积分/贡献计算 | 是 |
| 特殊标记 | `重点关注` `潜力专家` `核心骨干` `优先分配` | 运营标记 | 管理员手动 | 否 |
| 风险标记 | `疑似作弊` `多次返工` `投诉记录` `数据异常` | 风控标记 | 系统/人工判定 | 否 |
| 临时权限 | `临时质检` `临时审核` `临时小组长` | 临时授权 | 管理员授予 | 否 |
| 内部标记 | `内部员工` `测试账号` `合作伙伴` | 内部人员 | 管理员标记 | 否 |

### 3.3 标签授予节点

#### 专家全流程标签授予图

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
                                                       ↓
                                                   同时授予
                                                   领域职级
```

#### 节点1：注册

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 用户完成注册 | 注册渠道标签（官网/微信/APP） | source |
| 有邀请人 | 邀请来源标签（专家邀请/管理员邀请） | source |
| 有邀请岗位 | 岗位入口标签（待确认） | source |
| 有推广码 | 推广渠道标签 | source |

#### 节点2：资料填写与审核

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 填写学历 + 审核通过 | 学历标签（本科/硕士/博士） | identity |
| 教育经历 + 审核通过 | 学校类型标签（985/211/双一流） | identity |
| 选择领域 + 审核通过 | 领域标签（主领域/子领域） | qualification |
| 上传证书 + 审核通过 | 资质认证标签 | qualification |
| 填写语言技能 + 审核通过 | 语言能力标签 | qualification |

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

| 触发条件 | 授予标签 | 授予职级 | 标签分类 |
|----------|----------|----------|----------|
| AI审核通过 | 岗位配置的授予标签 | 岗位配置的领域职级 | 岗位配置 |
| 人工审核通过 | 岗位配置的授予标签 | 岗位配置的领域职级 | 岗位配置 |
| 审核通过 | `[岗位名]入口` | - | source |

#### 节点6：项目参与

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 加入项目 | `[项目名]参与者` | experience |
| 试标通过 | `[项目名]试标通过` | experience |
| 分配质检角色 | `质检员` | experience |
| 分配小组长 | `小组长` | experience |
| 完成项目考试 | `[项目名]认证` | qualification |

#### 节点7：任务执行

| 触发条件 | 授予标签 | 标签分类 |
|----------|----------|----------|
| 完成首题 | `任务类型标签`（如：对话标注） | experience |
| 累计100题 | `百题` | experience |
| 累计1000题 | `千题` | experience |
| 累计10000题 | `万题` | experience |
| 累计100000题 | `十万题` | experience |
| 质量评分 ≥ 4.5 | `S级` | behavior |
| 质量评分 ≥ 4.0 | `A级` | behavior |
| 质量评分 ≥ 3.5 | `B级` | behavior |
| 质量评分 ≥ 3.0 | `C级` | behavior |
| 质量评分 < 3.0 | `D级` | behavior |

#### 节点8：持续运营（系统定时任务）

| 触发条件 | 授予/更新标签 | 标签分类 |
|----------|--------------|----------|
| 7天内有活动 | `高活跃` | behavior |
| 30天内有活动 | `中活跃` | behavior |
| 30天无活动 | `低活跃` | behavior |
| 90天无活动 | `沉默用户` | behavior |
| 综合评估 | VIP等级更新 | operation |
| 管理员标记 | 特殊标记 | operation |
| 风控系统检测 | 风险标记 | operation |

### 3.4 标签数据库设计

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
    is_visible BOOLEAN DEFAULT true,     -- 是否对专家可见
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    
    -- 唯一约束
    CONSTRAINT uq_tag_category_group_value UNIQUE (category, tag_group, tag_value)
);

CREATE INDEX idx_tags_category ON tbl_tags(category) WHERE deleted = false;
CREATE INDEX idx_tags_group ON tbl_tags(tag_group) WHERE deleted = false;

COMMENT ON TABLE tbl_tags IS '标签定义表';
COMMENT ON COLUMN tbl_tags.category IS '标签大类: identity-身份认证, qualification-能力资质, behavior-行为表现, experience-项目经验, source-来源渠道, operation-运营管理';
COMMENT ON COLUMN tbl_tags.is_exclusive IS '是否互斥：同一tag_group下，专家只能拥有一个标签';

-- 触发器
CREATE TRIGGER trigger_update_time BEFORE UPDATE ON tbl_tags
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_tags TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_tags_tag_id_seq TO dlp;
```

```sql
-- 专家标签关联表
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
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

CREATE INDEX idx_expert_tags_user_id ON tbl_expert_tags(user_id) WHERE deleted = false;
CREATE INDEX idx_expert_tags_tag_id ON tbl_expert_tags(tag_id) WHERE deleted = false;
CREATE INDEX idx_expert_tags_status ON tbl_expert_tags(user_id, status) WHERE deleted = false;
CREATE INDEX idx_expert_tags_source ON tbl_expert_tags(grant_source, grant_source_id) WHERE deleted = false;

-- 唯一约束（同一用户同一标签只能有一条有效记录）
CREATE UNIQUE INDEX idx_expert_tags_unique ON tbl_expert_tags(user_id, tag_id) 
WHERE status = 'active' AND deleted = false;

COMMENT ON TABLE tbl_expert_tags IS '专家标签关联表';
COMMENT ON COLUMN tbl_expert_tags.grant_source IS '授予来源: system-系统自动, manual-人工授予, job-岗位审核, project-项目授予, exam-考试通过';

-- 触发器
CREATE TRIGGER trigger_update_time BEFORE UPDATE ON tbl_expert_tags
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_expert_tags TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_expert_tags_id_seq TO dlp;
```

```sql
-- 标签授予日志表
CREATE TABLE tbl_tag_grant_logs (
    log_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,                -- 专家用户ID
    tag_id INT NOT NULL,                 -- 标签ID
    
    -- 操作信息
    action VARCHAR(20) NOT NULL,         -- grant/revoke/update/expire
    
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

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_tag_grant_logs TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_tag_grant_logs_log_id_seq TO dlp;
```

---

## 四、岗位与职级/标签的关系

### 4.1 岗位发布时的配置

岗位发布时需要记录三个关键信息：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         岗位配置                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  岗位基本信息                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ 岗位名称: 金融数据标注专家                                        │   │
│  │ 推广奖金: 100元                                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ① 需求来源项目（推广奖励关联）                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ 关联项目: [金融研报分析项目]                                      │   │
│  │ 奖励条件: 必须在此项目完成2道题才发放推广奖金                       │   │
│  │ 目的: 避免高奖金拉人却做便宜题的套利行为                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ② 审核通过授予的职级（可多个领域）                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ 授予规则:                                                        │   │
│  │   规则1: 经济金融/投资 领域                                       │   │
│  │          简历匹配度 ≥ 80% → 授予 L2                               │   │
│  │          简历匹配度 ≥ 60% → 授予 L1                               │   │
│  │   规则2: 代码/Python 领域                                         │   │
│  │          有相关经验 → 授予 L1                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ③ 审核通过授予的标签                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ 授予标签:                                                        │   │
│  │   • [金融标注岗]入口  (source/岗位入口)                           │   │
│  │   • 金融项目准入资格  (qualification/项目认证)                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4.2 岗位配置数据结构

```sql
-- 岗位职级授予配置表
CREATE TABLE tbl_job_level_grants (
    id SERIAL PRIMARY KEY,
    job_id INT NOT NULL,                 -- 岗位ID
    
    -- 关联的需求项目（用于推广奖励）
    required_project_id INT,             -- 必须完成的项目ID
    required_task_count INT DEFAULT 2,   -- 需完成的题目数
    
    -- 领域信息
    domain_l1_id INT NOT NULL,
    domain_l2_id INT,
    domain_l3_id INT,
    
    -- 授予规则
    grant_level VARCHAR(20) NOT NULL,    -- 授予的职级: L1/L2
    grant_condition JSONB,               -- 授予条件（AI评估用）
    grant_priority INT DEFAULT 0,        -- 优先级（用于多规则匹配）
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

CREATE INDEX idx_job_level_grants_job ON tbl_job_level_grants(job_id) WHERE deleted = false;

COMMENT ON TABLE tbl_job_level_grants IS '岗位职级授予配置表';
COMMENT ON COLUMN tbl_job_level_grants.grant_condition IS '授予条件JSON，如: {"type":"score_threshold","field":"resume_match_score","operator":">=","value":80}';

-- 触发器
CREATE TRIGGER trigger_update_time BEFORE UPDATE ON tbl_job_level_grants
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_job_level_grants TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_job_level_grants_id_seq TO dlp;
```

```sql
-- 岗位标签配置表
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

-- 触发器
CREATE TRIGGER trigger_update_time BEFORE UPDATE ON tbl_job_tag_configs
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_job_tag_configs TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_job_tag_configs_id_seq TO dlp;
```

### 4.3 推广奖励优化

```
推广奖励流程（优化后）
═══════════════════════════════════════════════════════════════════════════

  专家A                              专家B
    │                                  │
    │  ① 生成邀请链接                  │
    │  (携带岗位ID)                    │
    │         ────────────────────►   │
    │                                  │
    │                              ② 注册
    │                              (记录邀请人+岗位)
    │                                  │
    │                              ③ 申请岗位
    │                                  │
    │                              ④ 审核通过
    │                              授予职级: 金融投资-L2, 代码-L1
    │                              授予标签: [金融标注岗]入口
    │                                  │
    │                              ⑤ 必须在指定项目完成2道题
    │                              （岗位关联的需求项目）
    │                                  │
    │                                  │ ← 避免套利：
    │                                  │    不能通过高奖金岗位进来
    │                                  │    却去做其他便宜题
    │                                  │
    │  ⑥ 发放推广奖金                  │
    │◄─────────────────────────────   │
    │                                  │

```

---

## 五、项目准入配置

### 5.1 项目环节与节点

```
项目工作流程
═══════════════════════════════════════════════════════════════════════════

  ┌──────────────────────────────────────────────────────────────────────┐
  │                           项目工作流                                  │
  │                                                                      │
  │   ┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐   │
  │   │  标注   │ ──►  │  质检1  │ ──►  │  质检2  │ ──►  │  终审   │   │
  │   │(labeling)│      │  (qa1)  │      │  (qa2)  │      │ (final) │   │
  │   └────┬────┘      └────┬────┘      └────┬────┘      └────┬────┘   │
  │        │                │                │                │        │
  │        ▼                ▼                ▼                ▼        │
  │   ┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐   │
  │   │ 职级要求 │      │ 职级要求 │      │ 职级要求 │      │ 职级要求 │   │
  │   │ L1及以上 │      │ L2及以上 │      │ L3及以上 │      │ 专家委员 │   │
  │   │   AND   │      │   AND   │      │   OR    │      │   OR    │   │
  │   │ 标签条件 │      │ 标签条件 │      │ 标签条件 │      │ 标签条件 │   │
  │   └─────────┘      └─────────┘      └─────────┘      └─────────┘   │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘

```

### 5.2 准入条件配置示例

```json
{
  "project_id": 1,
  "project_name": "金融研报分析项目",
  
  "stage_requirements": {
    "labeling": {
      "description": "标注环节准入要求",
      "level_requirement": {
        "domain": {"l1": 5, "l2": 12},
        "min_level": "L1",
        "operator": "gte"
      },
      "tag_requirement": {
        "must": [
          {"category": "identity", "group": "实名认证", "value": "已实名"}
        ],
        "should": [
          {"category": "behavior", "group": "质量评级", "values": ["S级", "A级", "B级"]}
        ],
        "must_not": [
          {"category": "operation", "group": "风险标记", "value": "疑似作弊"}
        ]
      },
      "combine_logic": "AND"
    },
    
    "qa1": {
      "description": "一级质检准入要求",
      "level_requirement": {
        "domain": {"l1": 5, "l2": 12},
        "min_level": "L2",
        "operator": "gte"
      },
      "tag_requirement": {
        "must": [
          {"category": "experience", "group": "项目角色", "value": "质检员"}
        ],
        "should": [
          {"category": "behavior", "group": "质量评级", "values": ["S级", "A级"]}
        ]
      },
      "combine_logic": "AND"
    },
    
    "qa2": {
      "description": "二级质检准入要求",
      "level_requirement": {
        "domain": {"l1": 5, "l2": 12},
        "min_level": "L3",
        "operator": "gte"
      },
      "tag_requirement": {
        "should": [
          {"category": "operation", "group": "特殊标记", "value": "核心骨干"}
        ]
      },
      "combine_logic": "OR"
    },
    
    "final": {
      "description": "终审准入要求",
      "level_requirement": {
        "domain": {"l1": 5},
        "min_level": "COMMITTEE",
        "operator": "gte"
      },
      "tag_requirement": null,
      "combine_logic": null
    }
  }
}
```

### 5.3 项目准入配置表

```sql
-- 项目环节准入配置表
CREATE TABLE tbl_project_stage_access (
    id SERIAL PRIMARY KEY,
    project_id INT NOT NULL,
    
    -- 环节信息
    stage VARCHAR(50) NOT NULL,          -- labeling/qa1/qa2/final/...
    stage_name VARCHAR(100),             -- 环节名称
    
    -- 职级要求
    level_domain_l1_id INT,              -- 要求的领域
    level_domain_l2_id INT,
    level_domain_l3_id INT,
    min_level_code VARCHAR(20),          -- 最低职级要求: L1/L2/L3/COMMITTEE
    
    -- 标签要求（JSON）
    tag_requirements JSONB,              -- 标签条件
    
    -- 组合逻辑
    combine_logic VARCHAR(10) DEFAULT 'AND',  -- AND/OR
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

CREATE INDEX idx_project_stage_access ON tbl_project_stage_access(project_id, stage) WHERE deleted = false;

COMMENT ON TABLE tbl_project_stage_access IS '项目环节准入配置表';
COMMENT ON COLUMN tbl_project_stage_access.combine_logic IS '组合逻辑: AND-职级和标签都要满足, OR-满足其一即可';

-- 触发器
CREATE TRIGGER trigger_update_time BEFORE UPDATE ON tbl_project_stage_access
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_project_stage_access TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_project_stage_access_id_seq TO dlp;
```

### 5.4 准入判断逻辑

```
项目准入判断逻辑
═══════════════════════════════════════════════════════════════════════════

  专家申请进入项目某环节
           │
           ▼
  ┌─────────────────────────────────────┐
  │     检查职级要求（主轨）              │
  │                                     │
  │  该专家在对应领域的职级 ≥ 要求职级?   │
  └───────────────┬─────────────────────┘
                  │
         ┌───────┴───────┐
         │               │
    满足 ▼          不满足 ▼
         │               │
         │    ┌──────────────────────────┐
         │    │ 组合逻辑 = OR ?           │
         │    └─────────┬────────────────┘
         │              │
         │      ┌───────┴───────┐
         │      │               │
         │   是 ▼            否 ▼
         │      │               │
         │      │          拒绝准入
         │      │               
         │      ▼               
  ┌──────┴──────────────────────┐
  │     检查标签要求（辅轨）       │
  │                             │
  │  must: 必须全部满足          │
  │  should: 优先级加分          │
  │  must_not: 不能有任何一个    │
  └─────────────┬───────────────┘
                │
       ┌────────┴────────┐
       │                 │
   满足 ▼            不满足 ▼
       │                 │
   允许准入         ┌────────────────┐
                   │ 组合逻辑 = AND? │
                   └───────┬────────┘
                           │
                   ┌───────┴───────┐
                   │               │
                是 ▼            否 ▼
                   │               │
              拒绝准入         允许准入

```

---

## 六、职级成长体系

### 6.1 成长要素

```
                        职级成长评估维度
═══════════════════════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────────────────────┐
  │                                                                     │
  │   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐       │
  │   │  做题量   │   │  通过率   │   │ Issue数  │   │  违规记录 │       │
  │   │          │   │          │   │          │   │          │       │
  │   │  权重30% │   │  权重35% │   │  权重20% │   │  权重15% │       │
  │   └────┬─────┘   └────┬─────┘   └────┬─────┘   └────┬─────┘       │
  │        │              │              │              │              │
  │        └──────────────┴──────────────┴──────────────┘              │
  │                              │                                      │
  │                              ▼                                      │
  │                    ┌─────────────────┐                             │
  │                    │    AI 评估      │                             │
  │                    │                 │                             │
  │                    │  综合得分计算    │                             │
  │                    │  晋升/降级建议   │                             │
  │                    └────────┬────────┘                             │
  │                             │                                      │
  │                             ▼                                      │
  │                    ┌─────────────────┐                             │
  │                    │    人工审核     │                             │
  │                    │   (L3及以上)    │                             │
  │                    └─────────────────┘                             │
  │                                                                     │
  └─────────────────────────────────────────────────────────────────────┘
```

### 6.2 晋升规则（自然语言定义）

```yaml
# L1 → L2 晋升条件
晋升条件_L1_to_L2:
  必须条件:
    - 在该领域累计完成 500 道及以上题目
    - 近30天题目通过率不低于 90%
    - 近30天无严重 Issue（影响等级为高的Issue）
    - 无任何违规违纪记录
  
  加分项:
    - 连续7天日均完成 20 题以上，+5分
    - 被评为优质答案 10 次以上，+5分
    - 主动报告问题被采纳 5 次以上，+3分
  
  AI评估:
    - 系统每周自动扫描符合条件的专家
    - 生成晋升建议报告
    - 自动执行晋升（无需人工审核）
    - 同步更新标签：授予"千题"等产出等级标签

# L2 → L3 晋升条件  
晋升条件_L2_to_L3:
  必须条件:
    - 在该领域累计完成 2000 道及以上题目
    - 近90天题目通过率不低于 95%
    - 近90天无任何 Issue
    - 无任何违规违纪记录
    - 作为质检员的准确率不低于 92%
  
  加分项:
    - 培训新人满意度 90% 以上，+10分
    - 参与标准制定，+5分
    - 发现重大问题，+5分
  
  AI评估:
    - 系统每月自动扫描符合条件的专家
    - 生成晋升建议报告
    - 需人工审核确认后执行晋升
    - 同步更新标签：授予"核心骨干"等特殊标记

# L3 → 专家委员会
晋升条件_L3_to_COMMITTEE:
  必须条件:
    - L3职级满 6 个月
    - 在该领域有突出贡献（人工判定）
    - 愿意参与平台治理
  
  流程:
    - 平台发起邀请
    - 本人同意
    - 现有委员会成员投票（过半数通过）
```

### 6.3 降级与处罚规则

```yaml
# 职级降级条件
降级条件:
  自动降级:
    - 连续30天题目通过率低于 70%，降一级
    - 月度 Issue 数超过 10 个，降一级
    - 90天内无任何做题记录，冻结职级
    - 同步更新标签：质量评级降级
  
  人工降级:
    - 发现作弊行为，直接降至 L1 或移除职级
    - 恶意行为，直接移除职级
    - 泄露数据，移除职级并加入黑名单
    - 同步更新标签：添加风险标记

# 职级冻结
冻结条件:
  - 连续90天无活动，职级冻结
  - 被投诉正在调查中，临时冻结
  - 重新激活需完成考核
  - 同步更新标签：活跃度标签更新为"沉默用户"

# 违规处罚
违规处罚:
  轻度违规（如：迟到、漏标）:
    - 扣除当期积分
    - 不影响职级
  
  中度违规（如：质量问题、多次返工）:
    - 限制领取任务
    - 职级考察期（30天）
    - 同步更新标签：添加"多次返工"风险标记
  
  重度违规（如：作弊、泄露数据）:
    - 立即降级或移除职级
    - 加入黑名单
    - 扣除未发放收入
    - 同步更新标签：添加"疑似作弊"风险标记
```

---

## 七、荣誉墙与激励

### 7.1 专家工作台荣誉展示

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         专家工作台 - 荣誉墙                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     我的职级                                     │   │
│  │                                                                  │   │
│  │   🏆 经济金融/投资           ⭐ 代码/Python         🌱 医疗健康  │   │
│  │      L3 - 大师                 L2 - 精英              L1 - 新锐  │   │
│  │                                                                  │   │
│  │   距离下一级还需:                                                 │   │
│  │   • 经济金融: 已达最高可自动晋升职级，等待专家委员会邀请           │   │
│  │   • 代码: 还需 500 题，通过率需保持 95%+                          │   │
│  │   • 医疗: 还需 200 题，通过率需达到 90%+                          │   │
│  │                                                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     我的标签                                     │   │
│  │                                                                  │   │
│  │   [已实名] [硕士] [985] [CFA] [高活跃] [S级质量] [万题达人]       │   │
│  │   [金融项目准入资格] [质检员] [核心骨干]                          │   │
│  │                                                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     成就徽章                                     │   │
│  │                                                                  │   │
│  │   🎯 万题达人    📈 连续优秀    🔍 问题猎人    ⚡ 效率之星       │   │
│  │   🌟 质检达人    💎 钻石会员    🏅 早期贡献者                     │   │
│  │                                                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     本周排行                                     │   │
│  │                                                                  │   │
│  │   🥇 张三  L3  2,345题    🥈 李四  L2  1,987题    🥉 王五  L2  1,654题   │
│  │                                                                  │   │
│  │   我的排名: 第 15 名                                              │   │
│  │                                                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 7.2 荣誉体系数据结构

```sql
-- 专家成就徽章表
CREATE TABLE tbl_expert_achievements (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    
    -- 成就信息
    achievement_code VARCHAR(50) NOT NULL,   -- 成就代码
    achievement_name VARCHAR(100) NOT NULL,  -- 成就名称
    achievement_icon VARCHAR(100),           -- 成就图标
    achievement_desc TEXT,                   -- 成就描述
    
    -- 获得信息
    earned_at TIMESTAMP NOT NULL,
    earned_data JSONB,                       -- 获得时的数据快照
    
    -- 通用字段
    deleted BOOLEAN NOT NULL DEFAULT false,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

CREATE INDEX idx_expert_achievements_user ON tbl_expert_achievements(user_id) WHERE deleted = false;

-- 成就定义表
CREATE TABLE tbl_achievement_definitions (
    achievement_code VARCHAR(50) PRIMARY KEY,
    achievement_name VARCHAR(100) NOT NULL,
    achievement_icon VARCHAR(100),
    achievement_desc TEXT,
    
    -- 获得条件（自然语言 + JSON规则）
    condition_desc TEXT,                     -- 条件描述
    condition_rule JSONB,                    -- 条件规则（AI评估用）
    
    -- 展示
    sort_order INT DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

-- 预置成就
INSERT INTO tbl_achievement_definitions VALUES
('TASK_1000', '千题达人', '🎯', '累计完成1000道题目', '累计完成题目数达到1000', '{"type":"task_count","value":1000}', 10, true, now()),
('TASK_10000', '万题达人', '🏅', '累计完成10000道题目', '累计完成题目数达到10000', '{"type":"task_count","value":10000}', 20, true, now()),
('QUALITY_STREAK', '连续优秀', '📈', '连续30天通过率95%+', '连续30天保持95%以上通过率', '{"type":"quality_streak","days":30,"rate":0.95}', 30, true, now()),
('ISSUE_HUNTER', '问题猎人', '🔍', '发现10个有效问题', '报告的问题被采纳10次', '{"type":"issue_report","value":10}', 40, true, now()),
('SPEED_STAR', '效率之星', '⚡', '单日完成100道题', '单日完成题目数达到100', '{"type":"daily_task","value":100}', 50, true, now()),
('QA_MASTER', '质检达人', '🌟', '质检准确率连续30天95%+', '作为质检员准确率连续30天保持95%以上', '{"type":"qa_accuracy_streak","days":30,"rate":0.95}', 60, true, now()),
('EARLY_BIRD', '早期贡献者', '🏅', '平台前1000名注册用户', '注册序号在前1000名', '{"type":"early_user","value":1000}', 70, true, now());

-- 授权
GRANT ALL PRIVILEGES ON TABLE tbl_expert_achievements TO dlp;
GRANT USAGE, SELECT ON SEQUENCE tbl_expert_achievements_id_seq TO dlp;
GRANT ALL PRIVILEGES ON TABLE tbl_achievement_definitions TO dlp;
```

---

## 八、系统运转机制

### 8.1 AI Native 设计理念

```
                        AI + 人工 协同运转
═══════════════════════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────────────────────┐
  │                                                                     │
  │                         自然语言规则                                 │
  │                              │                                      │
  │                              ▼                                      │
  │  ┌──────────────────────────────────────────────────────────────┐  │
  │  │                       AI 评估引擎                             │  │
  │  │                                                              │  │
  │  │  输入:                                                       │  │
  │  │    • 专家做题数据                                            │  │
  │  │    • 质量评分数据                                            │  │
  │  │    • Issue记录                                               │  │
  │  │    • 违规记录                                                │  │
  │  │    • 自然语言定义的职级规则                                   │  │
  │  │                                                              │  │
  │  │  输出:                                                       │  │
  │  │    • 职级晋升建议（含理由）                                  │  │
  │  │    • 职级降级预警（含理由）                                  │  │
  │  │    • 标签更新建议                                            │  │
  │  │    • 风险标记建议                                            │  │
  │  │                                                              │  │
  │  └──────────────────────────────────────────────────────────────┘  │
  │                              │                                      │
  │                              ▼                                      │
  │  ┌──────────────────────────────────────────────────────────────┐  │
  │  │                      执行层                                   │  │
  │  │                                                              │  │
  │  │   L1→L2: 自动执行                                            │  │
  │  │   L2→L3: 人工确认                                            │  │
  │  │   L3→专委: 委员会投票                                        │  │
  │  │   降级: 人工确认                                             │  │
  │  │   标签更新: 大部分自动，风险标签人工确认                      │  │
  │  │                                                              │  │
  │  └──────────────────────────────────────────────────────────────┘  │
  │                                                                     │
  └─────────────────────────────────────────────────────────────────────┘
```

### 8.2 定时任务设计

```yaml
定时任务:
  
  # 每日任务
  daily_level_check:
    schedule: "0 2 * * *"  # 每天凌晨2点
    description: 每日职级和标签检查
    actions:
      - 计算所有专家的当日数据
      - 检查是否触发降级条件
      - 更新活跃度标签（高活跃/中活跃/低活跃/沉默）
      - 更新质量评级标签
      - 生成预警通知
  
  # 每周任务
  weekly_promotion_scan:
    schedule: "0 3 * * 1"  # 每周一凌晨3点
    description: 每周晋升和标签扫描
    actions:
      - 扫描符合 L1→L2 晋升条件的专家
      - 自动执行晋升
      - 更新产出等级标签（百题/千题/万题）
      - 检查成就徽章获得条件
      - 发送晋升通知
      - 更新荣誉墙
  
  # 每月任务
  monthly_senior_review:
    schedule: "0 4 1 * *"  # 每月1日凌晨4点
    description: 每月资深专家评审
    actions:
      - 扫描符合 L2→L3 晋升条件的专家
      - 生成评审报告
      - 推送给管理员审核
      - 更新VIP等级标签
  
  # 实时任务
  realtime_violation_check:
    trigger: "on_issue_created"
    description: 实时违规检查
    actions:
      - 检查是否触发即时降级
      - 检查是否需要冻结职级
      - 添加风险标签
      - 发送预警通知
  
  # 节点触发任务
  on_task_completed:
    trigger: "on_task_completed"
    description: 任务完成时的标签更新
    actions:
      - 检查是否达到产出等级阈值
      - 更新任务类型标签
      - 检查成就获得条件
```

---

## 九、初始化数据

### 9.1 职级初始化

```sql
-- 初始化通用职级模板（适用于所有领域）
INSERT INTO tbl_domain_levels (domain_l1_id, level_code, level_name, level_order, honor_icon, honor_title, min_task_count, min_pass_rate, require_manual_review) VALUES
(NULL, 'L1', '初级专家', 1, '🌱', '新锐', 0, 0, false),
(NULL, 'L2', '进阶专家', 2, '⭐', '精英', 500, 90.00, false),
(NULL, 'L3', '资深专家', 3, '🏆', '大师', 2000, 95.00, true),
(NULL, 'COMMITTEE', '专家委员会', 4, '👑', '专家委员', NULL, NULL, true);
```

### 9.2 标签初始化

```sql
-- 身份认证类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system, is_exclusive, is_visible) VALUES
('identity', '实名认证', '已实名', true, true, true),
('identity', '学历认证', '高中及以下', true, true, true),
('identity', '学历认证', '大专', true, true, true),
('identity', '学历认证', '本科', true, true, true),
('identity', '学历认证', '硕士', true, true, true),
('identity', '学历认证', '博士', true, true, true),
('identity', '学历认证', '博士后', true, true, true),
('identity', '学校类型', '985', true, false, true),
('identity', '学校类型', '211', true, false, true),
('identity', '学校类型', '双一流', true, false, true),
('identity', '学校类型', '海外QS100', true, false, true),
('identity', '学校类型', '海外名校', true, false, true);

-- 能力资质类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system, is_exclusive, is_visible) VALUES
('qualification', '平台认证', '初级标注师', true, true, true),
('qualification', '平台认证', '中级标注师', true, true, true),
('qualification', '平台认证', '高级标注师', true, true, true);

-- 行为表现类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system, is_exclusive, is_visible) VALUES
('behavior', '活跃度', '高活跃', true, true, true),
('behavior', '活跃度', '中活跃', true, true, true),
('behavior', '活跃度', '低活跃', true, true, true),
('behavior', '活跃度', '沉默用户', true, true, true),
('behavior', '质量评级', 'S级', true, true, true),
('behavior', '质量评级', 'A级', true, true, true),
('behavior', '质量评级', 'B级', true, true, true),
('behavior', '质量评级', 'C级', true, true, true),
('behavior', '质量评级', 'D级', true, true, true),
('behavior', '效率评级', '高效', true, true, true),
('behavior', '效率评级', '正常', true, true, true),
('behavior', '效率评级', '偏慢', true, true, true),
('behavior', '信用等级', '优秀', true, true, true),
('behavior', '信用等级', '良好', true, true, true),
('behavior', '信用等级', '一般', true, true, true),
('behavior', '信用等级', '较差', true, true, true);

-- 项目经验类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system, is_exclusive, is_visible) VALUES
('experience', '项目角色', '标注员', true, false, true),
('experience', '项目角色', '质检员', true, false, true),
('experience', '项目角色', '审核员', true, false, true),
('experience', '项目角色', '小组长', true, false, true),
('experience', '产出等级', '百题', true, true, true),
('experience', '产出等级', '千题', true, true, true),
('experience', '产出等级', '万题', true, true, true),
('experience', '产出等级', '十万题', true, true, true);

-- 来源渠道类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system, is_exclusive, is_visible) VALUES
('source', '注册渠道', '官网', true, true, false),
('source', '注册渠道', '微信', true, true, false),
('source', '注册渠道', 'APP', true, true, false),
('source', '注册渠道', 'H5', true, true, false),
('source', '推广渠道', '校园大使', true, true, false),
('source', '推广渠道', '供应商', true, true, false),
('source', '推广渠道', '自然流量', true, true, false),
('source', '邀请来源', '专家邀请', true, true, false),
('source', '邀请来源', '管理员邀请', true, true, false),
('source', '邀请来源', '无邀请', true, true, false);

-- 运营管理类
INSERT INTO tbl_tags (category, tag_group, tag_value, is_system, is_exclusive, is_visible) VALUES
('operation', 'VIP等级', '普通', true, true, true),
('operation', 'VIP等级', '银牌', true, true, true),
('operation', 'VIP等级', '金牌', true, true, true),
('operation', 'VIP等级', '钻石', true, true, true),
('operation', '特殊标记', '重点关注', true, false, false),
('operation', '特殊标记', '潜力专家', true, false, true),
('operation', '特殊标记', '核心骨干', true, false, true),
('operation', '特殊标记', '优先分配', true, false, false),
('operation', '风险标记', '疑似作弊', true, false, false),
('operation', '风险标记', '多次返工', true, false, false),
('operation', '风险标记', '投诉记录', true, false, false),
('operation', '风险标记', '数据异常', true, false, false),
('operation', '内部标记', '内部员工', true, false, false),
('operation', '内部标记', '测试账号', true, false, false),
('operation', '内部标记', '合作伙伴', true, false, false);
```

---

## 十、实施计划

### 10.1 阶段规划

| 阶段 | 内容 | 优先级 |
|------|------|--------|
| **P0-职级基础** | 职级数据表创建、基础CRUD | P0 |
| **P0-标签基础** | 标签数据表创建（复用+扩展）、基础CRUD | P0 |
| **P0-岗位配置** | 岗位职级配置、岗位标签配置、审核授予 | P0 |
| **P0-项目准入** | 项目准入配置、职级+标签校验 | P0 |
| **P1-标签自动化** | 全流程节点标签自动授予 | P1 |
| **P1-职级成长** | 职级成长评估、AI评估引擎 | P1 |
| **P1-推广优化** | 推广奖励优化、项目关联 | P1 |
| **P2-荣誉系统** | 荣誉墙、成就系统 | P2 |
| **P2-运营看板** | 标签统计分析、职级分布分析 | P2 |

### 10.2 兼容性考虑

- 现有 `tbl_users.tags` 字段保留，逐步迁移到新的 `tbl_expert_tags` 表
- 现有 `tbl_project_jobs` 关系保留，新增职级授予配置
- 现有岗位申请流程保留，增加职级+标签授予逻辑
- 现有标签管理功能保留，扩展分类体系

---

## 十一、总结

### 11.1 双轨制优势

| 维度 | 优势 |
|------|------|
| **清晰的成长路径** | 职级体系让专家知道如何从 L1 成长到专家委员会 |
| **完整的专家画像** | 标签体系6大类全覆盖，构建精细化画像 |
| **精准的能力匹配** | 项目可以精确要求"金融L2 + 已实名 + S级质量" |
| **灵活的特例处理** | 标签处理各种 corner case，不破坏职级体系 |
| **可控的推广成本** | 岗位关联项目，必须完成指定任务才发奖金 |
| **AI Native** | 规则用自然语言定义，AI + 人工协同运转 |
| **激励驱动** | 荣誉墙让专家有成就感，愿意往上走 |
| **可追溯** | 职级变更和标签授予都有完整日志 |

### 11.2 核心设计要点

1. **职级 = 领域 × 等级**：每个专家可以有多个领域的职级
2. **标签 = 6大类全覆盖**：身份/能力/行为/经验/来源/运营
3. **先验 vs 后验**：L1/L2可先验授予，L3/专委会必须后验
4. **岗位三绑定**：绑定需求项目 + 绑定授予职级 + 绑定授予标签
5. **项目按职级+标签准入**：AND/OR 灵活组合
6. **全流程自动授予**：标签在注册/审核/考试/任务等节点自动授予
7. **成长靠数据说话**：做题量、通过率、Issue、违规
8. **AI + 人工**：低级自动，高级人工

---

> 文档版本：v2.0  
> 基于：职级体系 + 完整标签体系 融合设计  
> 日期：2026-01-23
