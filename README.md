# figma-to-ios-ui

`figma-to-ios-ui` is a public AI skill package for turning Figma design evidence into real iOS UI work.

It is built for agents and developers who need to:

- implement iOS UI from Figma
- audit existing UIKit, XIB, or SwiftUI screens against design
- repair visual drift without breaking the host codebase
- validate the final UI after implementation
- learn the target repo's UI patterns instead of forcing generic ones

## What this skill is for

Use this repository when the task is:

- Figma to UIKit
- Figma to XIB
- Figma to SwiftUI
- mixed UIKit and SwiftUI feature work
- shared UIKit component hardening
- post-implementation UI validation against the design

This skill is not the Figma extraction skill itself.

It expects `figma-mcp` to handle:

- target resolution
- MCP calls
- metadata and tokens
- screenshot capture
- `design_spec`

Then `figma-to-ios-ui` takes over and handles:

- lane selection
- iOS adaptation
- project-pattern learning
- implementation guidance
- post-implementation validation

## How to use it

Default flow:

1. Run `figma-mcp` first and get:
   - hierarchy
   - tokens
   - screenshot
   - `design_spec`
2. Load `figma-to-ios-ui`.
3. Choose the correct lane:
   - UIKit/XIB
   - SwiftUI
4. If the repo has custom UI patterns, use the companion sub-skill to learn them from the live codebase.
5. Implement or repair the UI.
6. Run the post-implementation validation loop to make sure the result still matches the design.

## What makes it different

This skill is designed to be:

- generic, not tied to one app
- evidence-first, not guess-based
- safe for hybrid UIKit and SwiftUI codebases
- design-system aware
- token-usage efficient

It avoids hard-coded project overlays in the main skill.
Instead, it learns project-specific UI patterns at task time using a small companion sub-skill.

## Repository layout

- [AGENTS.md](./AGENTS.md)
  - Repo-level instructions for AI agents.
  - Tells agents when to auto-load the skill without waiting for the developer to mention it.
- [CLAUDE.md](./CLAUDE.md)
  - Claude Code entrypoint.
  - Imports `AGENTS.md` so Claude and other agents stay aligned.
- [figma-to-ios-ui/SKILL.md](./figma-to-ios-ui/SKILL.md)
  - Main skill entrypoint.
- [figma-to-ios-ui/references/](./figma-to-ios-ui/references)
  - Lane guidance, validation rules, and supporting docs.
- [figma-to-ios-ui/scripts/](./figma-to-ios-ui/scripts)
  - Audit and scaffold helpers.
- [figma-to-ios-ui/templates/](./figma-to-ios-ui/templates)
  - Templates for design specs and validation briefs.
- [figma-to-ios-ui/subskills/project-ui-pattern-memory/](./figma-to-ios-ui/subskills/project-ui-pattern-memory)
  - Companion sub-skill that learns a repo's UI patterns and design system from the codebase.

## Main capabilities

### UIKit and XIB

- generation
- audit and repair
- runtime-aware troubleshooting
- shared component hardening
- token verification

### SwiftUI

- generation
- audit and repair
- design-system mapping
- state and presentation ownership checks
- accessibility, motion, and performance guidance
- project-specific pattern learning

### Validation

- post-implementation checks against `design_spec`
- screenshot-based parity checks
- smallest-scope validation modes
- incremental learning deltas instead of expensive full relearning

## Best place to start

If you are using this repo for the first time, read these in order:

1. [AGENTS.md](./AGENTS.md)
2. [figma-to-ios-ui/SKILL.md](./figma-to-ios-ui/SKILL.md)
3. The lane you need:
   - [UIKit/XIB lane](./figma-to-ios-ui/references/ios-uikit-xib-lane.md)
   - [SwiftUI lane](./figma-to-ios-ui/references/swiftui-lane-overview.md)
4. If the repo is pattern-heavy or hybrid:
   - [project-ui-pattern-memory](./figma-to-ios-ui/subskills/project-ui-pattern-memory/SKILL.md)
5. Before closing the task:
   - [post-implementation-validation-and-learning](./figma-to-ios-ui/references/post-implementation-validation-and-learning.md)

## In one sentence

This repository helps an AI agent take Figma design evidence, adapt it to a real iOS codebase, implement the right UIKit/XIB or SwiftUI solution, and then prove the result still matches the design.
