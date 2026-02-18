# Spring AI MCP 采样示例

此目录包含演示 Spring AI 中模型上下文协议 (MCP) 采样功能的示例。MCP 采样允许 MCP 服务器将某些请求委托给 LLM 提供者,从而实现多模型交互和创意内容生成。

## 概述

MCP 采样示例展示:
- MCP 服务器如何将 LLM 请求委托给客户端
- 客户端如何根据模型提示将请求路由到不同的 LLM 提供者
- 与多个 LLM 提供者 (OpenAI 和 Anthropic) 的集成
- 使用多个模型的创意内容生成
- 将来自不同 LLM 的响应组合为统一结果

## 项目

此目录包含几个演示 MCP 采样的项目:

### 主项目
1. **[mcp-sampling-server](./mcp-sampling-server)**: 一个提供天气信息并使用 MCP 采样生成创意内容的 MCP 服务器
2. **[mcp-sampling-client](./mcp-sampling-client)**: 一个处理采样请求并将它们路由到不同 LLM 提供者的 MCP 客户端

### 基于注解的示例

展示与主采样示例相同的 MCP 采样功能,但使用基于注解的方法进行简化开发 - 在 **[README](./annotations/README.md)** 中查找更多内容

3. **[annotations/mcp-sampling-server-annotations](./annotations/mcp-sampling-server-annotations)**: 使用 Spring AI 基于注解的方法的服务器实现
4. **[annotations/mcp-sampling-client-annotations](./annotations/mcp-sampling-client-annotations)**: 使用 Spring AI 基于注解的方法的客户端实现

## 什么是 MCP 采样?

MCP 采样是模型上下文协议的一个强大功能,它允许:

- MCP 服务器将某些请求委托给 LLM 提供者
- 指定模型偏好以定位特定的 LLM 提供者
- 基于模型提示路由请求
- 组合来自多个 LLM 的响应

这种方法使应用程序能够在单个工作流中利用多个 LLM 提供者,从而实现创意内容生成、模型比较和专业任务委托。

## 工作原理

这些示例中的 MCP 采样工作流程遵循以下步骤:

1. **客户端初始化**:
   - 客户端连接到 MCP 天气服务器
   - 它注册了一个可以路由请求到不同 LLM 提供者的采样处理程序

2. **用户查询**:
   - 用户向客户端发送与天气相关的查询
   - 客户端将查询转发到 MCP 天气服务器

3. **服务器处理**:
   - 服务器从 Open-Meteo API 检索天气数据
   - 它从工具上下文中提取 `McpSyncServerExchange`
   - 它创建具有不同模型偏好的采样请求:
     - 一个针对使用 `ModelPreferences.builder().addHint("openai").build()` 的 OpenAI
     - 一个针对使用 `ModelPreferences.builder().addHint("anthropic").build()` 的 Anthropic

4. **采样委托**:
   - 服务器将采样请求发送回客户端
   - 客户端的采样处理程序接收请求

5. **模型路由**:
   - 客户端从每个请求中提取模型提示
   - 它根据提示选择适当的 LLM 提供者
   - 它将提示转发到选定的 LLM

6. **响应生成**:
   - 每个 LLM 生成创意响应(在本例中,是关于天气的诗歌)
   - 客户端将响应返回给服务器

7. **结果组合**:
   - 服务器组合来自不同 LLM 的响应
   - 它将组合结果返回给用户

![MCP 采样序列图](./mvc-sampling-sq.svg)

## 服务器实现

MCP 天气服务器实现了 MCP 采样的服务器端:

```java
public String callMcpSampling(ToolContext toolContext, WeatherResponse weatherResponse) {
    StringBuilder openAiWeatherPoem = new StringBuilder();
    StringBuilder anthropicWeatherPoem = new StringBuilder();

    McpToolUtils.getMcpExchange(toolContext)
            .ifPresent(exchange -> {
                if (exchange.getClientCapabilities().sampling() != null) {
                    var messageRequestBuilder = McpSchema.CreateMessageRequest.builder()
                            .systemPrompt("你是一个诗人!")
                            .messages(List.of(new McpSchema.SamplingMessage(McpSchema.Role.USER,
                                    new McpSchema.TextContent(
                                            "请写一首关于这个天气预报的诗歌(温度为摄氏度)。使用 markdown 格式 :\n "
                                                    + ModelOptionsUtils.toJsonStringPrettyPrinter(weatherResponse)))));

                    // 从 OpenAI 请求诗歌
                    var openAiLlmMessageRequest = messageRequestBuilder
                            .modelPreferences(ModelPreferences.builder().addHint("openai").build())
                            .build();
                    CreateMessageResult openAiLlmResponse = exchange.createMessage(openAiLlmMessageRequest);
                    openAiWeatherPoem.append(((McpSchema.TextContent) openAiLlmResponse.content()).text());

                    // 从 Anthropic 请求诗歌
                    var anthropicLlmMessageRequest = messageRequestBuilder
                            .modelPreferences(ModelPreferences.builder().addHint("anthropic").build())
                            .build();
                    CreateMessageResult anthropicAiLlmResponse = exchange.createMessage(anthropicLlmMessageRequest);
                    anthropicWeatherPoem.append(((McpSchema.TextContent) anthropicAiLlmResponse.content()).text());
                }
            });

    // 组合响应
    return "OpenAI 关于天气的诗歌: " + openAiWeatherPoem.toString() + "\n\n" +
           "Anthropic 关于天气的诗歌: " + anthropicWeatherPoem.toString() + "\n" +
           ModelOptionsUtils.toJsonStringPrettyPrinter(weatherResponse);
}
```

## 客户端实现

MCP 采样客户端实现了采样请求的客户端处理:

```java
@Bean
McpSyncClientCustomizer samplingCustomizer(Map<String, ChatClient> chatClients) {
    return (name, spec) -> {
        spec.sampling(llmRequest -> {
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
    };
}
```

注意: 为防止循环依赖,您必须禁用 MCP 工具回调自动配置: `spring.ai.mcp.client.toolcallback.enabled=false`

## 运行示例

### 前置条件

- Java 17 或更高版本
- Maven 3.6+
- OpenAI API 密钥
- Anthropic API 密钥

### 步骤 1: 启动 MCP 天气服务器

```bash
cd mcp-sampling-server
./mvnw clean package -DskipTests
java -jar target/mcp-sampling-server-0.0.1-SNAPSHOT.jar
```

### 步骤 2: 设置环境变量

```bash
export OPENAI_API_KEY=your-openai-key
export ANTHROPIC_API_KEY=your-anthropic-key
```

### 步骤 3: 运行 MCP 采样客户端

```bash
cd mcp-sampling-client
./mvnw clean package
java -Dai.user.input='阿姆斯特丹现在天气如何?' -jar target/mcp-sampling-client-0.0.1-SNAPSHOT.jar
```

## 示例输出

当您运行应用程序时,您将看到来自 OpenAI 和 Anthropic 模型的创意响应,每个模型都生成一首关于阿姆斯特丹当前天气的诗歌。

## 相关项目

Spring AI 提供了几个 MCP 客户端和服务器实现:

- **[MCP 注解](../mcp-annotations)**: 使用 Spring AI 基于注解的 MCP 方法的示例
- **[天气服务器](../weather)**: 一个简单的天气 MCP 服务器示例
- **[SQLite 服务器](../sqlite)**: 一个提供 SQLite 数据库访问的 MCP 服务器
- **[文件系统服务器](../filesystem)**: 一个用于文件系统操作的 MCP 服务器

## 其他资源

* [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
* [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html)
* [MCP Client Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html)
* [模型上下文协议规范](https://modelcontextprotocol.github.io/specification/)
* [Spring Boot 文档](https://docs.spring.io/spring-boot/docs/current/reference/html/)
