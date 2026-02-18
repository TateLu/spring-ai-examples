# Spring AI MCP Weather STDIO Server

一个 Spring Boot starter 项目，演示如何使用国家气象服务（weather.gov）API 构建模型上下文协议（MCP）服务器，提供与天气相关的工具。该项目展示了 Spring AI MCP Server Boot Starter 的功能和 STDIO 传输实现。

有关更多信息，请参阅 [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html) 参考文档。

## 前置条件

- Java 17 或更高版本
- Maven 3.6 或更高版本
- 了解 Spring Boot 和 Spring AI 概念
- （可选）Claude Desktop 用于 AI 助手集成

## 关于 Spring AI MCP Server Boot Starter

`spring-ai-mcp-server-spring-boot-starter` 提供：
- MCP 服务器组件的自动配置
- 支持同步和异步操作模式
- STDIO 传输层实现
- 通过 Spring bean 进行灵活的工具注册
- 变更通知功能

## 项目结构

```
src/
├── main/
│   ├── java/
│   │   └── org/springframework/ai/mcp/sample/server/
│   │       ├── McpServerApplication.java    # 具有工具注册的主应用程序类
│   │       └── WeatherService.java          # 具有 MCP 工具的天气服务实现
│   └── resources/
│       └── application.properties           # 服务器和传输配置
└── test/
    └── java/
        └── org/springframework/ai/mcp/sample/client/
            └── ClientStdio.java             # 测试客户端实现
```

## 构建和运行

服务器使用 STDIO 传输模式，通常由客户端自动启动。要构建服务器 jar：

```bash
./mvnw clean install -DskipTests
```

## 工具实现

该项目演示如何使用 Spring 的依赖注入和自动配置来实现和注册 MCP 工具：

```java
@Service
public class WeatherService {
    @Tool(description = "获取特定纬度/经度的天气预报")
    public String getWeatherForecastByLocation(
        double latitude,   // 纬度坐标
        double longitude   // 经度坐标
    ) {
        // 实现
    }

    @Tool(description = "获取美国州的天气警报")
    public String getAlerts(
        String state  // 两个字母的美国州代码（例如 CA, NY）
    ) {
        // 实现
    }
}

@SpringBootApplication
public class McpServerApplication {
    @Bean
    public List<ToolCallback> weatherTools(WeatherService weatherService) {
        return ToolCallbacks.from(weatherService);
    }
}
```

自动配置会自动将这些工具注册到 MCP 服务器。您可以有多个 bean 生成 ToolCallback 列表，自动配置会将它们合并。

## 可用工具

### 1. 天气预报工具
```java
@Tool(description = "获取特定纬度/经度的天气预报")
public String getWeatherForecastByLocation(
    double latitude,   // 纬度坐标
    double longitude   // 经度坐标
) {
    // 返回详细预报，包括：
    // - 温度和单位
    // - 风速和风向
    // - 详细预报描述
}

// 示例用法：
CallToolResult forecast = client.callTool(
    new CallToolRequest("getWeatherForecastByLocation",
        Map.of(
            "latitude", 47.6062,    // 西雅图坐标
            "longitude", -122.3321
        )
    )
);
```

### 2. 天气警报工具
```java
@Tool(description = "获取美国州的天气警报")
public String getAlerts(
    String state  // 两个字母的美国州代码（例如 CA, NY）
) {
    // 返回活动警报，包括：
    // - 事件类型
    // - 受影响区域
    // - 严重程度
    // - 描述
    // - 安全说明
}

// 示例用法：
CallToolResult alerts = client.callTool(
    new CallToolRequest("getAlerts",
        Map.of("state", "NY")
    )
);
```

## 客户端集成

### Java 客户端示例

#### 手动创建 MCP 客户端

```java
// 创建服务器参数
ServerParameters stdioParams = ServerParameters.builder("java")
    .args("-Dspring.ai.mcp.server.transport=STDIO",
          "-Dspring.main.web-application-type=none",
          "-Dlogging.pattern.console=",
          "-jar",
          "target/mcp-weather-stdio-server-0.0.1-SNAPSHOT.jar")
    .build();

// 初始化传输和客户端
var transport = new StdioClientTransport(stdioParams);
var client = McpClient.sync(transport).build();
```

[ClientStdio.java](src/test/java/org/springframework/ai/mcp/sample/client/ClientStdio.java) 演示了如何手动实现 MCP 客户端。

为了获得更好的开发体验，建议使用 [MCP Client Boot Starters](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html)。这些 starter 可以自动配置多个与 MCP 服务器的 STDIO 和/或 SSE 连接。有关示例，请参阅 [starter-default-client](../../client-starter/starter-default-client) 和 [starter-webflux-client](../../client-starter/starter-webflux-client) 项目。

#### 使用 MCP Client Boot Starter

使用 [starter-default-client](../../client-starter/starter-default-client) 连接到天气 `starter-stdio-server`：

1. 按照 `starter-default-client` readme 说明构建 `mcp-starter-default-client-0.0.1-SNAPSHOT.jar` 客户端应用程序。

2. 使用配置文件运行客户端：

```bash
java -Dspring.ai.mcp.client.stdio.connections.server1.command=java \
     -Dspring.ai.mcp.client.stdio.connections.server1.args=-jar,/Users/christiantzolov/Dev/projects/spring-ai-examples/model-context-protocol/weather/starter-stdio-server/target/mcp-weather-stdio-server-0.0.1-SNAPSHOT.jar \
     -Dai.user.input='What is the weather in NY?' \
     -Dlogging.pattern.console= \
     -jar mcp-starter-default-client-0.0.1-SNAPSHOT.jar
```

### Claude Desktop 集成

要与 Claude Desktop 集成，将以下配置添加到您的 Claude Desktop 设置中：

```json
{
  "mcpServers": {
    "spring-ai-mcp-weather": {
      "command": "java",
      "args": [
        "-Dspring.ai.mcp.server.stdio=true",
        "-Dspring.main.web-application-type=none",
        "-Dlogging.pattern.console=",
        "-jar",
        "/absolute/path/to/mcp-weather-stdio-server-0.0.1-SNAPSHOT.jar"
      ]
    }
  }
}
```

将 `/absolute/path/to/` 替换为您构建的 jar 文件的实际路径。

## 配置

### 应用程序属性

所有属性都以 `spring.ai.mcp.server` 为前缀：

```properties
# 必需的 STDIO 配置
spring.main.web-application-type=none
spring.main.banner-mode=off
logging.pattern.console=

# 服务器配置
spring.ai.mcp.server.enabled=true
spring.ai.mcp.server.name=my-weather-server
spring.ai.mcp.server.version=0.0.1
# SYNC 或 ASYNC
spring.ai.mcp.server.type=SYNC
spring.ai.mcp.server.resource-change-notification=true
spring.ai.mcp.server.tool-change-notification=true
spring.ai.mcp.server.prompt-change-notification=true

# 可选文件日志记录
logging.file.name=mcp-weather-stdio-server.log
```

### 关键配置说明

1. **STDIO 模式要求**
   - 禁用 Web 应用程序类型（`spring.main.web-application-type=none`）
   - 禁用 Spring banner（`spring.main.banner-mode=off`）
   - 清除控制台日志模式（`logging.pattern.console=`）

2. **服务器类型**
   - `SYNC`（默认）：使用 `McpSyncServer` 进行简单的请求-响应模式
   - `ASYNC`：使用 `McpAsyncServer` 进行支持 Project Reactor 的非阻塞操作

## 其他资源

- [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
- [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html)
- [MCP Client Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-client-docs.html)
- [Model Context Protocol 规范](https://modelcontextprotocol.github.io/specification/)
- [Spring Boot 自动配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration)
