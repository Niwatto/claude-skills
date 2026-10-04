# claude-skills

Personal [Claude Code](https://claude.com/claude-code) skills.

## Install

Copy (or symlink) the skills you want into `~/.claude/skills/`:

```bash
git clone https://github.com/Niwatto/claude-skills.git
cp -R claude-skills/skills/debug-mantra ~/.claude/skills/
```

## Skills

| Skill | What it does |
|---|---|
| [`debug-mantra`](skills/debug-mantra/SKILL.md) | Four-mantra debugging discipline — reproduce, trace the fail path, falsify the hypothesis, cross-reference every breadcrumb. |
| [`management-talk`](skills/management-talk/SKILL.md) | Rewrite engineer-to-engineer content for engineering-org leadership (VPs, directors, PMs, release managers, execs in an engineering-savvy company) and shape it for the channel it is going to — JIRA comment, Slack post, async standup line, email, or meeting talking-points. |
| [`own-the-change`](skills/own-the-change/SKILL.md) | Guided session that helps the engineer understand a change an AI just made, well enough to explain the system, trace the data, and take the production-readiness questions to their team. |
| [`post-mortem`](skills/post-mortem/SKILL.md) | Write the canonical engineering record of a fixed bug — root cause, mechanism, fix, validation, and how it slipped through. |
| [`scrutinize`](skills/scrutinize/SKILL.md) | Outsider-perspective end-to-end review of a plan, PR, or code change. |
| [`sync-wiki`](skills/sync-wiki/SKILL.md) | Full rebuild of an Obsidian knowledge vault from a codebase. |
