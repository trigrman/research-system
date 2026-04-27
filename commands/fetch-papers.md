---
name: fetch-papers
description: Manually run the paper fetching script to retrieve new papers from arXiv, Semantic Scholar, and Google Scholar. Supports --days-back for historical searches.
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

Check if the user provided a number of days as an argument (e.g., `/fetch-papers 90`).

- If a number is provided, use it as the `--days-back` flag
- If no argument is provided, run with default config values (daily mode)

## Step 3: Run Fetch Script

1. **Run the fetch script:**

   If no days-back argument:
   ```bash
   cd ${CLAUDE_PLUGIN_ROOT}/scripts/automation && python3 fetch_papers.py
   ```

   If days-back argument provided (e.g., user said `/fetch-papers 90`):
   ```bash
   cd ${CLAUDE_PLUGIN_ROOT}/scripts/automation && python3 fetch_papers.py --days-back 90
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
- Semantic Scholar is free (no API key needed), rate limited to ~100 requests per 5 minutes
- Results are written to `daily-digests/YYYY-MM-DD.md`
- Duplicate papers (seen before) are automatically filtered out
- Check `.research-data/fetch_papers.log` for detailed execution history
