# Spring Boot OpenAI 流式集成

此 Spring Boot 应用程序演示了使用 Spring AI 与 OpenAI API 的实时流式集成。它提供了一个使用 Spring WebFlux 流式传输 AI 响应的简单端点。

## 前置条件

- Java 17 或更高版本
- Maven
- OpenAI API 密钥

所有必需的依赖项,包括 web、webflux 和 OpenAI 的 Spring Boot 启动器,都在 Maven pom.xml 中声明。

## 配置

1. 在 `src/main/resources` 中创建 `application.properties` 或 `application.yml` 文件
2. 添加您的 OpenAI API 密钥:

```properties
spring.ai.openai.api-key=your-api-key-here
```

## 运行应用程序

1. 克隆此仓库
2. 如上所述配置您的 OpenAI API 密钥
3. 运行应用程序:

```bash
./mvnw spring-boot:run
```

应用程序默认将在端口 8080 上启动。

## 使用

应用程序公开了一个流式端点,将 OpenAI 响应作为响应式流返回。

### 端点

```
GET /ai/generateStream
```

#### 参数
- `message` (可选): 发送到 OpenAI 的输入消息
    - 默认值: "Tell me a joke"

#### 使用 curl 测试

您可以使用 curl 测试流式端点。以下命令将流式传输响应并使用 `jq` 进行格式化:

```bash
curl localhost:8080/ai/generateStream | sed 's/data://' | jq .
```

## 实现细节

应用程序使用:
- Spring WebFlux 用于响应式流式传输
- Spring AI 的 OpenAI 集成用于 AI 模型交互

主要组件:
1. `OpenAiStreamingApplication`: Spring Boot 应用程序入口点
2. `ChatController`: 处理流式端点的 REST 控制器

### 代码详情

流式端点实现如下:

```java
@GetMapping(value = "/generateStream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<ChatResponse> generateStream(
       @RequestParam(value = "message", defaultValue = "Tell me a joke") String message) {
    return chatModel.stream(new Prompt(new UserMessage(message)));
}
```

关键点:
- 端点使用 `MediaType.TEXT_EVENT_STREAM_VALUE` 指示响应将作为文本事件流式传输。
- 返回一个 `Flux<ChatResponse>`,这是 Spring WebFlux 用于处理多个元素流的响应式类型。
- 接受一个可选的 `message` 参数,默认值为 "Tell me a joke"。
- 使用 Spring AI 的 `chatModel.stream()` 方法,该方法返回来自 OpenAI 的响应流。
- `Prompt` 和 `UserMessage` 类由 Spring AI 提供,用于构建对 OpenAI 的请求。

调用时,此端点:
1. 接受用户的输入消息(或使用默认值)
2. 将其包装在 Spring AI `Prompt` 对象中
3. 实时将 AI 的响应流式传输回客户端
4. 每个响应块自动序列化为 JSON
