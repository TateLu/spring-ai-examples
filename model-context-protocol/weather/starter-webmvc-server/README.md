# Spring AI MCP Weather Server Sample with WebMVC Starter

此示例项目演示如何使用 Spring AI MCP Server Boot Starter 和 WebMVC 传输创建 MCP 服务器。它实现了一个天气服务，公开使用国家气象服务 API 获取天气信息的工具。

有关更多信息，请参阅 [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html) 参考文档。

## 概述

该示例展示：
- 与 `spring-ai-mcp-server-webmvc-spring-boot-starter` 集成
- 支持 SSE（服务器发送事件）和 STDIO 传输
- 使用 Spring AI 的 `@Tool` 注解进行自动工具注册
- 两个与天气相关的工具：
  - 按位置（纬度/经度）获取天气预报
  - 按美国州获取天气警报

## 依赖项

项目需要 Spring AI MCP Server WebMVC Boot Starter：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-mcp-server-webmvc-spring-boot-starter</artifactId>
</dependency>
```

此 starter 提供：
- 使用 Spring MVC 的基于 HTTP 的传输（`WebMvcSseServerTransport`）
- 自动配置的 SSE 端点
- 可选的 STDIO 传输
- 包含的 `spring-boot-starter-web` 和 `mcp-spring-webmvc` 依赖项

## 构建项目

使用 Maven 构建项目：
```bash
./mvnw clean install -DskipTests
```

## 运行服务器

服务器支持两种传输模式：

### WebMVC SSE 模式（默认）
```bash
java -jar target/mcp-weather-starter-webmvc-server-0.0.1-SNAPSHOT.jar
```

### STDIO 模式
要启用 STDIO 传输，设置适当的属性：
```bash
java -Dspring.ai.mcp.server.stdio=true -Dspring.main.web-application-type=none -jar target/mcp-weather-starter-webmvc-server-0.0.1-SNAPSHOT.jar
```

## 配置

通过 `application.properties` 配置服务器：

```properties
# 服务器标识
spring.ai.mcp.server.name=my-weather-server
spring.ai.mcp.server.version=0.0.1

# 服务器类型（SYNC/ASYNC）
spring.ai.mcp.server.type=SYNC

# 传输配置
spring.ai.mcp.server.stdio=false
spring.ai.mcp.server.sse-message-endpoint=/mcp/message

# 变更通知
spring.ai.mcp.server.resource-change-notification=true
spring.ai.mcp.server.tool-change-notification=true
spring.ai.mcp.server.prompt-change-notification=true

# 日志记录（STDIO 传输需要）
spring.main.banner-mode=off
logging.file.name=./target/starter-webmvc-server.log
```

## 可用工具

### 天气预报工具
- 名称：`getWeatherForecastByLocation`
- 描述：获取特定纬度/经度的天气预报
- 参数：
  - `latitude`：double - 纬度坐标
  - `longitude`：double - 经度坐标

### 天气警报工具
- 名称：`getAlerts`
- 描述：获取美国州的天气警报
- 参数：
  - `state`：String - 两个字母的美国州代码（例如 CA, NY）

## 服务器实现

服务器使用 Spring Boot 和 Spring AI 的工具注解进行自动工具注册：

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

`WeatherService` 使用 `@Tool` 注解实现天气工具：

```java
@Service
public class WeatherService {
    @Tool(description = "获取特定纬度/经度的天气预报")
    public String getWeatherForecastByLocation(double latitude, double longitude) {
        // 使用 weather.gov API 实现
    }

    @Tool(description = "获取美国州的天气警报。输入是两个字母的美国州代码（例如 CA, NY）")
    public String getAlerts(String state) {
        // 使用 weather.gov API 实现
    }
}
```

## MCP 客户端

您可以使用 STDIO 或 SSE 传输连接到天气服务器：

### 手动客户端

#### WebMVC SSE 客户端

对于使用 SSE 传输的服务器：

```java
var transport = HttpClientSseClientTransport.builder("http://localhost:8080").build();
var client = McpClient.sync(transport).build();
```

#### STDIO 客户端

对于使用 STDIO 传输的服务器：

```java
var stdioParams = ServerParameters.builder("java")
    .args("-Dspring.ai.mcp.server.stdio=true",
          "-Dspring.main.web-application-type=none",
          "-Dspring.main.banner-mode=off",
          "-Dlogging.pattern.console=",
          "-jar",
          "target/mcp-weather-starter-webmvc-server-0.0.1-SNAPSHOT.jar")
    .build();

var transport = new StdioClientTransport(stdioParams);
var client = McpClient.sync(transport).build();
```

示例项目包含示例客户端实现：
- [SampleClient.java](src/test/java/org/springframework/ai/mcp/sample/client/SampleClient.java)：手动 MCP 客户端实现
- [ClientStdio.java](src/test/java/org/springframework/ai/mcp/sample/client/ClientStdio.java)：STDIO 传输连接
- [ClientSse.java](src/test/java/org/springframework/ai/mcp/sample/client/ClientSse.java)：SSE 传输连接

为了获得更好的开发体验，建议使用 [MCP Client Boot Starters](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html)。这些 starter 可以自动配置多个与 MCP 服务器的 STDIO 和/或 SSE 连接。有关示例，请参阅 [starter-default-client](../../client-starter/starter-default-client) 项目。

### Boot Starter 客户端

让我们使用 [starter-default-client](../../client-starter/starter-default-client) 客户端连接到我们的天气 `starter-webmvc-server`。

按照 `starter-default-client` readme 说明构建 `mcp-starter-default-client-0.0.1-SNAPSHOT.jar` 客户端应用程序。

#### STDIO 传输

1. 创建包含以下内容的 `mcp-servers-config.json` 配置文件：

```json
{
  "mcpServers": {
    "weather-starter-webmvc-server": {
      "command": "java",
      "args": [
        "-Dspring.ai.mcp.server.stdio=true",
        "-Dspring.main.web-application-type=none",
        "-Dlogging.pattern.console=",
        "-jar",
        "/absolute/path/to/mcp-weather-starter-webmvc-server-0.0.1-SNAPSHOT.jar"
      ]
    }
  }
}
```

2. 使用配置文件运行客户端：

```bash
java -Dspring.ai.mcp.client.stdio.servers-configuration=file:mcp-servers-config.json \
 -Dai.user.input='What is the weather in NY?' \
 -Dlogging.pattern.console= \
 -jar mcp-starter-default-client-0.0.1-SNAPSHOT.jar
```

#### SSE (WebMVC) 传输

1. 启动 `mcp-weather-starter-webmvc-server`：

```bash
java -jar mcp-weather-starter-webmvc-server-0.0.1-SNAPSHOT.jar
```

在端口 8080 上启动 MCP 服务器。

2. 在另一个控制台中启动配置了 SSE 传输的客户端：

```bash
java -Dspring.ai.mcp.client.sse.connections.weather-server.url=http://localhost:8080 \
 -Dlogging.pattern.console= \
 -Dai.user.input='What is the weather in NY?' \
 -jar mcp-starter-default-client-0.0.1-SNAPSHOT.jar
```

## 其他资源

* [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
* [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html)
* [MCP Client Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-client-docs.html)
* [Model Context Protocol 规范](https://modelcontextprotocol.github.io/specification/)
* [Spring Boot 自动配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration)
