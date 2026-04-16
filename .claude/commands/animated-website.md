---
description: Build a complete animated single-page website from a text brief using UI UX Pro Max, Firecrawl, 21st.dev Magic, Stitch, and Nano Banana 2.
---

# Animated Website Builder Pipeline

You are running the `/animated-website` pipeline. This is a 7-step workflow that turns a text brief into a single animated HTML file.

## Step 0 — Greet and collect brief

If the user has not yet provided a brief in `$ARGUMENTS`, respond with **exactly**:

> Animated website builder pipeline is ready. What would you like to build?

Then wait for the user to supply a brief in this shape:

```
Site name: <name>
Purpose: <one line>
Style: <mood, palette>
Sections: <comma-separated list>
Reference URL: <optional>
Image descriptions:
  - <section>: <description>
```

If `$ARGUMENTS` already contains a brief, parse it and proceed to Step 1.

## Step 1 — Design system (UI UX Pro Max)

Invoke the `ui-ux-pro-max` skill to generate a design system from the Style line: color palette, font pairing, spacing scale, shadow + radius tokens. Save the tokens to `design-system.json` in the project root. If the skill is unavailable, derive a reasonable token set inline and continue.

## Step 2 — Reference scrape (Firecrawl, optional)

If the brief includes a `Reference URL`, call `mcp__firecrawl-mcp__firecrawl_scrape` on it. Extract: hero copy patterns, section order, color cues, notable layout/animation ideas. Save a short summary to `reference-notes.md`. Skip this step entirely if no URL was given or the MCP is unavailable.

## Step 3 — Hero component (21st.dev Magic, optional)

If the brief explicitly asks for a 21st.dev hero ("use 21st.dev", "magic hero", etc.), call `mcp__magic__21st_magic_component_builder` with a prompt derived from the Hero section. Otherwise skip.

## Step 4 — Section screens (Stitch)

For each section in the brief:
1. On the first section, call `mcp__stitch__create_project` with the site name and apply the design system from Step 1 via `mcp__stitch__create_design_system` + `mcp__stitch__apply_design_system`.
2. Call `mcp__stitch__generate_screen_from_text` with a prompt describing the section.
3. Call `mcp__stitch__fetch_screen_code` to retrieve the markup.

If Stitch is unavailable, hand-author each section in the assembly step using the design tokens.

## Step 5 — Images (Nano Banana 2)

For each entry under `Image descriptions`, invoke the `nano-banana-2` skill to generate an image. Save outputs to `assets/images/<section>.png`. If the skill or `infsh` CLI is unavailable, generate inline SVG placeholders that match the palette and note this in the final summary.

## Step 6 — Assemble

Produce a single `index.html` that:
- Uses the design tokens from Step 1 as CSS custom properties.
- Inlines the section markup from Step 4 in the order given.
- References the images from Step 5.
- Includes scroll-triggered animations (IntersectionObserver + CSS keyframes), a sticky nav, smooth scroll, and respects `prefers-reduced-motion`.
- Is fully responsive (mobile-first, breakpoints at 640/1024px).
- Contains no external JS dependencies.

## Step 7 — Review

Run through this checklist and report results to the user:
- [ ] All requested sections present, in order
- [ ] Design tokens applied consistently (no hard-coded colors outside `:root`)
- [ ] Each section has at least one entrance animation
- [ ] Nav links scroll to the correct anchors
- [ ] Mobile layout (375px) has no horizontal scroll
- [ ] All images load (or fallbacks render)
- [ ] `prefers-reduced-motion` disables non-essential animation

End with a short summary: which steps ran, which were skipped (and why), and the path to `index.html`.

## Notes

- Do not invent reference URLs or scrape sites the user did not ask for.
- If a tool is missing, use the documented fallback rather than failing the pipeline.
- Keep the final HTML in one file unless the user asks to split it.

$ARGUMENTS
