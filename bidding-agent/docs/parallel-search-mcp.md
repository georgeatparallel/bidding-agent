# 用 Parallel Search MCP 核验最新采购规则

维护招投标知识库时，可以先用 Claude Code 搜索最新的政府采购规定，再抓取官方原文核对适用地区、发布日期和具体条款。[Parallel Search MCP](https://docs.parallel.ai/integrations/mcp/search-mcp) 提供 `web_search` 和 `web_fetch`，无需 Parallel 账号或 API key；匿名搜索默认使用 Fast 模式，适合轻量调研，有使用频率限制。

这是 **Claude Code 的可选开发工具**。应用内 Chat 页仍使用原有 Tavily/Exa 联网配置；这里的配置不会给后端增加搜索工具，也不需要修改 `bidding-agent/.env`。

## 1. 添加本地配置

先安装并登录 [Claude Code](https://code.claude.com/docs/en/setup)。在仓库根目录（`AGENTS.md` 与 `frontend/` 所在目录）执行：

```bash
claude mcp add --transport http --scope local parallel-search https://search.parallel.ai/mcp
claude mcp get parallel-search
claude mcp list
```

`--scope local` 只为当前用户的当前项目保存配置，不生成要提交的 `.mcp.json`，也不改变其他项目的设置。若已使用同名 MCP 服务，先用 `claude mcp get parallel-search` 检查，选择其他名称以保留原有配置。

在该目录重新启动 Claude Code，输入 `/mcp` 检查 `parallel-search` 的连接状态。连接成功后可使用 `web_search` 和 `web_fetch`；首次调用时按 Claude Code 提示授权所需工具。

## 2. 先搜索，再抓取原文

在 Claude Code 中输入，例如：

```text
请使用 parallel-search 的 web_search 调研：中国政府采购中，哪些现行规定约束
供应商质疑与投诉的期限？优先检索财政部和中国政府采购网，区分全国性规定与
地方规则。给出候选页面的标题、URL 和发布日期，不要仅根据搜索摘要下结论。
然后用同一服务的 web_fetch 读取候选官方页面，核对相关条款、适用范围和
生效日期。输出「规则 / 适用地区 / 期限 / 条款原文 / 来源 URL / 发布或生效日期」
表格；无法从原文确认的内容明确标为未确认。
```

确认工具调用记录中使用的是该 MCP 服务。`web_search` 返回候选来源，`web_fetch` 读取指定 URL 的正文；页面抓取失败时，保留来源链接并手动核验，不把摘要当作已核验原文。

核验后再整理成知识库的 `问` / `答` Excel，保留来源和日期以便后续复核。导入流程见[开发文档 §4.1](开发文档.md#41-知识库qdrant)；`python main.py ingest` 会全量重建向量库，确认数据后再执行。

## 3. 排障与移除

- `claude: command not found`：检查 Claude Code 安装及终端 PATH，重新打开终端后再试。
- 服务未连接或工具未出现：确认当前目录与添加配置时相同，用 `claude mcp list` 和会话中的 `/mcp` 检查连接；确认可访问 `https://search.parallel.ai/mcp`。
- 达到匿名使用限额：等待服务提示的重试时间，或按 [Parallel 配置说明](https://docs.parallel.ai/integrations/mcp/search-mcp)配置账号认证以获得更高限额；不要把密钥提交到仓库。

不再需要时，在同一仓库目录执行：

```bash
claude mcp remove parallel-search --scope local
```

作用域及 HTTP 传输的说明见 [Claude Code MCP 文档](https://code.claude.com/docs/en/mcp)。
