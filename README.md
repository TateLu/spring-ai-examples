# Spring AI 示例集合

[![集成测试](https://github.com/spring-projects/spring-ai-examples/actions/workflows/integration-tests.yml/badge.svg)](https://github.com/spring-projects/spring-ai-examples/actions/workflows/integration-tests.yml)

一个全面的 Spring AI 示例集合，展示如何使用 Spring Boot 构建基于大语言模型（LLM）的应用程序。这个多模块 Maven 项目展示了各种 AI 集成模式，从基础的聊天交互到高级的智能体工作流。

## 📚 所有已翻译的模块文档

以下所有模块的 README.md 文档已翻译为中文，您可以直接访问：

### 智能体模式 (Agentic Patterns)

- **[README.md](agentic-patterns/README.md)** - 智能体模式概述
- **[编排器-工作器](agentic-patterns/orchestrator-workers/README.md)** - Orchestrator-Workers 模式
- **[链式工作流](agentic-patterns/chain-workflow/README.md)** - Prompt Chaining 模式
- **[并行化工作流](agentic-patterns/parallelization-workflow/README.md)** - Parallelization 模式
- **[路由工作流](agentic-patterns/routing-workflow/README.md)** - Routing 模式
- **[评估器-优化器](agentic-patterns/evaluator-optimizer/README.md)** - Evaluator-Optimizer 模式

### 模型上下文协议 (Model Context Protocol - MCP)

- **[Brave 搜索](model-context-protocol/brave/README.md)** - Brave 搜索集成
- **[Brave Docker 网关](model-context-protocol/brave-docker-agents-gateway/README.md)** - Docker MCP 网关
- **[客户端 Starter - Default](model-context-protocol/client-starter/starter-default-client/README.md)** - 默认客户端
- **[客户端 Starter - WebFlux](model-context-protocol/client-starter/starter-webflux-client/README.md)** - WebFlux 客户端
- **[MCP 注解 - 服务器](model-context-protocol/mcp-annotations/mcp-annotations-server/README.md)** - 注解方式 MCP 服务器
- **[MCP 注解 - 概述](model-context-protocol/mcp-annotations/README.md)** - MCP 注解示例
- **[采样 - 概述](model-context-protocol/sampling/README.md)** - MCP 采样概述
- **[采样 - 注解](model-context-protocol/sampling/annotations/README.md)** - 注解方式 MCP 采样
- **[采样 - 服务器](model-context-protocol/sampling/mcp-sampling-server/README.md)** - MCP 采样服务器
- **[采样 - 客户端](model-context-protocol/sampling/mcp-sampling-client/README.md)** - MCP 采样客户端
- **[文件系统](model-context-protocol/filesystem/README.md)** - 文件系统访问
- **[SQLite - 聊天机器人](model-context-protocol/sqlite/chatbot/README.md)** - SQLite 聊天机器人
- **[SQLite - 简单](model-context-protocol/sqlite/simple/README.md)** - SQLite 简单示例
- **[Web 搜索 - 聊天机器人](model-context-protocol/web-search/brave-chatbot/README.md)** - Brave 搜索聊天机器人
- **[Web 搜索 - Starter](model-context-protocol/web-search/brave-starter/README.md)** - Brave 搜索 Starter
- **[天气 - OAuth2 WebMVC](model-context-protocol/weather/starter-webmvc-oauth2-server/README.md)** - OAuth2 天气服务器
- **[天气 - STDIO](model-context-protocol/weather/starter-stdio-server/README.md)** - STDIO 天气服务器
- **[天气 - WebMVC](model-context-protocol/weather/starter-webmvc-server/README.md)** - WebMVC 天气服务器
- **[天气 - 手动 WebFlux](model-context-protocol/weather/manual-webflux-server/README.md)** - 手动 WebFlux 服务器
- **[天气 - WebFlux](model-context-protocol/weather/starter-webflux-server/README.md)** - WebFlux 天气服务器
- **[动态工具更新](model-context-protocol/dynamic-tool-update/README.md)** - 动态工具更新

### 顾问模式 (Advisors)

- **[递归顾问演示](advisors/recursive-advisor-demo/README.md)** - 递归顾问模式
- **[工具参数增强演示](advisors/tool-argument-augmenter-demo/README.md)** - 工具参数增强
- **[评估递归顾问演示](advisors/evaluation-recursive-advisor-demo/README.md)** - LLM 评估演示

### 提示工程 (Prompt Engineering)

- **[提示工程模式](prompt-engineering/prompt-engineering-patterns/README.md)** - 提示工程模式

### 智能体 (Agents)

- **[反射](agents/reflection/README.md)** - 反射智能体

### Kotlin 示例

- **[Kotlin Hello World](kotlin/kotlin-hello-world/README.md)** - Kotlin Hello World 应用
- **[Kotlin 函数回调](kotlin/kotlin-function-callback/README.md)** - Kotlin 函数回调演示

### 杂项 (Misc)

- **[Spring AI Java 函数回调](misc/spring-ai-java-function-callback/README.md)** - Java 函数回调
- **[OpenAI 流式响应](misc/openai-streaming-response/README.md)** - 流式集成
- **[Document Forge](misc/claude-skills-demo/document-forge/README.md)** - Claude Skills 演示

### 基础模型示例 (Models)

- **[Hello World 聊天](models/chat/helloworld/README.md)** - 基础聊天模型

## 目录

- [快速开始](#快速开始)
- [项目结构](#项目结构)
- [学习路径](#学习路径)
- [模块分类](#模块分类)
- [运行示例](#运行示例)
- [环境要求](#环境要求)

## 快速开始

```bash
# 克隆仓库
git clone <repository-url>
cd spring-ai-examples

# 构建整个项目
./mvnw clean package

# 运行特定示例
cd <module-directory>
./mvnw spring-boot:run
```

## 项目结构

```
spring-ai-examples/
├── agentic-patterns/          # 5 种 LLM 工作流模式
├── model-context-protocol/    # MCP 客户端/服务器实现
├── advisors/                  # ChatClient 顾问模式
├── prompt-engineering/        # 提示工程模式
├── agents/                    # 高级智能体实现
├── kotlin/                    # Kotlin 示例
├── misc/                      # 函数回调、流式响应
├── models/                    # 基础模型示例
└── integration-testing/       # 集成测试框架
```

## 学习路径

### 初学者路径（第 1-3 周）

| 序号 | 模块 | 描述 |
|-----|------|------|
| 1 | `models/chat/helloworld` | 基础聊天模型 |
| 2 | `prompt-engineering/prompt-engineering-patterns` | 提示工程 |
| 3 | `misc/spring-ai-java-function-callback` | 函数回调 |
| 4 | `agentic-patterns/chain-workflow` | 顺序工作流 |
| 5 | `model-context-protocol/sqlite/simple` | MCP 基础 |
| 6 | `model-context-protocol/filesystem` | 文件系统访问 |

### 中级路径（第 4-6 周）

| 序号 | 模块 | 描述 |
|-----|------|------|
| 7 | `agentic-patterns/routing-workflow` | 分类路由 |
| 8 | `agentic-patterns/parallelization-workflow` | 并行处理 |
| 9 | `agentic-patterns/orchestrator-workers` | 协调器模式 |
| 10 | `advisors/recursive-advisor-demo` | 递归顾问 |
| 11 | `advisors/tool-argument-augmenter-demo` | 参数增强 |
| 12 | `model-context-protocol/weather/starter-webmvc-server` | Web 服务器 |
| 13 | `model-context-protocol/dynamic-tool-update` | 运行时工具更新 |

### 高级路径（第 7-9 周）

| 序号 | 模块 | 描述 |
|-----|------|------|
| 13 | `agents/reflection` | 自我反思智能体 |
| 14 | `agentic-patterns/evaluator-optimizer` | 迭代优化 |
| 15 | `kotlin/rag-with-kotlin` | RAG 实现 |
| 16 | `model-context-protocol/brave` | 网络搜索集成 |
| 17 | `model-context-protocol/weather/starter-webmvc-oauth2-server` | OAuth2 安全 |
| 18 | `model-context-protocol/sampling/annotations/README.md` | MCP 采样注解 |

## 模块分类

### 智能体模式 (Agentic Patterns)

基于 [Anthropic 研究](https://www.anthropic.com/research/building-effective-agents) 的 LLM 工作流模式：

| 模式 | 描述 | 使用场景 |
|------|------|----------|
| **链式工作流** | 顺序 LLM 调用 | 数据转换流水线 |
| **并行化** | 并发处理（分段/投票） | 批量文档处理 |
| **路由** | 基于分类的定向 | 客服工单路由 |
| **协调器-工作者** | 中央协调器 + 专门工作者 | 复杂代码生成 |
| **评估器-优化器** | 迭代生成 + 评估 | 代码审查、内容优化 |

### 模型上下文协议 (MCP)

[模型上下文协议](https://modelcontextprotocol.io) 的实现，用于标准化 LLM 交互：

| 类别 | 模块 |
|------|------|
| **SQLite** | `sqlite/simple`, `sqlite/chatbot` |
| **天气服务** | `weather/manual-webflux-server`, `weather/starter-*` (4 个变体) |
| **网络搜索** | `web-search/brave-starter`, `web-search/brave-chatbot` |
| **文件系统** | `filesystem` |
| **高级功能** | `dynamic-tool-update`, `mcp-annotations`, `sampling` |

### 顾问模式 (Advisors)

ChatClient 自定义模式：

| 模块 | 描述 |
|------|------|
| `recursive-advisor-demo` | 自我改进顾问 |
| `tool-argument-augmenter-demo` | 动态参数调整 |
| `evaluation-recursive-advisor-demo` | 需要评估的顾问（需要 Ollama） |

### 其他分类

| 类别 | 模块 |
|------|------|
| **提示工程** | `prompt-engineering-patterns` |
| **智能体** | `reflection` |
| **Kotlin** | `kotlin-hello-world`, `kotlin-function-callback`, `rag-with-kotlin` |
| **杂项** | `spring-ai-java-function-callback`, `openai-streaming-response`, `claude-skills-demo` |

## 运行示例

### 运行单个模块

```bash
cd <module-directory>
./mvnw clean package
./mvnw spring-boot:run
```

### 运行集成测试

```bash
# 运行所有集成测试
./integration-testing/scripts/run-integration-tests.sh

# 运行特定模块测试
./integration-testing/scripts/run-integration-tests.sh <module-name>

# 创建新的集成测试
python3 integration-testing/scripts/scaffold_integration_test.py <module-path>
```

### Spring AI 版本管理

```bash
# 更新所有模块的 Spring AI 版本
./scripts/update-spring-ai-version.sh 1.0.0

# 检查当前版本
./scripts/check-spring-ai-version.sh

# 从备份恢复
./scripts/restore-spring-ai-version.sh /path/to/backup
```

## 环境要求

- **Java**: 17+
- **Maven**: 已包含 Maven Wrapper (`./mvnw`)
- **API 密钥**:
  - `OPENAI_API_KEY`（大部分示例）
  - `ANTHROPIC_API_KEY`（AI 验证，部分示例）
  - `BRAVE_API_KEY`（网络搜索示例）

### 可选依赖

- **JBang**: 用于运行集成测试
- **Docker**: 用于 RAG (pgvector)、Ollama 示例

## 技术栈

- **Spring Boot**: 4.0.0
- **Spring AI**: 2.0.0-SNAPSHOT
- **Java**: 17
- **构建工具**: Maven（多模块项目）

## 文档

- [CLAUDE.md](CLAUDE.md) - 贡献者开发指南
- [集成测试指南](integration-testing/docs/README.md) - 测试框架文档
- 各目录下的模块级 README

## 许可证

本项目采用与 Spring AI 相同的许可证条款。

## 相关链接

- [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
- [Spring AI 聊天模型](https://docs.spring.io/spring-ai/reference/api/chatmodel.html)
- [Spring AI ChatClient](https://docs.spring.io/spring-ai/reference/api/chatclient.html)
- [构建有效智能体（Anthropic 研究）](https://www.anthropic.com/research/building-effective-agents)
- [模型上下文协议](https://modelcontextprotocol.io)
