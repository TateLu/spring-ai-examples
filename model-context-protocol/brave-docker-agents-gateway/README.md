# Spring AI - 模型上下文协议 (MCP) Brave 搜索示例

此示例演示了如何创建一个与 [Brave 搜索 MCP 服务器](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search) 通信的 Spring AI 模型上下文协议 (MCP) 客户端。该应用程序展示了如何构建一个 MCP 客户端,以启用与 Brave 搜索的自然语言交互,允许您通过对话界面执行互联网搜索。此示例使用 Spring Boot 自动配置通过配置文件设置 MCP 客户端。

运行时,应用程序通过询问特定问题来演示 MCP 客户端的功能:"Spring AI 是否支持模型上下文协议?请提供一些参考。" MCP 客户端使用 Brave 搜索查找相关信息并返回全面的答案。提供响应后,应用程序退出。

<img src="spring-ai-mcp-brave.jpg" width="600"/>

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- Docker Desktop 4.41 或更高版本
- Git
- OpenAI API 密钥
- Brave 搜索 API 密钥 (在 https://brave.com/search/api/ 获取)

## 设置

1. **启动 Docker 化服务:**
    确保 Docker (例如 Docker Desktop 4.41 或更高版本,或带有 Docker Compose CLI 的 Docker Engine) 已安装并运行。然后,在项目的根目录(其中 `compose.yml` 所在的位置)中,启动必要的服务:
    ```bash
    docker compose up
    ```

2. 克隆仓库:
   ```bash
   git clone https://github.com/spring-projects/spring-ai-examples.git
   cd model-context-protocol/brave-docker-agents-gateway
   ```

3. 设置您的 API 密钥:
   ```bash
   export OPENAI_API_KEY='your-openai-api-key-here'
   export BRAVE_API_KEY='your-brave-api-key-here'
   ```

4. 构建应用程序:
   ```bash
   ./mvnw clean install
   ```

## 运行应用程序

使用 Maven 运行应用程序:

```bash
./mvnw spring-boot:run
```

应用程序将执行单个查询,询问 Spring AI 对模型上下文协议的支持。它使用 Brave 搜索 MCP 服务器在互联网上搜索相关信息,通过 MCP 客户端处理结果,并在退出前提供详细的响应。

## 工作原理

应用程序通过 Spring Boot 自动配置将 Spring AI 与 Brave 搜索 MCP 服务器集成:

### MCP 客户端配置

使用配置文件配置 MCP 客户端:

1. `application.properties`:
```properties
spring.ai.mcp.client.see.gateway.url=http://localhost:8811
```

此配置:
1. 使用 Docker MCP 网关通过 Brave 搜索 MCP 服务器
2. Brave API 密钥从环境变量传递
3. 初始化与服务器的同步连接

### 聊天集成

在 Application 类中配置带有 MCP 工具回调的 ChatClient:

```java
var chatClient = chatClientBuilder
        .defaultToolCallbacks(new SyncMcpToolCallbackProvider(mcpSyncClients))
        .build();
```

此设置允许 AI 模型:
- 理解何时使用 Brave 搜索
- 适当地格式化查询
- 处理搜索结果并将其纳入响应中

## 依赖项

该项目使用:
- Spring Boot 4.0.0
- Spring AI 2.0.0-SNAPSHOT
- spring-ai-starter-model-openai
- spring-ai-starter-mcp-client
