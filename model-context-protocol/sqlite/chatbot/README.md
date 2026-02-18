# Spring AI 模型上下文协议 - SQLite 聊天机器人

一个演示应用程序,展示了使用模型上下文协议 (MCP) 与 SQLite 数据库的 Spring AI 集成。该应用程序通过命令行界面启用与 SQLite 数据库的自然语言交互。

它使用 [SQLite MCP 服务器](https://github.com/modelcontextprotocol/servers/tree/main/src/sqlite) 来启用运行 SQL 查询、分析业务数据和自动生成业务洞察备忘录。

该演示启动一个简单的聊天机器人,您可以在其中询问关于存储在数据库中的数据的问题。

例如:

> 你能连接到我的 SQLite 数据库并告诉我有哪些产品及其价格吗?

或即时执行一些数据聚合:

> 数据库中所有产品的平均价格是多少?

运行分析:

> 你能分析价格分布并建议任何定价优化吗?

甚至创建一个新表:

> 你能帮我设计并创建一个用于存储客户订单的新表吗?

## 功能

- SQLite 数据库的自然语言查询
- 用于动态数据库交互的交互式聊天模式
- 与 OpenAI 语言模型的无缝集成
- 基于 Spring AI 和模型上下文协议构建

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- uvx 包管理器
- Git
- OpenAI API 密钥
- SQLite (可选,用于数据库修改)

## 安装

1. 安装 uvx (通用包管理器):
   ```bash
   # 按照以下链接的安装说明操作:
   https://docs.astral.sh/uv/getting-started/installation/
   ```

2. 克隆仓库:
   ```bash
   git clone https://github.com/spring-projects/spring-ai-examples.git
   cd model-context-protocol/sqlite/chatbot
   ```

3. 设置您的 OpenAI API 密钥:
   ```bash
   export OPENAI_API_KEY='your-api-key-here'
   ```

## 示例 SQLite 数据库

SQLite 数据库文件可跨操作系统移植。此仓库包含一个名为 `test.db` 的示例数据库文件。

它有一个 `PRODUCTS` 表,是使用脚本 `create-database.sh` 创建的。

## 运行应用程序

### 交互式聊天

启用与数据库的实时对话:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=chat
```

## 架构概述

### MCP 客户端配置

应用程序使用同步 MCP 客户端与 SQLite 数据库通信:

```java
@Bean(destroyMethod = "close")
public McpSyncClient mcpClient() {

    var stdioParams = ServerParameters.builder("uvx")
            .args("mcp-server-sqlite", "--db-path", getDbPath())
            .build();

    var mcpClient = McpClient.sync(new StdioServerTransport(stdioParams),
            Duration.ofSeconds(10), new ObjectMapper());

    var init = mcpClient.initialize();

    System.out.println("MCP 已初始化: " + init);

    return mcpClient;
}
```

此配置:
1. 创建一个基于 stdio 的传输层,该层与 `uvx` MCP 服务器通信
2. 将 SQLite 指定为后端数据库及其位置
3. 为操作设置 10 秒超时
4. 使用 Jackson 进行 JSON 序列化
5. 初始化与 MCP 服务器的连接

`destroyMethod = "close"` 注解确保应用程序关闭时正确清理。

### 函数回调

应用程序使用函数回调向 Spring AI 注册 MCP 工具:

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

此 bean 负责:
1. 从客户端发现可用的 MCP 工具
2. 将每个工具转换为 Spring AI 函数回调
3. 使这些回调可用于 ChatClient

#### 工作原理

1. `mcpClient.listTools(null)` 向 MCP 服务器查询所有可用工具
   - `null` 参数表示分页游标
   - 为 null 时,返回结果的第一页
   - 可以提供游标字符串以获取该位置之后的结果
2. `.tools()` 从响应中提取工具列表
3. 每个工具使用 `.map()` 转换为 `McpFunctionCallback`
4. 这些回调使用 `.toArray(McpFunctionCallback[]::new)` 收集到数组中

#### 用法

注册的回调使 ChatClient 能够:
- 在对话期间访问 MCP 工具
- 处理 AI 模型请求的函数调用
- 针对 MCP 服务器执行工具(例如,SQLite 数据库)

## 文档参考

您可以按照特定版本的 github 中的此快速启动链接查找有关此示例应用程序的更多信息。

不幸的是,在 2024 年 12 月 10 日,快速启动已从 SQLite 更改为天气检索示例。

但是,这里是[链接](https://github.com/modelcontextprotocol/docs/blob/1024e03f83aa0b8badde9b50dfee4d2e4e7f9446/quickstart.mdx)到更改前的文档,如果您想了解一些详细信息。

例如,您可能想要创建其他表并安装 SQLite
