# Spring AI - 模型上下文协议（MCP）Brave 搜索示例

此示例演示如何将 Spring AI 的模型上下文协议（MCP）与 [Brave 搜索 MCP 服务器](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search) 一起使用，利用 Spring Boot 的自动配置功能。该应用程序实现与 Brave 搜索的自然语言交互，允许通过对话界面进行互联网搜索。运行时，应用程序通过询问一个特定问题来演示 MCP 客户端的功能："Does Spring AI supports the Model Context Protocol? Please provide some references." MCP 客户端使用 Brave 搜索查找相关信息并返回全面的答案。提供响应后，应用程序退出。

与手动配置方法不同，此示例使用 Spring Boot starter，它会自动为您创建 MCP 客户端，将配置移至 `application.properties` 和 `mcp-servers-config.json`。

## 依赖项

项目使用以下关键依赖项：

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

<img src="spring-ai-mcp-brave.jpg" width="600"/>

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- npx 包管理器
- Anthropic API 密钥
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
   cd model-context-protocol/brave-starter
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

应用程序将通过询问一个关于 Spring AI 和模型上下文协议的示例问题来演示集成，利用 Brave 搜索收集信息。

## 工作原理

应用程序使用 Spring Boot 的自动配置功能来设置 MCP 客户端。配置主要通过两个文件完成：

### 项目依赖项

项目使用 Spring AI 的 MCP 客户端 Spring Boot starter 和 Anthropic starter：

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

### 配置文件

MCP 客户端可以通过两种不同的方式进行配置，两者都能实现相同的行为：

#### 选项 1：直接在 application.properties 中配置

此方法直接在 application.properties 文件中配置 MCP 客户端：

```properties
spring.application.name=mcp
spring.main.web-application-type=none
spring.ai.anthropic.api-key=${ANTHROPIC_API_KEY}

# 直接 MCP 客户端配置
spring.ai.mcp.client.stdio.connections.brave-search.command=npx
spring.ai.mcp.client.stdio.connections.brave-search.args=-y,@modelcontextprotocol/server-brave-search
```

#### 选项 2：外部配置文件

或者，您可以将 MCP 配置移至外部文件。此方法类似于 anthropic 独立客户端的配置方式。

1. 在 `application.properties` 中，启用外部配置并注释掉直接配置：
```properties
spring.application.name=mcp
spring.main.web-application-type=none
spring.ai.anthropic.api-key=${ANTHROPIC_API_KEY}

# 使用外部配置文件
spring.ai.mcp.client.stdio.servers-configuration=classpath:/mcp-servers-config.json

# 使用外部文件时注释掉直接配置
# spring.ai.mcp.client.stdio.connections.brave-search.command=npx
# spring.ai.mcp.client.stdio.connections.brave-search.args=-y,@modelcontextprotocol/server-brave-search
```

2. 在 `mcp-servers-config.json` 中 - 定义 MCP 服务器配置：
```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-brave-search"
      ],
      "env": {}
    }
  }
}
```

### 应用程序代码

主应用程序（`Application.java`）使用自动配置的组件创建聊天客户端并执行单个问题：

```java
@SpringBootApplication
public class Application {
    @Bean
    public CommandLineRunner predefinedQuestions(ChatClient.Builder chatClientBuilder,
            List<ToolCallback> tools, ConfigurableApplicationContext context) {
        return args -> {
            var chatClient = chatClientBuilder
                    .defaultToolCallbacks(tools)
                    .build();

            String question = "Does Spring AI support the Model Context Protocol? Please provide some references.";
            System.out.println("QUESTION: " + question);
            System.out.println("ASSISTANT: " + chatClient.prompt(question).call().content());

            context.close();
        };
    }
}
```

应用程序使用 Spring Boot 的自动配置根据属性和配置文件自动创建和配置 MCP 客户端，无需为客户端本身显式定义 bean。


# MCP 配置
spring.ai.mcp.client.stdio.enabled=true
spring.ai.mcp.client.stdio.servers-configuration=classpath:/mcp-servers-config.json
```

2. `mcp-servers-config.json` - MCP 服务器配置：
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

### 自动配置

Spring Boot 的自动配置处理设置：
- 将应用程序配置为命令行工具（非 Web）
- 使用提供的 API 密钥设置 Anthropic 集成
- 启用 MCP STDIO 客户端与 Brave 搜索服务器通信
- 提供包含 Brave 搜索功能的 `List<ToolCallback>` bean，该 bean 会自动注入到 `CommandLineRunner` 中

### 主应用程序

`Application.java` 文件演示了一个简单的 Spring Boot 应用程序，它：

1. 使用 Spring Boot 的 `CommandLineRunner` 在应用程序启动时执行预定义问题
2. 创建一个具有自动配置的 MCP 工具（Brave 搜索功能）的 `ChatClient`
3. 询问一个关于 Spring AI 和模型上下文协议的特定问题
4. 将问题和 AI 助手的响应都打印到控制台
5. 收到响应后自动关闭应用程序

以下是 `Application.java` 中的关键代码：

```java
@Bean
public CommandLineRunner predefinedQuestions(ChatClient.Builder chatClientBuilder,
        List<ToolCallback> tools, ConfigurableApplicationContext context) {
    return args -> {
        var chatClient = chatClientBuilder
                .defaultToolCallbacks(tools)
                .build();

        String question = "Does Spring AI support the Model Context Protocol? "
                + "Please provide some references.";

        System.out.println("QUESTION: " + question);
        System.out.println("ASSISTANT: " + chatClient.prompt(question).call().content());

        context.close();
    };
}
```

应用程序自动：
- 注入 `ChatClient.Builder` 和 `ToolCallback` 列表（包含 Brave 搜索功能）
- 使用可用工具配置聊天客户端
- 执行预定义问题
- 在需要时使用 Brave 搜索收集信息以提供响应

此设置使 AI 模型能够：
- 理解何时使用 Brave 搜索
- 适当地格式化查询
- 处理并将搜索结果纳入响应
