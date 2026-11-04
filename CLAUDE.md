# Braid

Braid is an open-source visual workspace for building software with AI agents: one live map of a developer's repos with their agents working on it, line-level AI provenance, and review as a picture before the diff. Local-first: a Rust daemon (`braidd`) plus one Svelte web UI shown in every shell.

The full plan lives in `docs/plan/` (read the file for the area you touch before changing it):

- `docs/plan/01-vision.md` — what Braid is, principles, idea space, positioning
- `docs/plan/02-product.md` — personas, jobs, journeys, feature catalogue by release, MVP acceptance criteria, metrics
- `docs/plan/03-architecture.md` — Braid Graph, storage, event pipeline, map layout, agents, collision radar, provenance, change story, interfaces, tech choices, budgets, security, testing, ADR list
- `docs/plan/04-design.md` — design system rules, tokens, components, screens
- `docs/plan/05-execution.md` — milestones, next tasks, repo rules, launch plan
- `docs/design/tokens.json`, `docs/design/components.css` — the design system's tokens and `bd-` component classes

## Current state

- v0.1 (built 30 Oct 2026) is the base: about 7,500 lines of Rust, 52 tests. If the Rust workspace is not in this folder yet, import it from `braid.bundle` (keep its history) before anything else.
- v0.1 crates: `braid-git`, `braid-prov`, `braid-ws`, `braid-cli`, `braidd`, plus the Oxc-based JS/TS diff. Binaries: `braid` (CLI) and `braidd` (daemon on 127.0.0.1:7471).
- Works from the CLI: workspaces (`braid ws init`, `braid status`; M1 replaces `braid ws init` with `braid init` and keeps the old name as an alias), worktrees and collision checks (`braid wt`), capture from Claude Code, Codex and Cursor hooks, provenance sealed as Git AI Standard v3 notes that survive rebase, amend, squash and cherry-pick (`braid carry` after forge squash-merges), `braid blame`, `braid diff` with export surface and cross-repo blast radius, an MCP server with 8 tools, a Claude Code plugin.
- Current milestone: **M0 Foundations** (2–13 Nov 2026), then **M1 Map**. Tasks are in `docs/plan/05-execution.md`, section "The next two weeks".

## Architecture in brief

- One local daemon per machine owns git, file watching, parsing, provenance and storage. Every surface (desktop app, browser tab, VS Code, CLI) is a thin client.
- Everything is a node or edge in the **Braid Graph** (code, work, people and tools). Nodes have random stable IDs (ULID) that survive renames and moves; never derive an ID from a path.
- Storage: derived data in SQLite under `~/.braid` (rebuildable); sealed provenance in `refs/notes/ai` (Git AI compatible); Braid's own links in `refs/notes/braid`; intents and tasks as Markdown files in `.braid/intents/` in the repo; shared map anchors in `.braid/map.json`.
- Updates flow: source event → append-only event log → incremental graph update (salsa) → derived views → versioned diffs over one WebSocket stream.
- The map is a containment map (nested regions), anchored at the top tiers, ordered strip treemap inside packages. Links are drawn on demand. Rendering: deck.gl on WebGL2; WebGPU only as a later upgrade.
- New crates planned: `braid-graph`, `braid-map` (also built for WebAssembly), `braid-story`, `braid-intent`, `braid-stream`; plus `ui/` (Svelte 5 + TypeScript + Vite) and `desktop/` (Tauri 2).

## Rules

- Rust edition 2024. `cargo fmt`, `cargo clippy --workspace --all-targets -- -D warnings` and `cargo test --workspace` must pass before every commit. No `unwrap` outside tests.
- Anything touching git history gets an end-to-end test against real git. Every agent hook format gets a recorded payload fixture.
- Agent hooks must answer in under 20 ms: log the event, reply, do the work afterwards.
- The daemon binds 127.0.0.1 only, checks the Host header and requires the per-install token. No network calls the user did not configure. Telemetry is opt-in and counts only.
- Never auto-approve or answer an agent's permission prompt. The inbox routes requests; the agent decides.
- Braid holds no AI API keys. AI steps (sketch to plan, ask the map) run through the user's own agent CLI.
- Unknown authorship stays unknown: never count it as human. Every authorship claim carries an evidence grade (A captured, B observed, C imported, D weak signal, U unknown).
- UI: Svelte 5 and TypeScript. Colours, type, spacing, radii and shadows come only from `docs/design/tokens.json` (as CSS variables); reuse the `bd-` classes from `docs/design/components.css`. Authorship is always colour plus pattern (human solid, AI hatched, mixed dotted, unknown hollow). Every view works in light and dark and from the keyboard. Fonts (Archivo, IBM Plex Sans, IBM Plex Mono, all SIL OFL) are bundled as local files; the product never loads Google Fonts or any other remote asset, so don't copy the `@import` from the design system's previews.
- Licences: apps Apache-2.0, crates `MIT OR Apache-2.0`. Never add tldraw, `@cosmograph/*`, AGPL or other copyleft dependencies; EPL or MPL libraries only unmodified. `cargo-deny` gates Rust crates and a JavaScript licence checker gates npm packages in CI (tldraw and `@cosmograph/*` are npm packages, so `cargo-deny` alone never sees them).
- Commits carry a DCO sign-off (`git commit -s`). One change per commit, with the why in the message.
- An architecture change gets a new ADR in `docs/adr/`; don't rewrite old ones.
- Scope follows the release column in `docs/plan/02-product.md`. Don't build v1 or v2 features during MVP milestones; open an issue instead.

## Definition of done

The feature's acceptance criterion in `docs/plan/02-product.md` passes, tests cover it, the budgets in `docs/plan/03-architecture.md` still hold, it works in both themes and by keyboard, and it has been used on a real repo for a day.
