# MCP with OAuth

此示例演示如何使用 OAuth2 保护 MCP 服务器，符合 [MCP 规范](https://spec.modelcontextprotocol.io/specification/2025-03-26/basic/authorization/)。

## 快速开始

运行项目：

```
./mvnw spring-boot:run
```

通过调用 `/oauth2/token` 端点获取令牌：

```shell
curl -XPOST "http://localhost:8080/oauth2/token" \
  --data grant_type=client_credentials \
  --user "mcp-client:secret"
# 并复制粘贴访问令牌
# 或使用 JQ：
curl -XPOST "http://localhost:8080/oauth2/token" \
  --data grant_type=client_credentials \
  --user "mcp-client:secret" | jq -r ".access_token"
```

存储该令牌，然后启动 MCP 检查器：

```shell
npx @modelcontextprotocol/inspector@0.6.0
```

在 MCP 检查器中，粘贴您的令牌。点击连接，完成！

![MCP inspector](./mcp-inspector.png)

注意��令牌仅有效 5 分钟

## 实现注意事项

### 依赖项

在 Spring 中，MCP 服务器的 OAuth2 支持意味着添加：

1. [Spring Security](https://docs.spring.io/spring-security/)（安全基础设施）
2. [Spring Authorization Server](https://docs.spring.io/spring-authorization-server/)（颁发令牌）
3. [Spring Security: OAuth2 Resource Server](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html#page-title) (
   使用令牌进行身份验证）

注意，Spring Auth Server 不支持响应式栈，因此令牌颁发仅在 Servlet 中工作。
