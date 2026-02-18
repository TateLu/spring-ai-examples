# Spring AI - MCP Starter 客户端

此项目演示了如何在 Spring Boot 应用程序中使用 Spring AI MCP (模型上下文协议) 客户端启动器。它展示了如何连接到 MCP 服务器并将它们与 Spring AI 的工具执行框架集成。

请遵循 [MCP 客户端启动器](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html) 参考文档。

## 概述

该项目使用 Spring Boot 4.0.0 和 Spring AI 2.0.0-SNAPSHOT 创建一个演示 MCP 服务器集成的命令行应用程序。该应用程序:
- 使用 STDIO 和/或 SSE (基于 HttpClient) 传输连接到 MCP 服务器
- 与 Spring AI 的聊天功能集成
- 演示通过 MCP 服务器执行工具
- 通过 `-Dai.user.input` 命令行属性接受用户定义的问题,该属性映射到代码中的 Spring `@Value` 注解

例如,使用 `-Dai.user.input="Does Spring AI support MCP?"` 运行应用程序将通过 Spring 的属性注入将此问题注入到应用程序中,应用程序将使用它来查询 MCP 服务器。

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- Anthropic API 密钥 (Claude) (在 https://docs.anthropic.com/en/docs/initial-setup 获取)
- Brave 搜索 API 密钥 (用于 Brave 搜索 MCP 服务器) (在 https://brave.com/search/api/ 获取)

## 依赖项

该项目使用以下主要依赖项:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-mcp-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-anthropic</artifactId>
    </dependency>
</dependencies>
```

## 配置

### 应用程序属性

请查看 [MCP 客户端配置属性](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html#_configuration_properties) 文档。

应用程序可以通过 `application.properties` 或 `application.yml` 进行配置:

#### 通用属性
```properties
# 应用程序配置
spring.application.name=mcp
spring.main.web-application-type=none

# AI 提供者配置
spring.ai.anthropic.api-key=${ANTHROPIC_API_KEY}

# 启用 MCP 客户端工具回调自动配置
spring.ai.mcp.client.toolcallback.enabled=true
```

#### STDIO 传输属性

请遵循 [STDIO 配置属性](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html#_stdio_transport_properties) 文档。

为您连接的每个 STDIO 服务器配置一个单独的命名配置:

```properties
spring.ai.mcp.client.stdio.connections.brave-search.command=npx
spring.ai.mcp.client.stdio.connections.brave-search.args=-y,@modelcontextprotocol/server-brave-search
```

这里,`brave-search` 是您的连接名称。

或者,您可以使用 Claude Desktop 格式的外部 JSON 文件配置 STDIO 连接:

```properties
spring.ai.mcp.client.stdio.servers-configuration=classpath:/mcp-servers-config.json
```

示例 `mcp-servers-config.json`:

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-brave-search"
      ],
      "env": {
      }
    }
  }
}
```

#### SSE 传输属性

您还可以使用 HttpClient 连接到服务器发送事件 (SSE) 服务器。
请遵循 [SSE 配置属性](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html#_sse_transport_properties) 文档。

SSE 传输的属性以 `spring.ai.mcp.client.sse` 为前缀:

```properties
spring.ai.mcp.client.sse.connections.server1.url=http://localhost:8080
spring.ai.mcp.client.sse.connections.server2.url=http://localhost:8081
```

## 工作原理

该应用程序演示了使用 MCP 工具与 AI 模型的简单命令行交互:

1. 应用程序启动并配置多个 MCP 客户端(每个提供的 STDIO 或 SSE 连接配置一个)
2. 它使用配置的 MCP 工具构建 ChatClient
3. 向 AI 模型发送预定义的问题(通过 `ai.user.input` 属性设置)
4. 显示 AI 的响应
5. 自动关闭应用程序

## 运行应用程序

1. 设置必需的环境变量:
   ```bash
   export ANTHROPIC_API_KEY=your-api-key

   # 用于 Brave 搜索 MCP 服务器
   export BRAVE_API_KEY=your-brave-api-key
   ```

2. 构建应用程序:
   ```bash
   ./mvnw clean install
   ```

3. 运行应用程序:
   ```bash
   # 使用 application.properties 中的默认问题运行
   java -jar target/mcp-starter-default-client-0.0.1-SNAPSHOT.jar

   # 或指定自定义问题
   java -Dai.user.input='Does Spring AI support MCP?' -jar target/mcp-starter-default-client-0.0.1-SNAPSHOT.jar
   ```

应用程序将执行问题,使用配置的 MCP 工具来回答它,并显示 AI 助手的响应。

## 其他资源

- [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
- [MCP 客户端启动器](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html)
- [模型上下文协议规范](https://modelcontextprotocol.github.io/specification/)
- [Spring Boot 文档](https://docs.spring.io/spring-boot/docs/current/reference/html/)
