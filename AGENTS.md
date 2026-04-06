# AGENTS.md

Repository-level instructions for AI coding agents working in this repo.

## Purpose

This repository packages the public `figma-to-ios-ui` skill.

Agents should treat the nested skill folder as the product:

- `./figma-to-ios-ui/`

The root files exist to help agents select and use that skill correctly.

## Auto-Trigger Rules

Load `./figma-to-ios-ui/SKILL.md` automatically when the task involves any of these:

- Figma MCP extraction handoff into iOS implementation
- design-to-code for iOS
- UIKit or XIB UI generation, audit, or repair
- SwiftUI UI generation, audit, or repair
- hybrid UIKit and SwiftUI seams
- visual fidelity checks against design evidence
- shared UIKit component hardening
- post-implementation UI validation against Figma

Also load `./figma-to-ios-ui/subskills/project-ui-pattern-memory/SKILL.md` when:

- the target repo has project-specific SwiftUI or hybrid patterns
- the design system or ownership model is unclear
- the agent needs to learn local UI conventions from a live codebase

Do not wait for the developer to mention these skill names explicitly if the task clearly matches them.

## Repo Layout

- `./figma-to-ios-ui/SKILL.md`
  - Main public skill entrypoint.
- `./figma-to-ios-ui/references/`
  - Lane references and validation guidance.
- `./figma-to-ios-ui/scripts/`
  - Audit and scaffold helpers.
- `./figma-to-ios-ui/subskills/project-ui-pattern-memory/`
  - Companion sub-skill for repo-specific UI pattern learning.
- `./figma-to-ios-ui/templates/`
  - Structured templates for design specs and validation briefs.

## Editing Rules

- Keep the skill generic and public-facing.
- Do not hard-code project-specific overlays into the base skill.
- Prefer reusable guidance over narrow product-specific examples.
- Preserve the separation of responsibilities:
  - `figma-mcp` handles extraction discipline.
  - `figma-to-ios-ui` handles iOS translation, validation, and adaptation.
- Keep UIKit/XIB guidance evidence-driven and runtime-aware.
- Keep SwiftUI guidance architecture-neutral: preserve the target repo's proven patterns instead of imposing one.

## Validation Rules

When changing the skill:

- validate modified Python scripts with `python3 -m py_compile`
- keep JSON templates parseable
- check markdown references for broken internal paths
- prefer the smallest validation scope that proves the skill still works

## Publishing Rules

- Keep repo-root instructions concise so they do not waste agent context.
- Prefer links or references over duplicating large bodies of guidance.
- Treat everything here as public repo content: never include secrets or personal-only notes.
