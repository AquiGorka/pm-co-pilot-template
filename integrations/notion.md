# Notion Integration

## What it does

Read, search, and update Notion pages programmatically via the claude.ai Notion MCP server.

## Auth

OAuth via `/mcp` > "claude.ai Notion". The MCP server is built into Claude Code. No tokens or API keys needed — just approve the OAuth flow in your browser.

**Known issue:** The OAuth flow may hang after browser approval (Enter key does nothing). Workaround: restart Claude Code — the connection often initializes on its own after restart.

## Available tools

| Tool | Purpose |
|------|---------|
| notion-search | Semantic search across workspace and connected sources |
| notion-fetch | Get full page content as enhanced markdown |
| notion-update-page | Update page properties or content |
| notion-create-pages | Create new pages |
| notion-duplicate-page | Duplicate an existing page |
| notion-create-comment | Add comments to pages |
| notion-get-comments | Read page comments |

## Common operations

### Search for a page

```json
{
  "query": "project spec",
  "filters": {},
  "content_search_mode": "workspace_search",
  "page_size": 5
}
```

Use `workspace_search` for Notion-only results (faster). Use `ai_search` to include connected sources (Linear, Slack, Google Drive, etc.).

### Read a page

```json
{"id": "page-uuid-or-url"}
```

Returns full page content as enhanced markdown. Large pages (70K+ chars) may exceed token limits — process the saved file instead.

### Update specific text on a page (search-and-replace)

```json
{
  "page_id": "uuid",
  "command": "update_content",
  "content_updates": [
    {"old_str": "text to find", "new_str": "replacement text"},
    {"old_str": "another match", "new_str": "another replacement"}
  ]
}
```

Up to 100 replacements per call. Use `replace_all_matches: true` for terms appearing multiple times.

### Replace entire page content

```json
{
  "page_id": "uuid",
  "command": "replace_content",
  "new_str": "full markdown content"
}
```

**WARNING:** This destroys all native Notion blocks (images, file attachments, embeds). Only use on pages with no attachments. See "Critical limitations" below.

## Critical limitations

### Attachments cannot be recreated via the API

Notion stores images and file attachments as native blocks with internal `attachment:UUID:filename` references. These blocks:

- **Are preserved** by `update_content` (search-and-replace) — it only touches text, not attachment blocks
- **Are destroyed** by `replace_content` — it replaces everything, including attachment blocks
- **Cannot be recreated** by writing `attachment:UUID:filename` in markdown — Notion strips the protocol

**Rule: Never use `replace_content` on pages with images or attachments. Always use `update_content`.**

### S3 image URLs are temporary

When you `fetch` a page, images appear as signed S3 URLs (`https://prod-files-secure.s3.us-west-2.amazonaws.com/...`). These expire. Don't store or write them back — they won't resolve later.

### Multi-line matching is fragile

The API often fails to match multi-line strings due to indentation differences. Prefer single-line matches. If a match fails, try with more or less surrounding context.

### Large pages exceed token limits

Pages over ~50K chars can't be read into context directly. The `fetch` result is saved to a file — process it with Bash/Python or an agent.

## Recommended workflows

### Updating an existing page (with attachments)

1. Fetch the page and save locally as backup
2. Use `update_content` with targeted search-and-replace operations
3. Verify after each batch by fetching again
4. Never use `replace_content`

### Uploading new content to an empty page

1. Split content into ~8K char chunks at newline boundaries
2. Upload chunk 0 via `replace_content`
3. Append chunks 1-N via `update_content` (match end of previous chunk, replace with itself + new chunk)
4. Fix any attachment references via `update_content` afterwards (only works for attachments that already exist on the page)

### Before any destructive operation

1. Duplicate the page in Notion UI (native duplicate preserves attachments)
2. Work on the duplicate first to verify the approach
3. Apply to the real page only after verification

## Gotchas

- `fetch` returns a single massive JSON blob — use file-based processing for large pages
- Notion transforms content on storage (whitespace, URL signing, block formatting) — what you write is not exactly what you read back
- Empty string replacements (`"new_str": ""`) remove the text but leave empty blocks
- `notion-duplicate-page` may fail with permission error if you can't insert into the parent — duplicate manually instead
- `{toggle="true"}` on headings makes them collapsible in Notion
