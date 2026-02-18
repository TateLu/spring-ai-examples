# Spring AI MCP 采样服务器

此示例项目演示了如何使用 Spring AI MCP Server Boot Starter 和 WebMVC 传输创建 MCP 服务器。它实现了一个天气服务,该服务公开了使用 Open-Meteo API 检索天气信���的工具,并展示了 MCP 采样功能。

有关更多信息,请参阅 [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html) 参考文档。

## 概述

该示例展示:
- 与 `spring-ai-mcp-server-webmvc-spring-boot-starter` 集成
- 支持 SSE (服务器发送事件) 和 STDIO 传输
- 使用 Spring AI 的 `@Tool` 注解自动工具注册
- 演示 LLM 提供者路由的 MCP 采样实现
- 检索温度数据并使用多个 LLM 生成创意响应的天气工具

## MCP 采样实现

此项目演示了 MCP 采样功能,该功能允许 MCP 服务器将某些请求委托给 LLM 提供者。实现包括:

1. **服务器端采样**: `WeatherService` 类实现了 `callMcpSampling` 方法,该方法:
   - 从工具上下文中提取 `McpSyncServerExchange`
   - 创建具有不同模型偏好的两个单独的消息请求:
     - 一个针对使用 `ModelPreferences.builder().addHint("openai").build()` 的 OpenAI 模型
     - 一个针对使用 `ModelPreferences.builder().addHint("anthropic").build()` 的 Anthropic 模型
   - 发送两个请求以生成关于天气数据的创意诗歌
   - 将响应组合为单个结果

2. **客户端采样**: 配套客户端项目 (`mcp-sampling-client`) 实现:
   - 处理采样请求的 `McpSyncClientCustomizer`
   - 根据模型提示将请求路由到适当的 LLM 的逻辑
   - 与 OpenAI 和 Anthropic 模型的集成

此方法演示了如何在单个应用程序中利用多个 LLM 提供者,从而实现创意内容生成和模型比较。

## 依赖项

该项目需要 Spring AI MCP Server WebMVC Boot Starter:

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-server-webmvc</artifactId>
</dependency>
```

此启动器提供:
- 使用 Spring MVC (`WebMvcSseServerTransport`) 基于 HTTP 的传输
- 自动配置的 SSE 端点
- 可选的 STDIO 传输
- 包含的 `spring-boot-starter-web` 和 `mcp-spring-webmvc` 依赖项

## 构建项目

使用 Maven 构建项目:

```bash
./mvnw clean package -DskipTests
```

## 运行服务器

服务器支持两种传输模式:

### WebMVC SSE 模式(默认)

```bash
java -jar target/mcp-sampling-server-0.0.1-SNAPSHOT.jar
```

### STDIO 模式

要启用 STDIO 传输,请设置适当的属性:

```bash
java -Dspring.ai.mcp.server.stdio=true -Dspring.main.web-application-type=none -jar target/mcp-sampling-server-0.0.1-SNAPSHOT.jar
```

## 配置

通过 `application.properties` 配置服务器:

```properties
# 服务器标识
spring.ai.mcp.server.name=mcp-sampling-server
spring.ai.mcp.server.version=0.0.1

# 日志记录配置
spring.main.banner-mode=off
logging.file.name=./model-context-protocol/sampling/mcp-sampling-server/target/server.log

# 取消注释以使用 STDIO 传输
# spring.ai.mcp.server.stdio=true
# spring.main.web-application-type=none
# logging.pattern.console=
```

## 可用工具

### 天气温度工具
- 名称: `getTemperature`
- 描述: 获取特定位置的温度(摄氏度)
- 参数:
  - `latitude`: double - 位置纬度
  - `longitude`: double - 位置经度
  - `toolContext`: ToolContext - 由 Spring AI 自动提供

此工具不仅从 Open-Meteo API 检索当前温度,还使用 MCP 采样从 OpenAI 和 Anthropic 模型生成关于天气的创意诗歌。

## 服务器实现

服务器使用 Spring Boot 和 Spring AI 的工具注解进行自动工具注册:

```java
@SpringBootApplication
public class McpServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }

    @Bean
    public ToolCallbackProvider weatherTools(WeatherService weatherService){
      return MethodToolCallbackProvider.builder().toolObjects(weatherService).build();
    }
}
```

`WeatherService` 使用 `@Tool` 注解实现天气工具,并包括 MCP 采样功能:

```java
@Service
public class WeatherService {
    @Tool(description = "获取特定位置的温度(摄氏度)")
    public String getTemperature(@ToolParam(description = "位置纬度") double latitude,
                                @ToolParam(description = "位置经度") double longitude,
                                ToolContext toolContext) {
        // 从 Open-Meteo API 检索天气数据
        WeatherResponse weatherResponse = restClient
            .get()
            .uri("https://api.open-meteo.com/v1/forecast?latitude={latitude}&longitude={longitude}&current=temperature_2m",
                latitude, longitude)
            .retrieve()
            .body(WeatherResponse.class);

        // 使用 MCP 采样生成创意响应
        String responseWithPoems = callMcpSampling(toolContext, weatherResponse);

        return responseWithPoems;
    }

    public String callMcpSampling(ToolContext toolContext, WeatherResponse weatherResponse) {
        // 使用 McpToolUtils.getMcpExchange() 访问 MCP 交换
        // 发送带有 OpenAI 和 Anthropic 的模型偏好的采样请求
        // 返回来自两个模型的诗歌以及天气数据
    }
}
```

## MCP 客户端

您可以使用 STDIO 或 SSE 传输连接到天气服务器:

### 手动客户端

#### WebMVC SSE 客户端

对于使用 SSE 传输的服务器:

```java
var transport = HttpClientSseClientTransport.builder("http://localhost:8080").build();
var client = McpClient.sync(transport).build();
```

#### STDIO 客户端

对于使用 STDIO 传输的服务器:

```java
var stdioParams = ServerParameters.builder("java")
    .args("-Dspring.ai.mcp.server.stdio=true",
          "-Dspring.main.web-application-type=none",
          "-Dspring.main.banner-mode=off",
          "-Dlogging.pattern.console=",
          "-jar",
          "target/mcp-sampling-server-0.0.1-SNAPSHOT.jar")
    .build();

var transport = new StdioClientTransport(stdioParams);
var client = McpClient.sync(transport).build();
```

示例项目包括示例客户端实现:
- [SampleClient.java](src/test/java/org/springframework/ai/mcp/sample/client/SampleClient.java): 支持采样的手动 MCP 客户端实现
- [ClientSse.java](src/test/java/org/springframework/ai/mcp/sample/client/ClientSse.java): SSE 传输连接
- [ClientStdio.java](src/test/java/org/springframework/ai/mcp/sample/client/ClientStdio.java): STDIO 传输连接

### 采样客户端

配套项目 `mcp-sampling-client` 演示了如何实现处理 MCP 采样请求的客户端:

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

要运行采样客户端:

1. 启动 MCP 服务器
2. 设置必需的环境变量:
   ```bash
   export OPENAI_API_KEY=your-openai-key
   export ANTHROPIC_API_KEY=your-anthropic-key
   ```
3. 导航到客户端目录并运行:
   ```bash
   cd ../mcp-sampling-client
   java -jar target/mcp-sampling-client-0.0.1-SNAPSHOT.jar
   ```

## 其他资源

* [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
* [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html)
* [MCP Client Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-client-docs.html)
* [模型上下文协议规范](https://modelcontextprotocol.github.io/specification/)
* [Spring Boot 自动配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration)
