# Spring AI Java 函数回调演示与 OpenAI

此 Spring Boot 应用程序演示了使用 OpenAI API 和 Spring AI 实现函数回调。该应用模拟了使用模拟天气服务检索不同城市天气信息的功能。

## 概述

该���用程序展示如何:
- 将 OpenAI 的函数调用能力与 Spring Boot 集成
- 使用模拟天气服务处理天气相关查询
- 处理并响应天气信息的自然语言请求

## 组件

### 主应用程序 (SpringAiJavaFunctionCallbackApplication)

主应用程序类包含:
- Spring Boot 配置
- 用于演示执行的命令行运行器
- 模拟天气服务实现
- 温度单位枚举 (摄氏度/华氏度)

### 天气请求 (WeatherRequest)

天气请求数据模型包含以下字段:
- location: 城市和州 (例如 "San Francisco, CA")
- lat: 纬度坐标
- lon: 经度坐标
- unit: 温度单位 (C/F)

### 天气响应 (WeatherResponse)

天气信息数据模型包含:
- temp: 当前温度
- feels_like: "体感" 温度
- temp_min: 最低温度
- temp_max: 最高温度
- pressure: 大气压
- humidity: 湿度百分比
- unit: 温度单位 (C/F)

## 模拟天气服务

该模拟服务为三个城市提供模拟天气数据:
- 旧金山 (30.0°C)
- 东京 (10.0°C)
- 巴黎 (15.0°C)

## 使用示例

应用程序通过命令行运行器演示使用:

```java
ChatResponse response = chatClient
.prompt("旧金山、东京和巴黎的天气状况如何?")
.functions("WeatherInfo")
.call()
.chatResponse();
```

## 依赖项

该项目使用 Maven 进行依赖管理。所有必需的依赖项都包含在 pom.xml 文件中:
- Spring Boot Starter (spring-boot-starter)
- Spring AI OpenAI Starter (spring-ai-openai-spring-boot-starter)
- Spring Boot Starter Test (spring-boot-starter-test)
- 用于 JSON 处理的 Jackson 库
- 其他 Spring 框架依赖

有关确切的版本和依赖项的完整列表,请参考项目根目录中的 pom.xml 文件。

## 设置

克隆仓库

在应用程序配置中设置您的 OpenAI API 密钥:

```properties
openai.api.key=your_api_key_here
```

使用以下命令运行应用程序:

```bash
./mvnw spring-boot:run
```

## 注意事项

- 这是一个使用模拟数据的演示应用程序
- 实际实现需要与实际的天气 API 集成
- 温度值是硬编码的,仅用于演示目的
