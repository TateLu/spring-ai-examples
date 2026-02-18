# Spring AI MCP 注解服务器示例

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Java Version](https://img.shields.io/badge/Java-17%2B-orange)](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)

此示例项目演示了如何使用 Spring AI 的 MCP 注解创建 MCP 服务器。它展示了 MCP 服务器功能的全面实现,包括使用干净、声明式的 Java 注解方法实现的工具、资源、提示和自动完成。

有关更多信息,请参阅 [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html) 参考文档。

## 目录
- [概述](#概述)
- [功能](#功能)
- [依赖项](#依赖项)
- [构建项目](#构建项目)
- [运行服务器](#运行服务器)
- [配置](#配置)
- [服务器实现](#服务器实现)
- [MCP 功能](#mcp-功能)
  - [工具](#工具)
  - [资源](#资源)
  - [提示](#提示)
  - [自动完成](#自动完成)
- [MCP 客户端](#mcp-客户端)
  - [手动客户端](#手动客户端)
  - [Boot Starter 客户端](#boot-starter-客户端)
- [其他资源](#其他资源)

## 概述

该示例展示了全面的 MCP 服务器实现,具有:
- 与 `spring-ai-starter-mcp-server-webmvc` 集成
- 支持 SSE (服务器发送事件) 和 STDIO 传输
- 使用注解自动注册 MCP 功能:
  - `@Tool` 用于 Spring AI 工具注册
  - `@McpTool` 用于 MCP 特定工具注册
  - `@McpResource` 用于资源注册
  - `@McpPrompt` 用于提示注册
  - `@McpComplete` 用于自动完成注册
- 每种功能类型的全面示例

## 功能

此示例演示:

1. **天气工具** - 使用 Spring AI `@Tool` 和 MCP `@McpTool` 注解检索天气预报和警报的工具
2. **用户配置文件资源** - 使用各种 URI 模式访问用户配置文件信息的资源
3. **提示生成** - 不同用例的各种提示模板
4. **自动完成** - 用户名和国家的自动完成建议

## 依赖项

该项目需要 Spring AI MCP Server WebMVC Boot Starter 和 MCP 注解:

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-server-webmvc</artifactId>
</dependency>
```

这些依赖项提供:
- 使用 Spring MVC (`WebMvcSseServerTransport`) 基于 HTTP 的传输
- 通过 `spring.ai.mcp.server.protocol=...` 属性配置的自动配置的 SSE、Streamable-HTTP 或 Stateless 端点,默认为 `SSE`
- 如果设置了 `spring.ai.mcp.server.stdio=true`,则为可选的 STDIO 传输
- 基于 MCP 操作的注解式方法处理

## 构建项目

使用 Maven 构建项目:

```bash
./mvnw clean install -DskipTests
```

## 运行服务器

服务器支持两种传输模式:

### WebMVC SSE/Streamable-HTTP/Stateless 模式

模式取决于 `spring.ai.mcp.server.protocol=...` 设置。

```bash
java -Dspring.ai.mcp.server.protocol=STREAMABLE -jar target/mcp-annotations-server-0.0.1-SNAPSHOT.jar
```

### STDIO 模式

要启用 STDIO 传输,请设置适当的属性:

```bash
java -Dspring.ai.mcp.server.stdio=true -Dspring.main.web-application-type=none -jar target/mcp-annotations-server-0.0.1-SNAPSHOT.jar
```

## 配置

通过 `application.properties` 配置服务器:

```properties
# 服务器标识
spring.ai.mcp.server.name=my-weather-server
spring.ai.mcp.server.version=0.0.1
spring.ai.mcp.server.protocol=STREAMABLE
# spring.ai.mcp.server.protocol=STATELESS

# 传输配置(取消注释以启用 STDIO)
# spring.ai.mcp.server.stdio=true
# spring.main.web-application-type=none

# 日志记录(STDIO 传输需要)
spring.main.banner-mode=off
# logging.pattern.console=

# 日志文件位置
logging.file.name=./model-context-protocol/mcp-annotations/mcp-annotations-server/target/server.log
```

## 服务器实现

服务器使用 Spring Boot 和 MCP 注解自动注册功能:

```java
@SpringBootApplication
public class McpServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }

    // (可选)不是 MCP 注解。仅演示如何使用 @Tool 以及 @McpTool 来提供 MCP 服务器工具。
    @Bean
    public ToolCallbackProvider weatherTools(SpringAiToolProvider weatherService) {
        return MethodToolCallbackProvider.builder().toolObjects(weatherService).build();
    }
}
```

## MCP 功能

### 工具

该项目包括两个不同的工具提供者,演示了不同的注解方法:

#### SpringAiToolProvider (Spring AI @Tool 注解)

使用 Spring AI 的 `@Tool` 注解进行与天气相关的工具:

```java
@Service
public class SpringAiToolProvider {
    @Tool(description = "获取特定纬度/经度的天气预报")
    public String getWeatherForecastByLocation(double latitude, double longitude) {
        // 使用 weather.gov API 实现
    }

    @Tool(description = "获取美国州的天气警报。输入是两位数的美国州代码(例如 CA、NY)")
    public String getAlerts(String state) {
        // 使用 weather.gov API 实现
    }
}
```

#### McpToolProvider (MCP @McpTool 注解)

使用 MCP 特定的 `@McpTool` 注解进行温度检索:

```java
@Service
public class McpToolProvider {
    @McpTool(description = "获取特定位置的温度(摄氏度)")
    public WeatherResponse getTemperature(McpSyncServerExchange exchange,
            @McpProgressToken String progressToken
            @McpToolParam(description = "位置纬度") double latitude,
            @McpToolParam(description = "位置经度") double longitude,
            @McpToolParam(description = "城市名称") String city) {
        // 使用 open-meteo.com API 实现
    }
}
```

#### 可用工具

1. **天气预报工具** (Spring AI)
   - 名称: `getWeatherForecastByLocation`
   - 描述: 获取特定纬度/经度的天气预报
   - 参数:
     - `latitude`: double - 纬度坐标
     - `longitude`: double - 经度坐标

2. **天气警报工具** (Spring AI)
   - 名称: `getAlerts`
   - 描述: 获取美国州的天气警报
   - 参数:
     - `state`: String - 两位数的美国州代码(例如 CA、NY)

3. **温度工具** (MCP)
   - 名称: `getTemperature`
   - 描述: 获取特定位置的温度(摄氏度)
   - 参数:
     - `latitude`: double - 位置纬度
     - `longitude`: double - 位置经度
     - `city`: String - 城市名称

### 资源

`UserProfileResourceProvider` 使用 `@McpResource` 注解实现资源访问,并提供全面示例:

```java
@Service
public class UserProfileResourceProvider {
    @McpResource(uri = "user-profile://{username}",
                name = "User Profile",
                description = "为特定用户提供用户配置文件信息")
    public ReadResourceResult getUserProfile(ReadResourceRequest request, String username) {
        // 检索用户配置文件的实现
    }

    // 其他资源方法...
}
```

#### 可用资源

1. **用户配置文件**
   - URI: `user-profile://{username}`
   - 描述: 为特定用户提供用户配置文件信息

2. **用户详细信息**
   - URI: `user-profile://{username}`
   - 描述: 使用 URI 变量提供用户详细信息

3. **用户属性**
   - URI: `user-attribute://{username}/{attribute}`
   - 描述: 从用户配置文件中提供特定属性

4. **带 Exchange 的用户配置文件**
   - URI: `user-profile-exchange://{username}`
   - 描述: 提供带有服务器交换上下文的用户配置文件

5. **用户连接**
   - URI: `user-connections://{username}`
   - 描述: 为用户提供连接列表

6. **用户通知**
   - URI: `user-notifications://{username}`
   - 描述: 为用户提供通知

7. **用户状态**
   - URI: `user-status://{username}`
   - 描述: 为用户提供当前状态

8. **用户位置**
   - URI: `user-location://{username}`
   - 描述: 为用户提供当前位置

9. **用户头像**
   - URI: `user-avatar://{username}`
   - 描述: 为用户提供 base64 编码的头像图像
   - MIME 类型: `image/png`

#### 示例用户数据

提供者包括用户 `john`、`jane`、`bob` 和 `alice` 的示例数据,其配置文件包含名称、电子邮件、年龄和位置信息。

### 提示

`PromptProvider` 使用 `@McpPrompt` 注解实现提示生成:

```java
@Service
public class PromptProvider {
    @McpPrompt(name = "greeting", description = "简单的问候提示")
    public GetPromptResult greetingPrompt(
            @McpArg(name = "name", description = "要问候的名称", required = true) String name) {
        // 生成问候提示的实现
    }

    // 其他提示方法...
}
```

#### 可用提示

1. **问候**
   - 名称: `greeting`
   - 描述: 简单的问候提示
   - 参数:
     - `name`: String - 要问候的名称(必需)

2. **个性化消息**
   - 名称: `personalized-message`
   - 描述: 根据用户信息生成个性化消息
   - 参数:
     - `name`: String - 用户名称(必需)
     - `age`: Integer - 用户年龄(可选)
     - `interests`: String - 用户兴趣(可选)

3. **对话开场白**
   - 名称: `conversation-starter`
   - 描述: 提供与系统的对话开场白

4. **Map 参数**
   - 名称: `map-arguments`
   - 描述: 演示使用 map 作为参数

5. **单条消息**
   - 名称: `single-message`
   - 描述: 演示返回单个 PromptMessage
   - 参数:
     - `name`: String - 用户名称(必需)

6. **字符串列表**
   - 名称: `string-list`
   - 描述: 演示返回字符串列表
   - 参数:
     - `topic`: String - 要提供有关信息的主题(必需)

### 自动完成

`AutocompleteProvider` 使用 `@McpComplete` 注解实现自动完成:

```java
@Service
public class AutocompleteProvider {
    @McpComplete(uri = "user-status://{username}")
    public List<String> completeUsername(String usernamePrefix) {
        // 提供用户名自动完成的实现
    }

    // 其他自动完成方法...
}
```

#### 可用自动完成

1. **用户名自动完成**
   - URI: `user-status://{username}`
   - 根据前缀提供用户名的自动完成建议

2. **名称自动完成**
   - 提示: `personalized-message`
   - 为个性化消息提示中的名称提供自动完成建议

3. **国家名称自动完成**
   - 提示: `travel-planner`
   - 为国家名称提供自动完成建议
   - 返回: 带有自动完成详细信息的 `CompleteResult`

## MCP 客户端

您可以使用 STDIO 或 SSE 传输连接到服务器:

### Boot Starter 客户端

为了获得更好的开发体验,请考虑使用 [MCP Client Boot Starters](https://docs.spring.io/spring-ai/reference/1.1-SNAPSHOT/api/mcp/mcp-client-boot-starter-docs.html) 和相关的 [MCP Client Annotations](https://docs.spring.io/spring-ai/reference/1.1-SNAPSHOT/api/mcp/mcp-annotations-client.html) 支持。

MCP Client Boot Starter 提供:
- MCP 客户端连接的自动配置
- 用于服务器通知的声明式基于注解的处理程序
- 支持多种传输协议(SSE、Streamable-HTTP、STDIO)
- 客户端处理程序的自动注册

#### 客户端依赖项

将 MCP Client Boot Starter 添加到您的项目中:

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-client</artifactId>
</dependency>
```

#### 客户端注解

客户端支持几种用于处理服务器通知的注解:

- `@McpLogging` - 处理来自 MCP 服务器的日志消息通知
- `@McpProgress` - 处理长时间运行操作的进度通知
- `@McpSampling` - 处理来自 MCP 服务器的 LLM 完成采样请求
- `@McpElicitation` - 处理引诱请求以从用户收集附加信息
- `@McpToolListChanged` - 处理服务器工具列表更改时的通知
- `@McpResourceListChanged` - 处理服务器资源列表更改时的通知
- `@McpPromptListChanged` - 处理服务器提示列表更改时的通知

**重要**: 所有 MCP 客户端注解 **必须** 包含一个 `clients` 参数,以将处理程序与特定的 MCP 客户端连接关联。

#### 示例客户端实现

```java
@SpringBootApplication
public class McpClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(McpClientApplication.class, args).close();
    }

    @Bean
    public CommandLineRunner predefinedQuestions(List<McpSyncClient> mcpClients) {
        return args -> {
            for (McpSyncClient mcpClient : mcpClients) {
                System.out.println(">>> MCP 客户端: " + mcpClient.getClientInfo());

               // 调用发送进度通知的工具
               CallToolRequest toolRequest = CallToolRequest.builder()
                     .name("tool1")
                     .arguments(Map.of("input", "test input"))
                     .progressToken("test-progress-token")
                     .build();

               CallToolResult response = mcpClient.callTool(toolRequest);

               System.out.println("工具响应: " + response);
            }
        };
    }
}
```

#### 客户端处理程序提供者

```java
@Service
public class McpClientHandlerProviders {

    private static final Logger logger = LoggerFactory.getLogger(McpClientHandlerProviders.class);

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
        logger.info("MCP 采样: {}", llmRequest);
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

#### 客户端配置

在 `application.properties` 中配置客户端连接:

```properties
spring.application.name=mcp
spring.main.web-application-type=none

# Streamable-HTTP 传输配置
spring.ai.mcp.client.streamable-http.connections.server1.url=http://localhost:8080

# SSE 传输配置(替代)
# spring.ai.mcp.client.sse.connections.server1.url=http://localhost:8080

# 全局客户端设置
spring.ai.mcp.client.request-timeout=5m

# 日志记录配置
logging.level.io.modelcontextprotocol.client=WARN
logging.level.io.modelcontextprotocol.spec=WARN

# 可选:如果不需要则禁用工具回调
# spring.ai.mcp.client.toolcallback.enabled=false
```

#### 运行客户端示例

##### Streamable-HTTP (HttpClient) 传输

1. 启动 MCP 注解服务器:

```bash
java -Dspring.ai.mcp.server.protocol=STREAMABLE -jar mcp-annotations-server-0.0.1-SNAPSHOT.jar
```

2. 在另一个控制台中,启动配置了 Streamable-HTTP 传输的客户端:

```bash
java -Dspring.ai.mcp.client.streamable-http.connections.server1.url=http://localhost:8080 \
 -jar mcp-annotations-client-0.0.1-SNAPSHOT.jar
```

##### SSE 传输

1. 启动带有 SSE 协议的 MCP 注解服务器:

```bash
java -Dspring.ai.mcp.server.protocol=SSE -jar mcp-annotations-server-0.0.1-SNAPSHOT.jar
```

2. 启动配置了 SSE 传输的客户端:

```bash
java -Dspring.ai.mcp.client.sse.connections.server1.url=http://localhost:8080 \
 -jar mcp-annotations-client-0.0.1-SNAPSHOT.jar
```

**注意:** `@McpLogging`、`@McpSampling`、`@McpProgress` 和 `@McpElicitate` 注解的 `clients="server1"` 参数对应于您在 `spring.ai.mcp.client.streamable-http.connections.server1.url=` 或 `spring.ai.mcp.client.sse.connections.server1.url=` 配置中放入的连接名称。

##### STDIO 传输

1. 创建 `mcp-servers-config.json` 配置文件:

```json
{
  "mcpServers": {
    "annotations-server": {
      "command": "java",
      "args": [
        "-Dspring.ai.mcp.server.stdio=true",
        "-Dspring.main.web-application-type=none",
        "-Dlogging.pattern.console=",
        "-jar",
        "/absolute/path/to/mcp-annotations-server-0.0.1-SNAPSHOT.jar"
      ]
    }
  }
}
```

2. 使用配置文件运行客户端:

```bash
java -Dspring.ai.mcp.client.stdio.servers-configuration=file:mcp-servers-config.json \
 -Dlogging.pattern.console= \
 -jar mcp-annotations-client-0.0.1-SNAPSHOT.jar
```

客户端将自动:
- 连接到配置的 MCP 服务器
- 根据 `clients` 参数注册注解的处理程序
- 通过注解的方法处理服务器通知和请求
- 按配置提供日志记录和进度更新


## 其他资源

* [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
* [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html)
* [MCP Client Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html)
* [Model Context Protocol 规范](https://modelcontextprotocol.github.io/specification/)
* [MCP 注解项目](https://github.com/spring-ai-community/mcp-annotations)
* [Spring Boot 自动配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration)
