# Spring AI Kotlin Function Callback Demo

此项目演示了使用 Kotlin 在 Spring AI 中实现 Function Callbacks，特别是展示如何集成 OpenAI 的 API 来处理与天气相关的查询。

## 概述

该应用程序展示如何：
- 使用 Kotlin 设置 Spring AI
- 为天气信息实现 Function Callbacks
- 使用模拟天气服务进行演示
- 通过 Spring AI 处理 OpenAI API 集成

## 前置条件

- JDK 17 或更高版本
- Kotlin
- Spring Boot
- Spring AI 依赖项
- OpenAI API 密钥（在您的应用程序属性中配置）

## 关键组件

### 1. 主应用程序类

`KotlinFunctionCallbackApplication` 类初始化 Spring Boot 应用程序并设置与 OpenAI 的 API 交互的聊天客户端。它包括一个简单的演示，查询旧金山、东京和巴黎的天气状况。

### 2. 配置

`Config` 类设置函数回调所需的必要 bean：
- 配置天气函数信息
- 设置模拟天气服务
- 定义函数回调结构

### 3. 天气服务

`MockKotlinWeatherService` 提供一个简单的模拟实现，返回不同城市的预定义天气数据：

### 4. 数据模型

- `WeatherRequest`：表示天气查询参数
    - location（城市名称）
    - latitude
    - longitude
    - temperature unit（C/F）

- `WeatherResponse`：包含天气信息
    - current temperature
    - feels like temperature
    - minimum temperature
    - maximum temperature
    - pressure
    - humidity
    - unit（C/F）

## 用法

1. 将您的 OpenAI API 密钥添加到 `application.properties`：
```properties
spring.ai.openai.api-key=your-api-key-here
```

2. 运行应用程序：
```bash
mvn spring-boot:run
```

应用程序将执行旧金山、东京和巴黎的示例天气查询，演示函数回调与 Spring AI 的工作方式。

## 示例输出

```
响应：三个城市的当前天气状况如下：

旧金山：30°C
东京：10°C
巴黎：15°C
```

## 实现细节

1. 应用程序使用 Spring AI 的 `ChatClient` 与 OpenAI 的 API 交互
2. 函数回调通过 `FunctionCallback` 构建器注册
3. 天气查询由模拟服务处理，用于演示目的
4. 应用程序使用 Kotlin 数据类进行类型安全的请求/响应处理

## 自定义

要修改模拟天气服务：
1. 在 `MockKotlinWeatherService` 中更新温度值
2. 根据需要添加新的城市条件
3. 通过替换模拟服务实现实际的天气 API 集成
