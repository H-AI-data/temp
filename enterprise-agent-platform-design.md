# 企业级 AI Agent 平台架构设计文档

> 版本：v1.0  
> 日期：2026-02-10  
> 技术栈：OpenManus + Letta + LiteLLM + MCP

---

## 目录

1. [项目概述](#1-项目概述)
2. [整体架构](#2-整体架构)
3. [核心组件详解](#3-核心组件详解)
4. [上下文管理设计](#4-上下文管理设计)
5. [模型管理设计](#5-模型管理设计)
6. [Tool/Skill/MCP 管理设计](#6-toolskillmcp-管理设计)
7. [场景实现详解](#7-场景实现详解)
8. [API 设计](#8-api-设计)
9. [数据库设计](#9-数据库设计)
10. [部署架构](#10-部署架构)
11. [开发路线图](#11-开发路线图)

---

## 1. 项目概述

### 1.1 项目目标

构建一个企业级 AI Agent 平台，让业务人员通过自然语言完成复杂的数据处理、分析和报告生成任务。

### 1.2 核心能力

| 能力 | 说明 |
|------|------|
| **自然语言交互** | 用户通过对话描述需求，Agent 自动理解并执行 |
| **企业知识库** | 整合代码库、文档、数据库结构等企业上下文 |
| **自主任务执行** | Plan → Execute → Observe → Replan 循环 |
| **过程可视化** | 展示每步执行过程和结果 |
| **多场景支持** | 数据导出、报告生成、数据分析等 |

### 1.3 技术选型

| 层级 | 选型 | 理由 |
|------|------|------|
| **Agent 框架** | OpenManus | Python、MCP 原生支持、代码轻量 |
| **上下文管理** | Letta (MemGPT) | 分层记忆、Agent 自管理、向量存储 |
| **模型管理** | LiteLLM | 开源、100+ 模型、路由/Fallback |
| **工具协议** | MCP | 行业标准、工具互通 |
| **向量数据库** | pgvector | 与 PostgreSQL 统一 |
| **前端** | Vue 3 | 与现有项目一致 |

---

## 2. 整体架构

### 2.1 系统架构图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              用户层                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      Web 前端 (Vue 3)                            │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │   │
│  │  │ 对话界面  │  │ 任务面板  │  │ 结果展示  │  │ 管理后台         │ │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────┤
│                              网关层                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    API Gateway (FastAPI)                         │   │
│  │  - 认证鉴权 (JWT)                                                │   │
│  │  - 请求路由                                                      │   │
│  │  - WebSocket 连接管理                                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────┤
│                             Agent 层                                     │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    OpenManus Agent Core                          │   │
│  │                                                                   │   │
│  │   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │   │
│  │   │ Planner │ →  │Executor │ →  │Observer │ →  │Replanner│     │   │
│  │   │  规划器  │    │  执行器  │    │  观察器  │    │ 重规划器 │     │   │
│  │   └─────────┘    └─────────┘    └─────────┘    └─────────┘     │   │
│  │        ↑                                              │          │   │
│  │        └──────────────────────────────────────────────┘          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────┤
│                             能力层                                       │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────────┐   │
│  │  上下文管理    │  │   模型管理     │  │     Tool/Skill 管理       │   │
│  │   (Letta)     │  │  (LiteLLM)    │  │        (MCP)             │   │
│  │               │  │               │  │                           │   │
│  │ ┌───────────┐ │  │ ┌───────────┐ │  │ ┌─────────────────────┐  │   │
│  │ │Core Memory│ │  │ │Model Router│ │  │ │   Tool Registry     │  │   │
│  │ │ 核心记忆   │ │  │ │ 模型路由   │ │  │ │   工具注册表         │  │   │
│  │ └───────────┘ │  │ └───────────┘ │  │ └─────────────────────┘  │   │
│  │ ┌───────────┐ │  │ ┌───────────┐ │  │ ┌─────────────────────┐  │   │
│  │ │Recall Mem │ │  │ │ Fallback  │ │  │ │   MCP Servers       │  │   │
│  │ │ 召回记忆   │ │  │ │ 故障转移   │ │  │ │   MCP 服务端        │  │   │
│  │ └───────────┘ │  │ └───────────┘ │  │ └─────────────────────┘  │   │
│  │ ┌───────────┐ │  │ ┌───────────┐ │  │ ┌─────────────────────┐  │   │
│  │ │Archival   │ │  │ │Cost Track │ │  │ │   Skill Library     │  │   │
│  │ │ 档案记忆   │ │  │ │ 成本追踪   │ │  │ │   技能库            │  │   │
│  │ └───────────┘ │  │ └───────────┘ │  │ └─────────────────────┘  │   │
│  └───────────────┘  └───────────────┘  └───────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────┤
│                             执行层                                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │代码执行   │ │数据库查询 │ │文件操作   │ │API 调用   │ │浏览器自动化   │  │
│  │(Sandbox) │ │(SQL)     │ │(File)    │ │(HTTP)    │ │(Playwright)  │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────────┘  │
├─────────────────────────────────────────────────────────────────────────┤
│                             数据层                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌────────────┐  │
│  │  PostgreSQL  │  │   pgvector   │  │    Redis     │  │    MinIO   │  │
│  │  业务数据     │  │  向量存储     │  │  缓存/队列    │  │  文件存储   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └────────────┘  │
├─────────────────────────────────────────────────────────────────────────┤
│                           企业知识源                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │ 代码仓库  │ │ 飞书文档  │ │ 数据库   │ │ 标注数据  │ │ 会议视频     │  │
│  │  (Git)   │ │ (Feishu) │ │(Schema) │ │ (JSON)   │ │(Transcript) │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 数据流架构

```
用户输入
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│                        1. 意图理解                               │
│  - 解析用户自然语言                                              │
│  - 识别任务类型（数据导出/报告生成/数据分析等）                    │
│  - 提取关键参数                                                  │
└─────────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│                        2. 上下文检索                             │
│  - 从 Letta 核心记忆获取用户画像和偏好                           │
│  - 从向量库检索相关企业知识（文档、代码、数据结构）                │
│  - 构建增强 Prompt                                               │
└─────────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│                        3. 任务规划                               │
│  - LLM 生成执行计划                                              │
│  - 分解为多个子任务                                              │
│  - 确定工具调用顺序                                              │
└─────────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│                        4. 循环执行                               │
│                                                                  │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│   │ 执行步骤  │ →  │ 观察结果  │ →  │ 判断继续  │                  │
│   └──────────┘    └──────────┘    └──────────┘                  │
│        │                               │                         │
│        │         ┌──────────┐          │                         │
│        └─────────│ 重新规划  │←─────────┘                         │
│                  └──────────┘                                    │
│                                                                  │
│   每步执行结果实时推送到前端                                       │
└─────────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│                        5. 结果输出                               │
│  - 汇总所有步骤结果                                              │
│  - 生成最终输出（报告/图表/文件）                                 │
│  - 更新记忆（记住本次交互）                                       │
└─────────────────────────────────────────────────────────────────┘
    │
    ▼
用户获得结果 + 完整执行过程
```

---

## 3. 核心组件详解

### 3.1 Agent Core（基于 OpenManus）

```python
# agent/enterprise_agent.py
from app.agent.manus import Manus
from app.tool.tool_collection import ToolCollection
from letta import create_client
from litellm import completion

class EnterpriseAgent(Manus):
    """企业级 Agent，集成 Letta 记忆和 LiteLLM 模型管理"""
    
    def __init__(self, tenant_id: str, user_id: str):
        super().__init__()
        self.tenant_id = tenant_id
        self.user_id = user_id
        
        # 初始化 Letta 记忆系统
        self.letta_client = create_client()
        self.memory_agent = self._init_memory_agent()
        
        # 初始化工具集
        self.tools = self._load_tenant_tools()
        
        # 执行历史（用于过程展示）
        self.execution_history = []
    
    def _init_memory_agent(self):
        """初始化 Letta 记忆 Agent"""
        # 加载用户画像
        user_profile = self._load_user_profile()
        
        return self.letta_client.create_agent(
            name=f"enterprise_agent_{self.user_id}",
            memory=ChatMemory(
                persona=self._get_system_persona(),
                human=user_profile
            ),
            embedding_config=EmbeddingConfig(
                embedding_endpoint_type="openai",
                embedding_model="text-embedding-3-small"
            )
        )
    
    async def execute_task(self, user_input: str, websocket=None):
        """执行用户任务，返回过程和结果"""
        
        # 1. 检索相关上下文
        context = await self._retrieve_context(user_input)
        
        # 2. 生成执行计划
        plan = await self._generate_plan(user_input, context)
        await self._emit_step(websocket, "plan", plan)
        
        # 3. 循环执行
        for step in plan.steps:
            # 执行步骤
            result = await self._execute_step(step)
            
            # 记录历史
            self.execution_history.append({
                "step": step,
                "result": result,
                "timestamp": datetime.utcnow()
            })
            
            # 实时推送
            await self._emit_step(websocket, "execution", {
                "step": step,
                "result": result
            })
            
            # 判断是否需要重新规划
            if self._need_replan(result):
                plan = await self._replan(plan, result)
                await self._emit_step(websocket, "replan", plan)
        
        # 4. 生成最终结果
        final_result = await self._generate_final_result()
        
        # 5. 更新记忆
        await self._update_memory(user_input, final_result)
        
        return {
            "result": final_result,
            "execution_history": self.execution_history
        }
```

### 3.2 执行步骤推送（WebSocket）

```python
# api/websocket.py
from fastapi import WebSocket
import json

class AgentWebSocket:
    """Agent 执行过程的 WebSocket 推送"""
    
    def __init__(self, websocket: WebSocket):
        self.websocket = websocket
    
    async def emit(self, event_type: str, data: dict):
        """推送执行事件到前端"""
        await self.websocket.send_json({
            "type": event_type,
            "data": data,
            "timestamp": datetime.utcnow().isoformat()
        })

# 事件类型说明
EVENT_TYPES = {
    "plan": "任务规划完成",
    "thinking": "Agent 正在思考",
    "tool_call": "调用工具",
    "tool_result": "工具执行结果",
    "observation": "观察分析",
    "replan": "重新规划",
    "progress": "执行进度",
    "final": "最终结果",
    "error": "执行错误"
}
```

---

## 4. 上下文管理设计

### 4.1 Letta 记忆架构

```
┌─────────────────────────────────────────────────────────────────┐
│                      Letta 记忆系统                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   Core Memory (核心记忆)                   │  │
│  │                   ～2000 tokens                            │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │ System Persona (系统人设)                            │  │  │
│  │  │ - 你是企业数据分析助手                                │  │  │
│  │  │ - 擅长数据处理、报告生成、SQL 查询                    │  │  │
│  │  │ - 熟悉公司业务和数据结构                              │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │ User Profile (用户画像)                              │  │  │
│  │  │ - 姓名: 张三                                         │  │  │
│  │  │ - 部门: 数据运营部                                   │  │  │
│  │  │ - 偏好: 喜欢详细的分析报告，常用 Excel 格式           │  │  │
│  │  │ - 常用数据: 标注数据、用户行为数据                    │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              ↓                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                  Recall Memory (召回记忆)                  │  │
│  │                  历史对话存档，可搜索                       │  │
│  │                                                            │  │
│  │  conversation_search("上周的数据导出")                     │  │
│  │  → 返回相关历史对话片段                                    │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              ↓                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                Archival Memory (档案记忆)                  │  │
│  │                长期知识存储 (pgvector)                      │  │
│  │                                                            │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │ 企业知识库                                           │  │  │
│  │  │ - 代码仓库文档 (README, 注释)                        │  │  │
│  │  │ - 飞书文档 (产品文档, 技术规范)                       │  │  │
│  │  │ - 数据库表结构说明                                   │  │  │
│  │  │ - 标注规范和示例                                     │  │  │
│  │  │ - 会议纪要                                           │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                                                            │  │
│  │  archival_memory_search("用户标注数据表结构")             │  │
│  │  → 返回相关数据库表定义和字段说明                          │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 企业知识导入流程

```python
# context/knowledge_importer.py

class EnterpriseKnowledgeImporter:
    """企业知识导入器"""
    
    def __init__(self, letta_client, tenant_id: str):
        self.letta_client = letta_client
        self.tenant_id = tenant_id
    
    async def import_codebase(self, repo_path: str):
        """导入代码仓库"""
        for file_path in self._scan_code_files(repo_path):
            content = self._read_file(file_path)
            
            # 分块处理大文件
            chunks = self._chunk_content(content, max_tokens=500)
            
            for chunk in chunks:
                await self.letta_client.insert_archival_memory(
                    agent_id=self.agent_id,
                    text=chunk,
                    metadata={
                        "type": "code",
                        "file_path": file_path,
                        "tenant_id": self.tenant_id
                    }
                )
    
    async def import_feishu_docs(self, doc_ids: list):
        """导入飞书文档"""
        feishu_client = FeishuClient()
        
        for doc_id in doc_ids:
            doc = await feishu_client.get_document(doc_id)
            
            await self.letta_client.insert_archival_memory(
                agent_id=self.agent_id,
                text=doc.content,
                metadata={
                    "type": "feishu_doc",
                    "doc_id": doc_id,
                    "title": doc.title,
                    "tenant_id": self.tenant_id
                }
            )
    
    async def import_database_schema(self, db_config: dict):
        """导入数据库表结构"""
        schema_docs = await self._extract_schema_docs(db_config)
        
        for table_doc in schema_docs:
            await self.letta_client.insert_archival_memory(
                agent_id=self.agent_id,
                text=table_doc,
                metadata={
                    "type": "database_schema",
                    "tenant_id": self.tenant_id
                }
            )
    
    async def import_labeling_data(self, file_path: str):
        """导入标注数据示例"""
        data = self._load_json(file_path)
        
        # 提取数据结构说明
        schema_desc = self._generate_schema_description(data)
        
        # 提取示例数据
        samples = self._extract_samples(data, n=5)
        
        await self.letta_client.insert_archival_memory(
            agent_id=self.agent_id,
            text=f"标注数据结构:\n{schema_desc}\n\n示例数据:\n{samples}",
            metadata={
                "type": "labeling_data",
                "file_path": file_path,
                "tenant_id": self.tenant_id
            }
        )
    
    async def import_meeting_transcript(self, video_path: str):
        """导入会议视频（转录）"""
        # 使用 Whisper 转录
        transcript = await self._transcribe_video(video_path)
        
        # 分段存储
        segments = self._segment_transcript(transcript)
        
        for segment in segments:
            await self.letta_client.insert_archival_memory(
                agent_id=self.agent_id,
                text=segment.text,
                metadata={
                    "type": "meeting_transcript",
                    "video_path": video_path,
                    "start_time": segment.start,
                    "end_time": segment.end,
                    "tenant_id": self.tenant_id
                }
            )
```

### 4.3 上下文检索流程

```python
# context/retriever.py

class ContextRetriever:
    """上下文检索器"""
    
    async def retrieve(self, query: str, tenant_id: str) -> dict:
        """检索相关上下文"""
        
        # 1. 检索档案记忆（向量相似度搜索）
        archival_results = await self.letta_client.archival_memory_search(
            agent_id=self.agent_id,
            query=query,
            k=10
        )
        
        # 2. 检索历史对话
        conversation_results = await self.letta_client.conversation_search(
            agent_id=self.agent_id,
            query=query,
            k=5
        )
        
        # 3. 获取核心记忆
        core_memory = await self.letta_client.get_core_memory(
            agent_id=self.agent_id
        )
        
        # 4. 构建上下文
        return {
            "core_memory": core_memory,
            "relevant_knowledge": self._format_archival(archival_results),
            "relevant_history": self._format_conversation(conversation_results),
            "total_tokens": self._count_tokens(...)
        }
```

---

## 5. 模型管理设计

### 5.1 LiteLLM 配置

```yaml
# config/litellm_config.yaml

model_list:
  # 主力模型 - 复杂推理
  - model_name: main-reasoning
    litellm_params:
      model: anthropic/claude-3-5-sonnet-20241022
      api_key: os.environ/ANTHROPIC_API_KEY
      max_tokens: 8192
    model_info:
      description: "主力推理模型，用于复杂任务规划和分析"
      cost_per_1k_input: 0.003
      cost_per_1k_output: 0.015
      
  # 备用模型 - 复杂推理
  - model_name: backup-reasoning
    litellm_params:
      model: openai/gpt-4o
      api_key: os.environ/OPENAI_API_KEY
      max_tokens: 8192
    model_info:
      description: "备用推理模型"
      
  # 快速模型 - 简单任务
  - model_name: fast-task
    litellm_params:
      model: deepseek/deepseek-chat
      api_key: os.environ/DEEPSEEK_API_KEY
      max_tokens: 4096
    model_info:
      description: "快速任务模型，用于简单查询和格式化"
      cost_per_1k_input: 0.0001
      cost_per_1k_output: 0.0002
      
  # 代码模型 - 代码生成
  - model_name: code-gen
    litellm_params:
      model: deepseek/deepseek-coder
      api_key: os.environ/DEEPSEEK_API_KEY
      max_tokens: 8192
    model_info:
      description: "代码生成专用模型"

  # Embedding 模型
  - model_name: embedding
    litellm_params:
      model: openai/text-embedding-3-small
      api_key: os.environ/OPENAI_API_KEY

# 路由配置
router_settings:
  routing_strategy: "cost-based-routing"  # 成本优先
  num_retries: 3
  timeout: 60
  
  # Fallback 链
  fallbacks:
    main-reasoning: [backup-reasoning, fast-task]
    code-gen: [main-reasoning, fast-task]

# 限流配置
litellm_settings:
  max_budget: 100  # 每月预算上限 (USD)
  budget_duration: "monthly"
  
  # 按租户限流
  tenant_rpm_limit: 60
  tenant_tpm_limit: 100000
```

### 5.2 模型路由器

```python
# model/router.py
import litellm
from litellm import Router

class ModelRouter:
    """智能模型路由器"""
    
    def __init__(self, config_path: str):
        self.config = self._load_config(config_path)
        self.router = Router(
            model_list=self.config["model_list"],
            routing_strategy=self.config["router_settings"]["routing_strategy"],
            num_retries=self.config["router_settings"]["num_retries"]
        )
        
        # 任务类型到模型的映射
        self.task_model_map = {
            "planning": "main-reasoning",
            "analysis": "main-reasoning", 
            "code_generation": "code-gen",
            "simple_query": "fast-task",
            "formatting": "fast-task",
            "embedding": "embedding"
        }
    
    async def route_completion(
        self, 
        messages: list, 
        task_type: str = "planning",
        tenant_id: str = None
    ):
        """根据任务类型路由到合适的模型"""
        
        model_name = self.task_model_map.get(task_type, "main-reasoning")
        
        # 检查租户配额
        if tenant_id:
            await self._check_quota(tenant_id)
        
        try:
            response = await self.router.acompletion(
                model=model_name,
                messages=messages
            )
            
            # 记录用量
            if tenant_id:
                await self._record_usage(tenant_id, response)
            
            return response
            
        except Exception as e:
            # Fallback 已在 Router 中处理
            raise
    
    async def _check_quota(self, tenant_id: str):
        """检查租户配额"""
        usage = await self._get_tenant_usage(tenant_id)
        limit = await self._get_tenant_limit(tenant_id)
        
        if usage >= limit:
            raise QuotaExceededException(f"租户 {tenant_id} 已超出配额")
    
    async def _record_usage(self, tenant_id: str, response):
        """记录使用量"""
        await self.db.execute("""
            INSERT INTO tbl_model_usage 
            (tenant_id, model, input_tokens, output_tokens, cost, create_time)
            VALUES ($1, $2, $3, $4, $5, NOW())
        """, tenant_id, response.model, 
            response.usage.prompt_tokens,
            response.usage.completion_tokens,
            self._calculate_cost(response))
```

---

## 6. Tool/Skill/MCP 管理设计

### 6.1 工具注册表

```python
# tools/registry.py

class ToolRegistry:
    """工具注册表"""
    
    def __init__(self, db):
        self.db = db
        self.tools = {}  # 内存缓存
        self.mcp_clients = {}  # MCP 客户端
    
    async def register_tool(self, tool_config: dict):
        """注册新工具"""
        tool = Tool(
            id=tool_config["id"],
            name=tool_config["name"],
            description=tool_config["description"],
            type=tool_config["type"],  # builtin / mcp / custom
            parameters=tool_config["parameters"],
            handler=tool_config.get("handler"),
            mcp_server=tool_config.get("mcp_server")
        )
        
        # 存入数据库
        await self.db.execute("""
            INSERT INTO tbl_tools (id, name, description, type, config, tenant_id)
            VALUES ($1, $2, $3, $4, $5, $6)
            ON CONFLICT (id) DO UPDATE SET ...
        """, tool.id, tool.name, tool.description, 
            tool.type, json.dumps(tool_config), tool_config.get("tenant_id"))
        
        # 更新缓存
        self.tools[tool.id] = tool
        
        return tool
    
    async def get_tenant_tools(self, tenant_id: str) -> list:
        """获取租户可用的工具列表"""
        # 获取系统工具 + 租户自定义工具
        tools = await self.db.fetch("""
            SELECT * FROM tbl_tools 
            WHERE tenant_id IS NULL OR tenant_id = $1
            AND enabled = true
        """, tenant_id)
        
        return [self._to_tool_schema(t) for t in tools]
    
    def _to_tool_schema(self, tool_record) -> dict:
        """转换为 OpenAI function calling 格式"""
        return {
            "type": "function",
            "function": {
                "name": tool_record["name"],
                "description": tool_record["description"],
                "parameters": tool_record["config"]["parameters"]
            }
        }
```

### 6.2 内置工具列表

```python
# tools/builtin/__init__.py

BUILTIN_TOOLS = [
    # 数据库工具
    {
        "id": "sql_query",
        "name": "sql_query",
        "description": "执行 SQL 查询，支持 SELECT 语句。用于从数据库获取数据。",
        "type": "builtin",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "SQL 查询语句（仅支持 SELECT）"
                },
                "database": {
                    "type": "string",
                    "description": "数据库名称",
                    "default": "default"
                }
            },
            "required": ["query"]
        }
    },
    
    # 数据导出工具
    {
        "id": "export_data",
        "name": "export_data",
        "description": "将数据导出为文件（CSV/Excel/JSON）",
        "type": "builtin",
        "parameters": {
            "type": "object",
            "properties": {
                "data": {
                    "type": "array",
                    "description": "要导出的数据"
                },
                "format": {
                    "type": "string",
                    "enum": ["csv", "excel", "json"],
                    "description": "导出格式"
                },
                "filename": {
                    "type": "string",
                    "description": "文件名"
                }
            },
            "required": ["data", "format"]
        }
    },
    
    # Python 代码执行
    {
        "id": "python_execute",
        "name": "python_execute",
        "description": "在安全沙箱中执行 Python 代码，支持数据分析和图表生成",
        "type": "builtin",
        "parameters": {
            "type": "object",
            "properties": {
                "code": {
                    "type": "string",
                    "description": "Python 代码"
                }
            },
            "required": ["code"]
        }
    },
    
    # 图表生成
    {
        "id": "generate_chart",
        "name": "generate_chart",
        "description": "根据数据生成图表（柱状图/折线图/饼图等）",
        "type": "builtin",
        "parameters": {
            "type": "object",
            "properties": {
                "data": {
                    "type": "array",
                    "description": "图表数据"
                },
                "chart_type": {
                    "type": "string",
                    "enum": ["bar", "line", "pie", "scatter", "heatmap"],
                    "description": "图表类型"
                },
                "title": {
                    "type": "string",
                    "description": "图表标题"
                },
                "x_label": {"type": "string"},
                "y_label": {"type": "string"}
            },
            "required": ["data", "chart_type"]
        }
    },
    
    # 文件读取
    {
        "id": "read_file",
        "name": "read_file", 
        "description": "读取文件内容（支持 JSON/CSV/TXT/Excel）",
        "type": "builtin",
        "parameters": {
            "type": "object",
            "properties": {
                "file_path": {
                    "type": "string",
                    "description": "文件路径"
                }
            },
            "required": ["file_path"]
        }
    },
    
    # 报告生成
    {
        "id": "generate_report",
        "name": "generate_report",
        "description": "生成数据分析报告（Markdown/HTML/PDF）",
        "type": "builtin",
        "parameters": {
            "type": "object",
            "properties": {
                "title": {"type": "string"},
                "sections": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "heading": {"type": "string"},
                            "content": {"type": "string"},
                            "charts": {"type": "array"}
                        }
                    }
                },
                "format": {
                    "type": "string",
                    "enum": ["markdown", "html", "pdf"]
                }
            },
            "required": ["title", "sections"]
        }
    },
    
    # JSON 处理
    {
        "id": "process_json",
        "name": "process_json",
        "description": "处理 JSON 数据（过滤/转换/聚合）",
        "type": "builtin",
        "parameters": {
            "type": "object",
            "properties": {
                "data": {"type": "array"},
                "operations": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "type": {
                                "type": "string",
                                "enum": ["filter", "map", "group_by", "aggregate"]
                            },
                            "config": {"type": "object"}
                        }
                    }
                }
            },
            "required": ["data", "operations"]
        }
    }
]
```

### 6.3 MCP 工具集成

```python
# tools/mcp_manager.py
from mcp import ClientSession, StdioServerParameters

class MCPManager:
    """MCP 工具管理器"""
    
    def __init__(self):
        self.servers = {}
        self.sessions = {}
    
    async def register_mcp_server(self, server_config: dict):
        """注册 MCP 服务器"""
        server_id = server_config["id"]
        
        if server_config["type"] == "stdio":
            params = StdioServerParameters(
                command=server_config["command"],
                args=server_config.get("args", []),
                env=server_config.get("env", {})
            )
            
            session = await ClientSession(params).__aenter__()
            self.sessions[server_id] = session
            
            # 获取服务器提供的工具
            tools = await session.list_tools()
            self.servers[server_id] = {
                "config": server_config,
                "tools": tools
            }
            
            return tools
    
    async def call_mcp_tool(self, server_id: str, tool_name: str, arguments: dict):
        """调用 MCP 工具"""
        session = self.sessions.get(server_id)
        if not session:
            raise ValueError(f"MCP server {server_id} not found")
        
        result = await session.call_tool(tool_name, arguments)
        return result

# 预置 MCP 服务器配置
MCP_SERVERS = [
    {
        "id": "filesystem",
        "name": "文件系统",
        "type": "stdio",
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
    },
    {
        "id": "postgres",
        "name": "PostgreSQL",
        "type": "stdio", 
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-postgres"],
        "env": {
            "POSTGRES_URL": "postgresql://..."
        }
    },
    {
        "id": "github",
        "name": "GitHub",
        "type": "stdio",
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
            "GITHUB_TOKEN": "..."
        }
    }
]
```

---

## 7. 场景实现详解

### 7.1 场景一：数据导出与分析报告

**用户输入**: "帮我导出上周所有标注员的标注数据，生成数据分析报告"

#### 完整调用流程

```
┌─────────────────────────────────────────────────────────────────┐
│ Step 1: 意图理解 & 上下文检索                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  用户输入: "帮我导出上周所有标注员的标注数据，生成数据分析报告"     │
│                                                                  │
│  → Letta 检索相关上下文:                                         │
│    - 标注数据表结构 (tbl_annotations, tbl_labelers)              │
│    - 用户偏好 (喜欢 Excel 格式，详细报告)                         │
│    - 历史类似任务 (上次生成报告的格式)                            │
│                                                                  │
│  → 构建增强 Prompt                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 2: 任务规划                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LLM 生成执行计划:                                               │
│                                                                  │
│  Plan:                                                           │
│  ├── 1. 查询上周时间范围                                         │
│  ├── 2. 查询标注员列表                                           │
│  ├── 3. 查询标注数据 (sql_query)                                 │
│  ├── 4. 数据聚合分析 (python_execute)                            │
│  │      - 按标注员统计                                           │
│  │      - 按日期统计                                             │
│  │      - 质量分析                                               │
│  ├── 5. 生成图表 (generate_chart)                                │
│  │      - 标注量趋势图                                           │
│  │      - 标注员对比图                                           │
│  ├── 6. 导出原始数据 (export_data)                               │
│  └── 7. 生成报告 (generate_report)                               │
│                                                                  │
│  WebSocket 推送: { type: "plan", data: plan }                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 3: 执行 - 查询数据                                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Tool Call: sql_query                                            │
│  Arguments: {                                                    │
│    "query": """                                                  │
│      SELECT                                                      │
│        l.name as labeler_name,                                   │
│        DATE(a.create_time) as date,                              │
│        COUNT(*) as annotation_count,                             │
│        AVG(a.quality_score) as avg_quality                       │
│      FROM tbl_annotations a                                      │
│      JOIN tbl_labelers l ON a.labeler_id = l.id                  │
│      WHERE a.create_time >= NOW() - INTERVAL '7 days'            │
│      GROUP BY l.name, DATE(a.create_time)                        │
│      ORDER BY date, labeler_name                                 │
│    """                                                           │
│  }                                                               │
│                                                                  │
│  Result: [                                                       │
│    {"labeler_name": "张三", "date": "2026-02-03", ...},          │
│    {"labeler_name": "李四", "date": "2026-02-03", ...},          │
│    ...                                                           │
│  ]                                                               │
│                                                                  │
│  WebSocket 推送: { type: "tool_result", data: result }           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 4: 执行 - 数据分析                                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Tool Call: python_execute                                       │
│  Arguments: {                                                    │
│    "code": """                                                   │
│      import pandas as pd                                         │
│      import numpy as np                                          │
│                                                                  │
│      df = pd.DataFrame(data)                                     │
│                                                                  │
│      # 按标注员汇总                                               │
│      by_labeler = df.groupby('labeler_name').agg({               │
│          'annotation_count': 'sum',                              │
│          'avg_quality': 'mean'                                   │
│      }).round(2)                                                 │
│                                                                  │
│      # 按日期汇总                                                 │
│      by_date = df.groupby('date').agg({                          │
│          'annotation_count': 'sum'                               │
│      })                                                          │
│                                                                  │
│      # 计算统计指标                                               │
│      stats = {                                                   │
│          'total_annotations': df['annotation_count'].sum(),      │
│          'avg_daily': df.groupby('date')['annotation_count']     │
│                        .sum().mean(),                            │
│          'top_labeler': by_labeler['annotation_count'].idxmax(), │
│          'avg_quality': df['avg_quality'].mean()                 │
│      }                                                           │
│                                                                  │
│      result = {                                                  │
│          'by_labeler': by_labeler.to_dict(),                     │
│          'by_date': by_date.to_dict(),                           │
│          'stats': stats                                          │
│      }                                                           │
│    """                                                           │
│  }                                                               │
│                                                                  │
│  Result: {                                                       │
│    "by_labeler": {...},                                          │
│    "by_date": {...},                                             │
│    "stats": {                                                    │
│      "total_annotations": 1523,                                  │
│      "avg_daily": 217.6,                                         │
│      "top_labeler": "张三",                                      │
│      "avg_quality": 0.92                                         │
│    }                                                             │
│  }                                                               │
│                                                                  │
│  WebSocket 推送: { type: "tool_result", data: analysis }         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 5: 执行 - 生成图表                                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Tool Call: generate_chart                                       │
│  Arguments: {                                                    │
│    "data": by_date_data,                                         │
│    "chart_type": "line",                                         │
│    "title": "上周标注量趋势",                                     │
│    "x_label": "日期",                                            │
│    "y_label": "标注数量"                                         │
│  }                                                               │
│                                                                  │
│  Result: {                                                       │
│    "chart_url": "/files/charts/trend_20260210.png",              │
│    "chart_base64": "..."                                         │
│  }                                                               │
│                                                                  │
│  Tool Call: generate_chart (第二个图表)                          │
│  Arguments: {                                                    │
│    "data": by_labeler_data,                                      │
│    "chart_type": "bar",                                          │
│    "title": "标注员工作量对比"                                    │
│  }                                                               │
│                                                                  │
│  WebSocket 推送: { type: "tool_result", data: charts }           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 6: 执行 - 导出数据 & 生成报告                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Tool Call: export_data                                          │
│  Arguments: {                                                    │
│    "data": raw_data,                                             │
│    "format": "excel",                                            │
│    "filename": "标注数据_20260203-20260209.xlsx"                  │
│  }                                                               │
│                                                                  │
│  Result: {                                                       │
│    "file_url": "/files/exports/标注数据_20260203-20260209.xlsx"   │
│  }                                                               │
│                                                                  │
│  Tool Call: generate_report                                      │
│  Arguments: {                                                    │
│    "title": "标注数据周报 (2026.02.03 - 2026.02.09)",            │
│    "sections": [                                                 │
│      {                                                           │
│        "heading": "概述",                                        │
│        "content": "本周共完成标注 1523 条..."                    │
│      },                                                          │
│      {                                                           │
│        "heading": "标注量分析",                                  │
│        "content": "...",                                         │
│        "charts": [trend_chart, comparison_chart]                 │
│      },                                                          │
│      {                                                           │
│        "heading": "质量分析",                                    │
│        "content": "平均质量分数 0.92..."                         │
│      },                                                          │
│      {                                                           │
│        "heading": "建议",                                        │
│        "content": "建议关注..."                                  │
│      }                                                           │
│    ],                                                            │
│    "format": "html"                                              │
│  }                                                               │
│                                                                  │
│  Result: {                                                       │
│    "report_url": "/files/reports/标注周报_20260210.html"         │
│  }                                                               │
│                                                                  │
│  WebSocket 推送: { type: "tool_result", data: report }           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 7: 最终输出                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WebSocket 推送: {                                               │
│    type: "final",                                                │
│    data: {                                                       │
│      "summary": "已完成上周标注数据导出和分析报告生成",           │
│      "outputs": [                                                │
│        {                                                         │
│          "type": "file",                                         │
│          "name": "标注数据_20260203-20260209.xlsx",              │
│          "url": "/files/exports/...",                            │
│          "description": "原始标注数据"                           │
│        },                                                        │
│        {                                                         │
│          "type": "report",                                       │
│          "name": "标注周报_20260210.html",                       │
│          "url": "/files/reports/...",                            │
│          "description": "数据分析报告"                           │
│        }                                                         │
│      ],                                                          │
│      "execution_history": [                                      │
│        { "step": "查询数据", "duration": "1.2s", "status": "✓" },│
│        { "step": "数据分析", "duration": "0.8s", "status": "✓" },│
│        { "step": "生成图表", "duration": "2.1s", "status": "✓" },│
│        { "step": "导出数据", "duration": "0.5s", "status": "✓" },│
│        { "step": "生成报告", "duration": "1.5s", "status": "✓" } │
│      ],                                                          │
│      "total_time": "6.1s"                                        │
│    }                                                             │
│  }                                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### 代码实现

```python
# scenarios/data_export_report.py

class DataExportReportScenario:
    """场景一：数据导出与分析报告"""
    
    async def execute(self, agent: EnterpriseAgent, user_input: str, ws: WebSocket):
        """执行数据导出报告场景"""
        
        # 1. 检索上下文
        context = await agent.retrieve_context(user_input)
        
        # 2. 构建 Prompt
        prompt = f"""
你是一个数据分析助手。用户请求：{user_input}

相关上下文：
- 数据库表结构：{context['relevant_knowledge']}
- 用户偏好：{context['core_memory']['human']}

请生成执行计划，需要：
1. 确定时间范围
2. 查询相关数据
3. 进行数据分析
4. 生成可视化图表
5. 导出原始数据
6. 生成分析报告

输出 JSON 格式的执行计划。
"""
        
        # 3. 生成计划
        plan = await agent.model_router.route_completion(
            messages=[{"role": "user", "content": prompt}],
            task_type="planning"
        )
        
        await ws.send_json({"type": "plan", "data": plan})
        
        # 4. 执行计划
        results = []
        for step in plan.steps:
            result = await agent.execute_tool(step.tool, step.arguments)
            results.append({"step": step, "result": result})
            await ws.send_json({"type": "tool_result", "data": result})
        
        # 5. 生成最终输出
        final = await self._generate_final_output(results)
        await ws.send_json({"type": "final", "data": final})
        
        return final
```

---

### 7.2 场景二：个人周报生成

**用户输入**: "帮我生成我这周的工作周报"

#### 完整调用流程

```
┌─────────────────────────────────────────────────────────────────┐
│ Step 1: 意图理解 & 上下文检索                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  用户输入: "帮我生成我这周的工作周报"                             │
│                                                                  │
│  → Letta 检索:                                                   │
│    - 用户身份 (张三, 数据运营部)                                  │
│    - 用户本周活动数据源:                                         │
│      · Git 提交记录                                              │
│      · 任务系统完成情况                                          │
│      · 会议参与记录                                              │
│      · 飞书文档编辑记录                                          │
│    - 历史周报格式                                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 2: 任务规划                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Plan:                                                           │
│  ├── 1. 获取用户本周 Git 提交记录 (mcp: github)                  │
│  ├── 2. 获取任务系统完成情况 (sql_query)                         │
│  ├── 3. 获取会议参与记录 (mcp: calendar)                         │
│  ├── 4. 获取文档编辑记录 (mcp: feishu)                           │
│  ├── 5. 汇总分析工作内容 (python_execute)                        │
│  └── 6. 生成周报 (generate_report)                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 3-6: 执行各步骤                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 3: MCP Tool - github.get_commits                           │
│  Result: [                                                       │
│    {"message": "feat: 添加数据导出功能", "date": "02-05"},       │
│    {"message": "fix: 修复分页问题", "date": "02-06"},            │
│    ...                                                           │
│  ]                                                               │
│                                                                  │
│  Step 4: sql_query - 查询任务完成情况                            │
│  Result: [                                                       │
│    {"task": "数据标注平台优化", "status": "completed"},          │
│    {"task": "周报系统开发", "status": "in_progress"},            │
│  ]                                                               │
│                                                                  │
│  Step 5: MCP Tool - calendar.get_events                          │
│  Result: [                                                       │
│    {"title": "产品需求评审", "date": "02-04"},                   │
│    {"title": "技术方案讨论", "date": "02-06"},                   │
│  ]                                                               │
│                                                                  │
│  Step 6: python_execute - 汇总分析                               │
│  Result: {                                                       │
│    "commits_count": 12,                                          │
│    "tasks_completed": 3,                                         │
│    "meetings_attended": 5,                                       │
│    "highlights": ["完成数据导出功能", "修复3个Bug"]              │
│  }                                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 7: 生成周报                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Tool Call: generate_report                                      │
│  Result:                                                         │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    工作周报                              │    │
│  │                2026.02.03 - 2026.02.09                   │    │
│  │                    张三 | 数据运营部                      │    │
│  ├─────────────────────────────────────────────────────────┤    │
│  │ ## 本周工作总结                                          │    │
│  │                                                          │    │
│  │ ### 1. 开发工作                                          │    │
│  │ - 完成数据导出功能开发 (12 commits)                      │    │
│  │ - 修复分页显示问题                                       │    │
│  │ - 优化查询性能                                           │    │
│  │                                                          │    │
│  │ ### 2. 任务完成情况                                      │    │
│  │ - [x] 数据标注平台优化                                   │    │
│  │ - [x] API 文档更新                                       │    │
│  │ - [ ] 周报系统开发 (进行中)                              │    │
│  │                                                          │    │
│  │ ### 3. 会议参与                                          │    │
│  │ - 产品需求评审 (02-04)                                   │    │
│  │ - 技术方案讨论 (02-06)                                   │    │
│  │                                                          │    │
│  │ ## 下周计划                                              │    │
│  │ - 完成周报系统开发                                       │    │
│  │ - 开始用户反馈分析                                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### 7.3 场景三：批量评测结果处理

**用户输入**: "帮我处理这批评测结果 JSON，生成统计分析图表，列出问题数据"

#### 完整调用流程

```
┌─────────────────────────────────────────────────────────────────┐
│ Step 1: 意图理解 & 上下文检索                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  用户输入: "帮我处理这批评测结果 JSON，生成统计分析图表，         │
│            列出问题数据"                                         │
│                                                                  │
│  附件: evaluation_results_20260209.json                          │
│                                                                  │
│  → Letta 检索:                                                   │
│    - 评测数据结构说明                                            │
│    - 问题数据判定标准 (如: score < 0.6 为问题数据)               │
│    - 历史分析报告格式                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 2: 任务规划                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Plan:                                                           │
│  ├── 1. 读取 JSON 文件 (read_file)                               │
│  ├── 2. 解析数据结构 (python_execute)                            │
│  ├── 3. 统计分析 (python_execute)                                │
│  │      - 总体统计 (总数、通过率、平均分)                        │
│  │      - 分类统计 (按模型、按题型)                              │
│  │      - 问题数据提取 (score < 0.6)                             │
│  ├── 4. 生成图表 (generate_chart)                                │
│  │      - 分数分布直方图                                         │
│  │      - 模型对比柱状图                                         │
│  │      - 题型通过率饼图                                         │
│  ├── 5. 导出问题数据 (export_data)                               │
│  └── 6. 生成分析报告 (generate_report)                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 3: 读取并解析 JSON                                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Tool Call: read_file                                            │
│  Result: {                                                       │
│    "total_records": 5000,                                        │
│    "sample": [                                                   │
│      {                                                           │
│        "id": "eval_001",                                         │
│        "model": "gpt-4o",                                        │
│        "question_type": "reasoning",                             │
│        "score": 0.85,                                            │
│        "response": "...",                                        │
│        "expected": "..."                                         │
│      },                                                          │
│      ...                                                         │
│    ]                                                             │
│  }                                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 4: 统计分析                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Tool Call: python_execute                                       │
│  Code:                                                           │
│  ```python                                                       │
│  import pandas as pd                                             │
│  import numpy as np                                              │
│                                                                  │
│  df = pd.DataFrame(data)                                         │
│                                                                  │
│  # 总体统计                                                      │
│  stats = {                                                       │
│      'total': len(df),                                           │
│      'pass_rate': (df['score'] >= 0.6).mean(),                   │
│      'avg_score': df['score'].mean(),                            │
│      'std_score': df['score'].std()                              │
│  }                                                               │
│                                                                  │
│  # 按模型统计                                                    │
│  by_model = df.groupby('model').agg({                            │
│      'score': ['mean', 'std', 'count'],                          │
│      'id': lambda x: (df.loc[x.index, 'score'] >= 0.6).mean()    │
│  })                                                              │
│                                                                  │
│  # 问题数据                                                      │
│  problems = df[df['score'] < 0.6].to_dict('records')             │
│  ```                                                             │
│                                                                  │
│  Result: {                                                       │
│    "stats": {                                                    │
│      "total": 5000,                                              │
│      "pass_rate": 0.78,                                          │
│      "avg_score": 0.72,                                          │
│      "std_score": 0.18                                           │
│    },                                                            │
│    "by_model": {...},                                            │
│    "by_question_type": {...},                                    │
│    "problems_count": 1100,                                       │
│    "problems_sample": [...]                                      │
│  }                                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 5: 生成图表                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Chart 1: 分数分布直方图                                         │
│  ┌────────────────────────────────────┐                         │
│  │    ▓▓▓                              │                         │
│  │    ▓▓▓▓▓▓                           │                         │
│  │    ▓▓▓▓▓▓▓▓▓▓                       │                         │
│  │    ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓                  │                         │
│  │    ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓              │                         │
│  │ ───────────────────────────         │                         │
│  │ 0.0  0.2  0.4  0.6  0.8  1.0        │                         │
│  └────────────────────────────────────┘                         │
│                                                                  │
│  Chart 2: 模型对比柱状图                                         │
│  Chart 3: 题型通过率饼图                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 6: 导出问题数据                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Tool Call: export_data                                          │
│  Arguments: {                                                    │
│    "data": problems,                                             │
│    "format": "excel",                                            │
│    "filename": "问题数据_20260209.xlsx"                          │
│  }                                                               │
│                                                                  │
│  Result: {                                                       │
│    "file_url": "/files/exports/问题数据_20260209.xlsx",          │
│    "records_count": 1100                                         │
│  }                                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 7: 最终输出                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  {                                                               │
│    "summary": "评测结果分析完成",                                │
│    "key_findings": [                                             │
│      "总评测数: 5000 条",                                        │
│      "整体通过率: 78%",                                          │
│      "平均分: 0.72",                                             │
│      "问题数据: 1100 条 (22%)"                                   │
│    ],                                                            │
│    "outputs": [                                                  │
│      {                                                           │
│        "type": "chart",                                          │
│        "name": "分数分布图",                                     │
│        "url": "/files/charts/score_dist.png"                     │
│      },                                                          │
│      {                                                           │
│        "type": "chart",                                          │
│        "name": "模型对比图",                                     │
│        "url": "/files/charts/model_compare.png"                  │
│      },                                                          │
│      {                                                           │
│        "type": "file",                                           │
│        "name": "问题数据",                                       │
│        "url": "/files/exports/问题数据_20260209.xlsx"            │
│      },                                                          │
│      {                                                           │
│        "type": "report",                                         │
│        "name": "分析报告",                                       │
│        "url": "/files/reports/评测分析_20260209.html"            │
│      }                                                           │
│    ],                                                            │
│    "problems_preview": [                                         │
│      {"id": "eval_123", "model": "gpt-4o", "score": 0.35, ...}, │
│      {"id": "eval_456", "model": "claude", "score": 0.42, ...}, │
│      ...                                                         │
│    ],                                                            │
│    "execution_history": [...]                                    │
│  }                                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. API 设计

### 8.1 核心 API

```yaml
# API 列表

# 任务执行
POST   /api/v1/agent/execute          # 执行任务（HTTP）
WS     /api/v1/agent/execute/stream   # 执行任务（WebSocket 流式）

# 上下文管理
POST   /api/v1/context/import         # 导入知识
GET    /api/v1/context/search         # 搜索上下文
DELETE /api/v1/context/{id}           # 删除上下文

# 工具管理
GET    /api/v1/tools                  # 获取工具列表
POST   /api/v1/tools                  # 注册工具
PUT    /api/v1/tools/{id}             # 更新工具
DELETE /api/v1/tools/{id}             # 删除工具
POST   /api/v1/tools/{id}/test        # 测试工具

# MCP 服务管理
GET    /api/v1/mcp/servers            # 获取 MCP 服务器列表
POST   /api/v1/mcp/servers            # 注册 MCP 服务器
DELETE /api/v1/mcp/servers/{id}       # 删除 MCP 服务器
GET    /api/v1/mcp/servers/{id}/tools # 获取服务器工具

# 模型管理
GET    /api/v1/models                 # 获取模型列表
POST   /api/v1/models                 # 添加模型
PUT    /api/v1/models/{id}            # 更新模型配置
GET    /api/v1/models/usage           # 获取用量统计

# 任务历史
GET    /api/v1/tasks                  # 获取任务列表
GET    /api/v1/tasks/{id}             # 获取任务详情
GET    /api/v1/tasks/{id}/history     # 获取执行历史

# 文件管理
GET    /api/v1/files                  # 获取文件列表
GET    /api/v1/files/{id}/download    # 下载文件
DELETE /api/v1/files/{id}             # 删除文件
```

### 8.2 WebSocket 消息格式

```typescript
// 客户端发送
interface ClientMessage {
  type: "execute" | "cancel" | "feedback";
  data: {
    task?: string;          // 任务描述
    task_id?: string;       // 任务 ID（取消时使用）
    attachments?: string[]; // 附件 URL
  };
}

// 服务端推送
interface ServerMessage {
  type: "plan" | "thinking" | "tool_call" | "tool_result" | 
        "observation" | "replan" | "progress" | "final" | "error";
  data: any;
  timestamp: string;
  task_id: string;
}

// 示例消息流
[
  { type: "plan", data: { steps: [...] }, task_id: "task_001" },
  { type: "thinking", data: { content: "正在分析数据结构..." }, task_id: "task_001" },
  { type: "tool_call", data: { tool: "sql_query", args: {...} }, task_id: "task_001" },
  { type: "tool_result", data: { result: [...], duration: "1.2s" }, task_id: "task_001" },
  { type: "progress", data: { current: 2, total: 5, percent: 40 }, task_id: "task_001" },
  { type: "final", data: { summary: "...", outputs: [...] }, task_id: "task_001" }
]
```

---

## 9. 数据库设计

### 9.1 核心表结构

```sql
-- 租户表
CREATE TABLE tbl_tenants (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    config JSONB DEFAULT '{}',
    quota JSONB DEFAULT '{"monthly_tokens": 1000000}',
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    deleted BOOLEAN NOT NULL DEFAULT false
);

-- 用户表
CREATE TABLE tbl_users (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER REFERENCES tbl_tenants(id),
    username VARCHAR(100) NOT NULL,
    email VARCHAR(200),
    profile JSONB DEFAULT '{}',  -- 用户画像
    preferences JSONB DEFAULT '{}',  -- 偏好设置
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    deleted BOOLEAN NOT NULL DEFAULT false
);

-- 工具表
CREATE TABLE tbl_tools (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER REFERENCES tbl_tenants(id),  -- NULL 表示系统工具
    name VARCHAR(100) NOT NULL,
    description TEXT,
    type VARCHAR(50) NOT NULL,  -- builtin / mcp / custom
    config JSONB NOT NULL,
    enabled BOOLEAN DEFAULT true,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    deleted BOOLEAN NOT NULL DEFAULT false
);

-- MCP 服务器表
CREATE TABLE tbl_mcp_servers (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER REFERENCES tbl_tenants(id),
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50) NOT NULL,  -- stdio / sse
    config JSONB NOT NULL,
    status VARCHAR(50) DEFAULT 'active',
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    deleted BOOLEAN NOT NULL DEFAULT false
);

-- 模型配置表
CREATE TABLE tbl_models (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER REFERENCES tbl_tenants(id),
    name VARCHAR(100) NOT NULL,
    provider VARCHAR(50) NOT NULL,  -- openai / anthropic / deepseek
    model_id VARCHAR(100) NOT NULL,
    config JSONB NOT NULL,
    enabled BOOLEAN DEFAULT true,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    deleted BOOLEAN NOT NULL DEFAULT false
);

-- 模型使用量表
CREATE TABLE tbl_model_usage (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER REFERENCES tbl_tenants(id),
    user_id INTEGER REFERENCES tbl_users(id),
    model VARCHAR(100) NOT NULL,
    input_tokens INTEGER NOT NULL,
    output_tokens INTEGER NOT NULL,
    cost DECIMAL(10, 6),
    task_id VARCHAR(100),
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

-- 任务表
CREATE TABLE tbl_tasks (
    id SERIAL PRIMARY KEY,
    task_uuid VARCHAR(100) UNIQUE NOT NULL,
    tenant_id INTEGER REFERENCES tbl_tenants(id),
    user_id INTEGER REFERENCES tbl_users(id),
    input TEXT NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',  -- pending / running / completed / failed
    result JSONB,
    execution_history JSONB DEFAULT '[]',
    error_message TEXT,
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc')
);

-- 知识库表（向量存储元数据）
CREATE TABLE tbl_knowledge (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER REFERENCES tbl_tenants(id),
    type VARCHAR(50) NOT NULL,  -- code / document / schema / meeting
    source VARCHAR(500),
    title VARCHAR(500),
    content TEXT,
    embedding vector(1536),  -- pgvector
    metadata JSONB DEFAULT '{}',
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    update_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    deleted BOOLEAN NOT NULL DEFAULT false
);

-- 创建向量索引
CREATE INDEX idx_knowledge_embedding ON tbl_knowledge 
USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

-- 文件表
CREATE TABLE tbl_files (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER REFERENCES tbl_tenants(id),
    task_id INTEGER REFERENCES tbl_tasks(id),
    name VARCHAR(500) NOT NULL,
    type VARCHAR(50) NOT NULL,  -- export / report / chart
    path VARCHAR(1000) NOT NULL,
    size INTEGER,
    create_time TIMESTAMP NOT NULL DEFAULT (now() AT TIME ZONE 'utc'),
    deleted BOOLEAN NOT NULL DEFAULT false
);
```

---

## 10. 部署架构

### 10.1 Docker Compose 部署

```yaml
# docker-compose.yml
version: '3.8'

services:
  # API 服务
  api:
    build: ./server
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/agent_platform
      - REDIS_URL=redis://redis:6379
      - LITELLM_CONFIG=/app/config/litellm_config.yaml
    depends_on:
      - postgres
      - redis
    volumes:
      - ./config:/app/config
      - ./workspace:/app/workspace

  # LiteLLM Proxy
  litellm:
    image: ghcr.io/berriai/litellm:main-latest
    ports:
      - "4000:4000"
    volumes:
      - ./config/litellm_config.yaml:/app/config.yaml
    command: --config /app/config.yaml

  # 前端
  web:
    build: ./web
    ports:
      - "3000:80"
    depends_on:
      - api

  # PostgreSQL + pgvector
  postgres:
    image: pgvector/pgvector:pg16
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=agent_platform
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  # MinIO (文件存储)
  minio:
    image: minio/minio
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=password
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9001"

volumes:
  postgres_data:
  redis_data:
  minio_data:
```

### 10.2 生产环境架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        负载均衡 (Nginx)                          │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   API Server    │ │   API Server    │ │   API Server    │
│   (FastAPI)     │ │   (FastAPI)     │ │   (FastAPI)     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
              │               │               │
              └───────────────┼───────────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ LiteLLM Proxy   │ │    Redis        │ │   PostgreSQL    │
│ (模型网关)       │ │ (缓存/队列)      │ │  + pgvector     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

---

## 11. 开发路线图

### Phase 1: MVP (2-3 周)

| 任务 | 时间 | 产出 |
|------|------|------|
| OpenManus 环境搭建 | 2 天 | 基础 Agent 运行 |
| LiteLLM 集成 | 1 天 | 模型路由能力 |
| Letta 集成 | 3 天 | 记忆管理能力 |
| 基础工具开发 | 3 天 | SQL/导出/图表工具 |
| WebSocket 接口 | 2 天 | 流式输出 |
| 前端对话界面 | 5 天 | 基础交互 |
| 场景一实现 | 3 天 | 数据导出报告 |

### Phase 2: 完善 (2-3 周)

| 任务 | 时间 | 产出 |
|------|------|------|
| 场景二、三实现 | 4 天 | 更多场景支持 |
| MCP 工具扩展 | 3 天 | GitHub/飞书/日历 |
| 知识库导入 | 3 天 | 企业知识整合 |
| 管理后台 | 5 天 | 工具/模型管理 |
| 权限控制 | 2 天 | 多租户隔离 |

### Phase 3: 优化 (持续)

| 任务 | 说明 |
|------|------|
| 性能优化 | 并发处理、缓存优化 |
| 安全加固 | 沙箱隔离、输入校验 |
| 可观测性 | 日志、监控、告警 |
| 更多场景 | 根据用户反馈扩展 |

---

## 附录 A: 技术栈版本

| 组件 | 版本 | 说明 |
|------|------|------|
| Python | 3.12 | 运行时 |
| FastAPI | 0.109+ | Web 框架 |
| OpenManus | latest | Agent 框架 |
| Letta | 0.6+ | 记忆管理 |
| LiteLLM | 1.55+ | 模型网关 |
| PostgreSQL | 16 | 数据库 |
| pgvector | 0.7+ | 向量扩展 |
| Redis | 7 | 缓存 |
| Vue | 3.4+ | 前端框架 |
| Node.js | 22+ | MCP 运行时 |

---

## 附录 B: 参考资源

- [OpenManus GitHub](https://github.com/FoundationAgents/OpenManus)
- [Letta 文档](https://docs.letta.com/)
- [LiteLLM 文档](https://docs.litellm.ai/)
- [MCP 规范](https://modelcontextprotocol.io/)
- [LangGraph 文档](https://python.langchain.com/docs/langgraph)
