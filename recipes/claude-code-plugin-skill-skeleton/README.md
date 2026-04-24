# claude-code-plugin-skill-skeleton

Minimal Claude Code skill that loads on the right trigger, does one narrow job,
and exits non-zero with an actionable message when something's wrong.

## Problem

A skill with a fuzzy `description:` line triggers on everything or nothing. A
skill that writes to `CLAUDE_PLUGIN_ROOT` corrupts the bundle on re-install.
Both mistakes waste weeks of drift before you notice.

## Snippet

`skills/my-skill/SKILL.md`:

```markdown
---
name: my-skill
description: Use when the user asks to render a Mermaid diagram from a fenced `mermaid` block. Triggers on "render diagram", "mermaid to png", or a message that pastes a mermaid code block and asks for an image.
---

Run the bundled script:

```bash
bash "${CLAUDE_PLUGIN_ROOT}/skills/my-skill/run.sh" "$INPUT"
```

If `mmdc` is not installed, the script prints install instructions and exits 2.
Persistent cache lives at `$CLAUDE_PLUGIN_DATA/my-skill/` — never write to
`CLAUDE_PLUGIN_ROOT`.
```

`skills/my-skill/run.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
cache="${CLAUDE_PLUGIN_DATA:-$HOME/.claude}/my-skill"
mkdir -p "$cache"
command -v mmdc >/dev/null || { echo "install @mermaid-js/mermaid-cli" >&2; exit 2; }
mmdc -i "$1" -o "$cache/out.png"
echo "$cache/out.png"
```

## Why

The `description` field is the only thing the router sees. Keep it specific —
name the user phrases, the file types, the problem shape. `CLAUDE_PLUGIN_ROOT`
is read-only from the runtime's perspective; persistent state belongs in
`CLAUDE_PLUGIN_DATA`, and that split survives plugin re-install.

## When NOT to use

Don't ship a skill for a task the model can already do inline (summarizing
text, writing a regex). Skills are for shelling out to a specific tool or
enforcing a repeatable workflow — not for prompt templates.

## Reference

- https://docs.claude.com/en/docs/claude-code/plugins
