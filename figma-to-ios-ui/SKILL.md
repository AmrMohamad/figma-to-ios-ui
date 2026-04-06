---
name: figma-to-ios-ui
description: Translate Figma MCP design data into production iOS UI implementations in either UIKit/XIB or SwiftUI. Use when implementing, auditing, or repairing iOS UI from Figma, especially when the result must match both the design evidence and the target codebase's existing UI patterns, design system, and integration seams.
---

# Figma to iOS UI

## Core Role

Load `figma-mcp` first. This skill is the iOS implementation layer that consumes the extracted Figma design data and routes work into the correct implementation lane.

Keep the split strict:

- `figma-mcp` owns target resolution, MCP calls, extraction quality, screenshot verification, and `design_spec`
- `figma-to-ios-ui` owns lane selection, project adaptation, implementation guidance, and shared UIKit/XIB component hardening

Do not bypass `figma-mcp` when design data is incomplete.

## Public-Skill Rule

This package is intentionally public-facing and generic.

- Keep project-specific knowledge out of the base skill.
- Learn repo patterns at task time instead of baking static overlays into the skill.
- Prefer the target project's existing implementation seams, design system, and shared components over generic examples from the skill.

## Workflow

1. Confirm `figma-mcp` has already produced:
   - node hierarchy and IDs
   - token map
   - layout/style intent
   - screenshot
   - `design_spec`
2. Read `references/lane-selection.md`.
3. Route into exactly one implementation lane:
   - UIKit/XIB lane: `references/ios-uikit-xib-lane.md`
   - SwiftUI lane: `references/swiftui-lane-overview.md`
4. If SwiftUI is selected and project UI patterns are not already clear, load `subskills/project-ui-pattern-memory/SKILL.md` and build or refresh a project UI memory brief.
5. Load only the task-relevant reference files for the selected lane.
6. If UIKit/XIB work touches a shared cell, row, card, section, or nib-backed reusable component, also load:
   - `references/shared-uikit-component-hardening.md`
   - `references/component-spec-sheet-template.md`
   - `references/token-verification-workflow.md`
7. After implementation or repair, run `references/post-implementation-validation-and-learning.md`.
8. Emit final guidance only after the lane, project adaptation layer, and validation pass are fixed.

## Lane Selection

Use UIKit/XIB when the touched feature already follows `BaseVC + Presenter + XIB`, outlets/actions, or legacy reusable UIKit views.

Use SwiftUI when the touched feature already uses SwiftUI roots, SwiftUI-owned feature seams, hosted SwiftUI screens, or shared SwiftUI primitives and modifiers.

Use existing hybrid seams when the host screen mixes both. Do not migrate a host shell just because the new design is modern.

For hybrid screens:

- preserve the current root shell
- extend the matching child seam
- keep navigation, presentation, and state ownership where the repo already keeps them

## Project Pattern Memory

Static per-project SwiftUI overlays are intentionally not part of this public skill.

When the repo needs project-specific adaptation:

1. load the general SwiftUI lane references
2. run `subskills/project-ui-pattern-memory/SKILL.md`
3. build or refresh a project UI memory brief from the live codebase
4. use that brief as the task-local adaptation layer

Treat the resulting brief as working memory for the active repo, not as hard-coded skill doctrine.

## Output Contract

Always state:

- which lane was selected
- whether a project UI memory brief was built or reused
- which shared project primitives or existing components were reused
- what design-system or token mapping strategy was used
- what assumptions remain unresolved by design alone
- what validations were run or must still be run
- whether a validation brief or learning delta was produced

For UIKit/XIB work, preserve the XIB audit/runtime-diagnosis workflow and the merged shared-component hardening contract.

For SwiftUI work, include:

- state ownership choice
- navigation or presentation ownership choice
- component extraction plan
- modern API and availability choices
- token mapping plan
- accessibility and motion checks
- performance and update-risk checks
- screenshot parity checks
- whether the task is generation or audit/repair
- what the project UI memory brief learned or confirmed after validation

## Senior UIKit/XIB Interpretation Heuristics

Use these when the selected lane is UIKit/XIB and the design is a reused list row, card row, checkout option row, or any component with repeating siblings.

1. Identify the component family before editing.
- Decide whether the design is:
  - one reusable component with multiple content variants
  - multiple separate components
- Do not split a reused row into separate implementations unless the object anatomy truly changes.

2. Separate the primary lane from the secondary lane.
- Primary lane usually contains:
  - selection affordance
  - identity icon/chip
  - semantic text block
- Secondary lane usually contains:
  - trailing logos
  - accessory actions
  - disclosure metadata
- Many first-pass UIKit mistakes come from treating the secondary lane as part of the text block instead of a separate sibling lane.

3. Distinguish design intent from legacy implementation intent.
- If an existing shared component already ships in production, ask:
  - what old design goal was it optimizing for
  - what is actually broken now
  - what can be improved without rewriting the whole component
- Prefer the smallest patch that fixes the unstable lane before replacing the component skeleton.

4. Extract the extreme variants, not only the parent frame.
- For repeated rows, always inspect at least:
  - the compact variant
  - the multiline or densest variant
  - a variant with optional trailing disclosure/logos if present
- This catches geometry issues that full-frame extraction and token-only audits often miss.

5. Treat runtime screenshots as first-class evidence.
- Figma gives ideal anatomy.
- Runtime screenshots reveal:
  - data-driven combinations
  - resolver-driven icon/logo choices
  - truncation patterns
  - RTL or real-device density issues
- If screenshots conflict with the first implementation theory, update the theory before editing again.

6. Keep behavior ownership stable.
- In reused UIKit/XIB components, preserve:
  - presenter/view-model ownership
  - parent selection/navigation ownership
  - resolver/business contracts
- UI revamps should not silently move business logic into the view layer.

7. Localize state styling to the smallest object that changed in Figma.
- If selected state only changes the radio, do not redesign the full row as selected.
- If helper copy only changes text rhythm, do not redesign the entire row spacing model.
- This prevents over-correcting beyond the actual design evidence.

## References

- `references/lane-selection.md`
- `references/ios-uikit-xib-lane.md`
- `references/shared-uikit-component-hardening.md`
- `references/component-spec-sheet-template.md`
- `references/token-verification-workflow.md`
- `references/post-implementation-validation-and-learning.md`
- `references/swiftui-lane-overview.md`
- `references/swiftui-state-and-data-flow.md`
- `references/swiftui-view-structure-and-layout.md`
- `references/swiftui-navigation-sheets-and-routing.md`
- `references/swiftui-scroll-list-and-identity.md`
- `references/swiftui-design-system-and-token-mapping.md`
- `references/swiftui-modern-api-and-availability.md`
- `references/swiftui-accessibility-and-inclusive-design.md`
- `references/swiftui-performance-and-update-hygiene.md`
- `references/swiftui-animation-and-motion.md`
- `references/swiftui-fidelity-validation.md`
- `references/swiftui-audit-and-repair.md`
- `subskills/project-ui-pattern-memory/SKILL.md`
- `templates/ui_validation_brief.template.md`

## Validation Gates

Before closing an iOS design-to-code task, verify:

- the correct lane was chosen from existing implementation seams
- the output matches project conventions, not raw MCP React/Tailwind
- no extraction rules were duplicated locally instead of referenced from `figma-mcp`
- the active repo was adapted through live codebase evidence rather than a stale baked-in overlay
- interactive elements, dynamic regions, and visual fidelity checks are covered
- post-implementation validation was run at the smallest scope that can prove the result
- newly learned repo-specific rules were captured as deltas instead of forcing a full relearn
- the final guidance is implementation-ready for the selected framework
