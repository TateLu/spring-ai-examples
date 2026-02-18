# Spring AI Model Context Protocol Demo 应用程序

一个演示应用程序，展示 Spring AI 与 SQLite 数据库的集成，使用模型上下文协议（MCP）。该应用程序通过命令行界面实现与 SQLite 数据库的自然语言交互。

它使用 [SQLite MCP-Server](https://github.com/modelcontextprotocol/servers/tree/main/src/sqlite) 来执行 SQL 查询、分析业务数据和自动生成业务洞察备忘录。

## 功能

- SQLite 数据库的自然语言查询
- 预定义问题模式，用于自动化数据库分析
- 与 OpenAI 语言模型的无缝集成
- 基于 Spring AI 和模型上下文协议构建

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- uvx 包管理器
- Git
- OpenAI API 密钥
- SQLite（可选，用于数据库修改）

## 安装

1. 安装 uvx（通用包管理器）：
   ```bash
   # 按照以下链接的安装说明操作：
   https://docs.astral.sh/uv/getting-started/installation/
   ```

2. 克隆仓库：
   ```bash
   git clone https://github.com/spring-projects/spring-ai-examples.git
   cd model-context-protocol/sqlite/simple
   ```

3. 设置您的 OpenAI API 密钥：
   ```bash
   export OPENAI_API_KEY='your-api-key-here'
   ```

## 示例 SQLite 数据库

SQLite 数据库文件可在不同操作系统之间移植。该仓库包含一个名为 `test.db` 的示例数据库文件。

它有一个 `PRODUCTS` 表，是使用 `create-database.sh` 脚本创建的。

## 运行应用程序

### 预定义问题

运行一组预设的数据库查询：
```bash
./mvnw spring-boot:run
```

## 架构概述

Spring AI 与 MCP 的集成遵循简单的组件链：

1. **MCP 客户端** 提供与数据库的基础通信层
2. **函数回调** 将数据库操作作为 AI 可调用的函数公开
3. **聊天客户端** 将这些函数连接到 AI 模型

Bean 定义如下，从 `ChatClient` 开始

### 聊天客户端

```java
@Bean
@Profile("!chat")
public CommandLineRunner predefinedQuestions(ChatClient.Builder chatClientBuilder,
                                           List<McpFunctionCallback> functionCallbacks,
                                           ConfigurableApplicationContext context) {
    return args -> {
        var chatClient = chatClientBuilder.defaultFunctions(functionCallbacks.toArray(new McpFunctionCallback[0]))
                .build();
         // 运行预定义问题
         System.out.println(chatClient.prompt(
            "Can you connect to my SQLite database and tell me what products are available, and their prices?").call().content());
         // ...
    };
}
```

聊天客户端的设置非常简单——只需要从 MCP 工具自动创建的函数回调。Spring 的依赖注入处理所有连接，使集成无缝进行。

现在让我们详细查看其他 bean 定义...

### 函数回调

应用程序使用函数回调将 MCP 工具注册到 Spring AI：

```java
@Bean
public List<McpFunctionCallback> functionCallbacks(McpSyncClient mcpClient) {
    return mcpClient.listTools(null)
            .tools()
            .stream()
            .map(tool -> new McpFunctionCallback(mcpClient, tool))
            .toList();
}
```

#### 目的

此 bean 负责：
1. 从客户端发现可用的 MCP 工具
2. 将每个工具转换为 Spring AI 函数回调
3. 使这些回调可供 ChatClient 使用


#### 工作原理

1. `mcpClient.listTools(null)` 从 MCP 服务器查询所有可用工具
   - `null` 参数表示分页游标
   - 当为 null 时，返回第一页结果
   - 可以提供游标字符串以获取该位置之后的结果
2. `.tools()` 从响应中提取工具列表
3. 使用 `.map()` 将每个工具转换为 `McpFunctionCallback`
4. 使用 `.toArray(McpFunctionCallback[]::new)` 将这些回调收集到数组中

#### 用法

注册的回调使 ChatClient 能够：
- 在对话期间访问 MCP 工具
- 处理 AI 模型请求的函数调用
- 针对 MCP 服务器（例如 SQLite 数据库）执行工具


### MCP 客户端

应用程序使用同步 MCP 客户端与 SQLite 数据库通信：

```java
@Bean(destroyMethod = "close")
public McpSyncClient mcpClient() {
    var stdioParams = ServerParameters.builder("uvx")
            .args("mcp-server-sqlite", "--db-path", getDbPath())
            .build();

    var mcpClient = McpClient.sync(new StdioServerTransport(stdioParams),
            Duration.ofSeconds(10), new ObjectMapper());

    var init = mcpClient.initialize();
    System.out.println("MCP Initialized: " + init);

    return mcpClient;
}
```

此配置：
1. 创建一个基于 stdio 的传输层，与 `uvx` MCP 服务器通信
2. 指定 SQLite 作为后端数据库及其位置
3. 设置 10 秒的操作超时
4. 使用 Jackson 进行 JSON 序列化
5. 初始化与 MCP 服务器的连接

`destroyMethod = "close"` 注释确保应用程序关闭时进行适当清理。



## 文档参考

您可以通过以下快速入门链接了解此示例应用程序的更多详细信息。

遗憾的是，在 2024 年 12 月 10 日，快速入门从 SQLite 更改为天气检索示例。

但是，如果您想了解一些详细信息，这里有[更改前的文档链接](https://github.com/modelcontextprotocol/docs/blob/1024e03f83aa0b8badde9b50dfee4d2e4e7f9446/quickstart.mdx)。

例如，您可能想要创建其他表并安装 SQLite
