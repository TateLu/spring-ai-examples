# Spring AI Recursive Advisors Demo

一个 Spring Boot 演示项目，展示 Spring AI 1.1.0-M4+ 中的新 **Recursive Advisors** 功能，该功能可以在顾问链中循环多次以实现迭代 AI 工作流。

## 概述

此项目演示了使用 Spring AI 的 `ToolCallAdvisor` 的 **Recursive Advisors** 模式——这是一个在顾问链中处理工具调用循环的内置递归顾问。展示的主要功能：

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- Anthropic API 密钥

## 关键组件

### 1. 主应用程序（`RecursiveAdvisorDemoApplication.java`）

主类演示了递归顾问模式：

```java
ChatClient chatClient = chatClientBuilder
    .defaultTools(new MyTools())
    .defaultAdvisors(
        ToolCallAdvisor.builder().build(),  // 内置递归顾问
        new MyLogAdvisor())                 // 自定义日志记录顾问
    .build();
```

主要方面：
- **ToolCallAdvisor**：内置递归顾问，在所有工具调用完成之前循环
    - **用户控制的工具执行**：工具在顾问链内执行，而不是在模型内
- **顾问顺序**：多个顾问在链中一起工作

### 2. 自定义工具（`MyTools` 类）

```java
@Tool(description = "获取给定位置的当前天气")
public String weather(String location) {
    return location + " 的当前天气是晴朗的，温度为 25°C。";
}
```

演示如何创建 AI 在对话期间可以调用的自定义工具。

### 3. 自定义顾问（`MyLogAdvisor` 类）

实现非递归顾问以演示顾问链流程中的可观察性：

```java
static class MyLogAdvisor implements BaseAdvisor {
    @Override
    public ChatClientRequest before(ChatClientRequest request, AdvisorChain chain) {
        print("REQUEST", request.prompt().getInstructions());
        return request;
    }

    @Override
    public ChatClientResponse after(ChatClientResponse response, AdvisorChain chain) {
        print("RESPONSE", response.chatResponse().getResults());
        return response;
    }
}
```

此顾问在 `ToolCallAdvisor` 循环执行工具时记录每次迭代，提供对递归过程的可见性。

## 预期输出

```
REQUEST:[{"messageType":"USER","metadata":{"messageType":"USER"},"media":[],"text":"巴黎现在的天气怎么样？"}]

RESPONSE:[{"metadata":{"finishReason":"tool_use","contentFilters":[],"empty":true},"output":{"messageType":"ASSISTANT","metadata":{"messageType":"ASSISTANT"},"toolCalls":[],"media":[],"text":"我来为您查看巴黎的当前天气。"}},{"metadata":{"finishReason":"tool_use","contentFilters":[],"empty":true},"output":{"messageType":"ASSISTANT","metadata":{"messageType":"ASSISTANT"},"toolCalls":[{"id":"toolu_01HNde1Z7mwtwXh4qiK3tavZ","type":"function","name":"weather","arguments":"{\"location\":\"巴黎\"}"}],"media":[],"text":""}}]

REQUEST:[{"messageType":"USER","metadata":{"messageType":"USER"},"media":[],"text":"巴黎现在的天气怎么样？"},{"messageType":"ASSISTANT","metadata":{"messageType":"ASSISTANT"},"toolCalls":[{"id":"toolu_01HNde1Z7mwtwXh4qiK3tavZ","type":"function","name":"weather","arguments":"{\"location\":\"巴黎\"}"}],"media":[],"text":""},{"messageType":"TOOL","metadata":{"messageType":"TOOL"},"responses":[{"id":"toolu_01HNde1Z7mwtwXh4qiK3tavZ","name":"weather","responseData":"\"巴黎的当前天气是晴朗的，温度为 25°C。\""}],"text":""}]

RESPONSE:[{"metadata":{"finishReason":"end_turn","contentFilters":[],"empty":true},"output":{"messageType":"ASSISTANT","metadata":{"messageType":"ASSISTANT"},"toolCalls":[],"media":[],"text":"巴黎的当前天气是晴朗的，温度为 25°C。"}}]

巴黎的当前天气是晴朗的，温度为 25°C。
```
