# Agent tooling setup (literature + repos)

The autonomous agent in this case study is **Claude Code** (Anthropic **Claude
Opus 4-series**), run in a `tmux` session on the cluster login node. A
cheaper/faster model (Sonnet) was used for routine iteration when usage limits
bit; the strongest model was reserved for hard debugging.

## Which config matters for *you*

| How you run the agent | Where MCP must be configured |
|------------------------|------------------------------|
| **Claude Code / `claude` CLI in tmux** on the login node (your autoresearch loop) | **[Claude Code MCP](https://docs.anthropic.com/en/docs/claude-code/mcp)** — `claude mcp add`, **`~/.claude.json`**, and/or **project `.mcp.json`** in this repo. |
| **Cursor** chat only | **`~/.cursor/mcp.json`** — does **not** apply to the tmux `claude` CLI. |

If the autonomous agent only runs **inside tmux via Claude Code**, **`~/.cursor/mcp.json` is irrelevant** unless you also use Cursor. Configure MCP for **Claude Code** on the **same machine** where tmux runs (`claude mcp list`, `/mcp` inside a session).

---

## 1. Claude Code MCP (primary — tmux on login node)

Official guide: **[Connect Claude Code to tools via MCP](https://docs.anthropic.com/en/docs/claude-code/mcp)**.

### Quick adds (HTTP)

```bash
# DeepWiki — GitHub repo wikis / structure
claude mcp add --transport http deepwiki https://<wiki-server>/mcp

# alphaXiv — papers MCP (needs API key; create one in your alphaXiv account)
export ALPHAXIV_API_KEY="your-key-here"   # put in ~/.bashrc or session before agent
claude mcp add --transport http alphaxiv https://<lit-server>/mcp/v1 \
  --header "Authorization: Bearer ${ALPHAXIV_API_KEY}"
```

If `--header` does not expand env vars in your CLI version, paste the key once (avoid committing it) or use **project `.mcp.json`** below with `"Bearer ${ALPHAXIV_API_KEY}"`.

### arXiv MCP (stdio — local `uv`)

Current **arxiv-mcp-server** (0.3.x) needs **Python ≥3.11**; **`dwc-vit`** is often **3.10**, so use **`uv tool run`** from that conda env (see [blazickjp/arxiv-mcp-server](https://github.com/blazickjp/arxiv-mcp-server)):

```bash
conda activate /path/to/conda-env
pip install uv
uv tool install arxiv-mcp-server
mkdir -p /path/to/scratch/.arxiv-mcp-server/papers

claude mcp add --transport stdio arxiv-mcp-server -- \
  /path/to/conda-env/bin/uv tool run arxiv-mcp-server \
  --storage-path /path/to/scratch/.arxiv-mcp-server/papers
```

### Hugging Face MCP

Open **[<hub>/settings/mcp](https://<hub>/settings/mcp)** while logged in. If they offer **Claude Code** instructions, use those. Otherwise use **`claude mcp add-json`** with the JSON they provide (often `type` + `url` + `headers`), or add the equivalent **`claude mcp add --transport http ...`** line. **Do not commit** raw tokens; prefer env vars in `.mcp.json` (`${VAR}` — [supported in project `.mcp.json`](https://docs.anthropic.com/en/docs/claude-code/mcp#environment-variable-expansion-in-mcpjson)).

### Manage & verify

```bash
claude mcp list
claude mcp get deepwiki
# Inside an interactive Claude Code session:
#   /mcp
```

---

## 2. Project `.mcp.json` (this repo)

A **project-scoped** [`.mcp.json`](../.mcp.json) at the repo root is checked in so every clone shares the same **server list**; **secrets** stay in environment variables (e.g. `ALPHAXIV_API_KEY`). After pulling, run **`claude mcp reset-project-choices`** once if the agent asks for approval.

**Before starting the loop in tmux:**

```bash
export ALPHAXIV_API_KEY="..."   # required for alphaXiv MCP (401 without it)
export PATH="/path/to/conda-env/bin:$PATH"  # if needed for uv
```

---

## 3. GitHub CLI (`gh`)

Not an MCP server; available to the agent as **shell commands** if `gh` is on `PATH` and **`gh auth login`** was run on the login node.

```bash
gh auth status
```

---

## 4. Optional: Cursor-only MCP

If you **also** use Cursor, configure **`~/.cursor/mcp.json`** separately. Same URLs (DeepWiki, alphaXiv with Bearer header if the UI supports it, etc.) — see Cursor MCP docs. This does **not** wire tools into tmux `claude`.

---

## 5. Verification checklist (Claude Code loop)

- [ ] On the login node: `claude mcp list` shows **deepwiki**, **alphaxiv**, **arxiv-mcp-server**, and **Hugging Face** (if added).
- [ ] `ALPHAXIV_API_KEY` set when using alphaXiv (endpoint returns **401** without auth).
- [ ] `gh auth status` OK.
- [ ] Agent prompt still points at **`program.md`** + **`autoresearch/notes.md`**.

**Notes discipline:** Literature in `notes.md` = **motivation / context**; **Slurm + metrics** = validation. See **`program.md` § Traceability in `autoresearch/notes.md`**.
