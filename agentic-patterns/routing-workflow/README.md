# Routing Workflow Pattern

此项目实现了 [Anthropic 的 Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) 中描述的 Routing 工作流模式。该模式实现了根据内容分类将输入智能地路由到专门处理程序的功能。

![Routing Workflow](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F5c0c0e9fe4def0b584c04d37849941da55e5e71c-2401x1000.png&w=3840&q=75)

## 概述

Routing 工作流模式是为不同类型的输入由专门的流程处理得更好的复杂任务而设计的。它使用 LLM 分析输入内容并将其路由到最合适的专门提示或处理程序。

### 主要优势

- **关注点分离**：每个路由都可以针对特定类型的输入进行优化
- **提高准确性**：专门的提示处理它们最擅长的内容
- **可扩展的架构**：易于添加新路由和专门的处理程序
- **更好的资源利用**：将更简单的任务路由到更轻量的模型

### 何时使用

此工作流在以下情况下特别有效：

- 您有需要不同处理的明显不同类别的输入
- 分类可以由 LLM 准确处理
- 不同类型的输入需要不同的专业知识或处理方法

## 使用示例

```java
@Autowired
private ChatClient chatClient;

// 创建工作流
RoutingWorkflow workflow = new RoutingWorkflow(chatClient);

// 为不同类型的输入定义专门的提示
Map<String, String> routes = Map.of(
    "billing", "您是计费专家。帮助解决计费问题...",
    "technical", "您是技术支持工程师。帮助解决技术问题...",
    "general", "您是客户服务代表。帮助处理一般咨询..."
);

// 处理输入
String input = "我的账户上周被扣了两次";
String response = workflow.route(input, routes);
```

## 常见用例

1. **客户支持**
   - 将查询路由到适当的部门（计费、技术、一般）
   - 将紧急问题引导到优先处理
   - 将复杂案例转发给专业团队

2. **内容审核**
   - 将内容路由到适当的审核流程
   - 将敏感内容引导给人工审核员
   - 将常规内容通过自动化检查

3. **查询优化**
   - 将简单问题路由到更小、更快的模型
   - 将复杂查询引导到更强大的模型
   - 将专业主题转发到领域特定的处理程序

## 实现细节

该实现由两个主要组件组成：

1. `RoutingWorkflow`：实现路由逻辑的主要类
   - 使用 LLM 分析输入
   - 选择适当的路由
   - 使用专门的提示处理输入

2. `RouteResponse`：封装路由决策的记录类
   - 存储路由选择背后的推理
   - 维护选定的路由信息

## 参考

- [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) - Anthropic Research
- [Spring AI 文档](https://docs.spring.io/spring-ai/reference/1.0/api/chatclient.html)
