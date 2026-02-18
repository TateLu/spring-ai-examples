# Kotlin Spring AI Hello World 应用程序

一个用 Kotlin 编写的简单 "Hello World" Spring Boot 应用程序，演示 Spring AI 集成，从 OpenAI 获取笑话。

## 概述

此 hello world 应用程序展示了如何在 Kotlin Spring Boot 项目中开始使用 Spring AI 和 OpenAI。运行时，它连接到 OpenAI 的 API 并请求一个笑话，然后显示设置和笑点——这是对传统 "Hello World" 示例的有趣转折。该应用程序演示了 Kotlin 的扩展函数功能，使用 `org.springframework.ai.chat.client.entity` 自动将聊天响应转换为 `Joke` 数据对象，展示了 Kotlin 的类型安全构建器如何使 AI 响应处理更加优雅和类型安全。

## 前置条件

- JDK 17 或更高版本
- OpenAI API 密钥

## 依赖项

应用程序使用以下主要依赖项：
- Spring Boot
- Spring AI OpenAI starter
- Kotlin 标准库和反射
- Spring Web starter

## 配置

将您的 OpenAI API 密钥添加到 `application.properties` 或 `application.yml`：
```yaml
spring.ai.openai.api-key=your-api-key-here
```

## 工作原理

应用程序使用 Spring AI 的 OpenAI 集成来对 OpenAI 的服务进行 API 调用。主要组件包括：

1. 一个 Spring Boot 应用程序类（`KotlinHelloWorldApplication`）
2. 一个 `CommandLineRunner` bean，它：
- 创建一个聊天客户端
- 发送一个请求笑话的提示
- 将响应解析为 `Joke` 对象
- 打印设置和笑点

## 运行应用程序

```bash
./mvnw spring-boot:run
```

应用程序将启动，从 OpenAI 获取一个笑话，并在控制台中显示它。

## 示例输出

```
笑话：
设置：为什么程序员不喜欢大自然？
笑点：它有太多的 Bug！
```

## 注意事项

- 确保保持您的 OpenAI API 密钥安全，永远不要将其提交到版本控制
- 应用程序需要活动的互联网连接才能与 OpenAI 的 API 通信
- 根据您的 OpenAI 账户设置，可能会适用速率限制和 API 使用费用
