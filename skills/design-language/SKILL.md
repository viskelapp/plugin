---
name: design-language
description: Bring a product's design language from its code into a Viskel design system - tokens for every theme, the identity guide and the base stylesheet. Use when the user asks to import, bring in, extract or rebuild a design system from a repo, codebase, app.css, globals.css or Tailwind config into Viskel.
---

# Bring a design language in from code

Workflow for filling a Viskel design system from a repository. The Viskel server's own instructions and `update_design_system`'s description hold the rules and the shapes; this skill is the order of work. Components and patterns come after this, with `/viskel:components-from-code`.

## 1. Connect and pick the system

1. Call `list_design_systems`. If the Viskel tools are missing or answer with an auth error, sign in as `/viskel:design` step 1 says.
2. Use the system the user named. If there is none, create it with `update_design_system` path `starter`, `system` its name and patch `{title, description}` (no choices: the repo sets the look). Ignore its hint to send `starter` again with choices: the import fills the tokens. Use the space the user named, else the current one, and say which in the report.
3. `get_design_system(system)`, then `get_design_system(system, "checklist")`. Before each write, read what it changes (`tokens.<category>` after the import, the identity section before its write), so you send the shapes it takes.

## 2. Survey the repo

Find where the design language lives before you send anything. Note file and line for every value.

- **Token sources:** stylesheets with custom properties (`app.css`, `globals.css`, `:root`, `.dark`, `[data-theme]`, `@theme`), `tailwind.config.*` `theme`/`extend`, theme files (`theme.ts`, `tokens.json`), shadcn `components.json`.
- **Dead config:** a value that is defined but never used, or wrapped wrongly (an `oklch()` value inside `hsl(var(--x))`), does not describe the product. Check that the build loads the config: an `@config` or `@import` line pointing at it, a default export, and no `@theme inline` block that overrides the same names.
- **Real usage:** count what the templates and components use. Grep class names and arbitrary values (`bg-red-500`, `text-[10px]`, `rounded-xl`, `shadow-sm`, `gap-6`) across the source and keep the counts. Where usage and theme disagree, usage is the product: a status colour that only appears as `text-green-600` is still the success colour.
- **Fonts:** `@font-face`, font imports, `font-family` in the base CSS. A loaded font counts only when a `font-family` or a theme font token uses it. If none does, the system stack is the product's font; say so.
- **Icons:** count imports per icon library; the one with most imports is the icon set.
- **Density and shape:** the common control heights, paddings, gaps and radii from the counts.

## 3. Tokens

1. Send the stylesheet with `update_design_system` path `import` and `files: [{name: "app.css", content: "<the file's text>"}]` but no `apply`. Keep `@theme`/`@theme inline`, `:root` and the theme selectors (`.dark`, `[data-theme]`); leave out utilities and keyframes when the file is long. Read the report: mapped, custom, derived and unmapped rows, the mode rows for dark and other themes, and the derived neutral scale.
2. If the mapping is right, apply that dry run with `{import: "<id>", apply: true}` instead of sending the files again. If the draft changed, run the dry run again.
3. Check the neutral scale: when the repo names a base colour (shadcn `components.json` `baseColor`, a Tailwind palette used for greys), set `color.neutral.*` to that palette's steps. Status colours come in pairs: a `bg-<hue>-50` surface with `text-<hue>-600/700` maps to `color.<state>` and `color.<state>-foreground`.
4. Fill what the import could not: values from the Tailwind config or from usage (status colours, the neutral hue, fonts, shadows, radii). Send one `update_design_system` call with path `tokens` per category, every slot of that category in its patch: `{"color.success": {value, modes: {theme: {dark: …}}}, …}`. Values the system has no slot for (an app header height) go into the layout rules and the report.
5. `get_design_system(system, "tokens.<category>")` for colour, font, text, radius, spacing and shadow. A slot with status `default` while the repo defines another value is a miss: fix it. A default that equals the framework's own default the repo uses unchanged (Tailwind's shadows, type scale, spacing) is right; report it as found.

## 4. Identity

Write each identity section with `update_design_system` path `identity.<section>`: its choices first, then rules. The choice keys and their kinds (text, enum values, a slot reference or a list of them) are at the end of `update_design_system`'s description; `get_design_system(system, "checklist")` lists the choices each section still needs. A rule is `{text, severity, scope}`: the text names the thing itself and the severity gives the verb (`never` "raw palette colours for status", `prefer` "status colours on badges and alerts"); the scope says where it applies, in a word or two. Leave `check` out unless the description's check grammar fits it exactly. Rules state the product's intent from its conventions, with the evidence where it helps ("Status colours only on badges and alerts: 40 uses in components/ui"). Where the code breaks its own convention (`text-[10px]` against a 12px floor), write the intended rule and list the deviation in the report, not as a rule.

Cover colour use (which roles carry brand, how status shows), type roles and sizes, shape and depth, layout and density, icons, motion, accessibility, and the Never list: what the product never does.

## 5. Base stylesheet

`get_design_system(system, "components.base")`, then send `components.base` with the default fitted to the system: font family, size, colour and background, and the focus ring, all from the system's variables, never literals. Keep the shared resets there.

## 6. Check

`get_design_system(system, "checklist")` until the tokens, identity and `components.base` steps are done. Stop when only `components`, `patterns` and `publish` remain. Do not publish: the first release follows the components.

## 7. Report

End with a short report to the user:

- **Found:** each value you set and where it came from (file:line or a usage count).
- **Guessed:** each value the repo does not define and why you chose it.
- **Deviations:** places where the code breaks its own rules.
- **Next:** components with `/viskel:components-from-code`, then publish.
