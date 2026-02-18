# Document Forge

**使用 Spring AI 构建的 AI 驱动文档创作工作室**

*"描述它,下载它。"*

## 概述

Document Forge 是一个演示应用程序,展示了 [Claude Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) 与 [Spring AI](https://docs.spring.io/spring-ai/reference/2.0-SNAPSHOT/api/chat/anthropic-chat.html#_skills) 的集成。用自然语言描述文档,接收由 Claude 生成的真实可下载文件。

### 支持的文档类型

| 类型 | 扩展名 | 描述 |
|------|--------|------|
| Excel | `.xlsx` | 包含数据、公式和格式的电子表格 |
| PowerPoint | `.pptx` | 包含幻灯片和布局的演示文稿 |
| Word | `.docx` | 包含格式和结构的文档 |
| PDF | `.pdf` | 便携式文档格式 |

## 截图

![主表单](screenshots/01-main-form.png)
*选择文档类型并描述您想要的内容*

![生成中](screenshots/02-generating.png)
*Claude 使用 Skills 生成您的文档*

![结果](screenshots/03-result.png)
*下载生成的文档*

## 快速开始

### 前置条件

- Java 21+
- Maven 3.9+
- Anthropic API 密钥

### 运行

```bash
export ANTHROPIC_API_KEY=your-api-key-here
./mvnw spring-boot:run
```

打开 http://localhost:8080

## 功能

- **自然语言输入** - 用简单的英语描述您想要的内容
- **真实文件生成** - 获取实际可下载的文件,而不仅仅是描述
- **源文档上传** - 上传 PDF、TXT、CSV、JSON、XML 或 MD 文件供 Claude 转换
- **自定义 Skills 支持** - 通过 Anthropic 自定义技能应用自定义品牌或格式
- **历史记录侧边栏** - 查看过去的生成并重新下载文件
- **异步生成** - PowerPoint 演示文稿在后台运行,并在完成时通知

## 自定义 Skills 集成

Document Forge 演示了将预构建的 Anthropic 技能与自定义技能结合使用。
`custom-skills/watermark/SKILL.md` 中包含了一个示例自定义技能。

### 设置自定义 Skill

1. **将技能上传到 Anthropic:**

```bash
cd custom-skills/watermark

curl -X POST "https://api.anthropic.com/v1/skills" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: skills-2025-10-02" \
  -F "display_title=Document Watermark" \
  -F "files[]=@SKILL.md;filename=document-watermark/SKILL.md"
```

2. **从响应中复制技能 ID** (例如 `skill_01AbCdEfGhIjKlMnOpQrStUv`)

3. **设置环境变量:**

```bash
export CUSTOM_SKILL_ID=skill_01AbCdEfGhIjKlMnOpQrStUv
```

4. **运行应用程序** - UI 中现在将出现"应用自定义品牌"复选框

### 不使用自定义 Skills

如果未设置 `CUSTOM_SKILL_ID`,应用程序将正常工作,仅使用预构建的技能。
自定义品牌复选框将不会显示。

有关更多详细信息,请参阅 [Spring AI 自定义 Skills 文档](https://docs.spring.io/spring-ai/reference/2.0-SNAPSHOT/api/chat/anthropic-chat.html#_custom_skills)。

## 使用示例

**Excel - 销售报告**
> "为 2024 年第四季度创建销售报告,包含月度收入、支出和利润列。"

**PowerPoint - 演示文稿**
> "为一家名为 'GreenTech' 的初创公司创建一个 5 页的演示文稿,该公司生产可持续包装。"

**Word - 求职信**
> "为软件工程职位撰写一封专业的求职信。候选人拥有 5 年的 Java 和 Spring Boot 经验。"

**PDF - 发票**
> "创建一张 40 小时咨询服务的发票,每小时 $150,并附上标准付款条款。"

**转换文档**

上传会议笔记文本文件并提示:
> "创建一个操作事项跟踪电子表格,包含任务、负责人、到期日期和状态列。"

## 技术栈

| 层 | 技术 |
|-------|------------|
| 后端 | Spring Boot 4.0 |
| AI | Spring AI 2.0.0-SNAPSHOT + Anthropic Claude Skills |
| 模板 | Thymeleaf |
| 交互性 | HTMX |
| 样式 | Tailwind CSS |

## 配置

```yaml
spring:
  ai:
    anthropic:
      api-key: ${ANTHROPIC_API_KEY}
      chat:
        options:
          model: claude-sonnet-4-5-20250514
```

## 工作原理

```
用户提示 → Spring AI → Claude Skills → 文件生成 → 下载
```

1. 用户选择文档类型并输入自然语言提示
2. Spring AI 将请求发送到 Claude,并启用相应的技能
3. 可选地,添加自定义技能用于品牌或格式
4. Claude 使用代码执行生成实际的文档文件
5. `AnthropicSkillsResponseHelper` 从响应中提取文件 ID
6. 文件通过 Anthropic 的 Files API 下载并提供给用户

---

使用 Spring AI 构建。
