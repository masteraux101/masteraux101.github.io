---
name: AI Prompt Scheduler
description: Schedule recurring AI model calls with a fixed prompt via GitHub Actions cron, and email the response to the user automatically
---

# AI Prompt Scheduler

Schedule a cron-based AI model call that emails the result. You craft the prompt from the user's vague idea — never ask them to write it.

## When to Use

Keywords: "每天总结", "daily digest", "schedule a prompt", "recurring AI task", "定期分析", "每周报告"

## Context & Information

Read the **📋 Current Session Context** in your system instruction.

- **Auto-filled** (don't ask): Model name, Gemini API Key (✅), Resend API Key (✅), Notification email
- **Ask the user**: Schedule (e.g. "every day 9am UTC"), Task name slug (or auto-generate)
- Only ask about keys/email if marked ❌ in context

## Prompt Crafting

Turn a vague idea into a structured prompt with: **Role**, **Task**, **Scope**, **Format**, **Language** (match user's), **Length**, and **Date** (`{date}` placeholder, replaced at runtime).

Generate files directly — don't show the prompt separately for approval.

## ⚠️ Output Format — DEPLOY_BUNDLE

**ONLY output the bundle.** One short sentence before it, nothing after. No explanations, no setup steps.

### Format:

````markdown
<!--DEPLOY_BUNDLE:{"name":"SLUG","schedule":"CRON","scheduleText":"human text","description":"one-line summary"}-->

```python:ai-prompt-scheduler-SLUG.py
# script
```

```yaml:.github/workflows/ai-scheduler-SLUG.yml
# workflow
```

<!--/DEPLOY_BUNDLE-->
````

### Rules:
1. Valid JSON in `<!--DEPLOY_BUNDLE:...-->`
2. Every code block needs `language:filename` tag
3. No text between code blocks or after `<!--/DEPLOY_BUNDLE-->`
4. **⚠️ PATH RULE**: Workflow YAML must reference `artifacts/FILENAME.py` (not bare `FILENAME.py`). Code block tag stays bare.
5. Python script reads `GEMINI_API_KEY` from `os.environ`, uses `{date}` placeholder
6. Workflow must include `workflow_dispatch:` for manual testing
7. Email step: do NOT truncate output — send the full AI response

## Example

````markdown
✅ 已配置每日AI新闻摘要任务。

<!--DEPLOY_BUNDLE:{"name":"daily-ai-news","schedule":"0 9 * * *","scheduleText":"每天 9:00 UTC","description":"每天调用AI总结AI领域新闻并发送邮件"}-->

```python:ai-prompt-scheduler-daily-ai-news.py
#!/usr/bin/env python3
"""AI Prompt Scheduler: daily-ai-news"""
import urllib.request, json, os, sys
from datetime import datetime, timezone

API_KEY = os.environ["GEMINI_API_KEY"]
MODEL   = "gemini-2.5-flash-preview-05-20"

PROMPT_TEMPLATE = """你是一位资深科技分析师。今天是 {date}。
请撰写一份简洁的 AI 领域每日简报，涵盖最重要的 5 条进展。
分为：大模型、应用落地、开源动态三个板块。语言：中文。"""

PROMPT = PROMPT_TEMPLATE.replace("{date}", datetime.now(timezone.utc).strftime("%Y-%m-%d"))
url  = f"https://generativelanguage.googleapis.com/v1beta/models/{MODEL}:generateContent?key={API_KEY}"
body = json.dumps({"contents": [{"parts": [{"text": PROMPT}]}]}).encode()
req  = urllib.request.Request(url, data=body, headers={"Content-Type": "application/json"})
try:
    with urllib.request.urlopen(req) as resp:
        data = json.load(resp)
    print(data["candidates"][0]["content"]["parts"][0]["text"])
except urllib.error.HTTPError as e:
    print(f"API error {e.code}: {e.read().decode()}", file=sys.stderr)
    sys.exit(1)
```

```yaml:.github/workflows/ai-scheduler-daily-ai-news.yml
name: AI Scheduler — daily-ai-news
on:
  schedule:
    - cron: "0 9 * * *"
  workflow_dispatch:
jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run AI prompt
        id: ai
        env:
          GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
        run: |
          python3 artifacts/ai-prompt-scheduler-daily-ai-news.py 2>&1 | tee /tmp/ai_output.txt
      - name: Send email
        if: always()
        env:
          RESEND_API_KEY: ${{ secrets.RESEND_API_KEY }}
          NOTIFY_EMAIL: ${{ vars.NOTIFY_EMAIL }}
        run: |
          RESULT=$(cat /tmp/ai_output.txt)
          DATE=$(date -u '+%Y-%m-%d %H:%M UTC')
          SUBJECT="AI Digest: daily-ai-news — ${DATE}"
          python3 -c "
          import sys, json
          content = open('/tmp/ai_output.txt').read()
          html = '<pre style=\"font-family:monospace;white-space:pre-wrap\">' + content + '</pre>'
          payload = json.dumps({'from':'BrowserAgent <onboarding@resend.dev>','to':['${NOTIFY_EMAIL}'],'subject':'${SUBJECT}','html':html})
          sys.stdout.write(payload)
          " > /tmp/email_payload.json
          curl -s -X POST https://api.resend.com/emails \
            -H "Authorization: Bearer ${RESEND_API_KEY}" \
            -H "Content-Type: application/json" \
            -d @/tmp/email_payload.json
```

<!--/DEPLOY_BUNDLE-->
````

## Notes

- `{date}` is replaced with UTC date at runtime
- `onboarding@resend.dev` works on Resend free tier without domain verification
- Secrets & variables are auto-synced on deploy — never tell user to add them manually
- For OpenAI: swap API URL to `https://api.openai.com/v1/chat/completions`, use `Authorization: Bearer` header
- Cron reference: `0 9 * * *` (daily 9am), `0 8 * * 1` (Mon 8am), `0 */6 * * *` (every 6h)
- **NEVER add setup instructions or bullet lists in your response**
