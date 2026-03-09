---
name: Web Scraper
description: Generate Python scripts to scrape web content and extract structured data, executed via GitHub Actions
---

# Web Scraper Skill

**When to use:** "scrape website", "extract data from", "crawl page", "抓取网页", "爬取数据"

## Rules
1. Always generate a self-contained Python script that runs on GitHub Actions
2. Use `urllib.request` + `html.parser` for simple pages (no external deps)
3. For complex pages, use `requests` + `beautifulsoup4` (pip install in workflow)
4. Respect robots.txt — include a check when possible
5. Add proper User-Agent headers
6. Handle pagination if the user implies they want all results
7. Output structured data as JSON or CSV

## Output Format
Generate a script wrapped in a code block with `python:filename.py` tag.

```python
#!/usr/bin/env python3
"""Web scraper for [TARGET]"""
import urllib.request
import json
from html.parser import HTMLParser

URL = "https://example.com"
HEADERS = {"User-Agent": "BrowserAgent-Scraper/1.0"}

# ... scraping logic ...

# Output results
print(json.dumps(results, indent=2, ensure_ascii=False))
```

## Template Workflow Section
```yaml
- name: Install dependencies
  run: pip install requests beautifulsoup4
  
- name: Run scraper
  run: python artifacts/scraper.py | tee /tmp/_browseragent_output.txt
```

## Important
- Never scrape login-required pages without explicit credentials
- Rate-limit requests (add `time.sleep(1)` between requests)
- Handle HTTP errors gracefully
