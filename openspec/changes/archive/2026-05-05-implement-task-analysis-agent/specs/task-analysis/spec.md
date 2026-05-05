# Task Analysis Capability Specification

## ADDED Requirements

### Requirement: Analyze research intent
The system SHALL analyze user's research query and extract structured information including topic, keywords, research type, target audience, deliverables, depth, priority, and suggested approach.

#### Scenario: User provides clear research query
- **WHEN** user submits "我想了解 AI Agent 的最新发展趋势"
- **THEN** system extracts:
  - topic: "AI Agent 技术架构发展"
  - keywords: ["agent", "LLM", "autonomous", "multi-agent"]
  - researchType: "technical"
  - targetAudience: "研究人员/工程师"
  - deliverables: ["技术报告"]
  - depth: "medium"
  - priority: "medium"

#### Scenario: User provides insufficient information
- **WHEN** user submits "我想了解 AI"
- **THEN** system identifies missing information and generates clarification questions
- **AND** returns clarificationQuestions array with up to 3 questions
- **AND** does not proceed to full analysis until user provides more context

### Requirement: Extract research keywords
The system SHALL extract 3-10 relevant keywords from user's research query, including technical terms, concepts, and related entities.

#### Scenario: Extract technical keywords
- **WHEN** user submits "LLM 在代码生成中的应用"
- **THEN** system extracts keywords including: "LLM", "code generation", "CodeGen", "GitHub Copilot"

#### Scenario: Handle ambiguous terms
- **WHEN** user uses ambiguous terms like "AI" or "machine learning"
- **THEN** system SHALL generate clarification questions to narrow down
- **OR** include broader related terms as keywords

### Requirement: Classify research type
The system SHALL classify the research into one of four types: technical, competitive, academic, or market.

#### Scenario: Technical research query
- **WHEN** user asks about implementation details or technical architecture
- **THEN** system sets researchType to "technical"

#### Scenario: Academic research query
- **WHEN** user mentions papers, publications, or academic focus
- **THEN** system sets researchType to "academic"

#### Scenario: Competitive analysis query
- **WHEN** user asks about product comparisons or market positioning
- **THEN** system sets researchType to "competitive"

#### Scenario: Market analysis query
- **WHEN** user asks about market trends or business aspects
- **THEN** system sets researchType to "market"

### Requirement: Determine research depth
The system SHALL determine appropriate research depth based on user input or explicit preference: shallow (概览), medium (详细), or deep (深入).

#### Scenario: User specifies depth
- **WHEN** user explicitly mentions depth preference (如 "深入了解")
- **THEN** system sets depth accordingly

#### Scenario: User does not specify depth
- **WHEN** user does not mention depth preference
- **THEN** system defaults to "medium"
- **AND** may generate clarification question about desired depth

### Requirement: Generate clarification questions
When user input is insufficient for comprehensive analysis, the system SHALL generate up to 3 clarification questions to gather missing information.

#### Scenario: Missing research type context
- **WHEN** user query does not indicate research type
- **THEN** system generates question like "您主要关注的是技术实现还是实际应用？"

#### Scenario: Missing target audience
- **WHEN** user query does not indicate target audience
- **THEN** system generates question like "这份报告的主要受众是谁？"

### Requirement: Generate research approach suggestion
Based on the analysis, the system SHALL suggest an appropriate research approach.

#### Scenario: Academic research
- **WHEN** researchType is "academic"
- **THEN** system suggests approach like "按时间线梳理论文，突出研究方法对比"

#### Scenario: Technical research
- **WHEN** researchType is "technical"
- **THEN** system suggests approach like "按技术栈分层讲解，突出架构演进"

### Requirement: Save research plan
The system SHALL save the generated research plan to a persistent storage location for later use.

#### Scenario: Save successful
- **WHEN** analysis completes successfully
- **AND** user confirms the plan
- **THEN** system saves plan to `.individualAgent/research/plans/<timestamp>.json`

#### Scenario: Storage failure
- **WHEN** storage write fails
- **THEN** system returns error message
- **AND** allows retry without re-analysis

### Requirement: Integration with Supervisor
The system SHALL pass the completed research plan to the Supervisor Agent for downstream processing.

#### Scenario: Plan confirmed by user
- **WHEN** user confirms the research plan
- **THEN** system sends TaskAnalysisResult to Supervisor
- **AND** Supervisor proceeds to next agent (DataAcquisition)

#### Scenario: User requests modification
- **WHEN** user requests changes to the plan
- **THEN** system re-analyzes with modified input
- **AND** generates updated plan