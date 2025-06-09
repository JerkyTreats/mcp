
# 🧠 MCP System: Goals and Action Plan

## 🎯 Primary Goals

### 🧠 Context-aware Agent Infrastructure
- Build a **local MCP server** on your **GPU-enabled PC**
- Securely access it via **Tailscale** from VS Code or other dev machines
- Support **multiple repos/workspaces**, each with logically isolated context

### 🧩 Modular Context Sources
- Symbol-level indexing via **Tree-sitter**
- Semantic embedding + search via **Vector DB** (e.g., Qdrant or pgvector)
- Ingest `.agent/` folders containing design notes, feature briefs, and style guides

### 🔁 Automated Sync
- On `push to master`, re-index:
  - Symbol extraction
  - Code/doc embedding
  - Vector DB update
  - `.agent/` doc processing

### 🧪 Integration with Copilot Chat
- Use `.vscode/copilot-chat.json` per repo
- Deliver fast, scoped context via MCP API to GitHub Copilot Chat

---

## ✅ Action Plan

### 🗂 Phase 1: MCP Server Skeleton
- [ ] Choose backend: Go / Python / Node
- [ ] Implement JSON-RPC HTTP server
- [ ] Support endpoints: `getSymbols`, `getDefinition`, `getContext`
- [ ] Use per-workspace directory:
  ```
  workspaces/
    repo-a/
    repo-b/
  ```

### 🌲 Phase 2: Tree-sitter Symbol Indexing
- [ ] Parse Go files with Tree-sitter
- [ ] Build symbol table per workspace
- [ ] Store in SQLite or DuckDB
- [ ] Enable fast function/struct lookups

### 🔎 Phase 3: Semantic Search with Vector DB
- [ ] Chunk functions and `.agent/*.md` by section
- [ ] Embed using OpenAI or local GGUF model
- [ ] Store vectors in Qdrant or pgvector
- [ ] Add vector search endpoints (`searchContext`, `getSimilar`)

### 📝 Phase 4: `.agent/` Folder Integration
- [ ] Recursively parse `.agent/**/*.md`
- [ ] Index section headings and content
- [ ] Embed + tag as `agent_note`
- [ ] Add endpoints like `getAgentNotes(topic)`

### 🔄 Phase 5: Ingest Trigger Mechanism
- [ ] Create `/ingest` endpoint
- [ ] Trigger from:
  - Webhook (GitHub)
  - CLI (`curl`)
  - CI/CD job
- [ ] Refresh symbols, embeddings, and agent docs

### 🧩 Phase 6: VS Code + Tailscale Integration
- [ ] Set hostname like `mcpbox.tailnet.ts.net`
- [ ] Add to `.vscode/copilot-chat.json`:
  ```json
  {
    "contextServers": [
      {
        "url": "http://mcpbox.tailnet.ts.net:3000",
        "workspace": "repo-a",
        "languages": ["go"]
      }
    ]
  }
  ```

---

## 🔄 Optional / Future Steps

| Feature | Priority | Notes |
|--------|----------|-------|
| LLM summarization of `.agent/` + README | Medium | Auto-generated summaries |
| LangGraph / LangChain agent planning | Low | Optional for advanced workflows |
| ACL or multi-user auth layer | Medium | Useful for shared usage |
| Git diff-based incremental ingest | Medium | Optimize for large repos |
| Web UI dashboard | Low | For browsing workspaces and logs |

---

## 🧰 Next Steps

- Option to scaffold:
  - `mcp-server-go` or `mcp-server-py`
  - Ingest + vector search + agent note parser
