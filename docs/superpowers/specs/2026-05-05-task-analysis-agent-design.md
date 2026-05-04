# TaskAnalysisAgent Design - 任务分析智能体

**Date**: 2026-05-05  
**Change**: implement-task-analysis-agent  
**Status**: Design Approved

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    TaskAnalysisAgent 架构                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   用户输入                                                       │
│      │                                                          │
│      ▼                                                          │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  TaskAnalysisAgent                                        │  │
│   │  ├── analyze(userQuery) → TaskAnalysisResult            │  │
│   │  ├── generateClarificationQuestions()                   │  │
│   │  └── savePlan() / loadPlan()                             │  │
│   └────────────────────────────┬────────────────────────────┘  │
│                                │                                 │
│         ┌──────────────────────┼──────────────────────┐        │
│         ▼                      ▼                      ▼        │
│   ┌─────────────┐      ┌─────────────┐      ┌─────────────┐   │
│   │  Prompt     │      │  Schema     │      │  Tools      │   │
│   │  task-      │      │  TaskAnalysis- │    │  web_search │   │
│   │  analysis   │      │  Result       │    │  save_plan  │   │
│   │  .txt       │      │               │    │  load_plan  │   │
│   └─────────────┘      └─────────────┘      └─────────────┘   │
│                                │                                 │
│                                ▼                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  OpenCode Provider Service                               │  │
│   │  └── generateObject() with OpenAI Compatible API        │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Components

### 2.1 Schema: TaskAnalysisResult

```typescript
// packages/individualagent/src/agent/research/schema.ts

export const ResearchType = Schema.Literals(
  "technical",    // 技术调研
  "competitive",  // 竞品分析
  "academic",     // 学术研究
  "market"        // 市场分析
)

export const ResearchDepth = Schema.Literals(
  "shallow",      // 概览
  "medium",       // 详细
  "deep"          // 深入
)

export const Priority = Schema.Literals(
  "low",
  "medium",
  "high"
)

export const TaskAnalysisResult = Schema.Struct({
  topic: Schema.String,
  keywords: Schema.Array(Schema.String),
  researchType: ResearchType,
  targetAudience: Schema.String,
  deliverables: Schema.Array(Schema.String),
  depth: ResearchDepth,
  priority: Priority,
  suggestedApproach: Schema.String,
  clarificationQuestions: Schema.Array(Schema.String),
})
```

### 2.2 Agent Configuration

```yaml
# .individualAgent/agents/research/task-analysis.md
---
name: task-analysis
description: 分析用户研究需求，提取主题、关键词，生成研究计划
mode: subagent
model: ${USER_CONFIGURED_MODEL:-gpt-4o}
temperature: 0.5
permission:
  tools: [research_web_search, research_save_plan, research_load_plan]
  allow: []
---

# Agent 行为定义
```

### 2.3 Model Selection Strategy

| Depth | Default Model | Temperature | Use Case |
|-------|---------------|-------------|----------|
| shallow | gpt-4o-mini | 0.3 | 快速概览 |
| medium | gpt-4o | 0.5 | 标准分析 |
| deep | gpt-4o | 0.3 | 深度研究 |

用户可通过配置覆盖默认模型。

---

## 3. Data Flow

### 3.1 Analysis Flow

```
1. 用户输入查询
       │
       ▼
2. 检查是否有足够信息
       │
       ├─ 不足 → 生成澄清问题 → 返回给用户
       │
       └─ 足够 → 继续分析
                   │
                   ▼
3. 构建 Prompt
   - 加载 task-analysis.txt
   - 注入用户输入
   - 注入上下文（如果有）
       │
       ▼
4. 调用 LLM (generateObject)
   - 使用配置的模型
   - 使用对应的 temperature
   - 传入 TaskAnalysisResult Schema
       │
       ▼
5. 解析结果
   - 验证 Schema
   - 处理错误
       │
       ▼
6. 返回 TaskAnalysisResult
```

### 3.2 Storage Flow

```
研究计划保存:
  TaskAnalysisResult
       │
       ▼
  .individualAgent/research/plans/<timestamp>.json
       │
       ▼
  成功 → 返回确认
  失败 → 返回错误，允许重试
```

---

## 4. Tool Definitions

### 4.1 research_web_search

- **用途**: 搜索网络了解最新术语
- **参数**: query: string, maxResults?: number
- **权限**: 需要用户确认

### 4.2 research_save_plan

- **用途**: 保存研究计划
- **参数**: plan: TaskAnalysisResult
- **位置**: .individualAgent/research/plans/

### 4.3 research_load_plan

- **用途**: 加载已有计划
- **参数**: planId?: string (默认最新)
- **返回**: TaskAnalysisResult

### 4.4 research_list_authorities

- **用途**: 列出用户指定的权威来源
- **参数**: 无
- **返回**: string[]

---

## 5. Supervisor Integration

### 5.1 Message Format

```typescript
interface ResearchPlanMessage {
  type: "research_plan"
  content: TaskAnalysisResult
  status: "draft" | "confirmed" | "refined"
  createdAt: string  // ISO timestamp
}
```

### 5.2 Confirmation Flow

```
分析完成
    │
    ▼
展示计划给用户
    │
    ├─ 确认 → 发送给 Supervisor → 启动 DataAcquisition
    │
    ├─ 修改 → 重新分析
    │
    └─ 补充信息 → 多轮对话继续
```

---

## 6. Error Handling

| Error Type | Handling |
|------------|----------|
| Schema 验证失败 | 重试 2 次，失败后返回原始错误 |
| API 调用失败 | 返回错误信息，允许重试 |
| 存储失败 | 返回具体错误，不丢失数据 |
| 模型不可用 | 尝试备用模型，失败则报错 |

---

## 7. File Structure

```
packages/individualagent/src/agent/research/
├── index.ts              # TaskAnalysisAgent 主类
├── schema.ts             # Schema 定义
└── prompt/
    └── task-analysis.txt # 分析 Prompt

packages/individualagent/src/tool/research/
├── index.ts              # 工具导出
├── web-search.ts         # 搜索工具
├── save-plan.ts          # 保存工具
└── load-plan.ts          # 加载工具

.individualAgent/agents/research/
└── task-analysis.md      # Agent 配置

.individualAgent/research/
└── plans/                # 研究计划存储
    └── <timestamp>.json
```

---

## 8. Testing Strategy

| Test Type | Coverage |
|-----------|----------|
| Unit | Schema 验证、工具参数验证 |
| Integration | LLM 调用、Prompt 渲染 |
| E2E | 完整用户流程 |

---

## 9. Open Questions & Future Improvements

- [ ] 支持流式输出显示分析过程
- [ ] 研究计划版本控制
- [ ] 多语言支持
- [ ] 导出为 PDF/Markdown 格式