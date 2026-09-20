# PDF 工具箱 MCP 接入说明

[English](README.en.md)

[![MCP](https://img.shields.io/badge/MCP-Streamable%20HTTP-5B5BD6)](https://modelcontextprotocol.io/)
[![OAuth](https://img.shields.io/badge/Auth-OAuth%202.1%20%2B%20API%20Key-0F766E)](#3-oauth豆包推荐方式)
[![PDF Tools](https://img.shields.io/badge/Tools-PDF%20%2F%20Image%20Processing-EA580C)](#5-工具列表)
[![MCP Registry - com.tensormaster/pdf](https://img.shields.io/badge/MCP%20Registry-com.tensormaster%2Fpdf-111827?logo=modelcontextprotocol&logoColor=white)](https://registry.modelcontextprotocol.io/)
[![pdf-mcp MCP connector – tool definition quality and endpoint health on Glama](https://glama.ai/mcp/connectors/com.tensormaster.api/pdf-mcp/badges/score.svg)](https://glama.ai/mcp/connectors/com.tensormaster.api/pdf-mcp)
[![Smithery - PDF Toolkit](https://img.shields.io/badge/Smithery-PDF%20Toolkit-7C3AED?logo=smithery&logoColor=white)](https://smithery.ai/servers/bob-rsef/pdf-toolkit)

PDF 工具箱是一个面向 Agent 的 MCP 文件处理服务，通过 Streamable HTTP 提供 PDF/图片转 Word、Excel、Markdown、PDF 拆分合并、图片转 PDF、文字水印和异步任务查询能力。支持 OAuth 和手动 API Key 两种授权方式。

## 1. 快速选择

| 场景 | 推荐方式 |
| --- | --- |
| 豆包正式接入 | OAuth |
| MCP Client 支持浏览器授权 | OAuth |
| MCP Client 不支持 OAuth，但支持自定义 Header | 手动 API Key |
| curl、SDK、本地调试 | 手动 API Key |

## 2. 固定配置

| 项目 | 值 |
| --- | --- |
| MCP 服务地址 | `https://api.tensormaster.com/pdf/mcp` |
| 传输方式 | Streamable HTTP |
| Resource | `https://api.tensormaster.com/pdf/mcp` |
| OAuth 注册方式 | DCR（Dynamic Client Registration） |
| Scope | `mcp:tools offline_access`，使用 ASCII 空格分隔 |
| 客户端认证 | `none`，Public Client + PKCE |
| Response Type | `code` |
| Grant Type | `authorization_code`、`refresh_token` |
| PKCE | 必须使用 `S256` |
| Redirect URI | 注册时传入，后续精确匹配，不支持通配符 |
| OAuth Discovery | `https://miaodashi.com/.well-known/oauth-authorization-server` |
| Authorization Endpoint | `https://miaodashi.com/oauth/authorize` |
| Token Endpoint | `https://miaodashi.com/api/oauth/token` |
| Refresh Token Endpoint | 留空，复用 Token Endpoint |
| DCR 注册地址 | `https://miaodashi.com/api/oauth/register` |

## 3. OAuth：豆包推荐方式

在豆包中按以下步骤操作：

1. 在远程 MCP/Connector 中填写：

   ```text
   https://api.tensormaster.com/pdf/mcp
   ```

2. 点击“去授权”
3. 浏览器打开 MiaoDashi 授权页面；首次使用先注册，已有账号直接登录。
4. 核对客户端名称、PDF 工具箱和权限范围，点击“同意授权”。
5. 页面返回豆包后，确认状态显示“授权成功”或“已连接”。

DCR、`state`、PKCE、授权码兑换、Access Token 和 Refresh Token 均由豆包与 MiaoDashi 自动完成，用户无需手动复制授权码或 Token。Access Token 过期后，豆包使用 Refresh Token 自动续期。

### Claude 接入步骤

将以下指令发送给 Claude：

```text
帮我在设置里添加一个新的 MCP 连接器，地址是 https://api.tensormaster.com/pdf/mcp。添加后提醒我确认授权页面上的权限范围，再点同意。
```

然后按以下步骤完成授权：

1. 退出 Claude，再重新进入 Claude。
2. 输入 `/mcp`，选择 PDF 服务并进行授权。
3. 确认授权成功后，即可使用 PDF 工具箱。

### Codex（ChatGPT）接入步骤

1. 打开 **设置 → 插件 → 添加 → 添加 MCP 服务器**。
2. 填写：

   ```text
   名称：pdf
   类型：流式 HTTP
   URL：https://api.tensormaster.com/pdf/mcp
   ```

3. 点击“保存”。
4. 点击“进行身份验证”，注册或登录 MiaoDashi 网站，并在授权页面确认权限后点击“同意”。
5. 返回设置页，确认“进行身份验证”按钮消失，即表示授权成功，可以使用 PDF 工具箱。

> ChatGPT 的菜单名称可能因账号、工作区权限或版本略有差异；如果看不到添加 MCP 服务器入口，请先确认已启用相应的开发者模式或自定义连接器权限。

## 4. 手动 API Key：兼容方式

### 4.1 获取 API Key

1. 打开 [MiaoDashi API Keys](https://miaodashi.com/account/api-keys)，登录需要承担 PDF 费用的账号。
2. 点击“创建 API Key”，填写名称，例如 `pdf-mcp-local`。
3. 选择 PDF 工具需要的最小权限，至少需要文件处理/生成权限；按需设置过期时间。
4. 创建成功后立即复制完整 Key。完整 Key 通常只显示一次，请保存到密码管理器或 MCP Client 安全存储。

不要使用网页内部 Key、OAuth Token、Device Flow Key 或其他服务的 Key。

### 4.2 配置 MCP Client

MCP Gateway 已启用手动 Key 兼容入口时，使用：

```json
{
  "mcpServers": {
    "pdf-toolkit": {
      "type": "streamableHttp",
      "url": "https://api.tensormaster.com/pdf/mcp",
      "headers": {
        "Authorization": "Bearer md_api_<your_key>"
      }
    }
  }
}
```

也可以直接发送：

```text
Authorization: Bearer md_api_<your_key>
```

不要把 API Key 放到 URL、Query、日志或聊天内容中。先调用 `get_pdf_toolbox_capabilities` 验证连接，再提交转换任务。

## 5. 工具列表

| 工具 | 主要参数 | 说明 |
| --- | --- | --- |
| `convert_file_to_word` | `file_url`, `idempotency_key` | PDF/图片转 DOCX |
| `convert_file_to_excel` | `file_url`, `idempotency_key` | PDF/图片转 XLSX |
| `convert_file_to_markdown` | `file_url`, `idempotency_key` | PDF/图片转 Markdown |
| `split_pdf` | `file_url`, `pages_per_file`, `idempotency_key` | 拆分 PDF，结果通常为 ZIP |
| `merge_pdfs` | `file_urls`, `size`, `idempotency_key` | 按数组顺序合并 PDF |
| `images_to_pdf` | `image_urls`, `scale`, `idempotency_key` | 图片生成 PDF |
| `add_pdf_watermark` | `file_url`, 水印参数, `idempotency_key` | 添加文字水印 |
| `get_pdf_task` | `task_id` | 查询状态、结果 URL、错误和积分 |
| `list_pdf_tasks` | `cursor`, `limit` | 查询当前用户的任务列表 |
| `get_pdf_toolbox_capabilities` | 无 | 查询格式、数量、大小和服务限制；无副作用 |

### 参数和任务规则

- MCP 请求使用 JSON-RPC，不接收 `multipart/form-data`。
- 用户附件必须映射为临时 HTTPS URL：单文件使用 `file_url`，多文件使用 `file_urls` 或 `image_urls`。
- `split_pdf.pages_per_file` 为 `1..50`；`merge_pdfs` 支持 2～10 个文件；`images_to_pdf` 支持 1～10 张图片；`scale` 为 `(0, 1]`。
- 创建任务必须使用 `idempotency_key`；相同参数重试不会重复创建或扣费。
- 任务状态：`queued` → `running` → `succeeded`/`failed`。使用 `get_pdf_task` 轮询，不要重复创建任务。
- 任务只能由所属用户查询。

## 6. 常见错误

| 错误 | 处理方式 |
| --- | --- |
| `401` / `AUTH_REQUIRED` | OAuth 重新授权；手动模式检查 Key、过期和撤销状态 |
| `403` / `INSUFFICIENT_SCOPE` | 补充正确权限或重新授权 |
| `INVALID_INPUT` | 按 `tools/list` 返回的 schema 修正参数 |
| `UNSUPPORTED_FILE_TYPE` / `FILE_TOO_LARGE` | 查询 capabilities，转换格式或压缩文件 |
| `SOURCE_URL_FORBIDDEN` | 使用可访问的 HTTPS 临时 URL，不携带凭据 |
| `INSUFFICIENT_CREDITS` | 充值或切换有额度的账号 |
| `CONCURRENCY_LIMITED` | 等待已有任务完成 |
| `TASK_NOT_FOUND` | 检查 task ID；不能访问其他用户任务 |
| `UPSTREAM_UNAVAILABLE` | 稍后使用相同 `idempotency_key` 重试 |

## 7. 安全要求

- 全部 MCP、OAuth、Token 和发现地址使用 HTTPS。
- Access Token、Refresh Token 和 API Key 只能放在 `Authorization` Header。
- 不要记录完整凭据、内部 JWT、Supabase 密钥或签名下载 URL。
- 文件 URL 必须经过 HTTPS、大小、重定向、DNS/IP 和超时校验，防止 SSRF。
- 怀疑 API Key 泄露时，立即在 MiaoDashi API Keys 页面撤销并重新创建。
