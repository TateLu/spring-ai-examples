# Spring AI MCP 采样客户端

此项目演示了如何使用 Spring AI 实现用于 MCP (模型上下文协议) 采样功能的客户端。它展示了如何根据模型偏好将 LLM 请求路由到不同的提供者 (OpenAI 和 Anthropic)。

## 概述

MCP 采样客户端:
- 使用 SSE (服务器发送事件) 传输连接到 MCP 服务器
- 实现了一个将请求路由到不同 LLM 提供者的采样处理程序
- 与 OpenAI 和 Anthropic 模型集成
- 演示了如何使用模型提示选择适当的 LLM
- 将来自多个 LLM 的创意响应组合为单个结果
- 为 MCP 操作提供日志记录功能

## MCP 采样实现

MCP 采样是一个强大��功能,允许 MCP 服务器将某些请求委托给 LLM 提供者。此客户端实现了采样请求的客户端处理:

1. **采样处理程序注册**: 客户端使用 `McpSyncClientCustomizer` 注册采样处理程序:

```java
@Bean
McpSyncClientCustomizer samplingCustomizer(Map<String, ChatClient> chatClients) {
    return (name, mcpClientSpec) -> {

        mcpClientSpec = mcpClientSpec.loggingConsumer(logingMessage -> {
            System.out.println("MCP 日志记录: [" + logingMessage.level() + "] " + logingMessage.data());
        });

        mcpClientSpec.sampling(llmRequest -> {
            var userPrompt = ((McpSchema.TextContent) llmRequest.messages().get(0).content()).text();
            String modelHint = llmRequest.modelPreferences().hints().get(0).name();

            // 根据模型提示查找适当的聊天客户端
            ChatClient hintedChatClient = chatClients.entrySet().stream()
                    .filter(e -> e.getKey().contains(modelHint)).findFirst()
                    .orElseThrow().getValue();

            // 使用选定的模型生成响应
            String response = hintedChatClient.prompt()
                    .system(llmRequest.systemPrompt())
                    .user(userPrompt)
                    .call()
                    .content();

            return CreateMessageResult.builder().content(new McpSchema.TextContent(response)).build();
        });
        System.out.println("自定义 " + name);
    };
}
```

2. **模型路由**: 客户端使用请求中的模型提示选择适当的 LLM 提供者:
   - 如果提示是 "openai",则路由到 OpenAI 模型
   - 如果提示是 "anthropic",则路由到 Anthropic 模型

3. **聊天客户端管理**: 客户端创建和管理多个聊天客户端,每个 LLM 提供者一个:

```java
@Bean
public Map<String, ChatClient> chatClients(List<ChatModel> chatModels) {
    return chatModels.stream().collect(Collectors.toMap(
        model -> model.getClass().getSimpleName().toLowerCase(),
        model -> ChatClient.builder(model).build()));
}
```

4. **与 Spring AI 集成**: 客户端利用 Spring AI 的自动配置设置必要的组件:

```java
var mcpToolProvider = new SyncMcpToolCallbackProvider(mcpClients);

ChatClient chatClient = ChatClient.builder(openAiChatModel)
    .defaultToolCallbacks(mcpToolProvider)
    .build();
```

## 依赖项

该项目需要以下 Spring AI 依赖项:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-mcp-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-openai</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-anthropic</artifactId>
    </dependency>
</dependencies>
```

## 配置

### 应用程序属性

应用程序通过 `application.properties` 进行配置:

```properties
spring.application.name=mcp
spring.main.web-application-type=none

# 禁用聊天客户端自动配置,因为我们使用多个聊天模型
spring.ai.chat.client.enabled=false

# LLM 提供者的 API 密钥
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.anthropic.api-key=${ANTHROPIC_API_KEY}

# MCP 服务器连接
spring.ai.mcp.client.sse.connections.server1.url=http://localhost:8080

# 日志记录配置
logging.level.io.modelcontextprotocol.client=WARN
logging.level.io.modelcontextprotocol.spec=WARN

#禁用 MCP 工具回调
spring.ai.mcp.client.toolcallback.enabled=false
```

## 工作原理

该应用程序演示了使用与天气相关查询的 MCP 采样:

1. 客户端连接到 MCP 天气服务器
2. 它发送一个与天气相关的问题:"阿姆斯特丹现在天气如何?"
3. 服务器检索天气数据并将采样请求发送到客户端
4. 客户端根据模型提示将每个请求路由到适当的 LLM
5. 每个 LLM 生成关于天气的创意诗歌
6. 服务器组合响应并将它们返回给用户

## 运行应用程序

1. 首先,启动 MCP 天气服务器:
   ```bash
   cd ../mcp-sampling-server
   ./mvnw clean install -DskipTests
   java -jar target/mcp-sampling-server-0.0.1-SNAPSHOT.jar
   ```

2. 设置必需的环境变量:
   ```bash
   export OPENAI_API_KEY=your-openai-key
   export ANTHROPIC_API_KEY=your-anthropic-key
   ```

3. 构建并运行 MCP 采样客户端:
   ```bash
   cd ../mcp-sampling-client
   ./mvnw clean install
   java -Dai.user.input='阿姆斯特丹现在天气如何?' -jar target/mcp-sampling-client-0.0.1-SNAPSHOT.jar
   ```

应用程序将:
1. 连接到 MCP 天气服务器
2. 处理天气查询
3. 处理来自服务器的采样请求
4. 将每个请求路由到适当的 LLM
5. 显示组合的创意响应

## 示例输出

当您运行应用程序时,您将看到类似的输出:

```
> 用户: 阿姆斯特丹现在天气如何?
请包含所有 LLM 提供者的创意响应。
在其他提供者之后,添加一首综合所有其他提供者诗歌的诗。

> 助手: 我为您检查了阿姆斯特丹的当前天气。以下是来自不同 AI 提供者的创意响应:

OpenAI 关于天气的诗歌:
# 阿姆斯特丹的拥抱

在阿姆斯特丹,运河映照天空,
十六度的温暖轻轻拂过。
秋天的空气,清新而温柔,
抚摸着荷兰人的脸庞。

时间仿佛在这完美的时刻静止,
随着温和的阳光展现出温柔的力量。
城市带着平静的喜悦呼吸,
此刻,一切感觉都是对的。

Anthropic 关于天气的诗歌:
## 阿姆斯特丹今天

阿姆斯特丹的拥抱中,十六度,
完美的秋日以优雅展开。
运河映照着上方飘过的云朵,
当城市充满生活和爱时。

既不冷也不暖,而是恰到好处,
这荷兰场景的完美天气。
时间在这个时刻的温柔拥抱中被捕捉,
随着温和的阳光展示其微妙的力量。

城市呼吸着,充满平静的喜悦,
在那一刻,一切感觉恰到好处。

阿姆斯特丹的故事,继续展开,
拥抱其中的一切。

完美的一天,诗人无法忽视,
在这个我们都热爱的美丽城市。

当前天气数据显示阿姆斯特丹现在的温度为 16°C。
```

## 其他资源

* [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
* [MCP 客户端启动器](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html)
* [模型上下文协议规范](https://modelcontextprotocol.github.io/specification/)
* [Spring Boot 文档](https://docs.spring.io/spring-boot/docs/current/reference/html/)
