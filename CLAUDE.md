@AGENTS.md

## Claude Code

Use the packaged skills automatically when the task matches the auto-trigger rules in `AGENTS.md`.

For Figma-to-iOS tasks, use:

1. `./figma-mcp/SKILL.md`
2. `./figma-to-ios-ui/SKILL.md`

For tasks inside the skill trees:

- keep repo-shared instructions in this file or `AGENTS.md`
- keep product guidance inside `./figma-mcp/SKILL.md`, `./figma-to-ios-ui/SKILL.md`, and their references
- keep project-specific adaptation in `./figma-to-ios-ui/subskills/project-ui-pattern-memory/`

If this repository grows substantially, prefer splitting additional Claude-specific rules into `.claude/rules/` instead of making this file large.
