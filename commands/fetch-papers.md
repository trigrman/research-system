---
name: fetch-papers
description: Manually run the paper fetching script to retrieve new papers from arXiv, Semantic Scholar, and Google Scholar. Supports --days-back for historical searches and --query for ad hoc searches.
allowed-tools: [Read, Bash]
model: haiku
---

# Fetch Papers Command

Manually trigger paper fetching from arXiv, Semantic Scholar, and Google Scholar instead of waiting for the scheduled cron job.

## Step 1: Load Configuration

1. Read `~/.claude/research-system-config/config.yaml`
2. Extract `paths.research_root` for log file location
3. Set `log_file = research_root + "/.research-data/fetch_papers.log"`

## Step 2: Parse Arguments

Three modes are supported:

- **Daily mode** — no arguments: `/fetch-papers`
- **Backfill mode** — a number of days: `/fetch-papers 90` → `--days-back 90`
- **Ad hoc mode** — one or more `--query` strings, optionally with `--topic-name` and `--days-back`. Bypasses keywords.md. Example: `/fetch-papers --query "picture superiority effect" --query "dual coding theory" --topic-name aiviz-talk --days-back 365`

Pass through any `--query`, `--topic-name`, and `--days-back` flags the user provides directly to the script.

## Step 3: Run Fetch Script

1. **Run the fetch script:**

   Daily mode:
   ```bash
   cd ${CLAUDE_PLUGIN_ROOT}/scripts/automation && python3 fetch_papers.py
   ```

   Backfill mode (e.g., user said `/fetch-papers 90`):
   ```bash
   cd ${CLAUDE_PLUGIN_ROOT}/scripts/automation && python3 fetch_papers.py --days-back 90
   ```

   Ad hoc mode — pass through `--query` (repeatable), `--topic-name`, and any `--days-back`:
   ```bash
   cd ${CLAUDE_PLUGIN_ROOT}/scripts/automation && python3 fetch_papers.py --query "..." --query "..." --topic-name <name> --days-back <N>
   ```

2. **Capture output** - the script will show:
   - Which sources are being searched (arXiv, Semantic Scholar, Google Scholar)
   - Number of papers found per keyword/topic
   - Any errors or rate limiting messages
   - Path to created digest file

## Step 3: Report Results

**Provide summary to user:**

```
Fetch Papers Complete

Sources searched:
- arXiv: [X] keywords searched
- Semantic Scholar: [X] keywords searched
- Google Scholar: [Searched / Skipped (not Sunday) / No API key]

Results:
- New papers found: [X]
- Digest created: [path to daily-digests/YYYY-MM-DD.md]

[If any errors occurred, list them here]
```

**If script fails:**
- Show the error output
- Suggest checking `log_file` for details
- Common issues:
  - Network connectivity
  - Rate limiting (arXiv 429 errors)
  - Invalid SerpAPI key

## Error Handling

- **Script not found**: Run `/fix-scheduled-scripts` to repair plugin symlink
- **Config not found**: Run `/setup-research-automation` first
- **Python errors**: Show full error output for debugging
- **Rate limiting**: Note that arXiv has a 10-second delay between queries

## Notes

- **Daily mode** (no arguments): Searches arXiv and Semantic Scholar with config defaults (1 day, 10 results). Google Scholar only on Sundays.
- **Backfill mode** (`/fetch-papers 90`): Searches all sources for last N days with max_results bumped to 100. Google Scholar runs regardless of day.
- **Ad hoc mode** (`--query`): Bypasses keywords.md. Each `--query` becomes one search string under the topic named by `--topic-name` (default `ad-hoc`). Combine with `--days-back N` for historical scope.
- Semantic Scholar is free (no API key needed), rate limited to ~100 requests per 5 minutes
- Results are written to `daily-digests/YYYY-MM-DD.md`
- Duplicate papers (seen before) are automatically filtered out
- Check `.research-data/fetch_papers.log` for detailed execution history
