# Spring AI LLM-as-a-Judge with Recursive Advisors Demo

此项目演示如何使用 Spring AI 的 **Recursive Advisors** 实现 **LLM-as-a-Judge** 评估模式。它展示了自动质量评估和自我改进功能，使 AI 系统能够迭代地评估和改进自己的响应。

## 概述

该演示实现了一个 `SelfRefineEvaluationAdvisor`，它：
- 使用专用的判断模型评估 AI 响应
- 使用 4 点评分系统（1=差，4=优秀）
- 自动使用建设性反馈重试失败的响应
- 演示通过使用单独的模型进行生成和评估来减轻偏差
- 展示递归模式如何实现自我改进的 AI 系统

## LLM-as-a-Judge 模式

LLM-as-a-Judge 是一种评估技术，其中大语言模型评估其他模型生成的输出质量。这种方法：

- **无需人工干预即可扩展评估**
- **与人类判断的一致性高达 85%**
- **为迭代改进提供结构化反馈**
- **在生产系统中实现自动质量控制**

### 评估标准

顾问根据以下标准评估响应：
- **相关性** - 直接回应用户的问题
- **完整性** - 覆盖问题中的所有方面
- **准确性** - 所提供信息的正确性
- **清晰度** - 响应的可读性和连贯性

## 项目结构

```
src/main/java/com/example/advisor/
├── EvaluationAdvisorDemoApplication.java    # 具有配置的主应用程序
├── SelfRefineEvaluationAdvisor.java         # 递归顾问实现
└── spring-ai-llm-as-judge-blog-post.md     # 详细技术文章
```

## 关键组件

### SelfRefineEvaluationAdvisor

实现评估循环的核心递归顾问：

```java
SelfRefineEvaluationAdvisor.builder()
    .chatClientBuilder(ChatClient.builder(judgeModel))  // 单独的判断模型
    .maxRepeatAttempts(15)                              // 最大重试次数
    .successRating(4)                                   // 最低可接受的评级
    .order(0)                                          // 链中的高优先级
    .build()
```

**功能：**
- **递归评估**：使用 `callAdvisorChain.copy(this).nextCall()` 进行迭代改进
- **结构化反馈**：返回 `EvaluationResponse(rating, evaluation, feedback)`
- **智能跳过逻辑**：避免评估工具调用和非文本响应
- **偏差减轻**：使用单独的 ChatClient 实例进行评估
- **可配置阈值**：可自定义的成功评级和重试限制

### 演示应用程序

该应用程序演示了一个现实世界的场景：
- **主模型**：Anthropic Claude 用于响应生成
- **判断模型**：Ollama（本地模型）用于评估。[Judge Arena](https://huggingface.co/blog/arena-atla) 比较了 LLM 判断模型。
- **工具集成**：天气工具，具有故意变化的响应
- **日志记录**：完整的请求/响应可观察性

## 前置条件

- Java 17+
- Maven 3.6+
- 对 Anthropic Claude 的 API 访问
- 本地运行 Ollama（用于判断模型）

## 设置

### 1. 安装 Ollama

```bash
# 安装 Ollama (macOS)
brew install ollama

# 启动 Ollama 服务
ollama serve

# 拉取一个合适的判断模型
ollama pull avcodes/flowaicom-flow-judge:q4
```

### 2. 配置 API 密钥

将您的 Anthropic API 密钥设置为环境变量：

```bash
export ANTHROPIC_API_KEY=your_anthropic_api_key
```

或者添加到 `src/main/resources/application.properties`：

```properties
spring.ai.anthropic.api-key=${ANTHROPIC_API_KEY}

spring.ai.ollama.chat.options.model=avcodes/flowaicom-flow-judge:q4
spring.ai.ollama.chat.options.temperature=0

spring.ai.chat.client.enabled=false

```

### 3. 运行应用程序

```bash
mvn spring-boot:run
```

## 预期行为

应用程序将：

1. **生成响应**：询问 Claude 关于巴黎的天气
2. **工具执行**：调用天气工具（返回随机温度）
3. **评估响应**：判断模型对响应进行评分（1-4 级别）
4. **需要时重试**：如果评级 < 4，则使用反馈重试
5. **记录进度**：显示所有尝试和评估
6. **返回最终**：评估通过或达到最大尝试次数后的最佳响应

### 示例输出

```
REQUEST: [{"role":"user","content":"巴黎现在的天气怎么样？"}]

>>> 工具调用响应温度：-255
第 1 次尝试评估失败，评估：响应包含不切实际的温度数据，反馈：-255°C 的温度在物理上是不可能的，表明数据错误。

>>> 工具调用响应温度：15
第 2 次尝试评估通过，评估：具有现实天气数据的优秀响应

RESPONSE: 巴黎的当前天气是晴朗的，温度为 15°C。
```

## 配置选项

### 顾问配置

```java
SelfRefineEvaluationAdvisor.builder()
    .successRating(3)           // 最低评级（1-4）
    .maxRepeatAttempts(5)       // 最大重试次数
    .order(0)                   // 执行顺序
    .skipEvaluationPredicate((request, response) ->
        response.chatResponse().hasToolCalls())  // 跳过条件
    .promptTemplate(customTemplate)  // 自定义评估提示
    .build()
```

### 模型选择

为了获得最佳结果：
- **生成模型**：高质量模型（GPT-4、Claude、Gemini）
- **判断模型**：来自 [Judge Arena 排行榜](https://huggingface.co/spaces/AtlaAI/judge-arena) 的专用评估模型
- **偏差减轻**：始终使用不同的模型进行生成和评估


## 生产环境考虑

### 性能优化
- 设置合理的 `maxRepeatAttempts`（3-5）以平衡质量和延迟
- 在高吞吐量场景中使用更快的判断模型
- 为重复评估实现缓存

### 错误处理
- 配置适当的回退策略
- 为持续的评估失败设置警报
- 监控令牌使用情况和 API 配额


## 相关示例

- [Recursive Advisor Demo](../recursive-advisor-demo) - 基本的递归模式
- [Spring AI Advisors 文档](https://docs.spring.io/spring-ai/reference/api/advisors.html)
- [ChatClient API 指南](https://docs.spring.io/spring-ai/reference/api/chatclient.html)

## 了解更多

📖 **博客文章**：[使用 Spring AI 构建 LLM 评估系统](./spring-ai-llm-as-judge-blog-post.md)

🔬 **研究论文**：
- [使用 MT-Bench 和 Chatbot Arena 判断 LLM-as-a-Judge](https://arxiv.org/abs/2306.05685)
- [使用 GPT-4 和更好的对齐进行 NLG 评估 (G-Eval)](https://arxiv.org/abs/2303.16634)
- [LLMs-as-Judges: 一项综合调查](https://arxiv.org/abs/2412.05579)

🏆 **判断模型**：[Judge Arena 排行榜](https://huggingface.co/spaces/AtlaAI/judge-arena)

## 许可证

本项目采用 Apache License 2.0 许可证 - 有关详细信息，请参阅 [LICENSE](../../LICENSE) 文件。
