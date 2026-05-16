Personal AI agent skills for Claude Code, GitHub Copilot, Codex, and other agents.

---

## Install all skills

```sh
npx skills@latest add leonardo-luize/skills
```

## Install a specific skill

```sh
npx skills@latest add leonardo-luize/skills --skill memory-vault
```

---

## Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [memory-vault](#memory-vault) | Persistent atomic memory vault — always active, zero-token overhead | `--skill memory-vault` |

---

### memory-vault

> Persistent, atomic personal memory vault for AI agents.

Always active — no trigger needed. On first run, asks 3 questions to set up your vault (path, identity, language). Every session, reads your preferences and project context silently. After each response, detects and saves new context in nuclear YAML files optimized for minimal token usage.

**Features:**
- Atomic files: one file = one piece of information
- `core/` loaded every session (`INDEX.md`, `about-me.md`, `preferences.md`)
- Domain folders: `projects/`, `people/`, `decisions/`
- Auto-promotes projects from single file to indexed subfolder when they grow
- Surgical updates — only changed fields are rewritten

```sh
npx skills@latest add leonardo-luize/skills --skill memory-vault
```

→ Full spec: [`skills/memory-vault/SKILL.md`](skills/memory-vault/SKILL.md)

---

## Compatibility

Works with any agent supported by the [skills CLI](https://www.npmjs.com/package/skills): Claude Code, GitHub Copilot, Codex, Cursor, OpenCode, and [more](https://www.npmjs.com/package/skills#supported-agents).

## License

MIT
