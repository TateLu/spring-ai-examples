# Spring AI 模型上下文协议 - 文件系统演示

一个跨平台演示应用程序,展示了 Spring AI 与模型上下文协议 (MCP) 文件系统服务器的集成。该应用程序启用与本地文件系统的自然语言交互。

连接到 [文件系统 MCP 服务器](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) 并访问 `target` 目录。

## 平台支持

✅ **Windows** - npx 的自动 `cmd.exe` 包装器
✅ **Linux** - 直接 npx 执行
✅ **macOS** - 直接 npx 执行

应用程序 **自动检测**您的操作系统并相应地配置 MCP 客户端。

## 功能

- ✨ **跨平台** - 在 Windows、Linux 和 macOS 上无需修改即可工作
- 🤖 自然语言查询和更新本地文件系统
- 📝 用于自动分析的预定义问题模式
- 🔄 两种配置方法: 编程(默认)或基于 JSON
- 🚀 基于 Spring AI 和模型上下文协议构建

## 前置条件

- Java 17 或更高版本
- Maven 3.6+
- Node.js 和 npx (npm 随 Node.js 一起提供)
- OpenAI API 密钥

### 安装 Node.js/npx

**Windows:**
```cmd
# 从 https://nodejs.org 下载并安装
# npx 随 npm 一起提供
npx --version
```

**Linux (Ubuntu/Debian):**
```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
npx --version
```

**macOS:**
```bash
brew install node
npx --version
```

## 快速开始

1. **克隆仓库:**
   ```bash
   git clone https://github.com/spring-projects/spring-ai-examples.git
   cd spring-ai-examples/model-context-protocol/filesystem
   ```

2. **设置您的 OpenAI API 密钥:**

   **Linux/macOS:**
   ```bash
   export OPENAI_API_KEY='your-api-key-here'
   ```

   **Windows (命令提示符):**
   ```cmd
   set OPENAI_API_KEY=your-api-key-here
   ```

   **Windows (PowerShell):**
   ```powershell
   $env:OPENAI_API_KEY="your-api-key-here"
   ```

3. **创建示例测试文件:**

   **Linux/macOS:**
   ```bash
   ./create-text-file.sh
   ```

   **Windows:**
   ```cmd
   mkdir target
   echo 示例内容 > target\spring-ai-mcp-overview.txt
   ```

4. **运行应用程序:**

   **Linux/macOS:**
   ```bash
   ./mvnw spring-boot:run
   ```

   **Windows:**
   ```cmd
   .\mvnw.cmd spring-boot:run
   ```

## 配置方法

应用程序支持两种配置 MCP 客户端的方法:

### 选项 1: 编程配置(默认 - 推荐)

默认方法使用 `Application.java` 中的自动操作系统检测:

```java
@Bean(destroyMethod = "close")
@ConditionalOnMissingBean(McpSyncClient.class)
public McpSyncClient mcpClient() {
    ServerParameters stdioParams;

    if (isWindows()) {
        // Windows: cmd.exe /c npx 方法
        var winArgs = new ArrayList<>(Arrays.asList(
            "/c", "npx", "-y", "@modelcontextprotocol/server-filesystem", "target"));
        stdioParams = ServerParameters.builder("cmd.exe")
                .args(winArgs)
                .build();
    } else {
        // Linux/Mac: 直接 npx 方法
        stdioParams = ServerParameters.builder("npx")
                .args("-y", "@modelcontextprotocol/server-filesystem", "target")
                .build();
    }

    // 创建并初始化客户端...
}
```

**优势:**
- ✅ 在所有平台上开箱即用
- ✅ 无需配置文件
- ✅ 自动操作系统检测

**劣势:**
- ❌ 配置在 Java 中硬编码
- ❌ 对不同的服务器配置灵活性较低

### 选项 2: JSON 配置(可选)

为了获得更大的灵活性,您可以使用基于 JSON 的配置。编辑 `src/main/resources/application.properties`:

**对于 Windows:**
```properties
spring.ai.mcp.client.stdio.servers-configuration=classpath:/mcp-servers-config-windows.json
```

**对于 Linux/macOS:**
```properties
spring.ai.mcp.client.stdio.servers-configuration=classpath:/mcp-servers-config-linux.json
```

**优势:**
- ✅ 外部化配置
- ✅ 无需重新编译即可轻松修改
- ✅ 可以配置多个 MCP 服务器

**劣势:**
- ❌ 必须选择正确的操作系统特定 JSON 文件
- ❌ 需要手动配置

**⚠️ 重要:** 启用 JSON 配置时,通过 `@ConditionalOnMissingBean` 自动跳过编程的 `@Bean` 以避免冲突。

## 为什么 Windows 需要特殊处理

在 Windows 上,`npx` 作为 **批处理文件** (.cmd) 实现,而不是原生可执行文件。Java 的 `ProcessBuilder`(由 `StdioClientTransport` 内部使用) 无法直接执行批处理文件。

**解决方案:** 使用 `cmd.exe /c` 包装命令:

```java
// Windows
ServerParameters.builder("cmd.exe")
    .args("/c", "npx", "-y", "@modelcontextprotocol/server-filesystem", "target")

// vs. Linux/macOS
ServerParameters.builder("npx")
    .args("-y", "@modelcontextprotocol/server-filesystem", "target")
```

此模式适用于其他 Windows 批处理文件:`npm.cmd`、`node.cmd` 等。

## 架构概述

### 跨平台 MCP 客户端创建

应用程序使用 Spring 的 `@ConditionalOnMissingBean` 支持两种配置方法:

1. **编程 Bean** - 当未启用 JSON 配置时创建
2. **自动配置** - 当启用 JSON 配置时创建

```java
@Bean(destroyMethod = "close")
@ConditionalOnMissingBean(McpSyncClient.class)
public McpSyncClient mcpClient() {
    // 仅在自动配置不提供客户端时创建
}
```

### 支持双方法的 CommandLineRunner

`CommandLineRunner` 接受两种方法:

```java
@Bean
public CommandLineRunner predefinedQuestions(
        @Autowired(required = false) List<McpSyncClient> mcpSyncClients,  // 来自 JSON 配置
        @Autowired(required = false) McpSyncClient mcpClient,              // 来自编程
        ...) {

    // 使用可用的任何一种
    List<McpSyncClient> clients = (mcpSyncClients != null && !mcpSyncClients.isEmpty())
            ? mcpSyncClients
            : (mcpClient != null ? List.of(mcpClient) : List.of());
}
```

### 工具集成

MCP 工具被自动发现并与 Spring AI 集成:

```java
var chatClient = chatClientBuilder
    .defaultToolCallbacks(new SyncMcpToolCallbackProvider(clients))
    .build();
```

AI 模型随后可以通过自然语言调用 MCP 文件系统工具(read_file、write_file 等)。

## JSON 配置文件

### mcp-servers-config-windows.json

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "cmd.exe",
      "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-filesystem", "target"],
      "env": {}
    }
  }
}
```

### mcp-servers-config-linux.json

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "target"],
      "env": {}
    }
  }
}
```

## 路径处理

该示例使用 **相对路径** (`"target"`) 而不是绝对路径,以实现跨平台兼容性:

```java
// ✅ 推荐: 相对路径
.args("-y", "@modelcontextprotocol/server-filesystem", "target")

// ❌ 避免: 带有操作系统特定分隔符的绝对路径
.args("-y", "@modelcontextprotocol/server-filesystem", "/home/user/project/target")
```

MCP 服务器根据当前工作目录解析相对路径。

## 故障排除

### Windows: "无法运行程序 'npx'"

**原因:** npx 不在 PATH 中或 ProcessBuilder 无法直接执行的批处理文件。

**解决方案:** 确保应用程序使用 `cmd.exe` 包装器(使用默认编程方法应该是自动的)。

### Bean 冲突: "Sinks.many().unicast() sinks only allow a single Subscriber"

**原因:** 编程和 JSON 配置同时创建 MCP 客户端。

**解决方案:** 选择一种方法:
- 在 `application.properties` 中注释掉 JSON 配置(使用编程)
- 或启用 JSON 配置(编程将自动跳过)

### Linux/macOS: "npx: command not found"

**原因:** 未安装 Node.js/npm 或不在 PATH 中。

**解决方案:** 安装 Node.js:
```bash
# Linux
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS
brew install node
```

## 示例输出

```
检测到类 Unix 操作系统 - 直接使用 npx
MCP 已初始化: InitializeResult[protocolVersion=2024-11-05, ...]

使用 AI 模型响应运行预定义问题:

问题: 你能解释一下 target/spring-ai-mcp-overview.txt 文件的内容吗?
助手: 该文件包含模型上下文协议 (MCP) Java SDK 的概述...

问题: 请总结内容...并将其存储在 target/summary.md 中
助手: 我已创建一个摘要并将其保存在 target/summary.md 中...
```

## 了解更多

- [模型上下文协议规范](https://modelcontextprotocol.io)
- [MCP 文件系统服务器](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem)
- [Spring AI 文档](https://docs.spring.io/spring-ai/reference/)
- [Spring AI MCP 客户端文档](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-client-boot-starter-docs.html)
