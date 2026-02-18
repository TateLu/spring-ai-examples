# Spring AI Hello World 聊天应用程序

一个简单的命令行聊天应用程序，演示 Spring AI 的 ChatClient 功能与 AI 模型。

## 前置条件
- Java 17 或更高版本
- Maven

此示例使用 OpenAI 作为模型提供商。

在使用 AI 命令之前，请确保您拥有来自 OpenAI 的开发者令牌。

在 [OpenAI 注册](https://platform.openai.com/signup) 创建账户，并在 [API 密钥](https://platform.openai.com/account/api-keys) 生成令牌。

Spring AI 项目定义了一个名为 `spring.ai.openai.api-key` 的配置属性，您应该将其设置为您从 OpenAI 获得的 API 密钥的值。

导出环境变量是设置该配置属性的一种方法：

```shell
export SPRING_AI_OPENAI_API_KEY=<在此处插入密钥>
```



## 运行应用程序
1. 克隆仓库
2. 导航到项目目录
3. 使用 Maven wrapper 运行应用程序：
   `./mvnw spring-boot:run`

## 示例交互
启动后，您将看到 Spring Boot banner，然后是：

```text
让我们聊天！
USER: 给我讲个笑话
ASSISTANT: 为什么骷髅不会互相打架？
它们没有胆量。
```

## 技术细节

> **[架构文档 (Architecture Documentation)](docs/architecture.md)** - 查看 Spring AI ChatClient 的详细技术架构、实现原理和数据流程图

应用程序使用 Spring AI 的 ChatClient 与 AI 模型交互。以下演示代码展示了在 Spring Boot 应用程序中创建聊天交互的基本设置：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    CommandLineRunner cli(ChatClient.Builder builder) {
        return args -> {
            var chat = builder.build();
            var scanner = new Scanner(System.in);
            System.out.println("\n让我们聊天！");
            while (true) {
                System.out.print("\nUSER: ");
                System.out.println("ASSISTANT: " +
                        chat.prompt(scanner.nextLine()).call().content());
            }
        };
    }
}
```

# 解释
1. 应用程序入口点：

* `Application` 类使用 `@SpringBootApplication` 注解，标记它为 Spring Boot 应用程序的主入口点。
* `main` 方法通过调用 SpringApplication.run 启动应用程序。

2. 命令行运行器：

* 定义了一个 `CommandLineRunner` bean 来在应用程序启动后启动聊天功能。
* 一个 `ChatClient.Builder` 用于创建 ChatClient 实例，它作为 AI 交互的核心接口。


3. 用户交互：

* 使用 `Scanner` 捕获来自命令行的用户输入。
* 在连续循环中，应用程序从用户读取一行（``USER``）并将其作为提示发送给 `ChatClient`。
* 助手的响应使用 `chat.prompt(...).call().content()` 获取并显示为 `ASSISTANT'` 的回复。
* 这个简单的循环持续，直到用户终止应用程序。
