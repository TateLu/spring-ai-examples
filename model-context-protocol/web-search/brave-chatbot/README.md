# Spring AI - 模型上下文协议（MCP）Brave 搜索聊天机器人

此示例演示如何构建一个交互式聊天机器人，将 Spring AI 的模型上下文协议（MCP）与 [Brave 搜索 MCP 服务器](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search) 结合。该应用程序创建一个持久聊天机器人，使用 Spring AI 的内存顾问维护对话历史记录，实现跨多次交换的上下文交互。用户可以与机器人进行持续对话，该机器人可以通过 Brave 搜索执行互联网搜索，以提供最新信息。聊天机器人持续运行，直到使用 Ctrl-C 终止，并在整个会话期间维护对话状态。

该应用程序由 Anthropic 的 Claude AI 模型驱动，可以通过 Brave 搜索执行互联网搜索，实现与实时 Web 数据的自然语言交互，同时维护对话上下文。

<img src="spring-ai-mcp-brave.jpg" width="600"/>

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- npx 包管理器
- Anthropic API 密钥（Claude）（在 https://docs.anthropic.com/en/docs/initial-setup 获取）
- Brave 搜索 API 密钥（在 https://brave.com/search/api/ 获取）

## 设置

1. 安装 npx（Node Package eXecute）：
   首先，确保安装 [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
   然后运行：
   ```bash
   npm install -g npx
   ```

2. 克隆仓库：
   ```bash
   git clone https://github.com/spring-projects/spring-ai-examples.git
   cd spring-ai-examples/model-context-protocol/web-search/brave-chatbot
   ```

3. 设置您的 API 密钥：
   ```bash
   export ANTHROPIC_API_KEY='your-anthropic-api-key-here'
   export BRAVE_API_KEY='your-brave-api-key-here'
   ```

4. 构建应用程序：
   ```bash
   ./mvnw clean install
   ```

## 运行应用程序

使用 Maven 运行应用程序：
```bash
./mvnw spring-boot:run
```

应用程序将启动一个交互式聊天会话，您可以在其中提问。聊天机器人将在需要从互联网查找信息以回答您的查询时使用 Brave 搜索。

## 工作原理

应用程序通过多个组件将 Spring AI 与 Brave 搜索 MCP 服务器集成：

### MCP 客户端配置

1. pom.xml 中所需的依赖项：
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-anthropic</artifactId>
</dependency>
```

2. 应用程序属性（application.yml）：
```yaml
spring:
  ai:
    mcp:
      client:
        enabled: true
        name: brave-search-client
        version: 1.0.0
        type: SYNC  # 或 ASYNC 用于响应式应用程序
        request-timeout: 20s
        stdio:
          root-change-notification: true
          servers-configuration: classpath:/mcp-servers-config.json
    anthropic:
      api-key: ${ANTHROPIC_API_KEY}
```

3. MCP 服务器配置（mcp-servers-config.json）：
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
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

### 客户端类型

MCP 客户端支持两种类型的实现：

- **同步（默认）**：使用阻塞操作，适用于传统的请求-响应模式
- **异步**：使用非阻塞操作，适用于响应式应用程序

您可以使用 `spring.ai.mcp.client.type` 属性（SYNC 或 ASYNC）在这些类型之间切换。

### 聊天实现

聊天机器人使用 Spring AI 的 ChatClient 和 MCP 工具集成实现：

```java
var chatClient = chatClientBuilder
    .defaultSystem("You are a useful assistant, expert in AI and Java.")
    .defaultToolCallbacks((Object[]) mcpToolAdapter.toolCallbacks())
    .defaultAdvisors(new MessageChatMemoryAdvisor(new InMemoryChatMemory()))
    .build();
```

主要功能：
- 使用 Claude AI 模型进行自然语言理解
- 通过 MCP 集成 Brave 搜索，实现实时 Web 搜索功能
- 使用 InMemoryChatMemory 维护对话记忆
- 作为交互式命令行应用程序运行

聊天机器人可以：
- 使用其内置知识回答问题
- 在需要时使用 Brave 搜索进行 Web 搜索
- 记住对话中先前消息的上下文
- 结合来自多个来源的信息提供全面的答案

### 高级配置

MCP 客户端支持其他配置选项：

- 通过 `McpSyncClientCustomizer` 或 `McpAsyncClientCustomizer` 进行客户端自定义
- 多种传输类型：STDIO 和 SSE（服务器发送事件）
- 与 Spring AI 的工具执行框架集成
- 自动客户端初始化和生命周期管理

对于基于 WebFlux 的应用程序，您可以改用 WebFlux starter：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-client-webflux</artifactId>
</dependency>
```

这提供了类似的功能，但使用基于 WebFlux 的 SSE 传输实现，建议用于生产部署。
