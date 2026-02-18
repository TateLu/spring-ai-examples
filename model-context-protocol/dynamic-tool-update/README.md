# Dynamic Tool Update Example

此示例演示模型上下文协议（MCP）服务器如何在运行时动态更新其可用工具，以及客户端如何检测和使用这些更新的工具。

## 概述

MCP 协议允许 AI 模型通过标准化接口访问外部工具和资源。此示例展示了 MCP 的一个关键功能：在服务器端动态更新可用工具，客户端检测并利用这些更改。

## 关键组件

### 服务器

服务器应用程序由以下部分组成：

1. **WeatherService**：最初提供天气预报工具，用于检索特定位置的温度数据。
2. **MathTools**：包含数学运算（求和、乘法、除法），这些工具动态添加到服务器。
3. **ServerApplication**：管理服务器生命周期并处理动态工具更新过程。

### 客户端

客户端应用程序：

1. 连接到 MCP 服务器
2. 检索初始可用工具列表
3. 触发服务器更新其工具
4. 检测工具更改
5. 检索更新的工具列表

## 工作原理

1. **初始设置**：服务器启动时，仅公开天气预报工具。

2. **动态更新过程**：
   - 客户端向服务器的 `/updateTools` 端点发送请求
   - 服务器接收此信号并将数学工具添加到其可用工具
   - 服务器通知连接的客户端有关工具更改
   - 客户端接收通知，现在可以使用新工具

3. **工具发现**：客户端可以随时查询可用工具，演示工具列表已更新。

## 实现细节

### 服务器端实现

服务器使用 Spring AI 的 MCP 服务器实现：

```java
@Bean
public ToolCallbackProvider weatherTools(WeatherService weatherService) {
    return MethodToolCallbackProvider.builder().toolObjects(weatherService).build();
}

@Bean
public CommandLineRunner predefinedQuestions(McpSyncServer mcpSyncServer) {
    return args -> {
        logger.info("Server: " + mcpSyncServer.getServerInfo());

        latch.await(); // 等待更新信号

        // 动态添加数学工具
        List<SyncToolSpecification> newTools = McpToolUtils
                .toSyncToolSpecifications(ToolCallbacks.from(new MathTools()));

        mcpSyncServer.addTool(newTools.iterator().next());

        logger.info("Tools updated: ");
    };
}
```

### 客户端实现

客户端连接到服务器并注册回调，以便在工具更改时收到通知：

```java
@Bean
McpSyncClientCustomizer customizeMcpClient() {
    return (name, mcpClientSpec) -> {
        mcpClientSpec.toolsChangeConsumer(tv -> {
            logger.info("\nMCP TOOLS CHANGE: " + tv);
            latch.countDown();
        });
    };
}
```

客户端在更新前后检索可用工具：

```java
// 获取初始工具
List<ToolDescription> toolDescriptions = chatClientBuilder.build()
        .prompt("What tools are available?")
        .toolCallbacks(tools)
        .call()
        .entity(new ParameterizedTypeReference<List<ToolDescription>>() {});

// 通知服务器更新工具
String signal = RestClient.builder().build().get()
        .uri("http://localhost:8080/updateTools").retrieve().body(String.class);

// 等待工具更改通知
latch.await();

// 获取更新的工具
toolDescriptions = chatClientBuilder.build()
        .prompt("What tools are available?")
        .toolCallbacks(tools)
        .call()
        .entity(new ParameterizedTypeReference<List<ToolDescription>>() {});
```

## 关键洞察

该示例演示了 Spring AI 中 MCP 实现的关键方面：

> 客户端实现依赖于这样一个事实：MCP 的 `ToolCallbackProvider#getToolCallbacks` 实现将始终从服务器获取当前 MCP 工具列表。

这意味着，每当客户端请求可用工具时，它总是从服务器获取最新的列表，而无需重启或重新初始化客户端。

## 运行示例

1. 启动服务器应用程序：
   ```
   cd server
   ./mvnw spring-boot:run
   ```

2. 在单独的终端中，启动客户端应用程序：
   ```
   cd client
   ./mvnw spring-boot:run
   ```

3. 观察控制台输出，查看：
   - 初始工具列表（仅天气预报）
   - 工具更新通知
   - 更新的工具列表（天气预报 + 数学运算）

## MCP 协议规范

有关 MCP 协议的更多信息，请参阅官方规范：
[https://modelcontextprotocol.io/specification/2024-11-05/server/tools](https://modelcontextprotocol.io/specification/2024-11-05/server/tools)
