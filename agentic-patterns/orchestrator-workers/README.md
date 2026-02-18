# Orchestrator-Workers Workflow Pattern

此项目演示了用于构建有效的基于 LLM 的系统的 Orchestrator-Workers 工作流模式，如 [Anthropic 关于构建有效智能体的研究](https://www.anthropic.com/research/building-effective-agents) 中所述。

![Orchestration Workflow](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F8985fc683fae4780fb34eab1365ab78c7e51bc8e-2401x1000.png&w=3840&q=75)

## 概述

Orchestrator-Workers 模式是一种用于处理需要动态任务分解和专门化处理的复杂任务的灵活方法。它由三个主要组件组成：

- **协调器**：一个中央 LLM，用于分析任务并确定所需的子任务
- **工作者**：执行特定子任务的专业化 LLM
- **合成器**：将工作者输出组合成最终结果的组件

## 何时使用

此模式在以下情况下特别有效：

- 无法预先预测子任务的复杂任务
- 需要不同方法或视角的任务
- 需要自适应问题解决的情况
- 从专门化处理中受益的任务


## 实现

该实现使用 Spring AI 的 ChatClient 进行 LLM 交互，包括：

```java
public class OrchestratorWorkers {
    public WorkerResponse process(String taskDescription) {
        // 1. 协调器分析任务并确定子任务
        OrchestratorResponse orchestratorResponse = // ...

        // 2. 工作者并行处理子任务
        List<String> workerResponses = // ...

        // 3. 结果组合成最终响应
        return new WorkerResponse(/*...*/);
    }
}
```

### 使用示例

```java
ChatClient chatClient = // ... 初始化聊天客户端
OrchestratorWorkers agent = new OrchestratorWorkers(chatClient);

// 处理一个任务
WorkerResponse response = agent.process(
    "为 REST API 端点生成技术和用户友好的文档"
);

// 访问结果
System.out.println("分析: " + response.analysis());
System.out.println("工作者输出: " + response.workerResponses());
```

## 自定义

该模式可以通过以下方式进行自定义：

1. **自定义提示**：为协调器和工作者提供专门的提示
```java
agent = new OrchestratorWorkers(
    chatClient,
    customOrchestratorPrompt,
    customWorkerPrompt
);
```

2. **默认模板**：为常见用例修改默认提示
   - `DEFAULT_ORCHESTRATOR_PROMPT`：任务分析的模板
   - `DEFAULT_WORKER_PROMPT`：工作者处理的模板

## 依赖项

- Spring AI
- Spring Boot
- Java 17 或更高版本

## 参考

- [Building Effective Agents (Anthropic Research)](https://www.anthropic.com/research/building-effective-agents)
