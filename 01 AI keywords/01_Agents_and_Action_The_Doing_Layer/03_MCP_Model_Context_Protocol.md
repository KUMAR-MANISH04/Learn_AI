# 🔌 MCP — Model Context Protocol

> **A rapidly trending open standard that allows AI models to securely connect to external data sources and tools (like local files, databases, or specific software APIs) without developers needing to build custom integrations for every single app.**

---

## Phase 1: Core Foundations & Pre-requisites

### Prerequisites
- **Client-Server Architecture** — How clients request services from servers
- **REST APIs & JSON-RPC** — Communication protocols for web services
- **AI Agents & Tool Use** — How LLMs call external functions (see [01](01_Agents_Autonomous_Agents.md) & [04](04_Function_Calling_Tool_Use.md))

### Definition
**MCP (Model Context Protocol)** is an open standard created by **Anthropic** (November 2024) that provides a universal, standardized way for AI applications to connect to external data sources and tools.

Think of it as the **"USB-C for AI"** — just like USB-C provides one universal port for charging, data, and display, MCP provides one universal protocol for connecting any AI model to any tool or data source.

### The Problem It Solves

**Before MCP (the M×N Problem):**
```
Every AI app × Every tool = Custom integration needed

3 AI Apps × 5 Tools = 15 custom integrations 😱
10 AI Apps × 20 Tools = 200 custom integrations 🤯
```

| Before MCP | With MCP |
|-----------|----------|
| Each AI app builds custom connectors for each tool | Each tool builds ONE MCP server; every AI app connects via MCP client |
| Tight coupling between AI app and tool | Loose coupling; swap tools or AI apps freely |
| Duplicated authentication, error handling per integration | Standardized auth, error handling, capability negotiation |
| N apps × M tools = N×M integrations | N apps + M tools = N+M implementations |

### The Solution
MCP defines a **standard protocol** with three roles:
1. **MCP Host** — The AI application (Claude Desktop, VS Code, IDE)
2. **MCP Client** — Lives inside the Host; manages connections to servers
3. **MCP Server** — Exposes tools, resources, and prompts from an external system

```
┌────────────┐     MCP Protocol     ┌─────────────────┐
│  AI App    │◄────────────────────►│  MCP Server:    │
│  (Host +   │   (JSON-RPC 2.0)    │  GitHub          │
│   Client)  │                      │  - list_repos()  │
│            │◄────────────────────►│  - create_pr()   │
│            │                      └─────────────────┘
│            │◄────────────────────►┌─────────────────┐
│            │                      │  MCP Server:    │
│            │                      │  PostgreSQL      │
│            │                      │  - query()       │
└────────────┘                      │  - list_tables() │
                                    └─────────────────┘
```

### Real-World Example
**Scenario:** You're using Claude Desktop and want to query your company's PostgreSQL database.

**Without MCP:** You'd copy-paste table schemas into Claude, run queries manually in a SQL client, paste results back. Tedious.

**With MCP:**
1. Install the PostgreSQL MCP server (`npx @modelcontextprotocol/server-postgres`)
2. Configure it with your DB connection string in Claude Desktop's config
3. Now Claude can directly: list tables, read schemas, run SELECT queries — all within the chat

### Trade-off Table

| Dimension | Custom Integrations | MCP | Traditional APIs |
|-----------|-------------------|-----|-----------------|
| **Standardization** | ❌ Every app is different | ✅ Universal protocol | ⚠️ Per-API docs |
| **Setup effort (tool dev)** | 🔴 Build for each AI app | 🟢 Build once | 🟢 Build once |
| **Setup effort (AI app)** | 🔴 Build for each tool | 🟢 Standard client | 🔴 Per-tool connector |
| **Discoverability** | ❌ Manual | ✅ Auto-discovery | ❌ Manual |
| **Security** | ⚠️ Ad-hoc | ✅ Standardized scoping | ⚠️ Per-API |
| **Maturity** | ✅ Proven | 🟡 Young (2024) | ✅ Proven |

### 🧩 Mini-Quiz

> **Q1:** What is the "M×N Problem" that MCP solves?
> <details><summary>Answer</summary>Without MCP, M AI applications × N tools = M×N custom integrations. MCP reduces this to M + N implementations by standardizing the protocol.</details>

> **Q2:** What are the three roles in MCP architecture?
> <details><summary>Answer</summary>Host (AI application), Client (manages connections, lives in the Host), Server (exposes tools/resources from external systems).</details>

---

## Phase 2: Anatomy & Internal Mechanisms

### MCP Server Capabilities

An MCP server can expose three types of capabilities:

| Capability | Description | Example | Who Invokes |
|-----------|-------------|---------|-------------|
| **Tools** | Functions the AI can call | `run_query(sql)`, `create_issue(title, body)` | The AI model decides |
| **Resources** | Data the AI can read | File contents, DB schemas, API docs | The AI application |
| **Prompts** | Pre-built prompt templates | "Summarize this PR", "Explain this error" | The user selects |

### Protocol Architecture

![MCP Protocol Architecture](images/03_protocol_architecture.png)

### Transport Mechanisms

| Transport | How It Works | Best For |
|-----------|-------------|----------|
| **stdio** | Server runs as a local subprocess; communicates via stdin/stdout | Local tools (files, DB on localhost) |
| **HTTP + SSE** | Server runs remotely; uses HTTP POST for requests, Server-Sent Events for responses | Remote/cloud services, shared servers |

### MCP Message Lifecycle

![MCP Message Lifecycle](images/03_message_lifecycle.png)

### 🃏 Flashcard

> **Front:** How does MCP differ from direct API integration?
> <details><summary>Flip</summary>MCP is a <b>meta-protocol</b> — a standardized way to expose ANY API to ANY AI app. Instead of each AI app learning each API's unique auth, endpoints, and response format, MCP wraps all that behind a universal interface. The MCP server handles the API-specific logic; the AI app only speaks MCP.</details>

---

## Phase 3: Advanced / Enterprise Patterns & Pitfalls

### At Scale — Who's Adopting MCP (2025)

| Platform | MCP Support |
|----------|-------------|
| **Anthropic Claude** | Native — Claude Desktop, Claude Code, API |
| **OpenAI** | Adopted March 2025 — ChatGPT, Agents SDK |
| **Google** | Supported in ADK (Agent Development Kit) |
| **Microsoft** | Copilot Studio, VS Code integration |
| **Cursor** | Built-in MCP client |
| **Zed, Windsurf** | MCP client support |

### MCP vs. Google A2A (Agent-to-Agent)

| Feature | MCP | A2A |
|---------|-----|-----|
| **Focus** | Agent ↔ Tool/Data connection | Agent ↔ Agent communication |
| **Created by** | Anthropic | Google |
| **Use case** | "Give my agent access to GitHub" | "Let my agent talk to your agent" |
| **Relationship** | Complementary — not competing | Complementary — not competing |

### Security Considerations

| Risk | Mitigation |
|------|------------|
| **Data exfiltration** | MCP server should enforce read-only where possible; scope DB access |
| **Prompt injection via resources** | Sanitize resource content before injecting into LLM context |
| **Unauthorized tool execution** | MCP supports capability negotiation; hosts should require user consent |
| **Credential management** | Store API keys in env vars or secret managers, never in MCP config files |
| **Over-permissioned servers** | Apply principle of least privilege — expose only needed tools |

### Anti-Patterns

- ❌ **Exposing destructive tools without gates** — `DROP TABLE` via MCP → Always require confirmation
- ❌ **One mega-server** — 50 tools in one server → Split into focused servers (Git, DB, Files)
- ❌ **Ignoring error responses** — Tool fails silently → Return clear error messages the LLM can understand
- ❌ **No rate limiting** — LLM calls tool 100 times in a loop → Add rate limits and circuit breakers

---

## Phase 4: Practical Implementation

### Building an MCP Server (Python)

```python
# pip install mcp
from mcp.server import Server, stdio_transport
from mcp.types import Tool, TextContent

# 1. Create the server
server = Server("my-notes-server")

# 2. Define tools the server exposes
@server.tool()
async def search_notes(query: str) -> list[TextContent]:
    """Search through user's notes for relevant content.
    
    Args:
        query: The search term to look for in notes
    """
    # In production: search a vector DB or file system
    notes = [
        {"title": "Meeting Notes", "content": "Discussed Q4 roadmap..."},
        {"title": "Design Doc", "content": "New auth system architecture..."},
    ]
    results = [n for n in notes if query.lower() in n["content"].lower()]
    return [TextContent(type="text", text=str(results))]

@server.tool()
async def create_note(title: str, content: str) -> list[TextContent]:
    """Create a new note with the given title and content.
    
    Args:
        title: Title of the note
        content: Body content of the note
    """
    # In production: write to file system or database
    return [TextContent(type="text", text=f"Created note: {title}")]

# 3. Define resources (data the AI can read)
@server.resource("notes://list")
async def list_notes() -> str:
    """Returns a list of all available notes."""
    return "1. Meeting Notes\n2. Design Doc\n3. TODO List"

# 4. Run the server
if __name__ == "__main__":
    import asyncio
    asyncio.run(stdio_transport(server))
```

### Configuring in Claude Desktop

```json
// claude_desktop_config.json
{
  "mcpServers": {
    "my-notes": {
      "command": "python",
      "args": ["path/to/notes_server.py"],
      "env": {
        "NOTES_DIR": "/path/to/notes"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxxxxxxxxxxx"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost/mydb"
      }
    }
  }
}
```

### Popular Pre-Built MCP Servers

| Server | What It Connects To | Install |
|--------|-------------------|---------|
| **filesystem** | Local files & directories | `@modelcontextprotocol/server-filesystem` |
| **github** | GitHub repos, PRs, issues | `@modelcontextprotocol/server-github` |
| **postgres** | PostgreSQL databases | `@modelcontextprotocol/server-postgres` |
| **slack** | Slack messages & channels | `@modelcontextprotocol/server-slack` |
| **brave-search** | Web search via Brave | `@modelcontextprotocol/server-brave-search` |
| **memory** | Persistent memory store | `@modelcontextprotocol/server-memory` |

---

## Phase 5: Interview Preparation

### Q1: "Explain MCP and why the industry is converging on it."
<details><summary><b>Answer</b></summary>

MCP is an open protocol (by Anthropic, adopted by OpenAI, Google, Microsoft) that standardizes how AI models connect to tools and data. It solves the M×N integration problem: instead of every AI app building custom connectors for every tool, each tool exposes one MCP server and every AI app uses a standard MCP client. This is the same pattern that made USB-C universal — one plug for everything. The industry converges because it reduces integration cost, enables tool portability, and creates an ecosystem where tools are interchangeable.
</details>

### Q2: "MCP vs. Function Calling — what's the relationship?"
<details><summary><b>Answer</b></summary>

**Function Calling** is the LLM's ability to output structured tool calls (JSON). It's the *mechanism* by which the model says "I want to call this function."

**MCP** is the *protocol* that connects that function call to an actual external system. MCP handles: tool discovery, capability negotiation, transport, authentication, and execution.

They're **complementary**: Function Calling is *how the model asks*; MCP is *how the system delivers*.
</details>

### Q3: "How would you secure an MCP deployment in an enterprise?"
<details><summary><b>Answer</b></summary>

1. **Principle of Least Privilege** — Each MCP server exposes only the minimum tools needed
2. **User Consent Gates** — Dangerous actions (write, delete) require explicit user approval
3. **Network Isolation** — MCP servers for sensitive data run on internal networks only
4. **Credential Management** — API keys in secret managers (Vault, AWS Secrets), never in config
5. **Audit Logging** — Log every tool call with user, timestamp, parameters, and result
6. **Input Sanitization** — Prevent prompt injection through resource content
7. **Rate Limiting** — Prevent runaway agents from overwhelming backend systems
</details>

---

## Phase 6: Summary Cheatsheet & Action Plan

### 📋 TL;DR

| Concept | Key Point |
|---------|-----------|
| **MCP** | "USB-C for AI" — universal protocol for AI ↔ tools |
| **Architecture** | Host (AI app) → Client → Server (exposes tools/resources) |
| **Transport** | stdio (local) or HTTP+SSE (remote) |
| **Capabilities** | Tools (AI calls), Resources (AI reads), Prompts (user selects) |
| **Solves** | M×N integration problem → M+N |
| **Created by** | Anthropic (Nov 2024), adopted by OpenAI, Google, Microsoft |
| **Complements** | Function Calling (how model asks) + MCP (how system delivers) |

### 📖 Industry Reads
1. **Official:** [MCP Specification](https://modelcontextprotocol.io/) — The authoritative spec and docs
2. **Blog:** [Anthropic's MCP Announcement](https://www.anthropic.com/news/model-context-protocol) — Why they built it

### 🚀 Do These Now
1. **Install an MCP server (15 min):** Add the filesystem MCP server to Claude Desktop and browse your local files from chat
2. **Build a custom server (45 min):** Use the Python template above to build a server for your own notes or TODO list
3. **Explore the registry (15 min):** Browse [mcp.so](https://mcp.so/) to discover community-built MCP servers

### 🧭 Next Topic
> How does the LLM actually output those structured tool calls? → [04_Function_Calling_Tool_Use.md](04_Function_Calling_Tool_Use.md)
