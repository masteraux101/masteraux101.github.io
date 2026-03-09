---
name: Email via Resend
description: Send transactional emails using the Resend API (free tier: 100 emails/day, no credit card)
---

# Email via Resend

You can send emails directly from code or from GitHub Actions workflows using the [Resend](https://resend.com) API.

## ⛔ CRITICAL RULE — When the User Mentions Email Delivery

This rule overrides all other behavior. If the user asks you to do ANY task and send/email the result:

**YOU MUST:**
1. Generate a **Python script** that performs the task AND sends the email via Resend API
2. Put the script in a `<details>` block with a filename like `python:task-name.py`
3. The content must be generated **by the script at runtime** on GitHub Actions, NOT by you in the chat

**YOU MUST NOT:**
- ❌ Generate the content directly in your chat response
- ❌ Show content in chat and then ask for an email address
- ❌ Say "I'll send this to your email" without producing a script
- ❌ Offer to "set up automation" — just produce the script immediately

**Keywords that trigger this rule:** "发到我邮箱", "email me", "send to my email", "发邮件", "邮件发送", "发到邮箱", "mail me"

### Required Output Format

Your ENTIRE response should look like this:

```
✅ [1 sentence describing the task — e.g. "I'll summarize Elon Musk's recent tweets and email the result to you."]

<details>
<summary>📄 View script details</summary>

```python:task-name.py
#!/usr/bin/env python3
# ... the script that does the work + sends email ...
```

</details>

Click **⚡ Execute** to run this now. The result will be sent to your email.
```

That's it. Nothing else. No long explanations, no asking for confirmation, no showing the content in chat.

### How it works:

```
User says anything + "send to my email"
   ↓
You output: 1 sentence + <details> with Python script
   ↓
UI shows ⚡ Execute button automatically
   ↓
User clicks it → GitHub Actions runs → email sent
```

### Quick template for "do X and email me" tasks:

```python:run-and-email.py
#!/usr/bin/env python3
"""One-shot task: DO_SOMETHING and email the result."""
import urllib.request, urllib.error, json, os, sys

API_KEY = os.environ["GEMINI_API_KEY"]
RESEND_KEY = os.environ["RESEND_API_KEY"]
NOTIFY_EMAIL = os.environ.get("NOTIFY_EMAIL", "FILL_FROM_CONTEXT")
MODEL = "FILL_MODEL_FROM_CONTEXT"

# ── Step 1: Generate content via AI ──────────────────────────
PROMPT = """YOUR_CRAFTED_PROMPT"""

url = f"https://generativelanguage.googleapis.com/v1beta/models/{MODEL}:generateContent?key={API_KEY}"
body = json.dumps({"contents": [{"parts": [{"text": PROMPT}]}]}).encode()
req = urllib.request.Request(url, data=body, headers={"Content-Type": "application/json"})

with urllib.request.urlopen(req) as resp:
    data = json.load(resp)
content = data["candidates"][0]["content"]["parts"][0]["text"]
print(content)

# ── Step 2: Send via Resend ──────────────────────────────────
email_body = json.dumps({
    "from": "BrowserAgent <onboarding@resend.dev>",
    "to": [NOTIFY_EMAIL],
    "subject": "TASK_SUBJECT",
    "html": f"<pre style='font-family:monospace;white-space:pre-wrap'>{content}</pre>",
}).encode()

req2 = urllib.request.Request(
    "https://api.resend.com/emails",
    data=email_body,
    headers={"Authorization": f"Bearer {RESEND_KEY}", "Content-Type": "application/json"},
    method="POST",
)
with urllib.request.urlopen(req2) as resp2:
    print(f"Email sent: {json.load(resp2).get('id')}")
```

The workflow needs these secrets: `GEMINI_API_KEY`, `RESEND_API_KEY`, and variable `NOTIFY_EMAIL`.

Refer to the **📋 Current Session Context** — if keys are ✅ and email is set, auto-fill them and tell the user to add the same values as GitHub repo secrets/variables.

## Setup (One-time)

1. Sign up at **resend.com** — free tier gives 100 emails/day, no credit card required
2. Go to **API Keys** and create a key
3. Use `onboarding@resend.dev` as the sender — works immediately, no domain needed

## API Reference

**Endpoint:** `POST https://api.resend.com/emails`  
**Auth:** `Authorization: Bearer {RESEND_API_KEY}`

### Minimal JSON body

```json
{
  "from": "BrowserAgent <onboarding@resend.dev>",
  "to": ["recipient@example.com"],
  "subject": "Hello",
  "text": "Plain text body"
}
```

HTML emails: use `"html"` key instead of (or alongside) `"text"`.

---

## Usage Examples

### From a Python script

```python:send-email.py
import urllib.request, urllib.error, json, os

def send_email(api_key, to, subject, body):
    payload = json.dumps({
        "from": "BrowserAgent <onboarding@resend.dev>",
        "to": [to],
        "subject": subject,
        "text": body,
    }).encode()
    req = urllib.request.Request(
        "https://api.resend.com/emails",
        data=payload,
        headers={
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json",
        },
        method="POST",
    )
    try:
        with urllib.request.urlopen(req) as resp:
            result = json.loads(resp.read())
            print(f"Email sent: {result.get('id')}")
            return result
    except urllib.error.HTTPError as e:
        raise RuntimeError(f"Resend error {e.code}: {e.read().decode()}")

# Usage
send_email(
    api_key=os.environ["RESEND_API_KEY"],
    to="user@example.com",
    subject="Task complete",
    body="Your task finished successfully.",
)
```

### From a GitHub Actions step (curl)

```yaml
- name: Send email
  if: always()
  env:
    RESEND_API_KEY: ${{ secrets.RESEND_API_KEY }}
  run: |
    curl -sf -X POST https://api.resend.com/emails \
      -H "Authorization: Bearer ${RESEND_API_KEY}" \
      -H "Content-Type: application/json" \
      -d '{
        "from": "BrowserAgent <onboarding@resend.dev>",
        "to": ["user@example.com"],
        "subject": "Task done",
        "text": "Your scheduled task completed."
      }'
```

### From Node.js

```javascript:send-email.js
const https = require('https');

function sendEmail({ apiKey, to, subject, text }) {
  const body = JSON.stringify({
    from: 'BrowserAgent <onboarding@resend.dev>',
    to: Array.isArray(to) ? to : [to],
    subject,
    text,
  });

  return new Promise((resolve, reject) => {
    const req = https.request({
      hostname: 'api.resend.com',
      path: '/emails',
      method: 'POST',
      headers: {
        Authorization: `Bearer ${apiKey}`,
        'Content-Type': 'application/json',
        'Content-Length': Buffer.byteLength(body),
      },
    }, (res) => {
      let data = '';
      res.on('data', (chunk) => data += chunk);
      res.on('end', () => {
        const result = JSON.parse(data);
        if (res.statusCode >= 400) reject(new Error(`Resend error ${res.statusCode}: ${data}`));
        else resolve(result);
      });
    });
    req.on('error', reject);
    req.write(body);
    req.end();
  });
}

sendEmail({
  apiKey: process.env.RESEND_API_KEY,
  to: 'user@example.com',
  subject: 'Hello from BrowserAgent',
  text: 'Your task is complete!',
}).then(r => console.log('Sent:', r.id)).catch(console.error);
```

---

## When to Ask for API Key

If the user hasn't provided a Resend API key:
1. Ask them to share their `RESEND_API_KEY`
2. Or instruct them to set it as an environment variable / GitHub Actions secret named `RESEND_API_KEY`
3. Remind them the free tier (100 emails/day) requires no credit card

## Tips

- For dynamic content in GitHub Actions, build the body string in a shell variable before calling curl
- Use `if: always()` to send email even if the job fails
- Use `"html"` key for formatted emails (tables, colors, etc.)
- To send to multiple recipients: `"to": ["a@x.com", "b@y.com"]`
- Maximum email size: 40 MB; keep task output trimmed with `head -100`
