# Modernist design system

Last verified: 2026-09-24

Modernist is flat, architectural and set entirely in Archivo: deep navy with a warm gold accent on a light ground, a visible modular grid, zero corner radius and strong 2px rules. Nothing floats and nothing is decorated — alignment and the strength of the dividers do all the organising, labels sit flush left (even inside buttons), and photography prints in pure black and white.

## How to use this

- This repo ships two files: `styles.css` (the tokens and the component layer) and this guide. The app pulls the stylesheet in once — `app/globals.css` starts with `@import "../design-system/styles.css";`, and `app/layout.tsx` imports `globals.css` — so every page gets the tokens without a per-page `<link>`. Take every color, font, spacing, radius and shadow from the variables (`var(--color-*)`, `var(--font-*)`, `var(--space-*)`, `var(--radius-*)`, `var(--shadow-*)`). Never hard-code a hex, a font name or a px value the tokens already carry.
- Build with the classes below rather than inventing parallel ones. There are no component preview pages in this repo — the markup to copy is the class definitions themselves in `styles.css`, plus how `app/page.tsx` and `app/page.module.css` use them.
- To change the look, edit the tokens at the top of `styles.css` — the site and this guide both read from them. Keep this guide in step with what the CSS actually does; the tokens are the source of truth, not the prose.

## Direction

Modular grid layouts — content in equal-width cells, strong horizontal and vertical rhythm, visible structure. Use strong 2px dividers (`var(--color-divider)`) between major sections. Button labels are flush left — a button wider than its label starts the text at the left padding edge (trailing icon and all), never centered. Wrap hero and inline images in the `.grayscale` class — they print in pure black and white.

## Color

A light ground (`--color-bg` #f3f2f2) with `--color-text` #201e1d, a deep-navy accent #14233f (`--color-accent`) for structure, links and the poster field, and a warm-gold second accent #c99633 (`--color-accent-2`) for small emphasis, so the page reads as kin to the Hesed Home Care brand page. Each role carries a 100–900 tonal ramp (`--color-neutral-100` … `--color-accent-2-900`) generated in OKLCH on a shared perceptual lightness scale, so the same step of any ramp has the same visual weight. Use the light steps (100–300) for tinted fills, hovers and subtle borders, 500 as the role's base, and the dark steps (700–900) for text on tinted fills and for pressed states; prefer ramp steps over ad-hoc `color-mix()`. For elevation use `--shadow-sm/md/lg` (already tuned to the ground) rather than ad-hoc box-shadows.

## Type

Archivo for headings over Archivo for body text, loaded as `--font-heading` / `--font-body`. Density 1.00× and radius 0px are already baked into the `--space-*` / `--radius-*` scales — use the variables, not raw numbers.

## Icons

Use Lucide icons (https://lucide.dev) throughout.

## Interaction states

Interactive states are themed, never browser defaults: give every interactive element a `:hover` tint and a pressed state from the accent ramp (one step past the base — `--color-accent-600` on a light ground, `--color-accent-400` on a dark one, or a `color-mix()` tint for outlined/ghost variants), and style keyboard focus with `:focus-visible { outline: 2px solid var(--color-accent); outline-offset: 2px; }` — never leave the default blue focus ring.

## Components

Every class below is defined in `styles.css` (verified present 2026-09-24). The live page uses the
button, nav, tag and rule classes; the form, card, table and dialog classes are available and
currently unused — this site has no forms or modals.

| Class | What it is |
| --- | --- |
| `.btn` with `.btn-primary`, `.btn-secondary`, `.btn-ghost`, `.btn-icon`, `.btn-block` | Actions — the primary is a solid accent fill |
| `.tag` with `.tag-accent`, `.tag-accent-2`, `.tag-neutral`, `.tag-outline` | Small labels tinted from the ramps |
| `.field` + `label`, `.input`, `.radio` + `.dot`, `.seg` + `.seg-opt` | Form fields and choices on native elements — no script |
| `.card` with `.card-kicker`, `.card-title`, `.card-body`, `.card-meta`; `.elev-sm/md/lg` | Surface-filled content cards; elevation utilities |
| `.nav` + `.nav-brand` | The header bar |
| `.table` | Data tables with themed header and row rules |
| `.dialog-backdrop` + `.dialog` (+ `.dialog-title/-body/-actions`) | A modal at the top elevation |
| `.hr` | A strong 2px horizontal rule |
| `.grayscale` | The image wrapper — every content photograph goes through it (none on the page today; the closing band is the `public/capital-flow.html` canvas, already in brand colors) |
| `.grayscale` | The image wrapper — every content photograph goes through it |

States are built in: hovers and pressed states come from the accent ramp, keyboard focus is the 2px accent `:focus-visible` ring, `::selection` is an accent tint, and disabled controls drop to 45% opacity. Don't restyle them per page. The accent-to-ground pair is tuned to at least 3:1 — enough for icons, large text and interface chrome, not for body copy — so for paragraph-size text in the accent use a deep ramp step (`--color-accent-700` on this ground) rather than the accent itself.

## Do

- Let the grid show: equal-width cells, strong horizontal rules between sections, visible structure.
- Keep everything flush left — headings, copy, and the labels inside wide buttons.
- Use the accent sparingly, for the primary action and small emphasis; the system is mostly ink on ground. The one place the navy runs as a field is the poster statement — the landing's closing banner — where type stays display-grade and the accent carries the page.
- Print photographs in black and white with the `.grayscale` wrapper.

## Don't

- Do not round a corner anywhere — `--radius-md` is 0 on purpose.
- Do not center button labels or hero copy.
- Do not soften the rules into hairlines or drop them for whitespace.
- Do not tint or colorize imagery.

## Files

- `styles.css` — the only stylesheet: the token sheet (`:root` variables, ramps, base type) plus the
  component layer. Consumed once, by `app/globals.css`.
- `readme.md` — this guide.

That is the whole system as it lives in this repo. The preview pages (`foundations/`, `components/`,
`templates/`), `theme.json` and `thumbnail.html` that the original design-system kit carried were
never vendored here — the CSS and this guide are it.
