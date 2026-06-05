# Risk Assessment Skill

When invoked, analyze the most recent changes made in the current session and predict what could go wrong.

## Step 1: Identify Recent Changes

From conversation context, identify:
- What files were modified
- What behavior changed
- What was added, removed, or replaced

If an argument is provided (e.g., `/risk the cooldown approach`), focus the analysis on that specific aspect.

## Step 2: Analyze Failure Modes

Think through 3-5 concrete failure scenarios. For each, consider:
- **Race conditions** — async timing, stale state, interleaved operations
- **Edge cases** — empty states, boundary values, first/last items, rapid repeated input
- **Regression** — did removing old code break something that depended on it?
- **Assumptions** — what are we assuming about server behavior, timing, user behavior?
- **Silent failures** — things that won't crash but will produce wrong results

## Step 3: Present as a Testing Checklist

Format the output as a short, actionable list the user can verify by hand during live testing. No code, no deep technical detail — just what to DO and what to LOOK FOR.

### Format

```
## Risk Assessment

### What changed
[1-2 sentence summary of the change]

### What to watch for

1. **[Short label]** — [What to do] → [What bad behavior looks like if this fails]

2. **[Short label]** — [What to do] → [What bad behavior looks like if this fails]

3. ...

### Highest risk
[Which one you think is most likely to actually happen, and why — 1 sentence]
```

Keep it to 3-5 items max. Be specific and behavioral — "the highlight jumps backward" not "race condition in polling logic."
