# Spring AI MCP Sample Weather MCP Server

此示例项目演示 Spring AI 模型上下文协议（MCP）实现的用法。它展示了如何创建和使用具有不同传输模式和服务器的 MCP 客户端。

## 概述

该示例提供：
- 两种传输模式实现：Stdio 和 SSE（服务器发送事件）
- 服务器功能：
  - 支持列表变更通知的工具
  - 支持列表变更通知的资源（不支持订阅）
  - 支持列表变更通知的提示
- 示例实现：
  - 两个 MCP 工具：天气和计算器
  - 一个 MCP 资源：系统信息
  - 一个 MCP 提示：问候

## 构建项目

```bash
./mvnw clean package
```

## 运行服务器

服务器可以通过两种传输模式启动，由 `transport.mode` 属性控制：

### Stdio 模式（默认）

```bash
java -Dtransport.mode=stdio -jar model-context-protocol/mcp-weather-server/target/mcp-weather-server-0.0.1-SNAPSHOT.jar
```

Stdio 模式服务器由客户端自动启动 - 不需要显式启动服务器。
但您必须先构建服务器 jar：`./mvnw clean install -DskipTests`。

在 Stdio 模式下，服务器不得向控制台（例如标准输出）发出任何消息/日志，只能发出服务器产生的 JSON 消息。

### SSE 模式
```bash
java -Dtransport.mode=sse -jar model-context-protocol/mcp-weather-server/target/mcp-weather-server-0.0.1-SNAPSHOT.jar
```

## 示例客户端

项目包括两种传输模式的示例客户端：

### Stdio 客户端（ClientStdio.java）
```java
var stdioParams = ServerParameters.builder("java")
    .args("-Dtransport.mode=stdio", "-Dspring.main.web-application-type=none", "-jar",
            "model-context-protocol/mcp-weather-server/target/mcp-weather-server-0.0.1-SNAPSHOT.jar")
    .build();

var transport = new StdioClientTransport(stdioParams);
var client = McpClient.using(transport).sync();
```

### SSE 客户端（ClientWebFluxSse.java）
```java
var transport = new SseClientTransport(WebClient.builder().baseUrl("http://localhost:8080"));
var client = McpClient.using(transport).sync();
```

## 可用工具

### 天气工具
- 名称：`getWeatherForecastByLocation`
- 描述：按位置天气预报工具
- 参数：
  - `lat, long`：String - 位置的纬度、经度
- 示例：
```java
CallToolResult weatherForcastResult = client.callTool(new CallToolRequest("getWeatherForecastByLocation",
        Map.of("latitude", "47.6062", "longitude", "-122.3321")));
```


## 客户端使用示例

```java
// 初始化客户端
client.initialize();

// 测试连接
client.ping();

// 列出可用工具
ListToolsResult tools = client.listTools();
System.out.println("Available tools: " + tools);

CallToolResult weatherForcastResult = client.callTool(new CallToolRequest("getWeatherForecastByLocation",
        Map.of("latitude", "47.6062", "longitude", "-122.3321")));
System.out.println("Weather Forcast: " + weatherForcastResult);

CallToolResult alertResult = client.callTool(new CallToolRequest("getAlerts", Map.of("state", "NY")));
System.out.println("Alert Response = " + alertResult);


// 关闭客户端
client.closeGracefully();
```

## 服务器使用示例

```java
@Configuration
public class CustomMcpServerConfig {

    @Bean
	public WeatherApiClient weatherApiClient() {
		return new WeatherApiClient();
	}

	@Bean
	public McpSyncServer mcpServer(ServerMcpTransport transport, WeatherApiClient weatherApiClient) { // @formatter:off

		// 配置具有资源支持的服务器功能
		var capabilities = McpSchema.ServerCapabilities.builder()
			.tools(true) // 支持列表变更通知的工具
			.logging() // 日志支持
			.build();

		// 创建具有工具和资源功能的服务器
		McpSyncServer server = McpServer.sync(transport)
			.serverInfo("MCP Demo Weather Server", "1.0.0")
			.capabilities(capabilities)
			.tools(ToolHelper.toSyncToolRegistration(ToolCallbacks.from(weatherApiClient))) // 添加 @Tools
			.build();

		return server; // @formatter:on
	} // @formatter:on
}
```

## 配置

应用程序可以通过 `application.properties` 配置：

- `transport.mode`：要使用的传输模式（stdio/sse）
- `server.port`：SSE 模式的服务器端口（默认：8080）
- 各种可用于调试的日志记录配置
