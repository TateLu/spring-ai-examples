# Parallelization Workflow Pattern with Spring AI

此项目演示了使用 Spring AI 的 Parallelization Workflow 模式的实现，实现了多个大语言模型（LLM）操作的高效并发处理。该模式对于需要自动输出聚合的并行 LLM 调用场景特别有用。

![Parallelization Workflow](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F406bb032ca007fd1624f261af717d70e6ca86286-2401x1000.png&w=3840&q=75)

## 概述

Parallelization Workflow 模式以两种关键变体体现：

1. **分段**：将复杂任务分解为独立子任务以进行并发处理
2. **投票**：多次并行执行相同提示以获得不同视角或多数投票

## 主要优势

- 通过并发处理提高吞吐量
- 更好地利用 LLM API 容量
- 减少批量操作的总处理时间
- 通过多个视角提高结果质量（在投票场景中）

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- Ollama（本地 LLM 服务器）

## 设置

1. 克隆仓库
2. 构建项目：
   ```bash
   mvn clean install
   ```

## 使用示例

以下是使用 Parallelization Workflow 的基本示例：

```java
List<String> parallelResponse = new ParallelizationlWorkflow(chatClient)
    .parallel(
        "分析市场变化将如何影响此利益相关者群体。",
        List.of(
            "客户: ...",
            "员工: ...",
            "投资者: ...",
            "供应商: ..."
        ),
        4
    );
```

此示例演示了利益相关者分析的并行处理，其中每个利益相关者组都同时进行分析。

## 实现细节

`ParallelizationlWorkflow` 类提供核心实现，具有以下功能：

- 使用 `ExecutorService` 的固定线程池执行
- 匹配输入序列的有序结果保留
- 可配置的工作者线程数
- 内置的错误处理和资源管理
- 与 Spring AI 的 ChatClient 集成

### 何时使用

- 处理大量相似但独立的项目
- 需要多个独立视角或验证的任务
- 处理时间至关重要且任务可并行的场景
- 可以分解为独立子任务的复杂操作

### 实现注意事项

- 确保任务真正独立以避免一致性问题
- 在确定并行执行容量时考虑 API 速率限制
- 在扩展并行操作时监控资源使用（内存、CPU）
- 为并行任务失败实施适当的错误处理

## 许可证

本项目采用 Apache License 2.0 许可证 - 有关详细信息，请参阅 LICENSE 文件。

## 参考

- [Spring AI 文档](https://docs.spring.io/spring-ai/reference/1.0/api/chatclient.html)
- [Building Effective Agents - Anthropic](https://www.anthropic.com/research/building-effective-agents)
