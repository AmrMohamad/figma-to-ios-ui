---
name: figma-mcp
description: Authoritative workflow for robust Figma MCP usage across Codex and Claude Code. Use for any task involving Figma design extraction, FigJam parsing, node IDs, design tokens, screenshot-based validation, design-to-code, or prompts containing a Figma URL. Trigger on phrases like "use Figma MCP", "get design context", "get metadata", "extract variables", "implement this Figma", or "compare with Figma screenshot". Always load this skill first before framework-specific implementation skills. For iOS output, load `figma-to-ios-ui` after this skill.
---

# Figma MCP

## Core Rule

Do not invent layout or styling values. Source every spacing, color, typography value, radius, and hierarchy decision from MCP tool responses.

## Mental Model

```
Agent -> Figma MCP -> React+Tailwind reference -> extract values -> framework translation -> implementation
```

- Treat MCP as read-only design source.
- Treat `get_design_context` as design intent, not final production code.
- `get_design_context` always returns React JSX with Tailwind CSS classes regardless of target platform. Extract numeric values from the Tailwind classes and translate to the target stack. Never copy the React/Tailwind output verbatim.
- Translate output to the target stack instead of copying generated snippets verbatim.

## Server Modes

Select mode before the first call.

| Dimension | Desktop mode | Remote mode |
|---|---|---|
| Endpoint | `http://127.0.0.1:3845/mcp` | `https://mcp.figma.com/mcp` |
| Source | Current Figma desktop selection | URL/node passed in request |
| Selection behavior | Uses active selection | Requires explicit target |
| Auth | Desktop app session | OAuth/session auth |

Choose mode with this rule:
- Use desktop mode when user says selection is active in Figma desktop.
- Use remote mode when user gives a Figma URL.
- Ask mode only when target context is impossible to infer safely.

## Tool Availability Strategy

Do not assume every documented Figma tool is callable in every client.

- Start each task by checking callable tools exposed in the current MCP session.
- Treat non-callable prompt-like entries as client templates, not tool endpoints.
- Build the pipeline from tools that are actually callable now.

### Commonly callable core tools

- `get_metadata`
- `get_variable_defs`
- `get_design_context`
- `get_screenshot`
- `get_figjam`
- `create_design_system_rules`

### Optional tools (environment-dependent)

Use only if they are exposed as callable tools in the current session.

- `get_code_connect_map`
- `add_code_connect_map`
- `get_code_connect_suggestions`
- `send_code_connect_mappings`
- `generate_diagram`
- `generate_figma_design`
- `whoami`

## Required Pipeline

Run this default order for design-to-code tasks. Steps marked with the same letter can run in parallel.

1. (A) `get_metadata` — map hierarchy, collect node IDs
2. (A) `get_variable_defs` — collect color, typography, spacing tokens
3. (B) `get_design_context` — extract layout, alignment, padding, sizing
4. (B) `get_screenshot` — visual verification

Before step 1, resolve the target explicitly:

- If the user provides a Figma URL, parse the file key and node ID from the URL.
- If the user says the selection is active in Figma desktop, use desktop selection mode.
- Do not guess the target mode when the prompt is ambiguous.

Steps 1+2 have no dependencies on each other — run them in parallel.
Steps 3+4 have no dependencies on each other — run them in parallel after 1+2 complete.

If optional Code Connect tools are callable, insert them between step 2 and 3.

### Optional Code Connect branch

1. `get_code_connect_map` (if available)
2. Inspect mapped vs unmapped component nodes
3. Use `add_code_connect_map` or mapping suggestion flow when needed
4. Continue with `get_design_context`

### Code Connect fallback

If `get_design_context` returns a Code Connect prompt asking to run `get_code_connect_suggestions` but that tool is not available in the current session:
- Re-call `get_design_context` with the same `fileKey` and `nodeId` parameters to skip the suggestion and get the design data directly.
- Do not block on unavailable Code Connect tools.

## Tool-by-Tool Extraction

### `get_metadata`

Use first to map hierarchy and node scope.

Extract:
- Node IDs for focused downstream calls
- Node types and names
- Parent-child structure
- Dimensions and positional hints

If payload is huge:
- Split by child node IDs
- Continue with targeted per-section calls

### `get_variable_defs`

Use to collect tokenized values.

Extract:
- Color token names and values
- Typography token names and values
- Radius/spacing token names and values

Prompt with explicit wording when needed:
- "Get variable names and values used in this frame"

If response is `Nothing is selected`:
- Re-target by specific node ID or URL
- Avoid coding until tokens are resolved

### `get_design_context`

Use to extract implementation-level layout and styling intent.

**Critical:** This tool returns React JSX with Tailwind CSS classes. Do not use the React code directly. Extract the design values from the Tailwind classes:
- `gap-[16px]` → spacing of 16pt
- `px-[20px]` → horizontal padding of 20pt
- `pt-[16px]` → top padding of 16pt
- `rounded-[6px]` → corner radius of 6pt
- `text-[16.5px]` → font size 16.5pt
- `leading-[24px]` → line height 24pt
- `text-[#6f6f6f]` → color #6F6F6F
- `shadow-[0px_0px_32px_0px_rgba(0,0,0,0.04)]` → shadow offset 0, blur 32, color 4% black
- `font-['IBMPlexSans:SemiBold']` → IBM Plex Sans SemiBold

Extract:
- Container structure and nesting
- Alignment, padding, spacing, size behavior
- Text attributes and visual style
- Component boundaries and reusable chunks

Do not treat it as final code.

**Required parameter:** `dirForAssetWrites` — provide an absolute path for image/SVG asset output. Default to `/tmp/figma-assets` or the project's asset directory. The tool will fail without this parameter when assets are present.

If response is truncated:
- Re-run on smaller child nodes discovered via `get_metadata`

If response is `Nothing is selected`:
- Re-target explicitly; do not continue with inferred values

### `get_screenshot`

Use for visual validation after structural extraction.

Use it to:
- Verify spacing rhythm and component grouping
- Validate typography and visual balance
- Confirm interpretation before final code submission

Do not derive numeric layout from pixels if structured values are available.

## Asset Handling

When MCP returns assets:

- Use the MCP-provided asset source directly.
- Do not invent placeholder icons or images when MCP has already returned a concrete asset.
- Keep `dirForAssetWrites` stable across related calls so downloaded assets remain traceable during implementation.

### `get_figjam`

Use only for FigJam boards.

If tool reports FigJam-only restrictions:
- Switch to design-file tools for regular Figma design files

### `create_design_system_rules`

Use to bootstrap reusable project-level design instructions.

Use it:
- Once per repo initially
- Again after major design-system shifts

## Extraction Validation Rules

Run these checks after each pipeline step to catch bad data early. Do not proceed to code generation until all checks pass.

### Step 1: Validate `get_metadata` output

- Confirm the root node ID matches the requested target.
- Check for `hidden="true"` nodes — exclude them from downstream calls unless the user explicitly needs hidden layers.
- Verify the hierarchy depth is reasonable. A flat list with no children likely means the wrong node was targeted.
- Count child nodes. If the count seems too low compared to the screenshot, re-run on the parent node.

### Step 2: Validate `get_variable_defs` output

- Confirm at least one color token and one typography token were returned.
- If the response is empty or says "Nothing is selected", re-call with an explicit `nodeId` — do not proceed with zero tokens.
- Cross-check token names against known project token patterns (e.g., `Foundation/Mazaya/...` for Mazaya projects).
- Verify color hex values are 6-digit format (`#RRGGBB`). If 8-digit (`#RRGGBBAA`), extract the alpha separately.

### Step 3: Validate `get_design_context` output

This is the most error-prone step. The tool returns React+Tailwind that must be parsed carefully.

**Structure validation:**
- Verify every `data-node-id` in the output matches a node from `get_metadata`. Unknown node IDs mean stale or wrong data.
- Confirm the nesting depth (div inside div) matches the parent-child structure from metadata.
- Check that `data-name` attributes match the `name` attributes from metadata.

**Value extraction validation:**
- Parse every `gap-[Xpx]` value and list them. Verify they are consistent within sections (e.g., all cells in a card should share the same gap).
- Parse every `text-[Xpx]` font size. Cross-check against typography tokens from `get_variable_defs`. Flag any font size that does not match a known token.
- Parse every color value (`text-[#hex]`, `bg-[#hex]`, `text-black`). Cross-check against color tokens. Flag unrecognized colors.
- Parse every `font-['Family:Style']` value. Verify the font family and style match a typography token.
- Parse `rounded-[Xpx]` values. Verify consistency (all cards should share the same radius unless the design differs intentionally).
- Parse `shadow-[...]` values. Extract offset, blur, spread, and color components. Verify against the shadow token from `get_variable_defs`.

**Completeness validation:**
- Every visible text node from the screenshot should appear in the design context output. If text is missing, the response was likely truncated — re-run on smaller child nodes.
- Every Figma component/instance from metadata should appear in the output. Missing components mean truncation.

### Step 4: Validate `get_screenshot` against extracted data

- Count the major visual sections in the screenshot. This count should match the top-level containers in the design context.
- Verify text content visible in the screenshot matches the text strings in design context.
- Check that the visual spacing rhythm (tight, medium, wide gaps) matches the extracted gap values.
- Confirm colors visible in the screenshot (section titles, subtitles, accent colors) match the extracted color tokens.
- Look for elements visible in the screenshot that are absent from design context — these indicate truncation or missed nodes.

### Cross-tool consistency checks

Run these after all 4 steps complete:

| Check | Source A | Source B | Action if mismatch |
|---|---|---|---|
| Node count | `get_metadata` child count | `get_design_context` `data-node-id` count | Re-run design context on missing nodes |
| Colors used | `get_design_context` hex values | `get_variable_defs` token values | Flag untokenized colors in output brief |
| Font sizes | `get_design_context` `text-[Xpx]` | `get_variable_defs` typography tokens | Use token value, not raw Tailwind value |
| Dimensions | `get_metadata` width/height | `get_design_context` layout classes | Prefer metadata for absolute sizes |
| Visual match | `get_screenshot` visible elements | `get_design_context` structure | Re-extract if major elements missing |

### Anatomical decomposition checks (X/Y/Z)

Run these to understand the design anatomy before implementation:

- Build an `X-axis` map (horizontal anatomy):
  - leading/trailing anchors
  - center alignments
  - sibling-to-sibling horizontal gaps
  - width behavior (`fixed`, `fill`, `hug`)
- Build a `Y-axis` map (vertical anatomy):
  - top/bottom anchors
  - section rhythm (stack spacing and vertical cadence)
  - baseline and text block relationships
  - height behavior (`fixed`, `intrinsic`, `content-driven`)
- Build a `Z-axis` map (depth anatomy):
  - draw order candidates from node order and nesting
  - overlap regions from frame intersections
  - depth cues from shadow/blur/opacity/mask/clip
  - interaction precedence for layered elements (which layer receives taps)
- If Z-order is ambiguous, split extraction into smaller node-level calls and re-validate overlaps.

### Interaction and flow extraction checks

Run these before declaring extraction complete:

- Detect interactive candidates from `get_design_context` (`<button>`, `cursor-pointer`, underlined text+icon action rows, tappable-looking chips/cards, explicit action names in `data-name`).
- Build an interaction map per candidate:
  - `nodeId`
  - likely control type
  - likely action intent (download, view more, edit, navigate, submit, open modal, etc.)
  - confidence (`high|medium|low`)
- Extract a user-flow narrative from top to bottom:
  - primary path
  - secondary paths
  - terminal states implied by the design
- Flag unresolved interaction intent in an ambiguity list instead of guessing behavior.
- Detect repeated/repeatable visual structures (logo rows, cards, list-like blocks). Mark them as likely data-driven candidates for implementation.

### Platform chrome filtering

Before handing extraction to implementation, classify nodes as either app UI or device/system chrome.

- Treat status bars, home indicators, and device mock chrome as non-app UI unless explicitly required by the target platform.
- Keep custom in-app nav bars only when they are clearly part of product UI and not device framing.
- Record filtered nodes in the extraction brief so downstream implementation does not accidentally code them.

### When to re-extract

Re-run MCP tools when any of these are true:
- More than 2 colors in design context are not found in variable defs.
- A visible section from the screenshot is completely absent from design context.
- Node IDs in design context do not match metadata.
- The design context response ends abruptly (truncation).
- Any "Nothing is selected" response was received.

## Output Contract (Before Writing Code)

Produce a compact extraction brief with:

- Target: file/frame/node IDs used
- Tokens: colors, type, spacing, radii
- Layout tree: containers, gaps, alignments, sizing constraints
- Components: reusable vs one-off
- Assets: icons/images required
- Anatomy map: X-axis anchors/gaps, Y-axis rhythm, Z-axis layering and overlaps
- Interaction map: tappable nodes, hypothesized actions, confidence
- User-flow intent: primary path + secondary paths inferred from layout/actions
- Data model hints: static vs likely dynamic/repeated regions
- Business-logic open questions: details design cannot define alone
- Open gaps: unresolved values or ambiguous states

If critical fields are missing, call MCP again. Do not guess.

## Structured Extraction Artifact For Downstream iOS Work

Always produce a `design_spec` JSON artifact before handing off to downstream iOS implementation work.

Use it for:

- UIKit/XIB generation
- UIKit/XIB audit or patch-assist
- SwiftUI generation
- SwiftUI audit or repair
- hybrid-screen implementation planning

Required shape:

```json
{
  "node": {
    "file_key": "",
    "node_id": "",
    "node_name": "",
    "platform_chrome_filtered_nodes": []
  },
  "tokens": {
    "colors": [],
    "typography": [],
    "radii": [],
    "spacing": [],
    "shadows": []
  },
  "anatomy": {
    "x_axis": [],
    "y_axis": [],
    "z_axis": []
  },
  "layout_tree": [],
  "interactions": [],
  "flow": {
    "primary_path": [],
    "secondary_paths": [],
    "terminal_states": []
  },
  "business_assumptions": [],
  "unresolved": []
}
```

Rules:
- Do not invent unresolved fields. Set `status: "unknown"` and explain why.
- Every inferred interaction must include `confidence` (`high|medium|low`).
- Mark business-logic uncertainties explicitly in `business_assumptions`.
- Keep every node mapping traceable by `node_id`.

Anti-overengineering guardrail:
- If unresolved critical fields are `<= 2`, continue with explicit assumptions.
- If unresolved critical fields are `> 2`, re-extract only missing child nodes (do not rerun full-frame extraction).

## Existing UIKit Reality Input (When Files Are Provided)

If the user provides existing `.xib` and `.swift` implementation files:

1. Complete the normal MCP extraction flow first.
2. Produce `design_spec` from MCP evidence.
3. Forward implementation paths + `design_spec` to the translation-skill audit scripts:
   - `figma-to-ios-ui/scripts/extract_uikit_impl_snapshot.py`
   - `figma-to-ios-ui/scripts/figma_uikit_audit.py`
4. Use audit findings to generate patch hints only (no auto-editing).

This is an automatic UIKit-specific branch of the normal workflow, not a separate mode.

## Robustness Rules

- Prefer many small calls over one oversized call.
- Keep node IDs traceable in notes.
- Reuse token extraction across sibling sections when appropriate.
- Resolve conflicts by source priority:
  1. `get_variable_defs` for token identity
  2. `get_design_context` for layout/styling intent
  3. `get_screenshot` for visual confirmation

## Rate-Limit and Cost Control

Assume limits can exist and vary by account/server.

Apply these controls:
- Scope calls to component-level nodes
- Cache token results per screen/frame
- Avoid repeated full-frame `get_design_context`
- Retry once for transient failures, then narrow scope

## Failure Playbook

### Nothing is selected

- Confirm selection context (desktop mode)
- Pass explicit target node/URL (remote mode)
- Re-run with narrower scope

### Tool unavailable

- Continue with callable core tools
- Mark unavailable tools as optional in output notes
- Do not block if equivalent data can be obtained via core pipeline

### Code Connect prompt but tool missing

- Re-call `get_design_context` with the same fileKey and nodeId
- The tool will return the design data on the second call

### `dirForAssetWrites` error

- Pass `dirForAssetWrites` with an absolute path (e.g., `/tmp/figma-assets`)
- Create the directory first if it does not exist

### Permission/auth error

- Re-authenticate current MCP session
- Confirm file access rights for the authenticated account
- Re-run on the same target before changing assumptions

### Truncation/timeouts

- Split by child nodes from `get_metadata`
- Process sections independently (header/body/footer)

### Server/internal error

- Validate URL/node correctness
- Retry once
- Fall back to smaller scoped calls

## Prompt Patterns

Use explicit prompts to reduce mis-calls:

- "Use Figma MCP `get_metadata` on this node URL and return hierarchy + node IDs."
- "Use `get_variable_defs` and return token names and values, not raw CSS summary."
- "Use `get_design_context` for node X and focus on spacing, padding, alignment, and typography."
- "Use `get_screenshot` for visual verification only."

## Integration Rule

When the target is iOS UI implementation:

1. Load this `figma-mcp` skill first for extraction discipline.
2. Load `figma-to-ios-ui` second for implementation-lane translation.
3. Keep this split strict:
   - `figma-mcp`: tool orchestration and raw data quality
   - `figma-to-ios-ui`: iOS UIKit/XIB or SwiftUI implementation details

Both skills should be invoked together for Figma-to-iOS tasks. If only one is loaded, the agent should load the other automatically.

## How To Run With UIKit Audit Scripts

Use these commands after extraction when UIKit files exist:

```bash
python3 /Users/amrmohamad/.codex/skills/figma-to-ios-ui/scripts/extract_uikit_impl_snapshot.py \
  --xib /absolute/path/View.xib \
  --swift /absolute/path/View.swift \
  --presenter /absolute/path/Presenter.swift \
  --out /tmp/impl_snapshot.json

python3 /Users/amrmohamad/.codex/skills/figma-to-ios-ui/scripts/figma_uikit_audit.py \
  --design-spec /tmp/design_spec.json \
  --impl-snapshot /tmp/impl_snapshot.json \
  --report-out /tmp/analysis_report.md \
  --patch-hints-out /tmp/patch_hints.md
```
