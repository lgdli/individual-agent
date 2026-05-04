## Context

IndividualAgent 是一个基于 OpenCode 的个人 AI 研究智能体。TaskAnalysisAgent 是整个研究流程的入口 Agent，负责理解用户的研究需求并生成结构化的研究计划。

**当前状态：**
- OpenCode 架构已就绪，包含 Agent Service、Provider Service、Session System
- LLM 调用通过 OpenAI Compatible API 支持
- Agent 通过 `.md` 配置文件定义，使用 Prompt `.txt` 文件

**约束：**
- LLM 提供商：OpenAI Compatible API
- 数据存储：本地文件系统
- 模型选择：gpt-4o 或用户配置的其他模型
- 输出格式：JSON Structure Output（使用 ai 库的 generateObject）

## Goals / Non-Goals

**Goals:**
1. 实现 TaskAnalysisAgent，能够分析用户输入的研究需求
2. 提取结构化信息：主题、关键词、研究类型、目标受众、交付物、深度、优先级
3. 支持多轮对话：当信息不足时生成澄清问题
4. 与现有 OpenCode 架构无缝集成
5. 生成的研究计划可保存和加载

**Non-Goals:**
- 不实现资料获取功能（后续 Agent 负责）
- 不实现知识图谱构建（后续 Agent 负责）
- 不实现报告生成（后续 Agent 负责）
- 不支持多语言（初始仅支持中文）
- 不实现复杂的用户认证（使用本地配置）

## Decisions

### 1. 使用 generateObject 而非 streamObject

**决定：** 使用 `ai` 库的 `generateObject` 进行结构化输出

**理由：**
- 结构化输出确保 Schema 完整性
- 研究计划需要完整的数据结构，不适合流式
- 与 OpenCode 现有的 Agent 生成模式一致（见 `agent.ts` 第 380-399 行）

**替代方案考虑：**
- streamObject：适用于需要逐步显示分析过程的场景，但会增加复杂度
- 未来可扩展为支持流式输出

### 2. 模型选择策略

**决定：** 提供默认模型选择，可根据研究深度调整

**理由：**
- shallow: 使用 gpt-4o-mini（成本低，速度快）
- medium/deep: 使用 gpt-4o（质量优先）

**替代方案考虑：**
- 让用户完全手动选择：增加用户负担
- 统一使用最强模型：增加成本

### 3. 研究计划存储位置

**决定：** 存储在项目目录的 `.individualAgent/research/` 下

**理由：**
- 与 OpenCode 配置目录一致（`.individualAgent/`）
- 每个项目有独立的研究上下文
- 便于版本控制和共享

**替代方案考虑：**
- 全局存储：不便于项目隔离
- 数据库存储：增加复杂度，当前阶段不需要

### 4. Prompt 设计模式

**决定：** 遵循 OpenCode 现有模式，使用独立 `.txt` 文件

**理由：**
- 与现有 Agent Prompt 一致（`generate.txt`, `explore.txt` 等）
- 便于调试和迭代
- 支持热更新

## Risks / Trade-offs

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| LLM 输出不符合 Schema | 分析失败 | 添加 Schema 验证和重试逻辑 |
| 用户输入信息不足 | 分析结果不准确 | 实现澄清问题机制 |
| API 调用失败 | 服务中断 | 添加错误处理和降级策略 |
| 多次 API 调用增加成本 | 成本增加 | 支持配置使用更便宜的模型 |

### Trade-offs

- **质量 vs 成本**：使用更强的模型增加质量但增加成本
- **速度 vs 深度**：深度分析需要更多轮对话，增加等待时间
- **自动化 vs 控制**：完全自动化可能忽略用户特定需求

## Migration Plan

1. **Phase 1: 基础实现**
   - 创建 Agent 目录结构
   - 实现 Schema 定义
   - 编写基础 Prompt

2. **Phase 2: 核心逻辑**
   - 实现分析主逻辑
   - 集成 Provider Service
   - 实现工具定义

3. **Phase 3: 交互优化**
   - 实现多轮对话
   - 添加澄清问题机制
   - 集成测试

4. **Phase 4: 与 Supervisor 集成**
   - 定义消息传递协议
   - 实现状态机
   - 端到端测试

## Open Questions

1. **是否需要支持流式输出逐步显示分析过程？**
   - 当前设计使用一次性输出
   - 未来可扩展支持 streaming

2. **研究计划是否需要版本控制？**
   - 当前设计为单版本
   - 未来可添加版本历史

3. **是否需要支持导出为特定格式（PDF/Markdown）？**
   - 当前设计输出 JSON
   - 后续 Agent 负责格式转换

4. **如何处理用户中断分析的情况？**
   - 需要实现优雅的取消机制
   - 需要保存部分分析结果