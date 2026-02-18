# Spring AI - 模型上下文协议（MCP）Brave 搜索示例

此示例演示如何创建一个 Spring AI 模型上下文协议（MCP）客户端，该客户端与 [Brave 搜索 MCP 服务器](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search)通信。该应用程序展示了如何构建一个 MCP 客户端，实现与 Brave 搜索的自然语言交互，允许通过对话界面进行互联网搜索。此示例使用 Spring Boot 自动配置通过配置文件设置 MCP 客户端。

运行时，应用程序通过询问一个特定问题来演示 MCP 客户端的功能："Does Spring AI supports the Model Context Protocol? Please provide some references." MCP 客户端使用 Brave 搜索查找相关信息并返回全面的答案。提供响应后，应用程序退出。

<img src="spring-ai-mcp-brave.jpg" width="600"/>

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- npx 包管理器
- Git
- OpenAI API 密钥
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
   cd model-context-protocol/brave
   ```

3. 设置您的 API 密钥：
   ```bash
   export OPENAI_API_KEY='your-openai-api-key-here'
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

应用程序将执行单个查询，询问关于 Spring AI 对模型上下文协议的支持。它使用 Brave 搜索 MCP 服务器搜索互联网上的相关信息，通过 MCP 客户端处理结果，并在退出前提供详细响应。

## 工作原理

应用程序通过 Spring Boot 自动配置将 Spring AI 与 Brave 搜索 MCP 服务器集成：

### MCP 客户端配置

MCP 客户端使用配置文件进行配置：

1. `application.properties`：
```properties
spring.ai.mcp.client.stdio.servers-configuration=classpath:/mcp-servers-config.json
```

2. `mcp-servers-config.json`：
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

此配置：
1. 通过 npx 使用 Brave 搜索 MCP 服务器
2. Brave API 密钥从环境变量传递
3. 初始化与服务器的同步连接

### 聊天集成

ChatClient 在 Application 类中使用 MCP 工具回调进行配置：

```java
var chatClient = chatClientBuilder
        .defaultToolCallbacks(new SyncMcpToolCallbackProvider(mcpSyncClients))
        .build();
```

此设置使 AI 模型能够：
- 理解何时使用 Brave 搜索
- 适当地格式化查询
- 处理并将搜索结果纳入响应

## 依赖项

项目使用：
- Spring Boot 4.0.0
- Spring AI 2.0.0-SNAPSHOT
- spring-ai-starter-model-openai
- spring-ai-starter-mcp-client
