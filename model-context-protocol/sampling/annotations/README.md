# Spring AI MCP Sampling Examples - 基于注解的实现

此目录包含基于注解的示例，演示 Spring AI 中模型上下文协议（MCP）Sampling 功能。这些示例展示了与主要 sampling 示例相同的 MCP Sampling 功能，但使用 Spring AI 的基于注解的方法进行简化开发。

---

## 项目结构

```
annotations/
├── mcp-sampling-client-annotations/   # 示例 MCP 客户端（基于注解）
├── mcp-sampling-server-annotations/   # 示例 MCP 服务器（基于注解）
└── README.md
```

| 子项目                           | 类型   | 描述                                                              |
|------------------------------------|--------|--------------------------------------------------------------------|
| mcp-sampling-server-annotations    | 服务器 | 基于注解的 MCP 服务器，通过 LLM 暴露天气+诗歌工具             |
| mcp-sampling-client-annotations    | 客户端 | 基于注解的 MCP 客户端，聚合来自所有 LLM 的创造性响应          |

---

## 概述

基于注解的 MCP Sampling 示例演示：

- **简化开发**：使用注解（`@McpTool`、`@McpSampling`、`@McpLogging`）代替手动工具注册
- **声明式配置**：注解驱动的 MCP 服务器和客户端设置
- **多模型集成**：在 OpenAI 和 Anthropic 模型之间无缝路由
- **创造性内容生成**：使用多个 LLM 生成关于天气数据的诗歌
- **统一响应处理**：将来自不同模型的响应组合成连贯的结果

## 此实现的不同之处

基于注解的方法比传统实现提供了几个优势：

### 服务器端优势
- **`@McpTool`**：自动工具注册，无需手动 `ToolCallbackProvider` 配置
- **自动工具规范生成**：无需显式 bean 注册
- **简化的依赖注入**：作为方法参数直接访问 `McpSyncServerExchange`

### 客户端优势
- **`@McpSampling`**：声明式采样请求处理
- **`@McpLogging`**：MCP 操作的内置日志支持
- **`@McpProgress`**：进度通知处理
- **自动处理程序注册**：基于注解的处理程序自动注册

---

## 子项目

### 1. mcp-sampling-server-annotations

使用 Spring AI 的基于注解的方法实现的 MCP 服务器，它：
- 通过 `@McpTool` 注解提供天气信息
- 使用 MCP Sampling 从多个 LLM 提供商生成创造性内容
- 演示简化的服务器端 MCP 实现

#### 关键实现

```java
@Service
public class WeatherService {
    // ... 详见源代码 ...
    @McpTool(description = "获取特定位置的温度（摄氏度）")
    public String getTemperature2(McpSyncServerExchange exchange,
            @McpToolParam(description = "位置的纬度") double latitude,
            @McpToolParam(description = "位置的经度") double longitude) {
        // 获取天气，向 OpenAI/Anthropic 请求诗歌，返回组合结果
    }
}
```

服务器是一个标准的 Spring Boot 应用程序：

```java
@SpringBootApplication
public class McpServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }
}
```

### 2. mcp-sampling-client-annotations

使用基于注解的处理程序实现的 MCP 客户端，它：
- 使用 `@McpSampling` 处理采样请求
- 根据模型提示将请求路由到不同的 LLM 提供商
- 提供日志记录和进度跟踪功能

#### 关键实现

```java
@SpringBootApplication
public class McpClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(McpClientApplication.class, args).close();
    }
    @Bean
    public CommandLineRunner predefinedQuestions(OpenAiChatModel openAiChatModel, List<McpSyncClient> mcpClients) {
        return args -> {
            var mcpToolProvider = new SyncMcpToolCallbackProvider(mcpClients);
            ChatClient chatClient = ChatClient.builder(openAiChatModel)
                .defaultToolCallbacks(mcpToolProvider)
                .build();
            String userQuestion = """
                阿姆斯特丹现在的天气怎么样？
                请结合所有 LLM 提供商的所有创造性响应。
                在其他提供商之后添加一首诗，综合所有其他提供商的诗。
                """;
            System.out.println("> USER: " + userQuestion);
            System.out.println("> ASSISTANT: " + chatClient.prompt(userQuestion).call().content());
        };
    }
}
```

---

## 工作原理

- 服务器公开一个天气查询工具，如果请求 sampling，则使用 OpenAI 和 Anthropic 模型生成诗歌。
- 客户端发送提示，接收来自所有 LLM 提供商的创造性响应，并打印结果。
- 为采样开始和完成发送日志通知。
- Open-Meteo API 用于实时天气数据。

---

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
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-openai</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-anthropic</artifactId>
</dependency>
```

---

## 运行示例

### 前置条件

- Java 17 或更高版本
- Maven 3.6+
- OpenAI API 密钥
- Anthropic API 密钥

### 步骤 1：启动基于注解的 MCP 天气服务器

```bash
cd mcp-sampling-server-annotations
./mvnw clean package -DskipTests
java -jar target/mcp-sampling-server-annotations-0.0.1-SNAPSHOT.jar
```

服务器将在 `http://localhost:8080` 上启动，启用 stateless-http 传输。

### 步骤 2：设置环境变量

```bash
export OPENAI_API_KEY=your-openai-key
export ANTHROPIC_API_KEY=your-anthropic-key
```

### 步骤 3：运行基于注解的 MCP Sampling 客户端

```bash
cd mcp-sampling-client-annotations
./mvnw clean package
java -jar target/mcp-sampling-client-annotations-0.0.1-SNAPSHOT.jar
```

---

## 配置

### Stateless-HTTP 传输配置

这些示例演示了用于 MCP 通信的 **stateless-http 传输**，它比传统的 SSE（服务器发送事件）传输提供了几个优势：

- **无状态操作**：每个请求都是独立的，更容易扩展和调试
- **基于 HTTP**：使用标准的 HTTP 请求/响应，简化网络配置
- **更好的错误处理**：更可预测的错误处理和恢复
- **防火墙友好**：更好地与企业防火墙和代理配合

#### 关键配置属性

**服务器端：**
- `spring.ai.mcp.server.protocol=STREAMABLE` - 启用 stateless-http 服务器协议

**客户端：**
- `spring.ai.mcp.client.streamable-http.connections.server1.url=http://localhost:8080` - 配置 stateless-http 客户端连接到服务器

### 服务器配置（`application.properties`）

```properties
# 服务器标识
spring.ai.mcp.server.name=mcp-sampling-server-annotations
spring.ai.mcp.server.version=0.0.1

# 启用 stateless-http 服务器协议
spring.ai.mcp.server.protocol=STREAMABLE

# 禁用 banner 并配置日志记录
spring.main.banner-mode=off
logging.file.name=./model-context-protocol/sampling/mcp-sampling-server-annotations/target/server.log

# 取消注释以使用 STDIO 传输
# spring.ai.mcp.server.stdio=true
# spring.main.web-application-type=none
```

### 客户端配置（`application.properties`）

```properties
spring.application.name=mcp
spring.main.web-application-type=none

# 禁用默认聊天客户端自动配置以支持多个模型
spring.ai.chat.client.enabled=false

# API 密钥
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.anthropic.api-key=${ANTHROPIC_API_KEY}

# MCP 客户端连接使用 stateless-http 传输
spring.ai.mcp.client.streamable-http.connections.server1.url=http://localhost:8080

# 禁用工具回调以防止循环依赖
spring.ai.mcp.client.toolcallback.enabled=false

# 日志记录配置
logging.level.io.modelcontextprotocol.client=WARN
logging.level.io.modelcontextprotocol.spec=WARN
```

---

## 示例输出

当您运行基于注解的客户端时，您将看到类似于以下的输出：

```
> USER: 阿姆斯特丹现在的天气怎么样？
请结合所有 LLM 提供商的所有创造性响应。
在其他提供商之后添加一首诗，综合所有其他提供商的诗。

> ASSISTANT:
OpenAI 关于天气的诗：
**阿姆斯特丹的冬日低语**
*温度：4.2°C*

在阿姆斯特丹的怀抱中，运河映照着天空，
4.2 度的微风轻轻吹过...

Anthropic 关于天气的诗：
**运河边的沉思**
*当前状况：4.2°C*

沿着水道，自行车休息着，
冬日的空气考验着阿姆斯特丹...

天气数据：
{
  "current": {
    "time": "2025-01-23T11:00",
    "interval": 900,
    "temperature_2m": 4.2
  }
}
```

---

## 与传统实现的比较

| 方面 | 传统方法 | 基于注解的方法 |
|--------|---------------------|---------------------------|
| **工具注册** | 手动 `ToolCallbackProvider` | `@McpTool` 注解 |
| **Sampling 处理程序** | `McpSyncClientCustomizer` lambda | `@McpSampling` 方法 |
| **配置** | 编程式 bean 配置 | 注解驱动的规范 |
| **代码复杂度** | 更多样板代码 | 减少样板代码 |
| **类型安全** | 手动参数处理 | 自动参数注入 |
| **调试** | 复杂的堆栈跟踪 | 更清晰的基于注解的流程 |

---

## 基于注解的方法的优势

1. **减少样板代码**：需要更少的配置代码
2. **声明式风格**：通过注解清楚地表明意图
3. **类型安全**：自动参数验证和注入
4. **更好的 IDE 支持**：增强的代码完成和导航
5. **更容易测试**：简化单个处理程序的单元测试
6. **可维护性**：更清晰的关注点分离

---

## 相关项目

- **[主要 Sampling 示例](../)**：使用手动配置的传统实现
- **[MCP 注解](../../mcp-annotations)**：核心基于注解的 MCP 示例
- **[天气服务器](../../weather)**：简单的天气 MCP 服务器
- **[SQLite 服务器](../../sqlite)**：数据库访问 MCP 服务器
- **[文件系统服务器](../../filesystem)**：文件系统操作 MCP 服务器

---

## 其他资源

* [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
* [Spring AI MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html)
* [Spring AI MCP Client Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html)
* [Spring AI 注解](https://docs.spring.io/spring-ai/reference/1.1-SNAPSHOT/api/mcp/mcp-annotations-overview.html)
* [Java MCP 注解](https://github.com/spring-ai-community/mcp-annotations)
* [Model Context Protocol 规范](https://modelcontextprotocol.github.io/specification/)

---

## 故障排除和常见问题

- **问题：运行客户端时遇到连接被拒绝错误。**
  - 答案：确保服务器正在运行并且在配置的 URL 上可访问。
- **问题：遇到来自 OpenAI 或 Anthropic 的身份验证错误。**
  - 答案：确保您的 API 密钥在您的环境或 application.properties 中正确设置。
- **问题：如何切换到 STDIO 传输？**
  - 答案：取消注释 `application.properties` 中的相关属性，如上所示。
