# figma-to-ios-ui

This repository contains two skills that are designed to work together:

- `figma-mcp`
- `figma-to-ios-ui`

Use them as a pair when you want the best Figma-to-iOS results.

## What each skill does

### `figma-mcp`

This is the extraction skill.

Use it to:

- read Figma metadata
- collect tokens
- get layout and styling context
- capture screenshots
- build a reliable `design_spec`

It answers the question:

> What does the design actually say?

### `figma-to-ios-ui`

This is the iOS implementation skill.

Use it to:

- choose the correct iOS lane
- translate the design into UIKit, XIB, or SwiftUI
- adapt to the target repo's design system
- learn project-specific UI patterns from the live codebase
- validate the final result after implementation

It answers the question:

> Given the design evidence, what is the right iOS implementation here?

## Why they should be used together

Using only `figma-mcp` is not enough for iOS implementation.
Using only `figma-to-ios-ui` is risky if the design evidence was not extracted correctly.

The best flow is:

1. `figma-mcp` extracts the design evidence.
2. `figma-to-ios-ui` turns that evidence into the right iOS solution.
3. `figma-to-ios-ui` validates the final UI against the design after implementation.

From prior rollout experience, this split works better than one oversized skill because:

- extraction and translation stay cleanly separated
- the iOS skill can stay focused on UIKit, XIB, SwiftUI, and validation
- the final result is more accurate and easier to reason about

## Install both skills

Important:

- install the whole skill directories
- do not copy only `SKILL.md`
- keep support files such as `agents/`, `scripts/`, `templates/`, and `references/`

That full-directory rule matters. It is the safest pattern from prior skill migration work.

### Codex install

Copy both skill folders into `~/.codex/skills/`:

```bash
mkdir -p ~/.codex/skills

rsync -a --delete "/absolute/path/to/figma-to-ios-ui/figma-mcp/" \
  ~/.codex/skills/figma-mcp/

rsync -a --delete "/absolute/path/to/figma-to-ios-ui/figma-to-ios-ui/" \
  ~/.codex/skills/figma-to-ios-ui/
```

For this repo on your machine, the direct commands are:

```bash
mkdir -p ~/.codex/skills

rsync -a --delete "/Users/amrmohamad/Developer/iOS Skills/UI Skills/figma-to-ios-ui/figma-mcp/" \
  ~/.codex/skills/figma-mcp/

rsync -a --delete "/Users/amrmohamad/Developer/iOS Skills/UI Skills/figma-to-ios-ui/figma-to-ios-ui/" \
  ~/.codex/skills/figma-to-ios-ui/
```

### Verify the install

Verify both skill trees, not just the top-level file:

```bash
find ~/.codex/skills/figma-mcp -maxdepth 3 -type f | sort
find ~/.codex/skills/figma-to-ios-ui -maxdepth 4 -type f | sort
```

You should see:

- `SKILL.md`
- `agents/openai.yaml`
- `references/`
- `scripts/`
- `templates/`
- `subskills/` for `figma-to-ios-ui`

## How to use them together

Default flow:

1. Use `figma-mcp` first.
2. Get:
   - hierarchy
   - tokens
   - screenshot
   - `design_spec`
3. Then use `figma-to-ios-ui`.
4. Let it choose the correct lane:
   - UIKit/XIB
   - SwiftUI
   - hybrid adaptation
5. If the target repo has custom UI patterns, let `figma-to-ios-ui` use its companion sub-skill:
   - `project-ui-pattern-memory`
6. After implementation, run the post-implementation validation loop.

In short:

- `figma-mcp` finds the truth in the design
- `figma-to-ios-ui` finds the right iOS implementation

## Repository layout

- [AGENTS.md](./AGENTS.md)
  - Repo-level cross-agent instructions and auto-trigger rules.
- [CLAUDE.md](./CLAUDE.md)
  - Claude Code entrypoint that imports `AGENTS.md`.
- [figma-mcp/](./figma-mcp)
  - Figma extraction skill.
- [figma-to-ios-ui/](./figma-to-ios-ui)
  - iOS implementation skill.

## Best place to start

If you are new to this repository:

1. Read [AGENTS.md](./AGENTS.md)
2. Read [figma-mcp/SKILL.md](./figma-mcp/SKILL.md)
3. Read [figma-to-ios-ui/SKILL.md](./figma-to-ios-ui/SKILL.md)
4. For SwiftUI or hybrid repos, read:
   - [project-ui-pattern-memory](./figma-to-ios-ui/subskills/project-ui-pattern-memory/SKILL.md)
5. Before closing implementation work, read:
   - [post-implementation-validation-and-learning](./figma-to-ios-ui/references/post-implementation-validation-and-learning.md)

## One-line summary

This repo gives you one skill to extract trustworthy Figma evidence and one skill to turn that evidence into the right iOS UI implementation and validation flow.
