## Why

IndividualAgent 是一个基于 OpenCode 的个人 AI 研究智能体，旨在帮助用户进行深度研究。当前需要实现**任务分析 Agent (TaskAnalysisAgent)** 作为整个研究流程的入口，负责理解用户的研究需求、提取主题和关键词、生成结构化的研究计划。

没有任务分析，研究智能体无法准确理解用户意图，会导致后续资料获取、分析和报告生成的方向偏差。这是一个关键的初始步骤，决定了整个研究的质量和方向。

## What Changes

1. **新增 TaskAnalysisAgent** - 一个专门分析用户研究需求的子 Agent
2. **新增研究分析 Schema** - 定义任务分析输出的结构化数据格式
3. **新增分析 Prompt** - 用于引导 LLM 进行意图识别和主题提取
4. **新增研究计划存储工具** - 保存和加载研究计划的工具
5. **与 Supervisor 集成** - TaskAnalysisAgent 将结果传递给主控 Supervisor

## Capabilities

### New Capabilities
- `task-analysis`: 分析用户研究需求，提取主题、关键词，生成研究计划
  - 意图识别：理解用户真正想要研究什么
  - 主题提取：从用户描述中提取核心研究主题
  - 关键词抽取：识别关键技术术语
  - 研究类型判断：技术调研/竞品分析/学术研究/市场分析
  - 目标受众分析：确定报告的预期读者
  - 澄清问题生成：当信息不足时生成澄清问题

### Modified Capabilities
- (无) - 这是新增能力，不修改现有功能

## Impact

### 新增文件
- `packages/individualagent/src/agent/research/` - 研究 Agent 目录
- `packages/individualagent/src/agent/research/index.ts` - TaskAnalysisAgent 主逻辑
- `packages/individualagent/src/agent/research/schema.ts` - 分析结果 Schema
- `packages/individualagent/src/agent/research/prompt/task-analysis.txt` - 分析 Prompt
- `packages/individualagent/src/tool/research/` - 研究工具目录

### 修改文件
- `packages/individualagent/src/agent/agent.ts` - 注册新的 Agent
- `packages/individualagent/src/agent/research/agent.md` - Agent 配置文件

### 依赖
- OpenCode Provider Service (LLM 调用)
- OpenCode Session System (消息存储)
- OpenCode Effect Runtime (服务层)

### 系统
- 需要配置 OpenAI Compatible API 作为 LLM 提供商
- 研究计划存储在项目目录的 `.individualAgent/research/` 下