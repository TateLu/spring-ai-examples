# Spring AI MCP 注解示例

此目录包含演示使用 Spring AI 基于注解的方法的模型上下文协议 (MCP) 的示例。这些示例展示了 MCP 功能,包括使用声明式注解的工具、资源、提示、自动完成和客户端处理程序。

## 概述

MCP 注解示例演示:

- **注解驱动开发**: 使用 `@McpTool`、`@McpResource`、`@McpPrompt`、`@McpComplete` 和客户端注解简化 MCP 开发
- **声明式配置**: 通过注解自动注册和配置
- **客户端-服务器通信**: 完整的双向 MCP 通信模式
- **多个 MCP 功能**: 工具、资源、提示、自动完成和客户端处理程序
- **混合注解样式**: 同时支持 MCP 注解和 Spring AI `@Tool` 注解

## 项目

### 1. mcp-annotations-server
一个全面的 MCP 服务器实现,展示使用注解的所有主要 MCP 功能。

#### 工具
- **天气工具**: 使用 `@McpTool` 从 Open-Meteo API 获取温度数据
- **其他工具**: 使用 Spring AI `@Tool` 从 Weather.gov API 获取天气预报和警报

#### 资源
- **用户配置文件**: 具有多种访问模式的完整用户信息
- **用户属性**: 使用 URI 变量的单个属性访问
- **用户状态**: 具有不同返回类型的动态状态信息
- **用户连接**: 社交连接和关系
- **用户通知**: 动态通知生成
- **用户头像**: 带有自定义 MIME 类型的二进制数据处理
- **用户位置**: 简单的位置信息

#### 提示
- **问候提示**: 简单的参数化问候
- **个性化消息**: 具有多个参数和逻辑的复杂提示
- **对话开场白**: 多消息对话流程
- **动态内容**: 基于 map 的参数处理
- **单条消息**: 单条消息响应
- **字符串列表**: 基于列表的响应

#### 自动完成
- **用户名自动完成**: 用户状态 URI 的自动完成
- **名称自动完成**: 个性化消息提示的自动完成
- **国家自动完成**: 与旅行相关的国家名称自动完成

### 2. mcp-annotations-client
一个简单的 MCP 客户端实现,演示使用注解的客户端 MCP 功能。

#### 处理程序类型
- **进度处理程序**: 使用 `@McpProgress` 跟踪长时间运行的操作
- **日志处理程序**: 使用 `@McpLogging` 从 MCP 服务器集中记录日志
- **采样处理程序**: 使用 `@McpSampling` 进行 LLM 请求委托
- **引诱处理程序**: 使用 `@McpElicitation` 进行用户交互提示

## 架构概述

### 服务器架构

```java
@SpringBootApplication
public class McpServerApplication {
    // 传统 Spring AI 工具集成
    @Bean
    public ToolCallbackProvider weatherTools(SpringAiToolProvider weatherService) {
        return MethodToolCallbackProvider.builder().toolObjects(weatherService).build();
    }
}
```

服务器使用自动注解扫描来注册 MCP 提供者。所有 `@McpTool`、`@McpResource`、`@McpPrompt` 和 `@McpComplete` 注解的方法都会自动被发现和注册。

### 客户端架构

```java
@SpringBootApplication
public class McpClientApplication {
    @Bean
    public CommandLineRunner predefinedQuestions(List<McpSyncClient> mcpClients) {
        return args -> {
            for (McpSyncClient mcpClient : mcpClients) {
                // 调用工具并与 MCP 服务器交互
                CallToolRequest toolRequest = CallToolRequest.builder()
                    .name("tool1")
                    .arguments(Map.of("input", "test input"))
                    .progressToken("test-progress-token")
                    .build();

                CallToolResult response = mcpClient.callTool(toolRequest);
            }
        };
    }
}
```

客户端处理程序通过注解扫描自动注册。

## 关键实现示例

### 工具实现

```java
@Service
public class ToolProvider {
    @McpTool(description = "获取特定位置的温度(摄氏度)")
    public WeatherResponse getTemperature(
            @McpToolParam(description = "位置纬度") double latitude,
            @McpToolParam(description = "位置经度") double longitude,
            @McpToolParam(description = "城市名称") String city) {

        return restClient.get()
            .uri("https://api.open-meteo.com/v1/forecast?latitude={latitude}&longitude={longitude}&current=temperature_2m",
                latitude, longitude)
            .retrieve()
            .body(WeatherResponse.class);
    }
}
```

### 资源实现

```java
@Service
public class UserProfileResourceProvider {
    @McpResource(uri = "user-profile://{username}",
                 name = "User Profile",
                 description = "为特定用户提供用户配置文件信息")
    public ReadResourceResult getUserProfile(ReadResourceRequest request, String username) {
        String profileInfo = formatProfileInfo(userProfiles.getOrDefault(username.toLowerCase(), new HashMap<>()));
        return new ReadResourceResult(List.of(new TextResourceContents(request.uri(), "text/plain", profileInfo)));
    }

    @McpResource(uri = "user-attribute://{username}/{attribute}",
                 name = "User Attribute",
                 description = "从用户配置文件中提供特定属性")
    public ReadResourceResult getUserAttribute(String username, String attribute) {
        Map<String, String> profile = userProfiles.getOrDefault(username.toLowerCase(), new HashMap<>());
        String attributeValue = profile.getOrDefault(attribute, "Attribute not found");

        return new ReadResourceResult(
            List.of(new TextResourceContents("user-attribute://" + username + "/" + attribute,
                   "text/plain", username + "'s " + attribute + ": " + attributeValue)));
    }
}
```

### 提示实现

```java
@Service
public class PromptProvider {
    @McpPrompt(name = "personalized-message",
               description = "根据用户信息生成个性化消息")
    public GetPromptResult personalizedMessage(
            McpSyncServerExchange exchange,
            @McpArg(name = "name", description = "用户名称", required = true) String name,
            @McpArg(name = "age", description = "用户年龄", required = false) Integer age,
            @McpArg(name = "interests", description = "用户兴趣", required = false) String interests) {

        // 记录提示生成
        exchange.loggingNotification(LoggingMessageNotification.builder()
            .level(LoggingLevel.INFO)
            .data("personalized-message event").build());

        StringBuilder message = new StringBuilder();
        message.append("你好, ").append(name).append("!\n\n");

        if (age != null) {
            message.append("在 ").append(age).append(" 岁时,你有 ");
            if (age < 30) {
                message.append("很多未来可以期待。\n\n");
            } else if (age < 60) {
                message.append("获得了宝贵的生活经验。\n\n");
            } else {
                message.append("积累了可以与他人分享的智慧。\n\n");
            }
        }

        return new GetPromptResult("个性化消息",
            List.of(new PromptMessage(Role.ASSISTANT, new TextContent(message.toString()))));
    }
}
```

### 自动完成实现

```java
@Service
public class CompletionProvider {
    @McpComplete(uri = "user-status://{username}")
    public List<String> completeUsername(String usernamePrefix) {
        String prefix = usernamePrefix.toLowerCase();
        if (prefix.isEmpty()) {
            return List.of("输入用户名");
        }

        String firstLetter = prefix.substring(0, 1);
        List<String> usernames = usernameDatabase.getOrDefault(firstLetter, List.of());

        return usernames.stream()
            .filter(username -> username.toLowerCase().startsWith(prefix))
            .toList();
    }

    @McpComplete(prompt = "personalized-message")
    public CompleteResult completeCountryName(CompleteRequest request) {
        String prefix = request.argument().value().toLowerCase();
        String firstLetter = prefix.substring(0, 1);
        List<String> countries = countryDatabase.getOrDefault(firstLetter, List.of());

        List<String> matches = countries.stream()
            .filter(country -> country.toLowerCase().startsWith(prefix))
            .toList();

        return new CompleteResult(new CompleteCompletion(matches, matches.size(), false));
    }
}
```

### 客户端处理程序实现

```java
@Service
public class McpClientHandlerProviders {
    @McpProgress(clients = "server1")
    public void progressHandler(ProgressNotification progressNotification) {
        logger.info("MCP 进度: [{}] 进度: {} 总计: {} 消息: {}",
            progressNotification.progressToken(), progressNotification.progress(),
            progressNotification.total(), progressNotification.message());
    }

    @McpLogging(clients = "server1")
    public void loggingHandler(LoggingMessageNotification loggingMessage) {
        logger.info("MCP 日志记录: [{}] {}", loggingMessage.level(), loggingMessage.data());
    }

    @McpSampling(clients = "server1")
    public CreateMessageResult samplingHandler(CreateMessageRequest llmRequest) {
        String userPrompt = ((McpSchema.TextContent) llmRequest.messages().get(0).content()).text();
        String modelHint = llmRequest.modelPreferences().hints().get(0).name();

        return CreateMessageResult.builder()
            .content(new McpSchema.TextContent("响应 " + userPrompt + " 带有模型提示 " + modelHint))
            .build();
    }

    @McpElicitation(clients = "server1")
    public ElicitResult elicitationHandler(McpSchema.ElicitRequest request) {
        logger.info("MCP 引诱: {}", request);
        return new ElicitResult(ElicitResult.Action.ACCEPT, Map.of("message", request.message()));
    }
}
```

## 关键依赖项

### 服务器依赖项
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-server-webmvc</artifactId>
</dependency>
```

### 客户端依赖项
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-client</artifactId>
</dependency>
```

## 运行示例

### 前置条件

- Java 17 或更高版本
- Maven 3.6+

### 步骤 1: 启动 MCP 注解服务器

```bash
cd mcp-annotations-server
./mvnw clean package -DskipTests
java -jar target/mcp-annotations-server-0.0.1-SNAPSHOT.jar
```

服务器将在 `http://localhost:8080` 上启动,并启用 SSE 传输。

### 步骤 2: 运行 MCP 注解客户端

```bash
cd mcp-annotations-client
./mvnw clean package
java -jar target/mcp-annotations-client-0.0.1-SNAPSHOT.jar
```

## 可用功能

### 工具
- `getTemperature`: 使用 Open-Meteo API 获取温度数据

### 资源
- `user-profile://{username}`: 完整的用户配置文件信息
- `user-attribute://{username}/{attribute}`: 特定用户属性
- `user-status://{username}`: 用户在线状态
- `user-connections://{username}`: 用户社交连接
- `user-notifications://{username}`: 用户通知
- `user-location://{username}`: 用户位置信息
- `user-avatar://{username}`: 用户头像图像 (base64)

### 提示
- `greeting`: 带有名称参数的简单问候
- `personalized-message`: 带有年龄和兴趣的复杂个性化消息
- `conversation-starter`: 多消息对话流程
- `map-arguments`: 演示基于 map 的参数处理
- `single-message`: 单条消息响应
- `string-list`: 基于列表的响应

### 自动完成
- `user-status://` URI 的用户名自动完成
- `personalized-message` 提示的名称自动完成
- 旅行提示的国家名称自动完成

## 配置

### 服务器配置 (`application.properties`)

```properties
# 服务器标识
spring.ai.mcp.server.name=my-weather-server
spring.ai.mcp.server.version=0.0.1

# 禁用横幅以兼容 STDIO 传输
spring.main.banner-mode=off

# 日志记录配置
logging.file.name=./model-context-protocol/mcp-annotations/mcp-annotations-server/target/mcp-annotations-server.log

# 取消注释以使用 STDIO 传输
# spring.ai.mcp.server.stdio=true
# spring.main.web-application-type=none

# 取消注释以使用不同的协议
# spring.ai.mcp.server.protocol=STREAMABLE
# spring.ai.mcp.server.protocol=STATELESS
```

### 客户端配置 (`application.properties`)

```properties
spring.application.name=mcp
spring.main.web-application-type=none

# MCP 客户端连接
spring.ai.mcp.client.sse.connections.server1.url=http://localhost:8080

# 日志记录配置
logging.level.io.modelcontextprotocol.client=WARN
logging.level.io.modelcontextprotocol.spec=WARN
```

## 测试服务器

您可以使用包含的测试客户端测试服务器:

### SSE 客户端测试
```java
// 位于 src/test/java/.../client/ClientSse.java
var transport = HttpClientSseClientTransport.builder("http://localhost:8080").build();
var client = McpClient.sync(transport).build();
```

### STDIO 客户端测试
```java
// 位于 src/test/java/.../client/ClientStdio.java
var stdioParams = ServerParameters.builder("java")
    .args("-jar", "target/mcp-annotations-server-0.0.1-SNAPSHOT.jar")
    .build();
var transport = new StdioClientTransport(stdioParams);
var client = McpClient.sync(transport).build();
```

## 高级功能

### 混合注解样式
服务器演示了 MCP 特定注解 (`@McpTool`) 和 Spring AI 注解 (`@Tool`),展示了它们如何在同一应用程序中共存。

### URI 模板变量
资源支持具有多个变量的复杂 URI 模板,这些变量会自动提取并作为方法参数传递。

### 返回类型灵活性
方法可以返回各种类型:
- `ReadResourceResult` 用于完全控制
- `List<ResourceContents>` 用于多个内容项
- `ResourceContents` 用于单个内容项
- `String` 用于简单文本响应
- `List<String>` 用于多个文本项

### 自定义 MIME 类型
资源可以为不同的内容类型指定自定义 MIME 类型,包括二进制数据处理。

### 服务器交换集成
方法可以访问 `McpSyncServerExchange` 以进行高级服务器操作,如日志通知和进度跟踪。

## 示例数据

服务器包括用于测试的示例用户配置文件:
- **john**: John Smith (纽约,年龄 32)
- **jane**: Jane Doe (伦敦,年龄 28)
- **bob**: Bob Johnson (东京,年龄 45)
- **alice**: Alice Brown (悉尼,年龄 36)

## 相关项目

- **[采样示例](../sampling)**: 带有多个 LLM 提供者的 MCP 采样
- **[天气服务器](../weather)**: 简单的天气 MCP 服务器
- **[SQLite 服务器](../sqlite)**: 数据库访问 MCP 服务器
- **[文件系统服务器](../filesystem)**: 文件系统操作 MCP 服务器

## 其他资源

* [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
* [Spring AI MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html)
* [Spring AI MCP Client Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html)
* [Spring AI 注解](https://docs.spring.io/spring-ai/reference/1.1-SNAPSHOT/api/mcp/mcp-annotations-overview.html)
* [Java MCP 注解](https://github.com/spring-ai-community/mcp-annotations)
* [Model Context Protocol 规范](https://modelcontextprotocol.github.io/specification/)
