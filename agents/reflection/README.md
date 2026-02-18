# Spring AI Reflection Agent 应用程序

此项目演示了使用 Spring AI 创建自我改进代码生成系统的用法。Reflection Agent 使用两个 ChatClient 实例进行迭代循环——一个用于生成，一个用于批评——以生成高质量的 Java 代码。

它基于仓库 https://github.com/neural-maze/agentic_patterns 中的代码

该应用程序实现了一个基于反射的系统，其中 **Reflection Agent**：

- 使用一个 **生成 ChatClient** 根据用户提示创建代码
- 使用一个 **批评 ChatClient** 审查生成的代码
- 通过将批评反馈给 **生成 ChatClient** 来迭代地改进代码
- 继续此循环��直到 **批评 ChatClient** 对质量满意为止


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

## 项目结构
### 主要组件

* `Application.java`：提供命令行界面的主 Spring Boot 应用程序
* `ReflectionAgent.java`：管理生成和批评之间迭代的核心组件


## 工作原理

### 初始设置

Reflection Agent 创建两个 `ChatClient` 实例：

- `generateChatClient`：用于根据用户请求生成 Java 代码
- `critiqueChatClient`：用于审查和批评生成的代码

## 生成过程

- 用户输入请求
- 生成 `ChatClient` 创建初始代码
- 批评 `ChatClient` 审查代码
- 如果需要改进，生成 `ChatClient` 创建修订版本
- 这将持续最多 `maxIterations` 次或直到批评 `ChatClient` 批准（`<OK>`）
-
## ChatClient 配置

- **生成 ChatClient** 系统提示：

```text
您是一名 Java 程序员，负责生成高质量的
Java 代码。您的任务是为用户的请求
生成最好的内容。
```
- **批评 ChatClient** 系统提示：
```text
您负责为用户的生成内容
生成批评和建议。如果用户内容有
错误或需要改进，请输出建议
和批评的列表。
```

## 示例运行

在此示例运行中，用户请求为 `Person` 类创建一个 JUnit 5 测试。有关实际输出，请参阅文件 `JacksonTestAgent.md`。

### 初始生成

- 生成 `ChatClient` 创建了基本的 `Person` 类和测试类
- 包括序列化/反序列化功能
- 实现了基本测试用例

### 批评阶段

批评 `ChatClient` 识别了几项改进：

- 更好的错误处理
- 改进的代码可读性
- 需要边缘情况测试
- 更好的测试结构
- 扩展的测试覆盖范围
- 增强的 Java 类结构
- 现代 Java 功能的使用

### 最终结果

生成 `ChatClient` 创建了改进的代码，包括：

- 更好的模块化的分离测试方法
- 增强的错误处理和详细消息
- 添加了 null 值的边缘情况测试
- 改进的代码结构和可读性
- 更好的测试覆盖范围

## MergeSort

文件 `AgentMergeSort.md` 显示了类似的运行，用于创建归并排序算法。还有一个为生成的代码创建的 JUnit 测试，以演示其工作正常。
