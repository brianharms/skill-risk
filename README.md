# /risk

> ## ⚠️ Before you start
>
> Installing a Claude Code skill is just copying one folder into `~/.claude/skills/`. Your AI agent can do all of it. **The only human step:** after install, **restart Claude Code** so it picks up the new skill, then type `/risk`.


**Risk assessment for a proposed change before you make it.**

## What it does

`/risk` is a Claude Code skill that pauses before (or right after) a change and asks the only question that matters: *what could actually go wrong?* It reads the recent edits from the current session, reasons through 3–5 concrete failure modes — race conditions, edge cases, regressions, hidden assumptions, silent failures — and hands you back a short, behavioral testing checklist you can run by hand. No code dumps, no abstract theory. Just "do this, and if it breaks you'll see *that*," plus a one-line call on which risk is most likely to bite.

It's the skill you reach for after Claude says "done" and before you ship.

## Install

`/risk` is a single self-contained skill — one `SKILL.md`, no scripts, no build step. Drop it into your Claude Code skills directory:

```bash
# Clone the repo (or download the ZIP from GitHub)
git clone https://github.com/brianharms/skill-risk.git

# Copy the skill into your Claude Code skills directory
mkdir -p ~/.claude/skills/risk
cp skill-risk/SKILL.md ~/.claude/skills/risk/SKILL.md
```

When you're done, `~/.claude/skills/risk/SKILL.md` should exist. That's the whole install.

Then, in any Claude Code session, invoke it by typing:

```
/risk
```

## Usage

Make a change, then ask for the risk profile:

```
/risk
```

Claude looks at what just changed in the session and replies with something like:

```
## Risk Assessment

### What changed
Added a 300ms debounce to the search input so it only fires after the user stops typing.

### What to watch for

1. **Stale results** — Type fast, then delete back to a shorter query → an older,
   longer-query result list flashes in before the correct one settles.

2. **First keystroke lag** — Type a single character and wait → results should
   appear; if nothing happens, the debounce is swallowing the trailing call.

3. **Rapid clear** — Type, then immediately hit ⌘-A + Delete → the list should
   empty; watch for a ghost result set that never clears.

### Highest risk
#1 — the debounce reorders in-flight requests, so out-of-order responses are the
most likely thing to actually surface.
```

You can also scope the analysis to one aspect of the change by passing an argument:

```
/risk the cooldown approach
```

…and Claude focuses its failure-mode hunt on that specific piece instead of the whole diff.

## Requirements / Dependencies

- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** — this is a Claude Code skill and runs inside the CLI.
- **No other dependencies.** `/risk` is OS-agnostic, has no companion scripts, no external tools, and no sibling skills to install. It works entirely from the model's reasoning over your session context.

## For AI coding agents

This section is for an agent working *on* the `/risk` skill itself.

**Repo layout** (everything ships from the repo root):

```
skill-risk/
├── SKILL.md        # the entire skill — the contract Claude reads when /risk is invoked
├── LICENSE         # MIT
├── .gitignore      # ignores OS cruft + runtime/state artifacts
└── README.md       # this file
```

**What `SKILL.md` is.** It is the skill. When a user types `/risk`, Claude Code loads `SKILL.md` and follows it as instructions — there is no code, no entrypoint, no runtime. The file *is* the behavior. Treat it as a prompt contract: every line is something Claude will read and act on, so edits to wording change behavior directly.

**The contract has a fixed three-step shape** that downstream behavior depends on — don't reorder or collapse it:
1. **Identify Recent Changes** — pull the modified files / changed behavior from session context, and honor an optional argument (`/risk <aspect>`) by narrowing the analysis to that aspect.
2. **Analyze Failure Modes** — 3–5 scenarios across the five named lenses: race conditions, edge cases, regression, assumptions, silent failures. Keep these five categories; they're what makes the output non-generic.
3. **Present as a Testing Checklist** — output the exact `## Risk Assessment` template (What changed → What to watch for → Highest risk).

**Invariants — do not break these:**
- **Output stays behavioral, not technical.** The skill's whole value is "the highlight jumps backward," not "race condition in polling logic." If you edit the format, preserve the *What to do → What bad behavior looks like* arrow structure and keep the prohibition on code/deep technical detail.
- **Keep it to 3–5 items max.** The brevity is a feature; an exhaustive list defeats the purpose.
- **Preserve the `## Risk Assessment` heading and its three sub-sections.** Anything reading or screenshotting the output expects that structure.
- **Keep the `### Highest risk` single-sentence verdict.** It's the part the user acts on first.
- **No new dependencies.** This skill must stay zero-install beyond the copy step — no scripts, no external tools, no sibling skills. If you find yourself adding a `scripts/` dir, you're changing what this skill is.

**How to test a change.** Install it the same way a user would, then exercise it live:

```bash
mkdir -p ~/.claude/skills/risk
cp SKILL.md ~/.claude/skills/risk/SKILL.md
```

Open a Claude Code session, make a small real change to some project (or describe one), and run `/risk` and `/risk <some aspect>`. Verify: it grabs the right change, produces 3–5 behavioral items, scopes correctly when given an argument, and ends with a single Highest-risk call in the exact template.

## License

MIT © 2026 Brian Harms / Ritual Industries — [ritual.industries](https://ritual.industries)
