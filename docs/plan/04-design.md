<!-- Composed from the Braid design system (https://claude.ai/artifact/6QfVoaAyTJ9DGUZFNQqfMS) and the Braid Screens canvas (https://claude.ai/artifact/GipvFcyUhbhdMLKXUjP6fe) on 31 Oct 2026. Those artifacts are the living versions; tokens.json in docs/design/ is the source of truth for values. -->

# Braid · Phase 4 — Design

Phase 4 produced two artifacts: the **Braid design system** (tokens, rules and 13 components) and **Braid Screens**, six screens built only from that system. This file carries the rules and the token tables so code can follow them without opening the artifacts.

- Design system: https://claude.ai/artifact/6QfVoaAyTJ9DGUZFNQqfMS
- Screens: https://claude.ai/artifact/GipvFcyUhbhdMLKXUjP6fe
- Tokens as data: `docs/design/tokens.json`; component classes: `docs/design/components.css` (prefix `bd-`)

## Design system rules

Braid is a visual workspace for building software with AI agents. Its look is a survey map: calm paper and ink, nested regions you can find your way around, and a few strong marks for the things that matter: who or what wrote the code, what is waiting on a person, and the links Braid itself draws.

### Content fundamentals

- **Plain and specific.** Say what happened and where, with numbers: "Two agents will edit `useCart.ts`, lines 40–58", not "Potential conflict detected".
- **Sentence case everywhere**, including buttons and titles: "Send to Claude Code", "Look here first".
- **Address the person as you; Braid speaks as Braid,** never "I" or "we": "Braid found 2 callers that weren't updated."
- **Verbs first on actions:** "Approve", "Open change story", "Remove worktree".
- **No emoji, no exclamation marks, no hype words** ("magic", "supercharge", "AI-powered").
- **Name agents by their tool**, with the product's own spelling (Claude Code, Codex, Cursor), and call them agents, not bots or assistants.
- **Never blame people.** Authorship is a fact about lines, not a score for developers: "38% of these lines were written by an agent", never "Ana wrote only 16%".
- **Paths, SHAs, worktree and symbol names are set in `code-sm`**, never in quotes.

Real copy to follow:

| Where | Copy |
| --- | --- |
| Inbox | "Wants to run npm test" · "Ready for review: express-pay retries" |
| Overlap warning | "Claude Code (wt-2) and Cursor (wt-3) both edit `useCart.ts`, lines 40–58." |
| Empty inbox | "Nothing waiting on you." |
| Empty map | "Start an agent in a fresh worktree, or keep working: Braid will notice." |
| Unknown authorship | "No record of who wrote these lines." |
| Error | "Braid can't reach the daemon. Run `braid open` or restart the app." |

### Visual foundations

#### Paper and ink

The map sits on `surface`; panels, cards and label plates use `surface-raised`; inputs and the gutters between map regions use `surface-sunken`. Text is `ink`, secondary text `ink-muted`, timestamps and counts `ink-faint` (on `surface` and `surface-raised` only). Hairlines use `line`; anything a person must see as an edge (control borders, region outlines) uses `line-strong`.

#### The map

Regions nest: workspace, group, repo, package, folder, file. Alternate `region-a` and `region-b` by depth so nesting reads without heavy borders. Every label sits on a `surface-raised` plate in `map-label` (repo names in `display` at small size), never directly on a lens fill. Positions never move when a lens changes; only colour does.

#### Authorship

Four kinds, each with a colour **and** a pattern, so they are never told apart by hue alone:

| Kind | Token | Pattern | Meaning |
| --- | --- | --- | --- |
| Human | `human` | Solid | Written by a person |
| AI | `ai` | Diagonal hatch | Written by an agent |
| Mixed | `mixed` | Dots | Written by an agent, then changed by a person |
| Unknown | `unknown` | Hollow outline | No evidence; never counted as human |

Human and AI also differ in lightness in both themes. Authorship colours are marks, not text colours: label them in `ink`. Show the authorship legend the first time authorship appears in a view.

#### Braid's own marks

`accent` (teal) is reserved for what Braid adds: links it draws between things (change-set stitches, pinned notes, selection), the focus ring and the single primary action. Don't use it for decoration or for status. `accent-soft` backs selected rows and intent notes; text on it is `ink`.

#### Status

`ok`, `warn` and `crit` mean states only, and always come with a word or icon: `warn` for anything waiting on a person or overlapping, `crit` for conflicts and broken callers, `ok` for merged and passing. Never use them for authorship or decoration.

#### Evidence grades

Every authorship claim carries a grade: A captured, B observed, C imported, D weak signal, U unknown. Use the GradeBadge; strength reads from the fill, and the letter is always there.

#### Lenses

Lens scales run `lens-1` to `lens-5`, one hue, light to dark, so more always means darker. Every lens shows its legend and a one-line note on how it is measured.

#### Type

- `display`, `display-xl`, `title` in Archivo: names and big numbers. Wide and confident, used sparingly.
- `heading`, `body`, `body-sm`, `label`, `map-label` in IBM Plex Sans: all interface text.
- `code`, `code-sm`, `eyebrow` in IBM Plex Mono: code, paths, SHAs, counts in rows, and small capital eyebrows above titles.

The product bundles the three families as local font files (all under the SIL Open Font Licence) and never requests fonts from the network; only these design-system previews load them from Google Fonts. The stacks fall back to system faces.

#### Space, layout and shape

A 4px base: `space-1` to `space-8`. The top bar is `topbar-height`, the left rail `rail-width`, side panels `panel-width`, and the change story reads in a `story-width` column. The map fills everything else, edge to edge. Corners: `radius-sm` for chips and regions, `radius-md` for buttons, cards and panels, `radius-lg` for floating panels and the command palette, `radius-pill` for markers, status pills and the lens switcher.

#### Borders before shadows

Separate with `line` and space first. `shadow-raised` only for panels floating over the map, `shadow-pop` only for menus, tooltips and the command palette.

#### Strokes

`stroke-hairline` for dividers, `stroke-mark` for icons and connectors, `stroke-strong` for selection, the focus ring and links Braid draws.

#### Motion

Motion explains change, never decorates. Agent markers glide between files in 200 ms; panels slide in 160 ms; a lens fades in 120 ms; the layout never animates for more than 300 ms. With reduced motion, everything switches instantly.

#### Focus and accessibility

The focus ring is `stroke-strong` in `accent`, offset 2px, on every interactive element; it holds 3:1 on every surface in both themes. Text pairs named in token notes meet 4.5:1 in both themes. Every action is reachable from the keyboard and from the command palette (⌘K). Agent markers and map regions carry accessible names ("Claude Code in wt-2, waiting for permission").

### Iconography

Braid has no icon set of its own yet. Until it does, use Lucide (ISC licence) at 16px in rows and 20px in the rail, stroke `stroke-mark`, coloured `ink-muted` or `ink`. This is a substitute, not a brand decision; replace it when Braid's own set exists.

Agent tools are never shown with their vendors' logos. Use two-letter monograms in the AgentMarker: CC Claude Code, CX Codex, CU Cursor, GH Copilot, OC OpenCode.

The Braid mark is two crossing strands, one in `human` blue and one in `ai` magenta. Use the light version on light surfaces and the dark version on dark ones; minimum size 22 × 16 px. The wordmark is "Braid" set in Archivo at weight 800 next to the mark; there is no drawn wordmark file.


### Component families

- **Button**, **Badge**: actions and states.
- **GradeBadge**, **AuthorshipKey**, **ProvenanceRibbon**: who or what wrote the code, and how sure Braid is.
- **AgentMarker**, **SessionCard**, **InboxItem**: agents on the map, in the fleet and waiting in the inbox.
- **LensBar**, **MapRegion**: the living map.
- **ChangeChip**, **RiskItem**: the change story.
- **IntentNote**: the intent canvas (v1).

Components are CSS classes in `docs/design/components.css` with the prefix `bd-`; each component's guidelines give its markup. The product UI implements them as Svelte components with the same class names.

The Braid mark is two crossing sine strands (viewBox `0 0 22 16`): `M1 8 C6 0, 10 16, 15 8 S 21 2, 21 2` stroked in `human`, and `M1 8 C6 16, 10 0, 15 8 S 21 14, 21 14` stroked in `ai`, both 2px with round caps. Light surfaces use `#1f5c99` and `#d24fab`; dark surfaces `#4f86c9` and `#f7a8dd`.

## Tokens

### Colour

| Token | Light | Dark | Use |
| --- | --- | --- | --- |
| `surface` | `#edf1f0` | `#0c1318` | Page and map background, the paper everything sits on. |
| `surface-raised` | `#f9fbfa` | `#15212a` | Panels, cards, menus, label plates on the map. |
| `surface-sunken` | `#e1e8e6` | `#111b22` | Inputs, wells, the gutter between map regions. Body text on it uses ink or ink-muted, never ink-faint or accent. |
| `region-a` | `#f4f7f6` | `#132029` | Map regions at odd nesting depth (repo, folder). |
| `region-b` | `#e8eeec` | `#18262f` | Map regions at even nesting depth (package, file group). |
| `line` | `#c6d1cf` | `#2e3e4a` | Hairlines and dividers that carry no meaning. Never the only edge of a control. |
| `line-strong` | `#6f7e85` | `#5a6e7b` | Control borders and region outlines; at least 3:1 on surface, surface-raised and surface-sunken. |
| `ink` | `#13202a` | `#e3ebee` | Primary text on every surface, on accent-soft and on lens-1 to lens-3. |
| `ink-muted` | `#4a5a65` | `#9aacb5` | Secondary text and metadata on every surface. |
| `ink-faint` | `#5e6d75` | `#7e909a` | Tertiary text (timestamps, counts) on surface and surface-raised only. |
| `accent` | `#0b7a75` | `#3cc2b8` | Braid's own marks: links Braid draws (stitches, pins, selection), the focus ring, the primary action. As text, on surface and surface-raised only. |
| `accent-soft` | `#d2eae7` | `#103a39` | Selected rows, intent notes, the tint behind accent text or icons. |
| `on-accent` | `#ffffff` | `#06201f` | Text and icons on an accent fill. |
| `human` | `#1f5c99` | `#4f86c9` | Code written by a person. Always solid. A mark (3:1), not a text colour. |
| `ai` | `#d24fab` | `#f7a8dd` | Code written by an agent. Always drawn with the diagonal hatch. A mark, not a text colour. |
| `mixed` | `#7552c4` | `#a98bef` | Agent-written code a person then changed. Always drawn with the dot pattern. A mark, not a text colour. |
| `unknown` | `#737f86` | `#7f8e96` | No evidence of authorship. Always drawn hollow (outline only). Never counted as human. |
| `ok` | `#2a7350` | `#4cc38a` | Success states, always with a word or icon. As text on surface and surface-raised. |
| `warn` | `#8f5d06` | `#e8a93a` | Needs attention: waiting agents, overlaps. Always with a word or icon. |
| `crit` | `#b83226` | `#f0705f` | Conflicts, failures, broken callers. Always with a word or icon. |
| `lens-1` | `#ddefec` | `#12302f` | Lens scale, lowest step. Map labels sit on a surface-raised plate, never directly on lens fills. |
| `lens-2` | `#a9d8d1` | `#17504c` | Lens scale, step 2. |
| `lens-3` | `#6dbdb3` | `#1e7069` | Lens scale, step 3. |
| `lens-4` | `#2f9a90` | `#2e958c` | Lens scale, step 4. |
| `lens-5` | `#0b7a75` | `#3cc2b8` | Lens scale, highest step. |

### Type

| Family token | Stack | Use |
| --- | --- | --- |
| `font-display` | "Archivo", "Arial Narrow", system-ui, sans-serif | Names and big numbers |
| `font-sans` | "IBM Plex Sans", system-ui, -apple-system, "Segoe UI", sans-serif | All interface text |
| `font-mono` | "IBM Plex Mono", ui-monospace, SFMono-Regular, Menlo, Consolas, monospace | Code, paths, SHAs, counts, eyebrows |

| Style | Family | Size / line | Weight | Tracking | Use |
| --- | --- | --- | --- | --- | --- |
| `display-xl` | display | 56px / 56px | 800 | -0.02em | The product name on covers and the first-run screen. |
| `display` | display | 36px / 40px | 760 | — | Workspace name, big numbers on dashboards. |
| `title` | display | 22px / 28px | 700 | — | View titles, change-set names. |
| `heading` | sans | 17px / 24px | 600 | — | Panel and section headings. |
| `body` | sans | 15px / 22px | 400 | — | Default UI and prose text. |
| `body-sm` | sans | 13px / 18px | 400 | — | Dense rows, cards, metadata. |
| `label` | sans | 12px / 16px | 600 | 0.01em | Buttons, chips, map labels. |
| `map-label` | sans | 11px / 14px | 500 | — | Folder and file names on the map at small zoom; always on a label plate. |
| `code` | mono | 13px / 20px | 400 | — | Code, diffs, symbol names. |
| `code-sm` | mono | 12px / 16px | 400 | — | Paths, SHAs, worktree names, counts in rows. |
| `eyebrow` | mono | 11px / 14px | 500 | 0.12em | Small uppercase labels above titles. Set in capitals. |

### Spacing

| Token | Value | Use |
| --- | --- | --- |
| `space-1` | `4px` | Gap inside chips, between an icon and its label. |
| `space-2` | `8px` | Gap between related items, padding of small controls. |
| `space-3` | `12px` | Padding of rows and cards. |
| `space-4` | `16px` | Panel padding, gap between groups. |
| `space-5` | `24px` | Section spacing inside a panel. |
| `space-6` | `32px` | Outer margins of full views. |
| `space-7` | `48px` | Top bar height, large separations. |
| `space-8` | `64px` | Empty-state spacing. |

### Radius

| Token | Value | Use |
| --- | --- | --- |
| `radius-sm` | `4px` | Chips, grade badges, ribbon ends, map regions. |
| `radius-md` | `8px` | Buttons, cards, inputs, panels. |
| `radius-lg` | `12px` | Floating panels, the command palette, dialogs. |
| `radius-pill` | `999px` | Agent markers, status pills, the lens switcher. |

### Stroke

| Token | Value | Use |
| --- | --- | --- |
| `stroke-hairline` | `1px` | Dividers, region outlines at far zoom. |
| `stroke-mark` | `1.5px` | Icons, connectors, unknown-authorship outlines. |
| `stroke-strong` | `2px` | Selection, focus ring, links Braid draws (stitches, pins). |

### Layout

| Token | Value | Use |
| --- | --- | --- |
| `topbar-height` | `48px` | The one bar across the top of every view. |
| `rail-width` | `56px` | The left icon rail. |
| `panel-width` | `360px` | Side panels: inbox, fleet, details. |
| `story-width` | `720px` | Reading column of the change story. |

### Shadow

| Token | Value | Use |
| --- | --- | --- |
| `shadow-raised` | `light: 0 1px 0 #d9e1df, 0 18px 40px -26px rgba(10, 30, 40, 0.45) · dark: 0 1px 0 #1d2932, 0 18px 40px -26px rgba(0, 0, 0, 0.8)` | Floating panels over the map. Borders first; shadow only for things that float. |
| `shadow-pop` | `light: 0 8px 24px -8px rgba(10, 30, 40, 0.35) · dark: 0 8px 24px -8px rgba(0, 0, 0, 0.7)` | Menus, tooltips, the command palette. |

## Components

Known gap: a few component styles in `components.css` use type sizes outside the scale above (14px/20px semibold, 12px/16px in the display face, 10px/14px). When the Svelte components are built, map each to a type token or add the token to `tokens.json` first; don't copy the raw sizes.

### Button

Use a Button for an action a person takes; at most one **primary** per view, for the thing the view is for. Labels are a verb first in sentence case ("Approve", "Send to Claude Code"). Show the shortcut with `bd-kbd` when one exists.

- Markup: `<button class="bd-btn">`, plus `bd-btn--primary`, `bd-btn--danger`, `bd-btn--ghost` or `bd-btn--sm`.
- The consumer provides the label, an optional shortcut and the handler.
- Danger is for actions that lose work (remove a worktree with changes); it always asks to confirm.
- Never use colour alone to mark a button as primary: primary is also the only filled one.

### Badge

A Badge names a state in a word, with a dot in the state's colour; the word carries the meaning, the colour only reinforces it.

- Markup: `<span class="bd-badge bd-badge--ok">Merged</span>`; tones `--ok`, `--warn`, `--crit`, `--accent`, or none for neutral.
- `warn` is for anything waiting on a person or overlapping; `crit` for conflicts and broken callers; `accent` for Braid's own states such as "In plan".
- Keep the word to one or two: "Working", "Needs you", "Conflict", "Ready".

### GradeBadge

Shows the evidence grade behind an authorship claim: **A** captured from a hook, **B** observed during a session, **C** imported from another tool, **D** a weak signal, **U** unknown. Strength is shown by the fill (solid for A, fading to dotted for U) and the letter is always present.

- Markup: `<span class="bd-grade bd-grade--a" title="Captured from a hook">A</span>`.
- Always give a `title` or nearby text with the grade's meaning the first time it appears in a view.
- Never hide U: unknown authorship is shown, counted separately, and never treated as human.

### AuthorshipKey

The four authorship fills and their legend. Every authorship mark in Braid uses one of these fills, so colour is never the only difference: human is solid, AI is hatched, mixed is dotted, unknown is hollow.

- Markup: `<i class="bd-auth bd-auth--ai"></i>` for a swatch; wrap swatches and words in `bd-authkey` for a legend.
- Show the legend wherever authorship appears for the first time in a view.
- Mixed means an agent wrote it and a person changed it afterwards; say so in tooltips.

### ProvenanceRibbon

A thin bar showing who or what wrote a file or a pull request, segments in reading order with widths proportional to lines. Use it in change stories, file headers and the fleet.

- Markup: a `bd-ribbon` holding `bd-auth` segments, each with `style="flex: <lines>"`; in a list, put it in a `bd-ribbon-row` with the path and the counts.
- The consumer provides the segments (kind and line count) and the totals.
- Always pair it with numbers ("38% AI · 212 of 558 lines"); the bar alone is never the only statement.

### AgentMarker

Places an agent session on the map: a circle with the tool's two-letter monogram and a ring that shows its state. Never use a vendor's logo; monograms are CC (Claude Code), CX (Codex), CU (Cursor), GH (Copilot), OC (OpenCode), plus the first two letters for others.

- Markup: `<span class="bd-agent">CC</span>`; states `--waiting` (warn ring and a "!" badge), `--done` (quiet ring), `--conflict` (crit ring). Working is the default accent ring.
- A trail of the files it read and edited is drawn with `bd-agent-trail` on an SVG path.
- Give every marker an accessible name: "Claude Code in wt-2, waiting for permission".

### SessionCard

One agent session in one worktree, as shown in the fleet panel: tool and task, where it works, what it touches, how much it changed and for how long.

- Markup: `article.bd-session` with a `header` (marker, task name, state badge), `.bd-where` (repo and worktree), `.bd-files` (up to three paths, then "+n more") and a `footer` (added and removed lines, elapsed time).
- The consumer provides the session, its worktree, its state and its counts.
- Clicking the card selects the session on the map; it never opens the agent itself.

### InboxItem

Something waiting on a person: a permission prompt, a question, finished work ready for review, or a predicted conflict. The inbox is a list of these, newest first, and empty is the goal.

- Markup: `div.bd-inbox` with a marker, a `strong` line saying what is needed, a `.bd-ctx` line (repo, worktree, file) and `.bd-actions`.
- Say what is needed in plain words: "Wants to run npm test", not "PermissionRequest".
- Answering a permission prompt happens in the agent's own window; the action here jumps there. Braid never answers on the person's behalf.

### LensBar

Switches what the map is coloured by, and always shows the legend for the active lens. Lenses never change where things are, only their colour.

- Markup: `div.bd-lensbar` of `button.bd-lens` with `aria-pressed`; the legend is a `bd-legend` of `bd-l1` to `bd-l5` swatches between a low and a high label.
- Lens scales use `lens-1` to `lens-5` only; authorship lenses use the authorship fills instead.
- Every lens has a one-line "how it's measured" note, reachable from the legend.

### MapRegion

A region of the living map: a repo, package, folder or file, nested inside its parent. Regions alternate `region-a` and `region-b` by depth; a lens replaces the fill with a lens step; selection and overlap draw an inset ring.

- Markup: `div.bd-map` containing absolutely positioned `div.bd-region`, each with a `span.bd-plate` label (`bd-plate--repo` for repo names).
- Labels always sit on a plate, never directly on a lens fill, so they stay readable at every step.
- Selection uses `bd-region--sel` (accent ring); two agents overlapping uses `bd-region--overlap` (warn ring) plus a badge naming the overlap.

### ChangeChip

Names the kind of a change in a change story: formatting, rename, move, logic or API. Logic and API changes are the ones that need reading; formatting is dashed and collapsed by default.

- Markup: `<span class="bd-change bd-change--logic">logic</span>`; kinds `--format`, `--logic`, `--api`, or none for rename and move.
- The word is always shown in lower case in mono, so chips line up in lists.
- API marks a changed export; pair it with the number of callers not updated.

### RiskItem

One row of "look here first" in a change story: a risk score and the reasons that put it there. The score orders attention; it never blocks a merge.

- Markup: `div.bd-risk` with `.bd-score`, a `strong` title (symbol or file), and `.bd-why` holding ChangeChips and Badges for each signal.
- Every row says why it is there; a score without reasons is not shown.
- Rows link to the exact hunk, in dependency order.

### IntentNote

A note on the intent canvas, pinned to code: what we want, why, and the acceptance criteria an agent will be held to. Arrives in v1.

- Markup: `article.bd-note` with an `h4` title, a `.bd-pin` line naming what it is pinned to, and a `ul` of criteria (`li.done` once covered).
- A note whose code was deleted becomes `bd-note--orphan` (dashed warn border) until someone re-pins it.
- Keep criteria short and testable: "Never retries a 4xx response".

## Screens

Six screens on the Braid Screens canvas, each with a light/dark Theme tweak. All repos, people, agents and numbers are sample data (the acme-commerce workspace from the Concept Atlas).

| Screen | Release | What it shows |
| --- | --- | --- |
| Map home | MVP | The living map with the lens bar, agent markers, an overlap warning on a region, the fleet panel and the inbox. Answers J1–J4 from Phase 2. The Cursor marker (CU) is sample data; Cursor support arrives in v1. |
| Change story | MVP first version, full in v1 | Picture before diff, drawn in its full v1 form. The MVP first version is only what the Phase 2 criterion lists: touched areas on a mini-map, files grouped by area, a provenance ribbon per file, whether tests changed, then the diff. Change chips, "Look here first", criteria coverage and comments to agents arrive in v1. Answers J5, and J8 in v1. |
| Intent canvas and send to agent | v1 | Notes, frames and arrows drawn on the map, pinned to code, with acceptance criteria and a send-to-agent dialog with an overlap warning. Answers J6 and J7. |
| Line view and why-links | MVP blame, links in v1 | Code panel with the provenance gutter, evidence grades and why-links from lines to intents and sessions. Answers J9. |
| First run | MVP | The install-to-map path: workspace detection, the hook changes `braid install` will make, and the map filling in. The five-minute activation journey. |
| Braid card on a pull request | v1 | A 1200×630 image `braid card` renders in CI: what changed, where it lands across repos, AI share and the top risks. Answers J12. |

Open the canvas for the screens themselves: https://claude.ai/artifact/GipvFcyUhbhdMLKXUjP6fe
