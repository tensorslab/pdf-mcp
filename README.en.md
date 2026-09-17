# PDF Toolkit MCP Integration Guide

[中文](README.md)

[![MCP](https://img.shields.io/badge/MCP-Streamable%20HTTP-5B5BD6)](https://modelcontextprotocol.io/)
[![OAuth](https://img.shields.io/badge/Auth-OAuth%202.1%20%2B%20API%20Key-0F766E)](#3-oauth-recommended-for-doubao)
[![PDF Tools](https://img.shields.io/badge/Tools-PDF%20%2F%20Image%20Processing-EA580C)](#5-tool-list)

PDF Toolkit is an Agent-oriented MCP file-processing service. Over Streamable HTTP, it converts PDFs/images to Word, Excel, and Markdown, splits and merges PDFs, creates PDFs from images, adds text watermarks, and provides asynchronous task lookup. It supports OAuth and manual API Key authorization.

## 1. Choose an authorization method

| Scenario | Recommended method |
| --- | --- |
| Production Doubao connection | OAuth |
| MCP Client supports browser authorization | OAuth |
| MCP Client has no OAuth but supports custom headers | Manual API Key |
| curl, SDK, or local debugging | Manual API Key |

## 2. Fixed configuration

| Item | Value |
| --- | --- |
| MCP service URL | `https://api.tensormaster.com/pdf/mcp` |
| Transport | Streamable HTTP |
| Resource | `https://api.tensormaster.com/pdf/mcp` |
| OAuth registration | DCR (Dynamic Client Registration) |
| Scope | `mcp:tools offline_access`, separated by an ASCII space |
| Client authentication | `none`, Public Client + PKCE |
| Response type | `code` |
| Grant types | `authorization_code`, `refresh_token` |
| PKCE | Required: `S256` |
| Redirect URI | Supplied at registration and matched exactly; wildcards are not supported |
| OAuth Discovery | `https://miaodashi.com/.well-known/oauth-authorization-server` |
| Authorization Endpoint | `https://miaodashi.com/oauth/authorize` |
| Token Endpoint | `https://miaodashi.com/api/oauth/token` |
| Refresh Token Endpoint | Leave blank; reuse the Token Endpoint |
| DCR registration | `https://miaodashi.com/api/oauth/register` |

## 3. OAuth: recommended for Doubao

In Doubao:

1. Enter this remote MCP/Connector URL:

   ```text
   https://api.tensormaster.com/pdf/mcp
   ```

2. Click “Authorize”.
3. The browser opens the MiaoDashi authorization page. Register if needed; existing users can sign in.
4. Review the client name, PDF Toolkit service, and scopes, then click “Agree” or “Authorize”.
5. After the page returns to Doubao, confirm that the status is “Authorization successful” or “Connected”.

DCR, `state`, PKCE, authorization-code exchange, Access Token, and Refresh Token handling are automatic between Doubao and MiaoDashi. The user does not need to copy an authorization code or Token. Doubao renews an expired Access Token with the Refresh Token.

### Claude setup

Send the following instruction to Claude:

```text
Please add a new MCP connector in Settings at https://api.tensormaster.com/pdf/mcp. After adding it, remind me to review the permission scopes on the authorization page before clicking Agree.
```

Then complete authorization:

1. Exit Claude and open it again.
2. Enter `/mcp`, select the PDF service, and authorize it.
3. Confirm that authorization succeeded, then use PDF Toolkit.

### Codex (ChatGPT) setup

1. Open **Settings → Plugins → Add → Add MCP server**.
2. Enter:

   ```text
   Name: pdf
   Type: Streamable HTTP
   URL: https://api.tensormaster.com/pdf/mcp
   ```

3. Click **Save**.
4. Click **Authenticate**, register or sign in to MiaoDashi, review the permission scopes, and click **Agree**.
5. Return to Settings. When the **Authenticate** button disappears, authorization has succeeded and PDF Toolkit is ready to use.

> ChatGPT menu names may vary slightly by account, workspace permissions, or version. If the Add MCP server entry is unavailable, confirm that developer mode or custom-connector access is enabled.

## 4. Manual API Key: compatibility method

### 4.1 Create a Key

1. Open [MiaoDashi API Keys](https://miaodashi.com/account/api-keys) and sign in to the account that should pay for PDF processing.
2. Click **Create API Key** and use a recognizable name, such as `pdf-mcp-local`.
3. Select the least privilege required by PDF processing, including file-processing/generation permission; set an expiration date when appropriate.
4. Copy the complete Key immediately. It is normally shown only once; store it in a password manager or the MCP Client's secure storage.

Do not use a web-internal Key, OAuth Token, Device Flow Key, or a Key from another service.

### 4.2 Configure the MCP Client

When the MCP Gateway's manual-Key compatibility path is enabled, use:

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

Equivalent header:

```text
Authorization: Bearer md_api_<your_key>
```

Never put the API Key in a URL, query string, log, or chat message. Call `get_pdf_toolbox_capabilities` first to verify the connection.

## 5. Tool list

| Tool | Main parameters | Purpose |
| --- | --- | --- |
| `convert_file_to_word` | `file_url`, `idempotency_key` | PDF/image to DOCX |
| `convert_file_to_excel` | `file_url`, `idempotency_key` | PDF/image to XLSX |
| `convert_file_to_markdown` | `file_url`, `idempotency_key` | PDF/image to Markdown |
| `split_pdf` | `file_url`, `pages_per_file`, `idempotency_key` | Split a PDF, usually returning a ZIP |
| `merge_pdfs` | `file_urls`, `size`, `idempotency_key` | Merge PDFs in array order |
| `images_to_pdf` | `image_urls`, `scale`, `idempotency_key` | Create a PDF from images |
| `add_pdf_watermark` | `file_url`, watermark parameters, `idempotency_key` | Add a text watermark |
| `get_pdf_task` | `task_id` | Query status, result URL, errors, and credits |
| `list_pdf_tasks` | `cursor`, `limit` | List the current user's tasks |
| `get_pdf_toolbox_capabilities` | None | Query formats, counts, sizes, and limits; no side effect |

### Parameters and task rules

- MCP requests use JSON-RPC; `multipart/form-data` is not accepted.
- User attachments must be mapped to temporary HTTPS URLs: `file_url` for one file, `file_urls` or `image_urls` for multiple files.
- `split_pdf.pages_per_file` is `1..50`; `merge_pdfs` accepts 2–10 files; `images_to_pdf` accepts 1–10 images; `scale` is `(0, 1]`.
- Every task-creation tool requires `idempotency_key`; retries with the same parameters do not create or charge a duplicate task.
- Task states are `queued` → `running` → `succeeded`/`failed`. Poll with `get_pdf_task`; do not create repeated tasks.
- A task can be queried only by its owning user.

## 6. Common errors

| Error | Action |
| --- | --- |
| `401` / `AUTH_REQUIRED` | Re-authorize OAuth; for manual mode check the Key, expiry, and revocation |
| `403` / `INSUFFICIENT_SCOPE` | Add the required permission or authorize again |
| `INVALID_INPUT` | Correct arguments using the schema from `tools/list` |
| `UNSUPPORTED_FILE_TYPE` / `FILE_TOO_LARGE` | Check capabilities, convert the format, or compress the file |
| `SOURCE_URL_FORBIDDEN` | Use an accessible temporary HTTPS URL without credentials |
| `INSUFFICIENT_CREDITS` | Recharge or use an account with available credits |
| `CONCURRENCY_LIMITED` | Wait for existing tasks to finish |
| `TASK_NOT_FOUND` | Check the task ID; cross-user access is not allowed |
| `UPSTREAM_UNAVAILABLE` | Retry later with the same `idempotency_key` |

## 7. Security

- Use HTTPS for all MCP, OAuth, Token, and discovery endpoints.
- Send Access Tokens, Refresh Tokens, and API Keys only in the `Authorization` header.
- Never log complete credentials, internal JWTs, Supabase secrets, or signed download URLs.
- Validate HTTPS source URLs, size, redirects, DNS/IP, and timeouts to prevent SSRF.
- If an API Key may have leaked, revoke it immediately in MiaoDashi and create a replacement.
