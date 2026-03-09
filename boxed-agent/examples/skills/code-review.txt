---
name: Code Review
description: Systematic code review with actionable feedback
---

# Code Review Skill

When the user shares code for review, provide systematic, actionable feedback.

## Review Checklist

For every code snippet, evaluate:

1. **Correctness** — Does it do what it's supposed to do? Edge cases?
2. **Security** — XSS, injection, exposure of secrets, unsafe deserialization?
3. **Performance** — O(n²) when O(n) is possible? Unnecessary re-renders? Memory leaks?
4. **Readability** — Clear naming? Reasonable function length? Self-documenting?
5. **Maintainability** — Easy to modify? Proper separation of concerns?
6. **Error Handling** — Are errors caught and handled gracefully?

## Output Format

Structure feedback as:

### 🔴 Critical (must fix)
- Issue description + suggested fix with code

### 🟡 Improvement (should fix)
- Issue description + suggested fix with code

### 🟢 Good practices noticed
- What's done well (brief)

### 💡 Suggestions (optional)
- Nice-to-have improvements

## Rules

- Always provide a **corrected code snippet** for critical issues.
- Be specific — cite line numbers or function names.
- Don't nitpick style unless it affects readability.
- If the code is good, say so briefly. Don't invent problems.
- When suggesting improvements, explain **why**, not just **what**.
