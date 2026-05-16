---
name: memory-vault
description: |
  Persistent, atomic personal memory vault. Always active. Reads user preferences,
  project context, and decisions from nuclear files at session start. Silently saves
  new context after each response when confidence is high. Triggers onboarding on
  first run to collect vault path, identity, and language preference.
author: leonardo-luize
version: 1.0.0
date: 2026-05-16
---

# Memory Vault

**This skill is always active. Every session. No trigger required.**

## Vault Structure

```
<vault>/
  core/
    INDEX.md          # master index, always loaded every session
    about-me.md       # name, role, expertise, goals
    preferences.md    # language, tools, communication style
  projects/
    <name>.md         # single file when context fits in < 10 lines
    <name>/           # subfolder after promotion (see Project Promotion)
      INDEX.md
      stack.md
      decisions.md
      status.md
      context.md
  people/
    <name>.md
  decisions/
    <decision>.md
```

## Session Start: Bootstrap

First thing every session, before any other work:

1. Read `~/.claude/CLAUDE.md`
2. Look for a section matching this exact format:
   ```
   ## Memory Vault
   Path: <some-path>
   ```
3. If **no section found** → run [Onboarding](#onboarding)
4. If **section found** → extract the path value, then run [Session Read](#session-read)

## Onboarding

Ask these 3 questions one at a time. Wait for the user's answer before asking the next.

**Question 1:**
> "Where should I store your memory vault? I'll create the directory if it doesn't exist. (default: `~/memory-vault`)"

**Question 2:**
> "What's your name and role? (e.g., 'Leo, full-stack developer')"

**Question 3:**
> "What language do you prefer I communicate in? (default: English)"

After collecting all answers:

1. Expand the path (resolve `~` to the user's home directory)
2. Create the vault directory structure:
   ```
   <vault>/core/
   <vault>/projects/
   <vault>/people/
   <vault>/decisions/
   ```
3. Create `<vault>/core/about-me.md`:
   ```yaml
   ---
   created: <today-YYYY-MM-DD>
   updated: <today-YYYY-MM-DD>
   tags: [identity]
   summary: User name and role
   ---

   Name and role: <answer to Q2>
   ```
4. Create `<vault>/core/preferences.md`:
   ```yaml
   ---
   created: <today-YYYY-MM-DD>
   updated: <today-YYYY-MM-DD>
   tags: [preferences]
   summary: Communication and style preferences
   ---

   Language: <answer to Q3>
   ```
5. Create `<vault>/core/INDEX.md`:
   ```markdown
   # Vault Index

   ## Projects
   *(none yet)*

   ## People
   *(none yet)*

   ## Decisions
   *(none yet)*
   ```
6. Append to `~/.claude/CLAUDE.md` (create the file if it doesn't exist):
   ```

   ## Memory Vault
   Path: <expanded-vault-path>
   ```
7. Confirm to the user: "Memory vault created at `<expanded-path>`. I'll read and update it silently from now on."

## Session Read

After bootstrap confirms a vault path exists, read these three files:

- `<vault>/core/INDEX.md`
- `<vault>/core/about-me.md`
- `<vault>/core/preferences.md`

Apply the context silently. Do not announce that you read the vault. Do not summarize what you found. Just use the information to inform your responses for this session.

If any of the three core files is missing, recreate it with empty content and correct frontmatter before continuing.

## File Format

Every vault file except `INDEX.md` files uses YAML frontmatter + a short prose body:

```yaml
---
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
summary: Five-to-eight word description
---

Body in plain prose. Maximum 10 lines.
One topic per file. Not a collection of unrelated facts.
```

**Rules:**
- `summary` must be ≤ 8 words — it is copied verbatim into `INDEX.md`
- Body must be ≤ 10 lines — if a project file exceeds this, trigger [Project Promotion](#project-promotion)
- One file = one piece of information

## INDEX.md Format

The master index lives at `<vault>/core/INDEX.md`. Project subfolders each have their own `INDEX.md` in the same format.

```markdown
# Vault Index

## Projects
- projects/app-name.md | client web app | react, vite
- projects/complex-app/ | backend API | node, postgres

## People
- people/john-doe.md | client at Acme Corp | client

## Decisions
- decisions/chose-react.md | chose React over Vue | frontend
```

**Line format:** `- <relative-path> | <summary from frontmatter> | <tags comma-separated>`

When a category has no entries yet, write `*(none yet)*` instead of an empty section.

## Post-Response Analysis

After every response, silently decide whether anything is worth saving.

**Save when:**
- The user stated a preference explicitly: "I prefer X", "don't do Y", "I use Z"
- The user described a project, a person they work with, or a significant decision
- The user corrected the same behavior twice in one session (inferred preference)

**Do not save when:**
- The information is task-specific and won't apply in future sessions
- Confidence is below ~80% that this information generalizes beyond this conversation
- An existing vault file already accurately represents this information

**Process when saving:**
1. Determine the correct category folder (`projects/`, `people/`, `decisions/`)
2. Check if a relevant file already exists
3. If yes → apply [Update Rules](#update-rules)
4. If no → create a new nuclear file with correct frontmatter
5. Update `core/INDEX.md` to include or refresh the entry
6. Do all of this silently — no announcements

## Update Rules

When updating an existing vault file:

1. Read the current file content
2. Edit only the fields that changed — do not rewrite unchanged content
3. Always update the `updated:` date in frontmatter
4. If the `summary:` field changed, regenerate the corresponding line in `core/INDEX.md`

Never overwrite a file wholesale when only one field changed. Surgical edits only.

## Project Promotion

When a project file at `projects/<name>.md` would exceed 10 lines of body content:

1. Create directory `projects/<name>/`
2. Distribute context into atomic files inside the new directory. Use these names when applicable:
   - `stack.md` — technologies, frameworks, languages, infrastructure
   - `decisions.md` — architectural or product decisions made
   - `status.md` — current state, what is in progress, next steps
   - `context.md` — background, goals, stakeholders, constraints
3. Create `projects/<name>/INDEX.md` using the standard grouped format
4. Delete `projects/<name>.md`
5. Update `core/INDEX.md`: replace the entry `projects/<name>.md` with `projects/<name>/`

After promotion, `projects/<name>/INDEX.md` is the entry point for that project. Read it before reading individual files within the subfolder.

## Silence Rule

The vault is invisible by default:

- Do not announce vault reads at session start
- Do not announce when you save something to the vault
- Do not mention the vault in responses unless the user asks about it
- Do not confirm saves with phrases like "I've saved that to your vault"

**Exception:** when the user explicitly asks — "what do you know about me?", "check the vault for X", "did you save that?" — answer directly and accurately.
