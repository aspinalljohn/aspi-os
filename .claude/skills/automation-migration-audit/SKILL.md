---
name: automation-migration-audit
description: Scan a project or repo for scripts, cron entries, and configs that invoke a deprecated CLI or tool, classify each call site by blast radius (critical / nice-to-have / dead), and output a prioritized migration plan with the replacement command and a rollback note per item. Provider-agnostic and copy-paste runnable. The worked example is the Gemini CLI → Antigravity CLI sunset (June 18, 2026), but the audit takes any old-command/new-command pair. Use when an operator has wired a CLI into automations and that tool is being deprecated, renamed, or sunset, and they need to know exactly what breaks and what to port first before the cutover date.
when_to_use: When a CLI or tool an operator has wired into cron jobs, content pipelines, reporting scripts, or store automations is being deprecated or sunset, and they need an inventory of every call site plus a prioritized, rollback-safe migration plan before the cutover. Worked example is Gemini CLI → Antigravity CLI (ends June 18, 2026 for consumer tiers), but it generalizes to any deprecated-command migration.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Write
---

# Automation migration audit

Find every place a dying tool is invoked, rank those call sites by what breaks if they go silent, and emit a migration plan you can actually work through before the cutover date. Provider-agnostic. The default worked example is `gemini` → `antigravity`, but pass any old/new command pair.

## Inputs (ask if not given)

- `OLD_CMD` → the deprecated command/binary to hunt for (default `gemini`).
- `NEW_CMD` → the replacement command (default `antigravity`).
- `ROOT` → directory to scan (default current working directory).
- `CUTOVER` → the date the old tool stops working (default `2026-06-18`).

No private or client data, no machine-specific absolute paths. The audit reads the project it is pointed at and writes one report.

## What it does

1. **Inventory** → greps the project tree, plus the user's crontab and common scheduler/config locations, for any line that invokes `OLD_CMD`. Records file, line number, and the surrounding command.
2. **Classify by blast radius** → buckets each call site:
   - **critical** → runs on a schedule (cron / systemd timer / CI), or feeds a customer-facing output, or has no human in the loop. Silent breakage costs money or trust.
   - **nice-to-have** → a helper script or a manual one-off. Breaks quietly, you notice, you re-run by hand.
   - **dead** → commented out, in an archive/old/backup path, or in a file untouched for a long time. Probably safe to delete, not migrate.
3. **Plan** → for each live item, emit the replacement command (`OLD_CMD` swapped for `NEW_CMD`) and a one-line rollback note, ordered critical → nice-to-have, with dead items listed separately for deletion.

## Run it

The inventory pass is a single shell block. Drop it in, set the four variables, run.

```bash
#!/usr/bin/env bash
# automation-migration-audit → inventory pass
set -euo pipefail

OLD_CMD="${OLD_CMD:-gemini}"
NEW_CMD="${NEW_CMD:-antigravity}"
ROOT="${ROOT:-$(pwd)}"
CUTOVER="${CUTOVER:-2026-06-18}"
OUT="migration-audit-report.md"

echo "# Automation migration audit: $OLD_CMD -> $NEW_CMD" > "$OUT"
echo "" >> "$OUT"
echo "Cutover: $CUTOVER  |  Scanned: $ROOT  |  Generated: $(date +%Y-%m-%d)" >> "$OUT"
echo "" >> "$OUT"

# 1. Project files that invoke OLD_CMD (skip vendored / VCS dirs)
echo "## Project call sites" >> "$OUT"
echo "" >> "$OUT"
grep -rInE "(^|[^a-zA-Z0-9_-])${OLD_CMD}([^a-zA-Z0-9_-]|$)" "$ROOT" \
  --include="*.sh" --include="*.bash" --include="*.zsh" \
  --include="*.py" --include="*.js" --include="*.ts" \
  --include="*.yml" --include="*.yaml" --include="*.toml" \
  --include="*.json" --include="*.mjs" --include="Makefile" \
  --exclude-dir=.git --exclude-dir=node_modules \
  --exclude-dir=.venv --exclude-dir=dist \
  2>/dev/null | while IFS= read -r hit; do
    f="${hit%%:*}"; rest="${hit#*:}"; ln="${rest%%:*}"; code="${rest#*:}"
    # classify
    bucket="nice-to-have"
    case "$f" in
      */archive/*|*/old/*|*/backup/*|*.bak) bucket="dead" ;;
    esac
    case "$code" in
      \#*|//*|*"# "*"${OLD_CMD}"*) : ;;
    esac
    if printf '%s' "$code" | grep -qE '^\s*(#|//)'; then bucket="dead"; fi
    if printf '%s' "$f" | grep -qiE '(cron|ci|\.github/workflows|deploy|pipeline|report)'; then bucket="critical"; fi
    printf -- "- [%s] \`%s\`:%s\n      old: %s\n      new: %s\n      rollback: keep the original line commented for one cycle; re-enable if %s fails\n" \
      "$bucket" "$f" "$ln" "$(printf '%s' "$code" | sed 's/^[[:space:]]*//')" \
      "$(printf '%s' "$code" | sed "s/\b${OLD_CMD}\b/${NEW_CMD}/g" | sed 's/^[[:space:]]*//')" \
      "$NEW_CMD" >> "$OUT"
  done
echo "" >> "$OUT"

# 2. Scheduled jobs (highest blast radius → these break silently)
echo "## Scheduled jobs (treat as critical)" >> "$OUT"
echo "" >> "$OUT"
( crontab -l 2>/dev/null || true ) | grep -nE "(^|[^a-zA-Z0-9_-])${OLD_CMD}([^a-zA-Z0-9_-]|$)" \
  | while IFS= read -r cj; do
      echo "- [critical] crontab: \`${cj}\`" >> "$OUT"
      echo "      new: $(printf '%s' "$cj" | sed "s/\b${OLD_CMD}\b/${NEW_CMD}/g")" >> "$OUT"
      echo "      rollback: comment the cron line, keep the old job spec saved until $NEW_CMD runs clean twice" >> "$OUT"
    done || true
echo "" >> "$OUT"

echo "## Next steps" >> "$OUT"
echo "1. Port every [critical] item first. Test before $CUTOVER." >> "$OUT"
echo "2. Port [nice-to-have] only if you still use it." >> "$OUT"
echo "3. Delete [dead] items → do not migrate them." >> "$OUT"

echo "Report written to $OUT"
```

## Output

A single `migration-audit-report.md` with three sections: project call sites (each tagged critical / nice-to-have / dead, with old line, new line, and rollback note), scheduled jobs (always critical), and a next-steps order. Work it top to bottom. Critical before the cutover, nice-to-have if you still use it, dead gets deleted.

## Make it yours

- Different tool? Set `OLD_CMD` and `NEW_CMD` → works for any rename or sunset (`vercel` → `vc`, a renamed internal binary, a deprecated API wrapper).
- Add file types your stack uses to the `--include` list (`*.rb`, `*.go`, `*.php`).
- Tighten the **critical** rule to your reality: add your own path patterns (a `customer/` or `billing/` dir) to the classifier.
- Add a `systemctl list-timers` pass if you run systemd timers instead of cron.

## Gotchas

- A bare grep for the command name catches substrings (`geminispace`). The word-boundary pattern here avoids most, but eyeball the report before trusting it.
- Cron only shows the **current user's** jobs. Root jobs and other users' crontabs need a separate pass with the right permissions.
- The **dead** bucket is a guess based on path and comment state. Confirm before deleting.
- Replacement commands rarely have 1:1 flag parity. The `new:` line is a starting point, not a guarantee → test each ported command.
