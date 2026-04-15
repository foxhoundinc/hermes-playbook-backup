# Hermes Core Architecture — What to Know

**Source:** Hermes Agent Complete Guide v260408 (Orange Book) + gbrain docs

---

## The Three Layer Memory System

This is the heart of Hermes — it doesn't just chat, it *remembers*.

| Layer | What it stores | Lifetime |
|-------|---------------|----------|
| **Session** | Current conversation | Gone when chat ends |
| **Persistent** | Who you are — preferences, habits, project context | Forever, across sessions |
| **Skills** | How to do things — procedural knowledge distilled from recurring tasks | Auto-improves over time |

### Honcho (optional add-on)
Plastic Labs' dialectical user modeling. Infers 12 identity layers — not just what you said, but *who you are* as a person. Example: 3 weeks of Python tasks → Honcho infers "technical level: intermediate, prefers functional style, uses Cursor as main editor."

**Note:** Honcho is already installed in your setup.

---

## The Learning Loop

After every task, Hermes automatically:
1. Retrospective — what went wrong, what went right
2. Skill extraction — distill recurring patterns into `~/.hermes/skills/`
3. Memory update — persist new user preferences
4. Self-correction — flag what to avoid next time

This means Hermes gets better the more you use it. Not from you editing files — it does it automatically.

---

## Skills System

**Every skill is a markdown file** in `~/.hermes/skills/`. No framework, no API — just text.

```markdown
##-
name: my-skill
description: What this does
version: "1.0.0"
##-

# My Skill
## Trigger
When the user asks me to...

## Rules
1. Step one
2. Step two

## Example
Input: "..."
Output: "..."
```

Skills are already auto-improving from your 3 weeks of use (V0.40 → V0.80). Check what's in your skills folder.

---

## Toolsets — Pragmatic Security

Don't enable everything. Tools are grouped into toolsets:

```yaml
toolsets:
  - web        # web search
  - terminal   # terminal commands
  - file       # file operations
  - skills     # skill management
  - delegation # sub-agent spawning
  # - homeassistant  # disable what you don't need
```

**Rule:** Only enable what the current task needs. Lock down the rest.

---

## Sub-Agent Delegation

Can spawn up to **3 concurrent sub-agents** with:
- Isolated conversation context
- Restricted toolsets per sub-agent
- Parallel execution for research tasks

**Good for:** Competitive analysis, parallel research, splitting complex tasks
**Overkill for:** Tasks that fit in a single context window

Rule of thumb: If you're writing long consolidation instructions for the main agent, the decomposition is wrong.

---

## MCP Integration

Opens 6,000+ external apps. Two modes:

```yaml
# stdio mode
mcp_servers:
  github:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxxxx"

# HTTP mode
mcp_servers:
  remote-tools:
    url: "https://your-server.com/mcp"
    transport: "streamable-http"
```

**Per-server tool filtering** — expose only what the agent needs:

```yaml
mcp_servers:
  github:
    allowed_tools:
      - "list_issues"
      - "create_issue"
      # Block: delete_repo, change_settings
```

---

## Gateway — Multi-Platform

One process connects Telegram + Discord + all other platforms simultaneously. All platforms share the same brain.

---

## Deployment Options

| Option | Cost | Best for |
|--------|------|---------|
| Local install | Free | Development, trying it out |
| Docker | Free | Isolation without full VPS |
| $5 VPS + Telegram | $5/mo + API | 24/7 always-on personal agent |
| Serverless (Daytona/Modal) | ~$0 idle | Wake on message, cheapest |

**Your current:** NUC running 24/7 → equivalent to the VPS setup but free.

---

## The Thin Harness Philosophy

From gbrain's ethos docs:

> **Thin harness, fat skills.** Keep the core agent light with minimal hardcoded rules. Let rich skills handle complexity at the edges.

This means:
- Don't over-configure the agent
- Build domain-specific skills instead
- Skills are the competitive advantage, not the model choice

---

## Key Numbers

| Metric | Value |
|--------|-------|
| GitHub stars | 27,000+ (2 months) |
| Built-in tools | 40+ |
| Supported platforms | 14 |
| MCP integrations | 6,000+ apps |
| Sub-agent concurrency | Up to 3 |
| Memory usage (no local LLM) | <500MB |
| Minimum VPS cost | $5/month |
| License | MIT |

---

## Critical Limitations to Know

1. **Memory database grows forever** — no auto-expiration. Periodically check `~/.hermes/` size and clean up old skills.
2. **Memory pollution risk** — if Hermes learns something wrong early on, it can persist. Monitor and correct proactively.
3. **Self-improvement paradox** — you want it autonomous, but you also need to audit results. Set a rhythm for checking skill files.
4. **Skill conflicts** — two skills with overlapping triggers → Hermes picks highest match score. May not be what you expect.

---

## overlap with Enhanced Mind

Your Enhanced Mind project IS the "brain" layer for Hermes. The gbrain schema's MECE directory structure for people/companies/deals/projects maps directly onto how Enhanced Mind organizes cognitive optimization data.

**gbrain's "brain" structure:**
```
people/ → company/ → deals/ → projects/ → concepts/
```

**Enhanced Mind equivalent** (inference from your research library):
```
core-theory/ → elite-performance/ → neuroscience/ → skill-acquisition/
```

The gbrain approach of "compiled truth above, timeline below" is directly applicable to how you process Tim Ferriss / Huberman / JRE transcripts.
