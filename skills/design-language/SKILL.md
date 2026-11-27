---
name: design-language
description: Bring a product's design language from its code into a Viskel design system - tokens for every theme, the identity guide and the base stylesheet. Use when the user asks to import, bring in, extract or rebuild a design system from a repo, codebase, app.css, globals.css or Tailwind config into Viskel.
---

# Bring a design language in from code

Workflow for filling a Viskel design system from a repository. The Viskel server's own instructions and `update_design_system`'s description hold the rules and the shapes; this skill is the order of work. Components and patterns come after this, with `/viskel:components-from-code`.

## 1. Connect and pick the system

1. Call `list_design_systems`. If the Viskel tools are missing or answer with an auth error, sign in as `/viskel:design` step 1 says.
2. Use the system the user named. If there is none, ask which space, then create it with `update_design_system` path `starter` and no choices.
3. `get_design_system(system)`, then `get_design_system(system, "checklist")`. Read `tokens.color` and the identity section you are about to write before writing it, so you send the shapes it takes.

## 2. Survey the repo

Find where the design language lives before you send anything. Note file and line for every value.

- **Token sources:** stylesheets with custom properties (`app.css`, `globals.css`, `:root`, `.dark`, `[data-theme]`, `@theme`), `tailwind.config.*` `theme`/`extend`, theme files (`theme.ts`, `tokens.json`), shadcn `components.json`.
- **Dead config:** a value that is defined but never used, or wrapped wrongly (an `oklch()` value inside `hsl(var(--x))`), does not describe the product. Check that the config is actually loaded by the build.
- **Real usage:** count what the templates and components use. Grep class names and arbitrary values (`bg-red-500`, `text-[10px]`, `rounded-xl`, `shadow-sm`, `gap-6`) across the source and keep the counts. Where usage and theme disagree, usage is the product: a status colour that only appears as `text-green-600` is still the success colour.
- **Fonts:** `@font-face`, font imports, `font-family` in the base CSS. If the repo declares none, the system stack is the product's font; say so.
- **Icons:** count imports per icon library; the one with most imports is the icon set.
- **Density and shape:** the common control heights, paddings, gaps and radii from the counts.

## 3. Tokens

1. Send the stylesheet with `update_design_system` path `import` and `files` but no `apply`. Read the report: mapped, custom, derived and unmapped rows, the mode rows for dark and other themes, and the derived neutral scale.
2. If the mapping is right, apply that dry run with `{import: "<id>", apply: true}` instead of sending the files again. If the draft changed, run the dry run again.
3. Fill what the import could not: values from the Tailwind config or from usage (status colours, the neutral hue, fonts, shadows, radii). Send one `update_design_system` call with path `tokens` per category, every slot of that category in its patch.
4. `get_design_system(system, "tokens.<category>")` for colour, font, text, radius, spacing and shadow. A slot that still shows its Viskel default while the repo defines that value is a miss: fix it.

## 4. Identity

Write each identity section with `update_design_system` path `identity.<section>`: its choices first, then rules. Rules state the product's intent from its conventions, with the evidence where it helps ("Status colours only on badges and alerts: 40 uses in components/ui"). Where the code breaks its own convention (`text-[10px]` against a 12px floor), write the intended rule and list the deviation in the report, not as a rule.

Cover colour use (which roles carry brand, how status shows), type roles and sizes, shape and depth, layout and density, icons, motion, accessibility, and the Never list: what the product never does.

## 5. Base stylesheet

`get_design_system(system, "components.base")`, then send `components.base` with the default fitted to the system: font family, size, colour and background, and the focus ring, all from the system's variables, never literals. Keep the shared resets there.

## 6. Check

`get_design_system(system, "checklist")` until the tokens, identity and `components.base` steps are done. Do not publish: the first release follows the components.

## 7. Report

End with a short report to the user:

- **Found:** each value you set and where it came from (file:line or a usage count).
- **Guessed:** each value the repo does not define and why you chose it.
- **Deviations:** places where the code breaks its own rules.
- **Next:** components with `/viskel:components-from-code`, then publish.
