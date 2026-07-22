# pere's skills

Personal agent skills for [opencode](https://opencode.ai), Claude Code, and any agent compatible with the [Agent Skills](https://agentskills.io) spec. Discoverable via the [`skills` CLI](https://skills.sh).

## Skills

| Skill | Description |
| --- | --- |
| [`actualbudget`](skills/actualbudget/) | Access and analyze a self-hosted ActualBudget instance via the official `@actual-app/api` (headless, with ActualQL for aggregations). |
| [`shop-cli`](skills/shop-cli/) | Reverse-engineer an e-commerce site/app and ship an unofficial, agent-friendly Go CLI for it, end-to-end. |
| [`traductor-catala`](skills/traductor-catala/) | Translate software into Catalan following Softcatalà standards, style guide, ISO norms, and terminology resources. |

## Install

Install a single skill (recommended):

```bash
npx skills add pmontp19/skills --skill actualbudget
npx skills add pmontp19/skills --skill shop-cli -a opencode -g
```

Install all:

```bash
npx skills add pmontp19/skills
```

Or, in opencode, register this repo as a skills directory (scanned recursively for `SKILL.md`):

```json
{ "skills": { "paths": ["~/Developer/skills"] } }
```

## Layout

```
skills/<name>/SKILL.md   # one folder per skill
```

## License

MIT
