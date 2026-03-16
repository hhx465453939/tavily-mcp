## Tavily MCP Server（中文说明）

Tavily MCP Server 是基于 Tavily Web 搜索 API 的 Model Context Protocol（MCP）服务端，实现了以下能力：

- **tavily_search**：实时 Web 搜索，支持多种搜索深度、时间范围和国家权重
- **tavily_extract**：从指定 URL 提取网页内容，可输出为 Markdown 或纯文本
- **tavily_crawl**：网站爬取，按深度/广度限制抓取页面内容
- **tavily_map**：网站结构映射，返回站点 URL 拓扑
- **tavily_research**：长任务研究型检索，自动汇总多来源信息

适用于接入 Claude Desktop、Cursor 等支持 MCP 的客户端，为助手提供“联网搜索 + 内容抽取 + 爬虫 + 网站地图 + 研究型检索”等一站式能力。

---

### 有用链接

- Tavily 官网与 API Key 申请：`https://www.tavily.com`
- Tavily 文档中心：`https://docs.tavily.com`
- MCP 规范文档：`https://modelcontextprotocol.io`

---

## 一、本地源码部署

### 1. 环境要求

- **Node.js**：推荐 `v20` 及以上  
- **npm**：随 Node.js 安装  
- **Git**：用于克隆代码仓库  
- **Tavily API Key**：在 Tavily 网站注册并获取

### 2. 克隆与安装依赖

```bash
git clone https://github.com/tavily-ai/tavily-mcp.git
cd tavily-mcp
npm install
```

### 3. 构建与本地启动（作为 MCP Server）

项目入口为 `src/index.ts`，编译后入口为 `build/index.js`，通过 stdio 与 MCP 客户端通信。

```bash
# 构建 TypeScript
npm run build

# 直接以 Node 启动（推荐给本地 config 使用）
node build/index.js
```

你也可以全局 link 成可执行命令：

```bash
npm run build
npm link   # 之后可以使用 tavily-mcp 命令
```

此时：

```bash
tavily-mcp
```

即可以 MCP Server 形式通过 stdio 运行。

---

## 二、环境变量配置

### 1. 必需环境变量

- **TAVILY_API_KEY**：Tavily 的 API Key，用于鉴权

示例（Linux / macOS）：

```bash
export TAVILY_API_KEY="your-api-key-here"
```

示例（Windows PowerShell）：

```powershell
$env:TAVILY_API_KEY="your-api-key-here"
```

### 2. 默认参数 `DEFAULT_PARAMETERS`（可选）

服务端会从环境变量 `DEFAULT_PARAMETERS` 中读取一段 JSON，用作工具调用的默认参数。  
例如可以让所有搜索默认开启图片返回、提高结果数量等：

```bash
export DEFAULT_PARAMETERS='{
  "include_images": true,
  "max_results": 15,
  "search_depth": "advanced"
}'
```

在 `src/index.ts` 中，会自动解析 `DEFAULT_PARAMETERS`，并在未显式传参时使用这些默认值。

---

## 三、本地源码部署后的 MCP config 示例

下面分别给出 **Claude Desktop / Claude Code** 与 **Cursor** 在本地源码部署场景下的典型配置方式。核心思路都是：

- **command**：指向本地的 Node 可执行（或 `tavily-mcp` 命令）
- **args**：指向 `build/index.js`（或不需要 args，直接用 bin 命令）
- **env**：设置 `TAVILY_API_KEY` 和可选的 `DEFAULT_PARAMETERS`

> **注意**：路径请根据你本机实际仓库路径进行调整。

### 1. Claude Desktop / Claude Code（`mcpServers` 示例）

假设你把源码放在：

- `E:/Development/tavily-mcp`

且已执行过：

```bash
cd E:/Development/tavily-mcp
npm install
npm run build
```

那么在你的 MCP 配置（例如 `claude_desktop_config.json` 或其他 MCP 客户端统一的配置文件）中，可以这样写：

```json
{
  "mcpServers": {
    "tavily-mcp-local": {
      "command": "node",
      "args": [
        "E:/Development/tavily-mcp/build/index.js"
      ],
      "env": {
        "TAVILY_API_KEY": "your-api-key-here",
        "DEFAULT_PARAMETERS": "{\"include_images\": true, \"max_results\": 15, \"search_depth\": \"advanced\"}"
      }
    }
  }
}
```

说明：

- **command**：使用系统的 `node`
- **args**：传入本地 `build/index.js` 的绝对路径，Windows 下 `/` 与 `\` 皆可，建议与实际一致
- **env.TAVILY_API_KEY**：填入你自己的 Tavily API Key
- **env.DEFAULT_PARAMETERS**：是一个 JSON 字符串，内部再是 JSON 对象

如果你已经使用 `npm link` 把二进制命令 `tavily-mcp` 链接到全局，也可以改写为：

```json
{
  "mcpServers": {
    "tavily-mcp-local": {
      "command": "tavily-mcp",
      "args": [],
      "env": {
        "TAVILY_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

### 2. Cursor 中的本地 MCP 配置（`mcp.json` 示例）

在 Cursor 中，你可以通过编辑项目根目录的 `mcp.json` 来接入本地 Tavily MCP。

示例（源码本地运行）：

```json
{
  "mcpServers": {
    "tavily-mcp-local": {
      "command": "node",
      "args": [
        "E:/Development/tavily-mcp/build/index.js"
      ],
      "env": {
        "TAVILY_API_KEY": "your-api-key-here",
        "DEFAULT_PARAMETERS": "{\"include_images\": true, \"max_results\": 15, \"search_depth\": \"advanced\"}"
      }
    }
  }
}
```

如果你在项目目录内编辑 `mcp.json`，也可以使用相对路径（假设 `mcp.json` 与 `build` 在同一层）：

```json
{
  "mcpServers": {
    "tavily-mcp-local": {
      "command": "node",
      "args": [
        "build/index.js"
      ],
      "env": {
        "TAVILY_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

> **提示**：在 Windows 下，如果路径包含空格（例如 `C:/Users/Your Name/...`），建议：
> - 在 `args` 中使用不带引号的完整路径（MCP 客户端会负责正确处理），或
> - 把仓库放在无空格路径下（例如 `E:/Development/...`），避免转义问题。

---

## 四、与远程 Tavily MCP 的关系

本项目既可以：

- 通过 **npm 包 + npx** 方式直接使用远程 Tavily API，也可以  
- 通过 **源码本地运行** 的方式，作为你自己的本地 MCP Server，方便调试或本地开发集成。

如果你更倾向于免部署、免运维，可以继续使用官方提供的远程 MCP 地址：

```text
https://mcp.tavily.com/mcp/?tavilyApiKey=<your-api-key>
```

而当你希望：

- 调整本地代码逻辑
- 跟踪 / 调试 MCP 请求
- 与自己的其他服务组合开发

则可以使用本 README 中的“本地源码部署 + MCP config 示例”方案，将本仓库直接作为本地 MCP Server 使用。

---

## 五、工具一览

服务端会向 MCP 客户端暴露以下工具（见 `src/index.ts`）：

- **tavily_search**：通用 Web 搜索
- **tavily_extract**：网页内容抽取
- **tavily_crawl**：网站爬虫
- **tavily_map**：网站结构映射
- **tavily_research**：研究型长任务检索

所有工具均依赖 `TAVILY_API_KEY`，部分行为可通过 `DEFAULT_PARAMETERS` 进行全局默认配置。

---

## 致谢

- **Model Context Protocol**：提供统一的工具接入标准  
- **Anthropic / Claude**：提供强大的 LLM 与 MCP 客户端  
- **Tavily 团队**：提供高质量的 Web 搜索与抽取能力

