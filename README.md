# AuditAE MCP server

A hosted [Model Context Protocol](https://modelcontextprotocol.io) server for SEO. It gives Claude, ChatGPT, Codex, Cursor and other MCP clients 99 tools: Google Search Console and Analytics reports, site crawls, live search results, keyword data, rank tracking, AI citation checks (ChatGPT, Perplexity, Gemini, Google AI Overviews) and WordPress edits.

- **Server URL:** `https://auditae.app/api/mcp` (streamable HTTP)
- **Sign-in:** OAuth, or an API key
- **Pricing:** pay per call from a prepaid balance. No subscription, no per-site or per-seat fee. The first audit is free and a new account needs no card.
- **Website:** [auditae.app/seo-mcp](https://auditae.app/seo-mcp) · **Docs:** [auditae.app/docs/mcp](https://auditae.app/docs/mcp)
- **MCP registry:** `app.auditae/mcp`

This repository holds the setup instructions and the tool list. The server is hosted, so there is nothing to install or run.

## Connect

### Claude (web, desktop)

**Customize → Connectors → Add custom connector**, paste `https://auditae.app/api/mcp`, then sign in to AuditAE or create an account in the same step.

### ChatGPT

Open `chatgpt.com/plugins`, select **+**, then **Add custom MCP server**. Name it AuditAE, paste `https://auditae.app/api/mcp`, choose OAuth, and sign in when asked. Needs a plan that allows custom MCP servers.

### Claude Code

```bash
claude mcp add --transport http --scope user auditae "https://auditae.app/api/mcp"
```

Then run `/mcp`, pick `auditae` and choose **Authenticate**.

### Codex (CLI, IDE extension, desktop app)

```bash
codex mcp add auditae --url "https://auditae.app/api/mcp"
```

Or add the block to `~/.codex/config.toml` and run `codex mcp login auditae`:

```toml
[mcp_servers.auditae]
url = "https://auditae.app/api/mcp"
tool_timeout_sec = 300
```

`tool_timeout_sec = 300` gives crawls and multi-engine citation checks room to finish. Codex stops a tool call after 60 seconds by default.

### VS Code

`.vscode/mcp.json`, or your user MCP configuration:

```json
{
  "servers": {
    "auditae": {
      "type": "http",
      "url": "https://auditae.app/api/mcp"
    }
  }
}
```

### Cursor and other clients that take a key

Create an API key in the AuditAE dashboard, then:

```json
{
  "mcpServers": {
    "auditae": {
      "url": "https://auditae.app/api/mcp",
      "headers": {
        "Authorization": "Bearer ae_live_…"
      }
    }
  }
}
```

Current steps for every client, with fixes for common errors: [auditae.app/docs/mcp](https://auditae.app/docs/mcp#client-setup).

**First prompt to try:** `What's my AuditAE balance and what can you do?`

## Running many sites

One workspace holds every site you add, and adding a site is free. You sign in with Google once, and each site is linked to its matching Search Console property. Every site tool takes a `site_id` or a `domain`, so one prompt can run the same check across all of them:

> For every site in my AuditAE workspace, compare clicks for the last 28 days with the 28 days before. List the five sites that lost the most, with the pages behind each loss.

The agent calls `list_sites`, then `query_search_console` once per site.

More prompts: [Codex MCP servers for SEO](https://auditae.app/blog/codex-mcp-servers-for-seo) and [Claude for SEO](https://auditae.app/blog/claude-for-seo).

If your sites live in a team workspace, use an API key from that workspace. OAuth sign-in connects your personal workspace.

## How billing works

- Each tool's description starts with its price, and every paid result returns `charged_cents`.
- Errors are not billed. The same Search Console report again within ten minutes is free, and so is the same citation check again within an hour.
- The agent is told to get your OK before a paid call. Several tools take `dry_run: true` to quote a price first.
- When the balance runs out, paid calls stop and the agent can hand you a top-up link (`create_top_up_link`, free).

## Tools

Generated from the live server on 2026-11-24. The server's own `tools/list` response and [auditae.app/seo-mcp](https://auditae.app/seo-mcp) are the current source for names and prices.

### Google Search Console and Analytics (5)

| Tool | What it does | Price |
|---|---|---|
| `query_search_console` | Query Google Search Console for a site | 1¢/100 rows |
| `find_search_opportunities` | Rank a site's pages by the search traffic and revenue they could win | 15¢ |
| `inspect_url` | Inspect a URL in Google Search Console | 1¢ |
| `query_google_analytics` | Run a Google Analytics 4 report for a site | 1¢/100 rows |
| `get_ai_referral_traffic` | AI assistant referral traffic from Google Analytics | 1¢ |

### Crawls and technical checks (13)

| Tool | What it does | Price |
|---|---|---|
| `run_technical_audit` | Run a technical SEO check on one page | 2¢/page |
| `crawl_site` | Crawl a whole site (up to 10,000 pages) and run a Site Audit | 1¢/10 pages |
| `get_latest_crawl` | Fetch a site's latest crawl: Site Health, theme scores and failing checks | free |
| `get_crawl_issue` | Get one Site Audit check's affected URLs, evidence and fix | free |
| `compare_crawls` | Compare two crawls of a site: new and fixed issues, Site Health change | free |
| `schedule_crawl` | Re-crawl a site weekly or monthly, with a summary email | 1¢/10 pages when run |
| `optimize_image` | Optimize images: resize and compress to WebP, with download links | 1¢/5 images |
| `check_agent_readiness` | Check whether AI agents can discover, read and buy from a site | free |
| `detect_tech_stack` | Detect a site's CMS / platform / tech stack | free |
| `get_backlink_summary` | Get a domain's off-page authority (backlink summary) | free |
| `submit_urls` | Tell Bing and other IndexNow engines that pages changed | free |
| `check_ai_crawlers` | Check which AI crawlers can reach a site | free |
| `check_llms_txt` | Validate a site's llms.txt | free |

### Keywords and rank tracking (4)

| Tool | What it does | Price |
|---|---|---|
| `keyword_research` | Research keywords from a seed term | 25¢ |
| `track_keywords` | Track Google rankings for keywords | 5¢ |
| `list_tracked_keywords` | List a site's tracked keywords and rankings | free |
| `untrack_keyword` | Stop tracking a keyword | free |

### Search results, backlinks and domain data (5)

| Tool | What it does | Price |
|---|---|---|
| `get_serp` | Google results for any keyword | 5¢ |
| `get_domain_keywords` | Keywords any domain ranks for in Google | 10¢ |
| `get_competing_domains` | A domain's organic search competitors | 10¢ |
| `get_keyword_metrics` | Search volume, CPC and difficulty for keywords | 10¢ |
| `get_referring_domains` | Who links to a domain: referring domains and anchors | 20¢ |

### AI citations and fix plans (15)

| Tool | What it does | Price |
|---|---|---|
| `check_brand_citation` | Check brand citation across AI engines | 5–12¢/check + playbook |
| `get_share_of_voice` | Compute share of voice vs competitors | 5–12¢/check + playbook |
| `explore_llm_mentions` | Pull historical AI citation corpus for a domain or keyword | 75¢ |
| `mine_competitor_prompts` | Find prompts where competitors are winning citations and you're not | 5¢ |
| `find_content_gap` | Find Google-search keywords a competitor ranks for that this site doesn't | 25¢ |
| `get_fix_plan` | Get a fix plan: exact edits that get the site cited by AI engines | 50¢ |
| `extract_competitors_from_text` | Extract competitors and sentiment from AI-generated text | free |
| `explore_web_mentions` | Scan open-web brand mentions with sentiment | 25¢ |
| `get_audit` | Read a citation audit's per-prompt, per-engine results | free |
| `schedule_recheck` | Schedule a one-time recheck of prompts after publishing fixes | 5–12¢/check when run |
| `get_recheck` | Get a recheck's status and before-vs-after citation changes | free |
| `cancel_recheck` | Cancel a scheduled recheck before it runs | free |
| `monitor_citations` | Monitor a site's AI citations weekly or monthly | 5–12¢/check when run |
| `get_changes` | What changed since last time: weekly briefing, citations and Google ranks | free |
| `find_citation_sources` | Find the pages AI engines cite instead of the brand, as outreach targets | 10¢ |

### Account, sites and change history (9)

| Tool | What it does | Price |
|---|---|---|
| `list_sites` | List the workspace's sites, site_ids and competitor domains | free |
| `get_balance` | Get the workspace's AuditAE credit balance | free |
| `create_top_up_link` | Create a Stripe checkout link to add AuditAE credit | free |
| `add_site` | Add a website to this workspace | free |
| `get_site_brief` | Read a site's brief: brand, customers, competitors, strategy | free |
| `update_site_brief` | Save to a site's brief | free |
| `record_change` | Record an edit to a live page so get_changes can report what happened next | free |
| `get_client_portal` | Get a site's client portal link and status | free |
| `update_client_portal` | Share, unshare or rebrand a site's client portal | free |

### Structured data (3)

| Tool | What it does | Price |
|---|---|---|
| `discover_brand_entities` | Look up a brand's authoritative entity record on Wikidata and Google's Knowledge Graph | free |
| `validate_schema` | Validate a schema.org JSON-LD payload against the AuditAE-supported types | free |
| `generate_schema` | Build a validator-ready JSON-LD payload from minimal inputs | free |

### WordPress: read and edit (28)

| Tool | What it does | Price |
|---|---|---|
| `wp_list_sites` | List paired WordPress sites for this AuditAE org | free |
| `wp_site_status` | Fetch live status of a paired WordPress site | free |
| `wp_create_post` | Create a post on a paired WordPress site | 5¢ |
| `wp_update_post` | Update an existing post on a paired WordPress site | 5¢ |
| `wp_trash_post` | Move a post on a paired WordPress site to the Trash, or restore it | 5¢ |
| `wp_get_seo_settings` | Read Yoast SEO settings from a paired WordPress site | free |
| `wp_set_seo_settings` | Write Yoast SEO settings on a paired WordPress site | 5¢ |
| `wp_list_posts` | List recent posts on a paired WordPress site | free |
| `wp_get_post` | Read a single post on a paired WordPress site | free |
| `wp_get_post_seo` | Read per-post SEO meta (Yoast, Rank Math, All in One SEO, or AuditAE's own) on a paired WordPress site | free |
| `wp_set_post_seo` | Write per-post SEO meta (Yoast, Rank Math, All in One SEO, or AuditAE's own) on a paired WordPress site | 5¢ |
| `wp_get_faq_schema` | Read the FAQ JSON-LD schema stored on a WordPress post | free |
| `wp_set_faq_schema` | Write FAQ JSON-LD schema onto a WordPress post | 5¢ |
| `wp_delete_faq_schema` | Remove the FAQ JSON-LD schema from a WordPress post | 5¢ |
| `wp_get_post_schema` | Read the per-post Schema Autopilot JSON-LD blocks for a WordPress post | free |
| `wp_set_post_schema` | Write per-post Schema Autopilot JSON-LD onto a WordPress post | 5¢ |
| `wp_delete_post_schema` | Remove the per-post Schema Autopilot JSON-LD from a WordPress post | 5¢ |
| `wp_get_organization_schema` | Read the site-wide Organization + WebSite Schema Autopilot JSON-LD | free |
| `wp_set_organization_schema` | Write site-wide Organization + WebSite Schema Autopilot JSON-LD | 5¢ |
| `wp_delete_organization_schema` | Remove the site-wide Organization + WebSite Schema Autopilot JSON-LD | 5¢ |
| `wp_probe_schema` | Probe a paired WordPress site's homepage for existing JSON-LD | free |
| `wp_get_llms_txt` | Read the paired site's /llms.txt config + rendered manifest | free |
| `wp_set_llms_txt` | Curate the paired site's /llms.txt config | 5¢ |
| `wp_read_page_blocks` | Read the raw Gutenberg block markup of a WordPress page or post | free |
| `wp_design_page` | Design or redesign a WordPress page using Gutenberg blocks (Page Composer) | 25¢ |
| `wp_reset_llms_txt` | Reset the paired site's /llms.txt config to defaults | 5¢ |
| `wp_optimize_media` | Compress images in a WordPress Media Library in place | 5¢ + 1¢/5 images |
| `wp_restore_media` | Restore the original images wp_optimize_media replaced | free |

### WordPress: SEO, links, media and redirects (17)

| Tool | What it does | Price |
|---|---|---|
| `wp_list_crawler_hits` | Read the AI crawler log of a paired WordPress site | free |
| `wp_list_redirects` | List the redirect rules on a paired WordPress site | free |
| `wp_suggest_redirects_from_404s` | Find the 404s AI crawlers hit on a paired WordPress site | free |
| `wp_add_redirect` | Add or update a redirect on a paired WordPress site | 5¢ |
| `wp_delete_redirect` | Delete a redirect on a paired WordPress site | 5¢ |
| `wp_get_link_graph` | Read the internal-link graph of a paired WordPress site | free |
| `wp_get_post_links` | Read the internal links to and from one WordPress post | free |
| `wp_insert_internal_link` | Add one internal link to a WordPress post | 5¢ |
| `wp_list_media` | List images in a paired WordPress Media Library | free |
| `wp_get_media` | Read one image from a paired WordPress Media Library | free |
| `wp_update_media` | Set an image's alt text, title, caption or description | 5¢ |
| `wp_upload_media` | Add an image from a URL to a paired WordPress Media Library | 5¢ |
| `wp_rename_media` | Rename an image file in a paired WordPress Media Library | 5¢ |
| `wp_bulk_update_media_alt` | Set the alt text of many images at once | 5¢ |
| `wp_list_broken_links` | List the broken internal links on a paired WordPress site | free |
| `wp_fix_broken_link` | Fix one broken link in a WordPress post | 5¢ |
| `wp_get_robots_audit` | Check which AI crawlers a WordPress site's robots.txt blocks | free |

## Support

Email support@auditae.app, or open an issue here for setup problems. Product code is not in this repository.

AuditAE is made by AuditAE LLC.
