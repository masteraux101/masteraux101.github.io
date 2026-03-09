---
name: GitHub Actions Scheduler
description: Schedule recurring tasks via GitHub Actions cron workflows, with email notification on completion
---

# GitHub Actions Scheduler

Schedule recurring code execution on GitHub Actions with email notification.

## When to Use

Keywords: "run every day", "schedule a task", "cron job", "定时运行", "每天执行"

## Context & Information

Read the **📋 Current Session Context** for auto-filled settings.

- **Auto-filled** (don't ask): Resend API Key (✅), Notification email
- **Ask the user**: Schedule, Task description or code

## Cron Quick Reference

| When | Cron |
|------|------|
| Daily 9am UTC | `0 9 * * *` |
| Monday 8am | `0 8 * * 1` |
| Every hour | `0 * * * *` |
| Every 30 min | `*/30 * * * *` |
| 1st of month 9am | `0 9 1 * *` |

## ⚠️ Output Format — DEPLOY_BUNDLE

**ONLY output the bundle.** One short sentence before it, nothing after. No explanations, no setup steps.

### Rules:
1. Valid JSON in `<!--DEPLOY_BUNDLE:...-->`
2. Every code block needs `language:filename` tag
3. No text between code blocks or after `<!--/DEPLOY_BUNDLE-->`
4. **⚠️ PATH RULE**: Workflow YAML must reference `artifacts/FILENAME.py` (not bare `FILENAME.py`). Code block tag stays bare.
5. Always include `workflow_dispatch:` for manual testing
6. Email step: send full output — do NOT truncate with `head`
7. Use `python3 -c` to build JSON payload and write to file, then `curl -d @file` to avoid shell escaping issues

## Example

````markdown
✅ 已配置定时任务。

<!--DEPLOY_BUNDLE:{"name":"daily-report","schedule":"0 9 * * *","scheduleText":"每天 9:00 UTC","description":"每天运行报告脚本并发送邮件"}-->

```python:daily-report.py
# ... script content ...
```

```yaml:.github/workflows/scheduler-daily-report.yml
name: Scheduled Task — daily-report
on:
  schedule:
    - cron: '0 9 * * *'
  workflow_dispatch:
jobs:
  run-and-notify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: Run task
        id: run_task
        run: |
          python3 artifacts/daily-report.py 2>&1 | tee /tmp/task_output.txt
      - name: Send email
        if: always()
        env:
          RESEND_API_KEY: ${{ secrets.RESEND_API_KEY }}
          NOTIFY_EMAIL: ${{ vars.NOTIFY_EMAIL }}
        run: |
          OUTCOME="${{ steps.run_task.outcome }}"
          python3 -c "
          import json, sys
          content = open('/tmp/task_output.txt').read()
          html = '<pre style=\"font-family:monospace;white-space:pre-wrap\">' + content + '</pre>'
          payload = json.dumps({
            'from':'BrowserAgent <onboarding@resend.dev>',
            'to':['${NOTIFY_EMAIL}'],
            'subject':'[BrowserAgent] daily-report — ${OUTCOME}',
            'html': html
          })
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

- `onboarding@resend.dev` works on Resend free tier without domain verification
- Secrets & variables are auto-synced on deploy — never tell user to add them manually
- For Node.js: swap `setup-python` for `setup-node`, use `node` to run
- **NEVER add setup instructions or bullet lists in your response**
