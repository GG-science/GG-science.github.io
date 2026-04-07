---
title: "Why I Built an MCP Server for SQL"
date: 2026-03-27
draft: false
tags: ["mcp", "sql", "data-engineering", "ai"]
description: "Giving AI raw database access produces bad queries faster. The missing piece is a knowledge layer that teaches the AI your data model and learns from real sessions."
---

Every data scientist I know has the same workflow: ask an AI a data question, leave the conversation, open a SQL console, write the query, copy the results back. The round-trip kills flow state and makes iterative analysis painful.

I built an [MCP server](https://github.com/GG-science/data-bench-mcp) that lets the AI query the database directly. But that alone isn't interesting — raw SQL access without context just produces bad queries faster. The AI doesn't know your column conventions, your partition keys, your dedup rules, or the query your teammate ran last week.

The interesting part is the knowledge layer.

## The 4-Layer Knowledge System

| Layer | Format | How It Grows |
|-------|--------|--------------|
| **Schemas** | `.md` | You document tables, columns, join patterns |
| **Templates** | `.md` | You add parameterized query skeletons for recurring analysis |
| **Rules** | `.md` | You codify data gotchas ("always dedup by request_id") |
| **Learned Patterns** | `.json` | The AI saves validated queries from real sessions — with your approval |

The AI checks **templates → rules → learned patterns** before writing any SQL. After a successful query, it asks: *"Want to save this as a pattern?"* You approve, it extracts `{{parameters}}`, and next session it adapts that pattern instead of writing from scratch.

The server gets smarter the more you use it.

## In Practice

**Before:** You ask for customer segments. The AI guesses column names, writes a query with wrong join logic, you debug three times.

**After:**

```
You: "Top customers by completed order revenue"

AI: [finds customer_segmentation template]
    [checks rules — PII masking, row limits]
    [runs query, returns results]
    "Want to save this as a learned pattern?"
```

One turn. No tab-switching. No copy-paste.

## Why MCP and Not Just a Python Script

A script solves the query execution problem. MCP solves the *integration* problem. The server registers tools (`run_query`, `describe_table`, `get_rules`, etc.) that any MCP client — Claude Code, Cursor, VS Code — can call natively. No glue code, no custom integrations, no API wrappers.

It also loads schema docs and rules as MCP resources, so the AI has your data model in context before it writes a single query. The knowledge isn't bolted on — it's structural.

Two backends today: DuckDB for local work, Athena for cloud data lakes. SELECT-only enforcement at the SQL validation layer, not just a prompt instruction.

Repo: [**GG-science/data-bench-mcp**](https://github.com/GG-science/data-bench-mcp). 42 tests. MIT licensed.
