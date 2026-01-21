# Obsidian MCP Server 配置指南

> [!NOTE]
> MCP（Model Context Protocol）是一种标准协议，可以让 AI 系统连接外部工具和数据源。通过配置 Obsidian MCP Server，可以让 Antigravity 直接读写 Obsidian vault 中的笔记。

---

## 1. 在 Obsidian 启用 Local REST API

### 1.1 安装插件

1. 打开 Obsidian → 设置 → 第三方插件
2. 关闭"安全模式"
3. 点击"浏览社区插件"
4. 搜索 **Local REST API**
5. 安装并启用

### 1.2 获取 API Key

1. 进入插件设置（设置 → 第三方插件 → Local REST API）
2. 复制显示的 **API Key**
3. 确认 API 地址为 `http://127.0.0.1:27123`

> [!IMPORTANT]
> API Key 是敏感信息，请妥善保管，不要提交到公开仓库。

---

## 2. 在 PowerShell 测试 MCP Server（可选）

如果想手动测试 MCP Server 是否正常工作，可以在 PowerShell 中运行：

```powershell
npx -y @huangyihe/obsidian-mcp
```

但通常不需要手动运行，因为 Antigravity 会根据配置自动启动。

---

## 3. 在 Antigravity 配置 Custom MCP Server

### 3.1 配置文件位置

```
C:\Users\<用户名>\.gemini\antigravity\mcp_config.json
```

或者，可以执行以下步骤打开配置文件：

1. 找到 Antigravity IDE 的侧边面板右上角的 **...**，点击它。
2. 点击 **Manage MCP Servers**。
3. 点击 **View raw config**，即可编辑配置文件。

### 3.2 配置内容

```json
{
    "mcpServers": {
        "obsidian": {
            "command": "npx",
            "args": [
                "-y",
                "@huangyihe/obsidian-mcp"
            ],
            "env": {
                "OBSIDIAN_API_KEY": "<你的 API Key>",
                "OBSIDIAN_API_URL": "http://127.0.0.1:27123",
                "OBSIDIAN_VAULT_PATH": "<你的 Vault 绝对路径>"
            }
        }
    }
}
```

### 3.3 关键配置项说明

| 配置项                   | 说明                       | 示例                               |
| --------------------- | ------------------------ | -------------------------------- |
| `OBSIDIAN_API_KEY`    | 从 Local REST API 插件获取的密钥 | `a1...`                          |
| `OBSIDIAN_API_URL`    | Obsidian REST API 地址     | `http://127.0.0.1:27123`         |
| `OBSIDIAN_VAULT_PATH` | Vault 的绝对路径（使用双反斜杠）      | `C:\\projects\\my-tech-notebook` |

> [!CAUTION]
> Windows 路径需要使用双反斜杠 `\\` 或正斜杠 `/`。

---

## 4. 验证配置

配置完成后，在 Antigravity 中可以使用以下功能：

- `mcp_obsidian_list_notes` - 列出所有笔记
- `mcp_obsidian_read_note` - 读取笔记内容
- `mcp_obsidian_create_note` - 创建新笔记
- `mcp_obsidian_update_note` - 更新笔记
- `mcp_obsidian_search_vault` - 搜索 vault

---

## 5. 常见问题

### Q: 重启电脑后需要重新运行 npx 命令吗？

**A:** 不需要。Antigravity 会根据 `mcp_config.json` 自动启动 MCP Server。但需要确保：
1. Obsidian 应用程序正在运行
2. Local REST API 插件已启用

### Q: 连接失败怎么办？

**A:** 检查以下几点：
1. Obsidian 是否正在运行
2. Local REST API 插件是否启用
3. API Key 是否正确
4. Vault 路径是否正确

---

*创建日期: 2026-01-21*
