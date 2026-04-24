# chrome-multi-profile-bash

Pick the right Chrome profile without asking the user, using Chrome's own
`Local State` catalog. Works with bash 3.2 (macOS default) — no `declare -A`,
no `mapfile`.

## Problem

You have Personal, Work, and School Chrome profiles open at the same time.
Automating "go to Gmail in the Work profile" means matching a window to a
profile directory, and `osascript` only gives you window titles.

## Snippet

```bash
#!/usr/bin/env bash
set -euo pipefail

local_state="$HOME/Library/Application Support/Google/Chrome/Local State"

# Emit "dir<TAB>email" per profile (bash 3.2 safe — no associative arrays).
profile_rows() {
  jq -r '
    .profile.info_cache
    | to_entries[]
    | [.key, (.value.user_name // "")]
    | @tsv
  ' "$local_state"
}

# Pick profile dir whose email matches the query.
resolve_profile() {
  local query="$1" dir email
  while IFS=$'\t' read -r dir email; do
    [[ "$email" == *"$query"* ]] && { printf '%s\n' "$dir"; return 0; }
  done < <(profile_rows)
  return 1
}

profile="$(resolve_profile "work@example.com")"
open -na "Google Chrome" --args --profile-directory="$profile" "https://mail.google.com/"
```

## Why

`Local State` is the authoritative profile catalog — Chrome itself reads it on
startup, so it reflects every profile rename and re-order without separate
tracking. The `--profile-directory` flag is documented and stable; parsing the
window title to guess the profile is not.

## When NOT to use

Don't rely on `Local State` for bulk operations that hold profiles open across
macOS versions you don't control — schema has shifted between major Chrome
releases. Parse with `jq`, version-check, and fail loudly on unexpected shape.

## Reference

- https://chromium.googlesource.com/chromium/src/+/main/chrome/common/pref_names.cc
