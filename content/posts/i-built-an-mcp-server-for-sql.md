---
title: "I Built an MCP Server That Turns Any SQL Database Into an AI-Native Data Layer"
date: 2026-03-27
draft: false
tags: ["mcp", "ai", "sql", "data-engineering", "developer-tools"]
description: "The highest-friction point in data work is the copy-paste loop between AI and SQL. I built an MCP server that eliminates it — with a knowledge system that learns your data model over time."
---

The most painful part of using AI for data work isn't the AI. It's the round-trip.

You're in a conversation with Claude or Cursor. You need data. So you leave the conversation, open a SQL console, write a query, run it, copy the results, paste them back. By the time you're done, you've lost the thread.

I hit this wall daily at work — running Athena queries against a data lake with dozens of tables, each with their own gotchas. So I built an MCP server that lets the AI query the database directly.

Then I open-sourced a generic version: [**data-bench-mcp**](https://github.com/GG-science/data-bench-mcp).

---

## What is MCP?

[Model Context Protocol](https://modelcontextprotocol.io/) is Anthropic's open standard for connecting AI assistants to external tools. Think of it as a USB-C port for AI — any MCP client (Claude Code, Cursor, VS Code) can talk to any MCP server using the same interface.

The key primitives:
- **Tools** — functions the AI can call (like `run_query`)
- **Resources** — read-only context the AI loads automatically (like schema docs)

The server runs locally on your machine. No cloud infra. Just stdio.

---

## The Problem With Raw SQL Access

Giving AI raw database access sounds simple. But in practice, it's useless without context.

The AI doesn't know your column naming conventions. It doesn't know that `event_date` is a partition key you should always filter on. It doesn't know that table X has duplicates you need to deduplicate by `request_id`. It doesn't know that the query your teammate ran last week already solved this exact problem.

Raw access without knowledge just produces bad queries faster.

---

## The 4-Layer Knowledge System

This is what makes data-bench-mcp different from a generic SQL tool. It has a knowledge system that teaches the AI your data model:

| Layer | What It Is | How It Grows |
|-------|-----------|--------------|
| **Schemas** | Table definitions, column docs, join patterns | You write `.md` files when onboarding tables |
| **Templates** | Parameterized query skeletons | You add them for recurring analysis types |
| **Rules** | Mandatory guardrails | You add them when you discover data gotchas |
| **Learned Patterns** | Validated queries from real sessions | The AI saves them (with your approval) |

The AI is instructed to check **templates → rules → learned patterns** before writing any SQL. If nothing matches, it uses schema docs for column names and types.

The critical part: **the server gets smarter over time.** After a successful query, the AI asks if you want to save it as a reusable pattern. You approve, and next session, it finds and adapts that pattern instead of writing from scratch.

---

## How It Looks In Practice

**Before:** You ask the AI to analyze customer segments. It guesses column names, writes a query with wrong join logic, you debug it, re-run three times, eventually get results.

**After:**

```
You: "Show me top customers by completed order revenue"

AI: [checks templates → finds customer_segmentation template]
AI: [checks rules → finds general_safety rules about PII and row limits]
AI: [adapts template with your parameters, runs query]
AI: "Here are the results. Want to save this as a learned pattern?"
```

One turn. No copy-paste. No debugging column names.

---

## Architecture

```
AI Client (Claude Code / Cursor / VS Code)
    │ stdio (JSON-RPC)
    ▼
┌──────────────────────────────┐
│  data-bench-mcp              │
│                              │
│  9 Tools:                    │
│  ├─ run_query                │
│  ├─ list_databases           │
│  ├─ list_tables              │
│  ├─ describe_table           │
│  ├─ get_query_template       │
│  ├─ get_rules                │
│  ├─ get_learned_patterns     │
│  ├─ save_learned_pattern     │
│  └─ version                  │
│                              │
│  Knowledge:                  │
│  ├─ schemas/*.md             │
│  ├─ templates/*.md           │
│  ├─ rules/*.md               │
│  └─ learned/*.json           │
└──────────────┬───────────────┘
               │
       ┌───────┴───────┐
       ▼               ▼
    DuckDB          AWS Athena
   (local)        (boto3 creds)
```

Two backends today — DuckDB for local work, Athena for cloud data lakes. The backend is pluggable, so adding Postgres or BigQuery is straightforward.

Safety is structural: **SELECT-only enforcement** at the SQL validation layer (not just a prompt instruction). Write operations are blocked before they reach the database.

---

## Setup (3 Minutes)

```bash
git clone https://github.com/GG-science/data-bench-mcp.git
cd data-bench-mcp
cp .env.example .env
# Edit .env — point DATA_BENCH_DUCKDB_PATH to your database
```

Add to your Claude Code config:

```json
{
  "mcpServers": {
    "data-bench": {
      "command": "uv",
      "args": ["run", "--directory", "/path/to/data-bench-mcp", "data-bench-mcp"]
    }
  }
}
```

Restart. Ask: *"What version is the data-bench server?"*

That's it.

---

## What I Learned Building This

1. **Knowledge beats access.** Raw SQL access is table stakes. The value is in the knowledge layers — schemas, templates, rules, and especially learned patterns. Without them, the AI writes plausible-looking queries that are subtly wrong.

2. **Human-in-the-loop learning works.** The AI asks before saving patterns. No silent logging. This builds trust and means the learned pattern library is curated, not noisy.

3. **MCP is underrated for data teams.** Most MCP servers I see are for dev tools — GitHub, Slack, file systems. But the biggest pain point for data teams is the SQL round-trip, and MCP solves it cleanly.

4. **Start with SELECT-only.** I was tempted to add write operations. Don't. Read-only access with a knowledge layer is the 90% use case. Write operations are a different trust model entirely.

---

## What's Next

- More backends (Postgres, BigQuery)
- Query history and cost estimation tools
- A `reload` tool so you can add knowledge files without restarting
- Prompt templates for common analysis workflows

The repo is at [**GG-science/data-bench-mcp**](https://github.com/GG-science/data-bench-mcp). 42 tests passing. MIT licensed. PRs welcome.

---

*If you're on a data team and the copy-paste loop between AI and SQL is killing your flow — try it. Three minutes to set up, and the knowledge system compounds from there.*
