# Implementation Tasks - TaskAnalysisAgent

## 1. Project Setup

- [ ] 1.1 Create research agent directory: `packages/individualagent/src/agent/research/`
- [ ] 1.2 Create prompt directory: `packages/individualagent/src/agent/research/prompt/`
- [ ] 1.3 Create tool directory: `packages/individualagent/src/tool/research/`
- [ ] 1.4 Create agent config directory: `.individualAgent/agents/research/`

## 2. Schema Definition

- [ ] 2.1 Define TaskAnalysisResult schema in `packages/individualagent/src/agent/research/schema.ts`
- [ ] 2.2 Define ResearchType enum (technical, competitive, academic, market)
- [ ] 2.3 Define ResearchDepth enum (shallow, medium, deep)
- [ ] 2.4 Define Priority enum (low, medium, high)
- [ ] 2.5 Add schema to OpenCode's Effect schema system

## 3. Prompt Implementation

- [ ] 3.1 Create task-analysis.txt prompt file
- [ ] 3.2 Define system role and capabilities
- [ ] 3.3 Add input analysis framework
- [ ] 3.4 Define output format with JSON structure
- [ ] 3.5 Add examples for different research types

## 4. Core Agent Logic

- [ ] 4.1 Create TaskAnalysisAgent class in `packages/individualagent/src/agent/research/index.ts`
- [ ] 4.2 Implement analyze() method with LLM call
- [ ] 4.3 Integrate with Provider Service for model selection
- [ ] 4.4 Use generateObject for structured output
- [ ] 4.5 Handle schema validation and error cases

## 5. Model Selection

- [ ] 5.1 Implement model selector based on research depth
- [ ] 5.2 Configure gpt-4o-mini for shallow research
- [ ] 5.3 Configure gpt-4o for medium/deep research
- [ ] 5.4 Add temperature configuration per research type

## 6. Tools Implementation

- [ ] 6.1 Create web search tool for term discovery
- [ ] 6.2 Implement save_research_plan tool
- [ ] 6.3 Implement load_research_plan tool
- [ ] 6.4 Implement list_authorities tool
- [ ] 6.5 Register tools with OpenCode tool registry

## 7. Multi-turn Conversation

- [ ] 7.1 Implement clarification question generation
- [ ] 7.2 Add state management for ongoing analysis
- [ ] 7.3 Handle user feedback and re-analysis
- [ ] 7.4 Implement conversation context preservation

## 8. Storage Integration

- [ ] 8.1 Create `.individualAgent/research/` directory structure
- [ ] 8.2 Implement plan save functionality (JSON format)
- [ ] 8.3 Implement plan load functionality
- [ ] 8.4 Add timestamp-based file naming
- [ ] 8.5 Handle storage errors gracefully

## 9. Supervisor Integration

- [ ] 9.1 Define TaskAnalysisResult message format
- [ ] 9.2 Implement result passing to Supervisor
- [ ] 9.3 Add confirmation flow before proceeding
- [ ] 9.4 Handle user modification requests

## 10. OpenCode Registration

- [ ] 10.1 Register TaskAnalysisAgent in `packages/individualagent/src/agent/agent.ts`
- [ ] 10.2 Create agent config file `.individualAgent/agents/research/task-analysis.md`
- [ ] 10.3 Add agent to the research agent group
- [ ] 10.4 Configure permissions and tools access

## 11. Testing

- [ ] 11.1 Write unit tests for schema validation
- [ ] 11.2 Write integration tests for LLM calls
- [ ] 11.3 Test prompt with various input scenarios
- [ ] 11.4 Test storage read/write operations
- [ ] 11.5 End-to-end test with sample research queries

## 12. Documentation

- [ ] 12.1 Add API documentation for TaskAnalysisAgent
- [ ] 12.2 Document configuration options
- [ ] 12.3 Create usage examples
- [ ] 12.4 Document error handling and troubleshooting