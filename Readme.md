<div align="center">

# mcp-business-toolkit

A production Model Context Protocol server. **20+ real tools.** Lifted out of [Microns Hub](https://micronshub.eu) — the AI-native European manufacturing marketplace I built solo — and packaged so you can fork, configure, and run it.

![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![MCP](https://img.shields.io/badge/Model%20Context%20Protocol-1.0-6E56CF)
![Node](https://img.shields.io/badge/Node-20%2B-339933?logo=node.js&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-Postgres%20%7C%20Auth-3ECF8E?logo=supabase&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

*Single source file: src/index.ts — ~1,950 lines of working tool definitions.*

</div>

---

## What this is

This is the **actual MCP server** that runs in Microns Hub's production system. I lifted it into its own repository to make the pattern shareable and to serve as a portfolio piece. No simplification, no toy version — the only changes are renaming the package, sanitising env-var examples, and writing this README.

It exposes Microns Hub's lead-monitoring system, Google Search Console integration, and Supabase-backed business operations to Claude (via Claude Desktop, Claude Code, or any other MCP-aware client). Twenty-plus tools, three live resources, two prompt templates. Both stdio and HTTP transports.

If you're a hiring engineer reading this: the value isn't the toolkit, it's the architecture. See Architecture & decisions below.

## Tool surface

### Lead-monitoring tools

| Tool | What it does |
|------|---|
| `get_leads` | Filter the pipeline by source / score / status / date range / industry / keyword |
| `get_lead_detail` | Full lead record + matched keywords + activity history |
| `score_lead` | Manual high/medium/low override of the auto-score |
| `update_lead_status` | new → reviewed → contacted → saved → dismissed → converted |
| `save_response_draft` | Persist a drafted reply against a lead |
| `add_lead_note` | Free-text note, appended to lead and activity log |
| `get_lead_stats` | Volume + score-mix trend over a time period |
| `search_leads` | Full-text search across titles and bodies |
| `manage_keywords` | List / add / remove / toggle monitored keywords |
| `manage_subreddits` | List / add / remove / toggle monitored sources |

### Google Search Console tools

| Tool | What it does |
|------|---|
| `gsc_search_analytics` | Raw Search Analytics query |
| `gsc_get_top_queries` | Shortcut: top N queries by clicks |
| `gsc_get_top_pages` | Shortcut: top N pages, optional language filter |
| `gsc_compare_periods` | Compare totals between two equal-length windows |
| `gsc_inspect_url` | URL Inspection API on a single URL |
| `gsc_get_unindexed_pages` | List monitored URLs not PASS in cache |
| `gsc_submit_for_indexing` | Submit URLs to the Indexing API (200/day quota aware) |
| `gsc_get_indexing_quota` | Quota used today |
| `gsc_list_sitemaps` | List sitemaps for the property |
| `gsc_submit_sitemap` | Submit a new sitemap |

### Resources + Prompts

`leads://today`, `leads://keywords`, `leads://subreddits` — live snapshots of the pipeline.
`draft_lead_response`, `daily_lead_review` — prompt templates.

## Architecture & decisions

### One server, monolithic file
src/index.ts is the whole tool surface. ~1,950 lines, no clever decomposition. Considered: split per tool, split per domain, plugin architecture. Rejected because at this scale the cognitive cost of navigating between files outweighs scrolling. Each tool is 30–60 lines and self-contained. When the file hits 3,000 lines I'll reconsider.

### GSC credentials live in the database, not in env
A `gsc_config` table holds the OAuth refresh tokens. The server reads them at request time via the existing SUPABASE_SERVICE_KEY — no separate env var. Rotation is a single SQL update, picked up on the next call.

### Stdio AND HTTP transports
Stdio by default (Claude Desktop), HTTP-with-SSE when invoked with `--http --port`. Same tools, same code path. Two consumers, one integration.

### Manual overrides over auto-classification
Every lead has `auto_score` (set by the ingestion pipeline) and `manual_score` (human override). Reads prefer `manual_score` when set. The override path is first-class.

### Activity log on every state change
Every write tool appends to `lead_activity`. Append-only by RLS policy. Every change auditable.

### Validation via Zod, not at the SDK boundary
Each tool's schema is `z.object({...})`. The MCP SDK validates before the handler runs. No `if (!args.foo) throw` boilerplate.

## Install

```bash
git clone https://github.com/dimitrisvard/mcp-business-toolkit
cd mcp-business-toolkit
npm install
cp .env.example .env       # fill in real values
npm run build
```

## Run

**Stdio (Claude Desktop):** merge `examples/claude-desktop-config.json` into your `claude_desktop_config.json`, restart.

**Stdio (Claude Code):** `claude mcp add business-toolkit -- node /absolute/path/to/build/index.js`

**HTTP (remote agent runners):** `npm run start:http` (or `node build/index.js --http --port 3001`).

## Trade-offs

- **TypeScript over Python.** MCP SDK is TS-first. `npm install` beats virtualenv setup.
- **Postgres-backed everything, no Redis.** Adding a second datastore at this size doubles operational surface for negligible gain.
- **Telegram for alerts, not email or Slack.** Free, frictionless, no spam-filter wars.

## What I'd do differently

- **Land an eval harness for the prompt tools.** `draft_lead_response` and `daily_lead_review` deserve the same eval gate every other LLM-touching feature has.
- **Split index.ts at the 2,000-line mark.** GSC tools are a coherent cluster ready to extract.
- **Add a `dry_run` parameter to write tools.** Would let me debug at the terminal without polluting the activity log.

## License

MIT — see [LICENSE](./LICENSE).

## Contact

Dimitris Vardalachakis · `dimitrisvard@hotmail.com` · [github.com/dimitrisvard](https://github.com/dimitrisvard) · [linkedin.com/in/dimitrisvard](https://www.linkedin.com/in/dimitrisvard)

Built while running [Microns Hub](https://micronshub.eu). Open to remote AI Product Engineer / Founding Engineer roles in Europe.

