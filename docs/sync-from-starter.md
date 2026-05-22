# Sync Claude Code workflow from AI_Coding_Starter

**Goal:** update workflow definitions (`.claude/`, `.agents/` framework, `.gitignore`, `.mcp.json.example`) in this project, using <https://github.com/mrozmk/AI_Coding_Starter> as the source of truth. **Preserve all project-specific content** — replace workflow files only.

**Assumption:** the project already has `.claude/` and `CLAUDE.md` (even in an older version). If they don't exist — stop and tell me this is a bootstrap, not a sync (use GitHub "Use this template" instead of this prompt).

---

## Step 1: clone the starter to a temporary directory

```bash
rm -rf /tmp/ai-coding-starter-sync
git clone --depth 1 https://github.com/mrozmk/AI_Coding_Starter /tmp/ai-coding-starter-sync
```

Save the starter's commit hash (`git -C /tmp/ai-coding-starter-sync rev-parse --short HEAD`) — you'll use it in the final commit message.

## Step 2: file classification

### Category A — **overwrite entirely from the starter** (workflow definitions, project-agnostic)

- `.claude/commands/*.md` — all slash commands
- `.claude/agents/*.md` — subagent definitions
- `.claude/skills/**` — skill definitions (e.g. `jira/SKILL.md`)
- `.claude/templates/*.md` — templates (e.g. `CLAUDE-template.md`)

### Category B — **merge carefully**, show diff first and ask

- `.claude/settings.json` — the project may have its own permissions. Strategy: take the **union** of `permissions.allow` / `permissions.deny` entries from the starter and the project. Do not remove entries that exist in the project but not in the starter. Show me the diff before saving.
- `.agents/memory/index.md` — `Quick Reference` and `Loader Convention` from the starter, but `When to Read` may have project-specific rows added by `/create-CLAUDE_MD`. Strategy: overwrite with the starter structure, then restore project-specific rows (those not in the starter).
- `.gitignore` — add missing entries from the starter (e.g. `.claude/audit.log`, `.mcp.json`, `.agents/memory/archive/`), **do not remove** project-specific ones.
- `.mcp.json.example` — overwrite if the starter has a newer version.

### Category C — **do not touch** (project content)

- `CLAUDE.md` — project rules. If the starter has a new section structure, report it and propose a patch — but do not overwrite automatically.
- `.agents/memory/architecture.md`, `project-brief.md`, `domain/*.md` — regenerated from the project (`/create-CLAUDE_MD`, `/refresh-brief`).
- `.agents/memory/errors.md`, `decisions.md`, `api.md`, `patterns.md` — append-only, project history.
- `.agents/sources/`, `.agents/specs/`, `.agents/plans/`, `.agents/reference/`, `.agents/wiki/`, `.agents/memory/archive/` — project content.
- `README.md`, `CHANGELOG.md`, `docs/`, any code files — project documentation and code.

## Step 3: dry-run report (BEFORE any change)

Show me:

1. **Category A — diff:**
   - Files **new** (in starter, not in project) → full path list
   - Files **changed** (content differs) → list + brief note "what changed" (1-2 lines per file)
   - Files **identical** → count only, no list
   - Files **in project but not in starter** → list, mark as "project custom command? check if needed" — do NOT delete automatically

2. **Category B — proposed merge:**
   - `settings.json`: show which `allow`/`deny`/`hooks` entries the starter adds, and which project entries remain untouched
   - `index.md`: show which `When to Read` rows are project-specific and will be carried over
   - `.gitignore`: show lines to add

3. **Category C — signals:**
   - If a section name in the starter's `CLAUDE.md` doesn't match the project's CLAUDE.md (e.g. starter added "Loader Convention" to "Automatic Behaviors") — report as a suggestion, don't force

**Wait for my approval. Do not write anything to disk before confirmation.**

## Step 4: apply (after my approval)

In order:

1. Copy Category A (new + changed) — `cp -r` from `/tmp/ai-coding-starter-sync/` to the project
2. Perform Category B merge — first `settings.json`, then `index.md`, then `.gitignore`
3. **Sanity check:**
   - All links in copied commands resolve (`rg -o '\[.*?\]\(.*?\.md.*?\)' .claude/commands/`)
   - `.agents/memory/index.md` contains the `Loader Convention` section
   - Project's `CLAUDE.md` still references active commands (those in `.claude/commands/`)

4. Clean up: `rm -rf /tmp/ai-coding-starter-sync`

## Step 5: final report

Show:

- **Added files:** list
- **Updated files:** list
- **Merged files:** list (settings.json, index.md, .gitignore)
- **Skipped (Category C):** count
- **Action suggestions:** e.g. "regenerate `architecture.md` with `/create-CLAUDE_MD` if format changed", "run `/prime` to validate the new context"

Propose a commit message:

```text
chore(workflow): sync .claude commands and skills from AI_Coding_Starter@<short-hash>
```

## Critical rules

- NEVER remove entries from `.claude/settings.json` that don't exist in the starter — those are project permissions
- NEVER overwrite Category C files
- NEVER delete project slash commands from `.claude/commands/` — report and ask
- NEVER commit automatically — show the message and wait for `/commit`
- Always dry-run before apply
- Clean up `/tmp/ai-coding-starter-sync` at the end
