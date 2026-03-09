---
name: Summary & Digest
description: Summarize long texts, articles, documents, or conversations into concise, structured digests
---

# Summary & Digest Skill

**When to use:** "summarize", "TLDR", "digest", "总结", "摘要", "概括"

## Rules
1. Always preserve the key points and conclusions
2. Match the summary length to the content length (10-20% of original)
3. Use bullet points for multi-topic content
4. Maintain the original tone and intent
5. Flag any important caveats or nuances that might be lost in summarization

## Output Formats

### Quick Summary (for short content)
```
**TL;DR:** [1-2 sentence summary]

**Key Points:**
- Point 1
- Point 2
- Point 3
```

### Structured Digest (for long content)
```
## 📋 Summary: [Title]

**Overview:** [2-3 sentence high-level summary]

### Key Findings
1. [Finding with brief explanation]
2. [Finding with brief explanation]

### Notable Details
- [Detail 1]
- [Detail 2]

### Action Items / Takeaways
- [ ] [Actionable item]
- [ ] [Actionable item]

**Source:** [Original link/reference if provided]
```

### Comparison Summary (for multiple inputs)
```
| Aspect | Source A | Source B |
|--------|---------|---------|
| Main point | ... | ... |
| Stance | ... | ... |
| Evidence | ... | ... |
```

## Special Cases
- **Meeting notes:** Extract decisions, action items, and owners
- **Research papers:** Focus on methodology, findings, and limitations
- **News articles:** Lead with the 5W1H (who, what, when, where, why, how)
- **Code/Technical docs:** Summarize functionality, API surface, and gotchas
