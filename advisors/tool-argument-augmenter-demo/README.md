# Recursive Advisor with Memory and Tool Argument Augmentation

此示例演示如何使用 Spring AI 构建 **可解释的 AI 智能体**，通过在工具调用期间捕获 LLM 推理并与聊天记忆集成，以增强跨对话的上下文。

## 概述

在构建具有工具调用功能的 AI 智能体时，理解 **为什么** LLM 选择特定工具对于调试、可观察性和构建值得信赖的 AI 系统至关重要。此演示展示了 Spring AI [Tool Argument Augmenter](https://docs.spring.io/spring-ai/reference/2.0-SNAPSHOT/api/tools.html#tool-argument-augmentation) 工具，它实现：

- **捕获 LLM 推理**：在工具执行期间提取内部思维、置信度水平和记忆笔记
- **透明模式增强**：动态扩展工具模式，而无需修改底层工具实现
- **记忆增强推理**：在对话历史中持久化推理洞察，以在扩展交互中改进决策

## 工作原理

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          Tool Argument Augmenter Flow                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 用户询问："巴黎现在的天气怎么样？"                      │
│                         │                                               │
│                         ▼                                               │
│  2. 模式增强                                          │
│     原始：{ location: string }                                      │
│     增强：{ location: string, innerThought: string,                │
│                  confidence: string, memoryNotes: string[] }            │
│                         │                                               │
│                         ▼                                               │
│  3. 带推理的 LLM 响应                                         │
│     {                                                                   │
│       "location": "巴黎",                                              │
│       "innerThought": "用户想要巴黎的天气信息...",           │
│       "confidence": "high",                                             │
│       "memoryNotes": ["用户对巴黎天气感兴趣"]               │
│     }                                                                   │
│                         │                                               │
│                         ▼                                               │
│  4. 参数消费者处理推理（日志记录、记忆存储）     │
│                         │                                               │
│                         ▼                                               │
│  5. 原始工具仅接收：{ "location": "巴黎" }                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 关键组件

### AgentThinking 记录

定义要从 LLM 捕获的额外参数：

```java
public record AgentThinking(
    @ToolParam(description = "您调用此工具的分步推理",
               required = true)
    String innerThought,

    @ToolParam(description = "对此工具选择的置信度（低、中、高）",
               required = false)
    String confidence,

    @ToolParam(description = "需要为未来交互记住的关键洞察",
               required = true)
    List<String> memoryNotes
) {}
```

### AugmentedToolCallbackProvider

包装现有工具以透明地增强其模式：

```java
AugmentedToolCallbackProvider<AgentThinking> provider = AugmentedToolCallbackProvider
    .<AgentThinking>builder()
    .toolObject(new MyTools())
    .argumentType(AgentThinking.class)
    .argumentConsumer(event -> {
        AgentThinking thinking = event.arguments();
        logger.info("LLM Reasoning: {}", thinking.innerThought());
        logger.info("Confidence: {}", thinking.confidence());
        logger.info("Memory Notes: {}", thinking.memoryNotes());
    })
    .removeExtraArgumentsAfterProcessing(true)
    .build();
```

### 与顾问集成

将工具增强与 Spring AI 的顾问链结合：

```java
ChatClient chatClient = chatClientBuilder
    .defaultToolCallbacks(provider)
    .defaultAdvisors(
        ToolCallAdvisor.builder()
            .conversationHistoryEnabled(false).build(),
        MessageChatMemoryAdvisor.builder(chatMemory).build(),
        new MyLogAdvisor())
    .build();
```

## 运行示例

### 前置条件

- Java 17 或更高版本
- OpenAI API 密钥（或 Anthropic API 密钥）

### 配置

将您的 API 密钥设置为环境变量：

```bash
export OPENAI_API_KEY=your-api-key
# 或
export ANTHROPIC_API_KEY=your-api-key
```

### 构建和运行

```bash
cd advisors/recursive-advisor-with-memory
./mvnw spring-boot:run
```

## 示例输出

运行示例时，您将在每次工具调用之前看到捕获的 LLM 推理：

```
LLM Reasoning: 用户询问巴黎的当前天气。我需要调用天气工具...
Confidence: high
Memory Notes: [用户对巴黎天气感兴趣, 可能需要后续关于活动的信息]
Tool: weather

REQUEST: [{"type":"USER","text":"巴黎现在的天气怎么样？"}]

RESPONSE: [{"output":{"text":"巴黎的当前天气是晴朗的，温度为 25°C。"}}]
```

## 用例

- **调试**：理解您的 AI 智能体为何做出特定的工具选择
- **可观察性**：在生产环境中记录和监控智能体推理
- **记忆增强**：存储洞察以在未来的对话中改进上下文
- **多智能体协调**：在智能体之间传递协调信号
- **分析**：跟踪工具使用模式和决策质量

## 资源

- [可解释的 AI 智能体博客文章](https://spring.io/blog/2025/12/21/explainable-ai-agents-capture-llm-tool-call-reasoning-with-spring-ai)
- [Spring AI 工具调用文档](https://docs.spring.io/spring-ai/reference/api/tools.html)
- [Spring AI 顾问指南](https://docs.spring.io/spring-ai/reference/api/advisors.html)
- [Tool Argument Augmenter](https://docs.spring.io/spring-ai/reference/2.0-SNAPSHOT/api/tools.html#tool-argument-augmentation)
