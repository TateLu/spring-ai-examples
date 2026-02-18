# Prompt Chaining Workflow Example

此项目演示了使用 Spring AI 的大语言模型（LLM）的 Prompt Chaining 工作流模式。该模式将复杂任务分解为一系列步骤，其中每个 LLM 调用都处理前一个调用的输出。

![Prompt Chaining Workflow](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F7418719e3dab222dccb379b8879e1dc08ad34c78-2401x1000.png&w=3840&q=75)

## 概述

Prompt Chaining 模式在以下情况下特别有用：
- 复杂任务可以分解为更简单的、顺序的步骤
- 每个步骤的输出都需要验证或转换
- 过程需要维护清晰的转换链

此实现展示了一个用于处理文本中数值数据的四步工作流：
1. 提取数值和指标
2. 标准化为百分比格式
3. 按降序排序
4. 格式化为 markdown 表格

## 技术要求

- Java 17 或更高版本
- Spring Boot 4.0.0
- Spring AI 2.0.0-SNAPSHOT
- Ollama（用于 LLM 集成）

## 快速开始

1. 按照 [ollama.ai](https://ollama.ai) 的说明安装并启动 Ollama

2. 构建项目：
   ```bash
   ./mvnw clean install
   ```

3. 运行应用程序：
   ```bash
   ./mvnw spring-boot:run
   ```

## 使用示例

该示例通过一系列提示处理 Q3 性能报告。以下是示例输入：

```text
Q3 Performance Summary:
Our customer satisfaction score rose to 92 points this quarter.
Revenue grew by 45% compared to last year.
Market share is now at 23% in our primary market.
Customer churn decreased to 5% from 8%.
New user acquisition cost is $43 per user.
Product adoption rate increased to 78%.
Employee satisfaction is at 87 points.
Operating margin improved to 34%.
```

该工作流通过四个步骤处理：

1. **提取值**：提取数值和指标
   ```
   92: customer satisfaction
   45%: revenue growth
   23%: market share
   5%: customer churn
   43: user acquisition cost
   78%: product adoption
   87: employee satisfaction
   34%: operating margin
   ```

2. **标准化格式**：将值转换为百分比（如适用）
   ```
   92%: customer satisfaction
   45%: revenue growth
   23%: market share
   5%: customer churn
   78%: product adoption
   87%: employee satisfaction
   34%: operating margin
   ```

3. **排序**：按降序排列值
   ```
   92%: customer satisfaction
   87%: employee satisfaction
   78%: product adoption
   45%: revenue growth
   34%: operating margin
   23%: market share
   5%: customer churn
   ```

4. **格式化**：创建 markdown 表格
   ```markdown
   | Metric | Value |
   |:--|--:|
   | Customer Satisfaction | 92% |
   | Employee Satisfaction | 87% |
   | Product Adoption | 78% |
   | Revenue Growth | 45% |
   | Operating Margin | 34% |
   | Market Share | 23% |
   | Customer Churn | 5% |
   ```

## 实现细节

该工作流在两个主要类中实现：

1. `ChainWorkflow.java`：包含 prompt chaining 模式的核心逻辑，包括：
   - 每个转换步骤的系统提示
   - 链执行逻辑
   - 步骤之间的门验证

2. `Application.java`：提供 Spring Boot 设置和示例用法：
   - 示例输入数据
   - Spring AI 配置
   - 用于演示的命令行运行器

链中的每个步骤都充当一个门，在进入下一步之前验证和转换输出，确保过程保持正轨。

## 参考

此实现基于 Anthropic 的研究论文 [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) 中描述的 prompt chaining 模式。
