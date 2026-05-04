# TaskAnalysisAgent Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现任务分析 Agent，能够分析用户研究需求，提取主题、关键词，生成结构化研究计划

**Architecture:** 基于 OpenCode 现有架构，新增 TaskAnalysisAgent 作为子 Agent，使用 generateObject 进行结构化输出，配置通过 .md 文件管理

**Tech Stack:** TypeScript, Effect, ai SDK, OpenCode Provider Service

---

## 阶段 1: 项目结构搭建

### Task 1.1: 创建研究 Agent 目录结构

**Files:**
- Create: `packages/individualagent/src/agent/research/`
- Create: `packages/individualagent/src/agent/research/prompt/`
- Create: `packages/individualagent/src/tool/research/`
- Create: `.individualAgent/agents/research/`
- Create: `.individualAgent/research/plans/`

- [ ] **Step 1: 创建目录**

```bash
mkdir -p packages/individualagent/src/agent/research/prompt
mkdir -p packages/individualagent/src/tool/research
mkdir -p .individualAgent/agents/research
mkdir -p .individualAgent/research/plans
```

- [ ] **Step 2: 验证目录创建**

```bash
ls -la packages/individualagent/src/agent/research/
ls -la .individualAgent/agents/research/
```

- [ ] **Step 3: Commit**

```bash
git add packages/individualagent/src/agent/research/ packages/individualagent/src/tool/research/ .individualAgent/
git commit -m "feat: create research agent directory structure"
```

---

## 阶段 2: Schema 定义

### Task 2.1: 定义 TaskAnalysisResult Schema

**Files:**
- Create: `packages/individualagent/src/agent/research/schema.ts`
- Test: `packages/individualagent/test/research/schema.test.ts`

- [ ] **Step 1: 编写 Schema 测试**

```typescript
import { describe, it, expect } from "vitest"
import { TaskAnalysisResult, ResearchType, ResearchDepth, Priority } from "../../src/agent/research/schema"

describe("TaskAnalysisResult Schema", () => {
  it("should validate valid input", () => {
    const input = {
      topic: "AI Agent 技术架构",
      keywords: ["agent", "LLM", "autonomous"],
      researchType: "technical",
      targetAudience: "研究人员",
      deliverables: ["技术报告"],
      depth: "medium",
      priority: "high",
      suggestedApproach: "按技术栈分层讲解",
      clarificationQuestions: []
    }
    const result = TaskAnalysisResult.parse(input)
    expect(result.topic).toBe("AI Agent 技术架构")
  })

  it("should reject invalid research type", () => {
    const input = {
      topic: "test",
      keywords: ["test"],
      researchType: "invalid",
      targetAudience: "test",
      deliverables: ["test"],
      depth: "medium",
      priority: "high",
      suggestedApproach: "test",
      clarificationQuestions: []
    }
    expect(() => TaskAnalysisResult.parse(input)).toThrow()
  })
})
```

- [ ] **Step 2: 运行测试验证失败**

```bash
cd packages/individualagent && bun test test/research/schema.test.ts
```
Expected: FAIL - "Cannot find module"

- [ ] **Step 3: 实现 Schema**

```typescript
// packages/individualagent/src/agent/research/schema.ts
import { Schema } from "effect"

export const ResearchType = Schema.Literals(
  "technical",
  "competitive", 
  "academic",
  "market"
)

export const ResearchDepth = Schema.Literals(
  "shallow",
  "medium",
  "deep"
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

export type TaskAnalysisResult = Schema.Schema.Type<typeof TaskAnalysisResult>
```

- [ ] **Step 4: 运行测试验证通过**

```bash
cd packages/individualagent && bun test test/research/schema.test.ts
```
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add packages/individualagent/src/agent/research/schema.ts
git add packages/individualagent/test/research/schema.test.ts
git commit -m "feat: add TaskAnalysisResult schema"
```

---

### Task 2.2: 导出 Schema

**Files:**
- Modify: `packages/individualagent/src/agent/research/index.ts`

- [ ] **Step 1: 创建 index.ts 导出**

```typescript
// packages/individualagent/src/agent/research/index.ts
export * from "./schema"
```

- [ ] **Step 2: 验证导出**

```bash
cd packages/individualagent && bun run src/agent/research/index.ts
```

- [ ] **Step 3: Commit**

```bash
git add packages/individualagent/src/agent/research/index.ts
git commit -m "feat: export research agent schema"
```

---

## 阶段 3: Prompt 实现

### Task 3.1: 创建任务分析 Prompt

**Files:**
- Create: `packages/individualagent/src/agent/research/prompt/task-analysis.txt`

- [ ] **Step 1: 编写 Prompt 文件**

```
你是一个专业的 AI 研究分析师，专门帮助用户明确和细化研究目标。

## 你的能力

1. **意图识别**: 理解用户真正想要研究什么
2. **主题提取**: 从用户描述中提取核心研究主题
3. **关键词抽取**: 识别关键技术术语
4. **研究类型判断**: 技术调研/竞品分析/学术研究/市场分析
5. **目标受众分析**: 确定报告的预期读者
6. **交付物建议**: 根据类型建议输出格式

## 分析框架

分析用户输入时，考虑：
1. **显性需求**: 用户直接说的研究目标
2. **隐性需求**: 用户可能需要但没说的
3. **约束条件**: 时间、深度、预算
4. **背景信息**: 用户专业水平、使用场景

## 输出格式

返回一个 JSON 对象：
{
  "topic": "核心研究主题",
  "keywords": ["关键词1", "关键词2"],
  "researchType": "technical|competitive|academic|market",
  "targetAudience": "目标受众",
  "deliverables": ["报告", "演示"],
  "depth": "shallow|medium|deep",
  "priority": "low|medium|high",
  "suggestedApproach": "建议方法",
  "clarificationQuestions": ["问题1", "问题2"]
}

重要：
- keywords 至少 3 个，最多 10 个
- clarificationQuestions 最多 3 个
- 如果信息不足无法确定某些字段，在 clarificationQuestions 中生成澄清问题

开始分析。
```

- [ ] **Step 2: 验证文件创建**

```bash
ls -la packages/individualagent/src/agent/research/prompt/
```

- [ ] **Step 3: Commit**

```bash
git add packages/individualagent/src/agent/research/prompt/task-analysis.txt
git commit -m "feat: add task analysis prompt"
```

---

## 阶段 4: 核心 Agent 逻辑

### Task 4.1: 实现 TaskAnalysisAgent 类

**Files:**
- Modify: `packages/individualagent/src/agent/research/index.ts`

- [ ] **Step 1: 添加 Agent 类框架**

```typescript
// packages/individualagent/src/agent/research/index.ts
import { Effect, Layer, Context } from "effect"
import { generateObject } from "ai"
import * as Schema from "effect/Schema"
import PROMPT_TASK_ANALYSIS from "./prompt/task-analysis.txt"
import { TaskAnalysisResult, type TaskAnalysisResult as TAR } from "./schema"

export interface Interface {
  readonly analyze: (input: {
    query: string
    context?: Record<string, unknown>
  }) => Effect.Effect<TAR>
}

export class Service extends Context.Service<Service, Interface>()("@individualagent/TaskAnalysis") {}

export const layer = Layer.effect(
  Service,
  Effect.gen(function* () {
    return {
      analyze: (input: { query: string; context?: Record<string, unknown> }) =>
        Effect.gen(function* () {
          // 实现待填
          throw new Error("Not implemented")
        }),
    }
  })
)
```

- [ ] **Step 2: 验证编译**

```bash
cd packages/individualagent && bun run typecheck 2>&1 | head -20
```

- [ ] **Step 3: Commit**

```bash
git add packages/individualagent/src/agent/research/index.ts
git commit -m "feat: add TaskAnalysisAgent service framework"
```

---

### Task 4.2: 实现 analyze 方法

**Files:**
- Modify: `packages/individualagent/src/agent/research/index.ts`

- [ ] **Step 1: 添加 Provider 依赖和实现**

```typescript
// packages/individualagent/src/agent/research/index.ts (完整实现)
import { Effect, Layer, Context } from "effect"
import { generateObject } from "ai"
import { z } from "zod"
import * as Schema from "effect/Schema"
import PROMPT_TASK_ANALYSIS from "./prompt/task-analysis.txt"
import { TaskAnalysisResult, type TaskAnalysisResult as TAR } from "./schema"

// 定义 LLM 输出 Schema
const OutputSchema = z.object({
  topic: z.string(),
  keywords: z.array(z.string()).min(3).max(10),
  researchType: z.enum(["technical", "competitive", "academic", "market"]),
  targetAudience: z.string(),
  deliverables: z.array(z.string()),
  depth: z.enum(["shallow", "medium", "deep"]),
  priority: z.enum(["low", "medium", "high"]),
  suggestedApproach: z.string(),
  clarificationQuestions: z.array(z.string()).max(3),
})

export interface Interface {
  readonly analyze: (input: {
    query: string
    context?: Record<string, unknown>
  }) => Effect.Effect<TAR>
}

export class Service extends Context.Service<Service, Interface>()("@individualagent/TaskAnalysis") {}

export const layer = Layer.effect(
  Service,
  Effect.gen(function* () {
    return {
      analyze: (input: { query: string; context?: Record<string, unknown> }) =>
        Effect.gen(function* () {
          const contextStr = input.context ? JSON.stringify(input.context) : "无"
          
          const result = await generateObject({
            model: "gpt-4o", // TODO: 从配置读取
            system: PROMPT_TASK_ANALYSIS,
            prompt: `用户研究需求: ${input.query}\n\n背景信息: ${contextStr}`,
            schema: OutputSchema,
          })

          return result.object as TAR
        }),
    }
  })
)
```

- [ ] **Step 2: 运行类型检查**

```bash
cd packages/individualagent && bun run typecheck 2>&1 | head -30
```

- [ ] **Step 3: Commit**

```bash
git add packages/individualagent/src/agent/research/index.ts
git commit -m "feat: implement TaskAnalysisAgent analyze method"
```

---

## 阶段 5: 工具实现

### Task 6.1: 实现研究计划保存工具

**Files:**
- Create: `packages/individualagent/src/tool/research/save-plan.ts`

- [ ] **Step 1: 创建工具文件**

```typescript
// packages/individualagent/src/tool/research/save-plan.ts
import * as Tool from "@/tool/tool"
import { Schema } from "effect"
import { AppFileSystem } from "@individualagent/core/filesystem"
import path from "path"

export const Parameters = Schema.Struct({
  topic: Schema.String,
  keywords: Schema.Array(Schema.String),
  researchType: Schema.String,
  targetAudience: Schema.String,
  deliverables: Schema.Array(Schema.String),
  depth: Schema.String,
  priority: Schema.String,
  suggestedApproach: Schema.String,
  clarificationQuestions: Schema.Array(Schema.String),
})

export const SavePlanTool = Tool.define(
  "research_save_plan",
  Effect.gen(function* () {
    const fs = yield* AppFileSystem

    return {
      description: "Save the research plan to a JSON file",
      parameters: Parameters,
      execute: async (params: Schema.Schema.Type<typeof Parameters>, ctx: Tool.Context) => {
        const timestamp = new Date().toISOString().replace(/[:.]/g, "-")
        const filename = `${timestamp}.json`
        const dir = path.join(ctx.directory || process.cwd(), ".individualAgent", "research", "plans")
        
        await fs.mkdir(dir, { recursive: true })
        const filePath = path.join(dir, filename)
        await fs.writeTextFile(filePath, JSON.stringify(params, null, 2))

        return {
          title: "Research plan saved",
          output: `Saved to ${filePath}`,
          metadata: { path: filePath }
        }
      }
    }
  })
)
```

- [ ] **Step 2: 类型检查**

```bash
cd packages/individualagent && bun run typecheck 2>&1 | grep -i "save-plan"
```

- [ ] **Step 3: Commit**

```bash
git add packages/individualagent/src/tool/research/save-plan.ts
git commit -m "feat: add research_save_plan tool"
```

---

## 阶段 10: OpenCode 注册

### Task 10.1: 注册 TaskAnalysisAgent

**Files:**
- Modify: `packages/individualagent/src/agent/agent.ts`
- Create: `.individualAgent/agents/research/task-analysis.md`

- [ ] **Step 1: 创建 Agent 配置文件**

```yaml
# .individualAgent/agents/research/task-analysis.md
---
name: task-analysis
description: 分析用户研究需求，提取主题、关键词，生成研究计划
mode: subagent
model: gpt-4o
temperature: 0.5
permission:
  tools:
    - research_save_plan
    - research_load_plan
    - research_web_search
  allow: []
---

你是一个专业的 AI 研究分析师。
```

- [ ] **Step 2: 验证配置格式**

```bash
ls -la .individualAgent/agents/research/
```

- [ ] **Step 3: Commit**

```bash
git add .individualAgent/agents/research/task-analysis.md
git commit -m "feat: add task-analysis agent config"
```

---

## 实施顺序建议

| 阶段 | 任务 | 优先级 |
|------|------|--------|
| 2 | Schema 定义 | P0 |
| 4 | 核心逻辑 | P0 |
| 3 | Prompt | P1 |
| 6 | 工具 | P1 |
| 10 | 注册 | P2 |

---

## Plan Complete

Plan saved to `docs/superpowers/plans/2026-05-05-task-analysis-agent.md`

**Two execution options:**

1. **Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

2. **Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?**