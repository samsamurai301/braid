<!-- Exported from https://claude.ai/artifact/7u6PxvFDQcCgcdMtWjCHCQ on 31 Oct 2026. The doc is the living version. -->

# Braid · Phase 5 — Execution Plan

Oct 31, 2026 · @Sam

## Summary

Braid reaches a private MVP alpha in mid-March 2027 and a public beta with a Show HN in mid-June 2027, through four build milestones (M0 to M3) that each end in something you use yourself. The first two weeks decide whether the plan holds: a layout spike, eight user interviews and getting v0.1 onto GitHub.

The dates rest on three assumptions, each checked at the first milestone:

1. **About 20 hours a week of your time, with agents writing most of the code.** v0.1 (about 7,500 lines and 52 tests) was built by agents in one night, so the Phase 2 effort estimates are upper bounds.
2. **v0.1 is the base,** not a throwaway: its capture, provenance, worktree and diff code becomes the daemon's core.
3. **The layout spike passes** the stability test in its first two weeks. If it doesn't, Milestone 1 moves by up to four weeks.

This is the last of five planning docs: [Phase 1 Vision](https://claude.ai/artifact/WMvKpKb71EKkniYqHy2kJ5), [Phase 2 Product](https://claude.ai/artifact/ARXGEtnr8MhKRVCNb8HcAm), [Phase 3 Architecture](https://claude.ai/artifact/Jwg4EG9GrZeiF3QjDvgY6o), Phase 4 Design ([design system](https://claude.ai/artifact/6QfVoaAyTJ9DGUZFNQqfMS) and [screens](https://claude.ai/artifact/GipvFcyUhbhdMLKXUjP6fe)) and this one.

## Roadmap

The three feature milestones (M1 to M3) overlap by a week each: the week you use one milestone is the first week of the next; the layout spike and the interviews run alongside the foundations in the first two weeks.

*[Diagram: Roadmap · November 2026 to June 2027, to scale. See it in the doc: https://claude.ai/artifact/7u6PxvFDQcCgcdMtWjCHCQ]*

The dashed line marks the MVP alpha. Everything to its left is building before any users; v1 then runs for three months to the public beta.

## Milestones

Each milestone ends with something you use on your own repos for at least a week, which overlaps the next milestone's first week; its exit check decides whether that next milestone carries on or stops while you fix what failed.

| Milestone | Dates | Scope (Phase 2 features) | Exit check |
| --- | --- | --- | --- |
| M0 Foundations | 2–13 Nov 2026 | v0.1 pushed to GitHub; CI with fmt, clippy, tests and licence gate; Apache-2.0 and MIT-or-Apache headers; DCO; `CLAUDE.md`; ADR-001 to ADR-010 written; planning docs in `docs/` | Clean CI on `main`; Claude Code can work in the repo from `CLAUDE.md` alone |
| Layout spike | 2–13 Nov 2026 | Ordered strip treemap with anchors, circle packing and Voronoi treemap, each replayed over 100 commits of three real repos | One layout passes the stability test (under 5% average movement); otherwise M1 moves by up to four weeks |
| Validation interviews | 2–20 Nov 2026 | Eight interviews with agent conductors (test A1) | At least six of eight describe a recent overview or conflict incident; otherwise rethink the MVP's centre |
| M1 Map | 16 Nov 2026 – 8 Jan 2027 | Braid Graph and stable IDs, workspace manifest and braid init, file-level living map, stable layout, three lenses, web UI and desktop shell, streaming protocol | You open the map daily for a week on Braid's own repo and one work codebase (the first week of test A2) |
| M2 Fleet | 4 Jan – 12 Feb 2027 | Agent lanes, approval inbox, agents on the map, collision radar on the map, worktree per task in the UI, launch an agent with a brief, activity stream | Two agents run a full day across two repos with no terminal visit needed just to check status |
| M3 Review and polish | 8 Feb – 12 Mar 2027 | Change story first version, provenance blame view, cross-repo package links, fly-to search, snapshot export, MCP read tools, install and uninstall, privacy defaults, performance budgets | Every MVP acceptance criterion in Phase 2 passes; ten real PRs reviewed for test A4 |
| MVP alpha | from 15 Mar 2027 | No announcement yet; install script and a Homebrew tap; ten to fifteen design partners via Discord, enough to reach the gate | Alpha gate: 10 workspaces completing the loop each week |
| v1 build | 15 Mar – 11 Jun 2027 | Intent canvas, pinned notes, criteria, send with visual context, semantic diff and blast radius in the UI, Braid timeline, why-links, Cursor, VS Code extension, Braid cards, public atlas | The core journey from Phase 2 runs end to end, all thirteen steps |
| Public beta | from 14 Jun 2027 | Show HN, VS Code Marketplace and Open VSX, Homebrew | Beta gate: 500 workspaces completing the loop each week, before v2 starts |

## The next two weeks

Fourteen tasks, in this order. The first three take an evening; the spike and the interviews run side by side after that.

**Foundations (M0)**

- [ ] Import v0.1 into `~/Documents/experiments/Braid` from `braid.bundle`, keeping its history, and push to `samsamurai301/braid` (public, not announced)
- [ ] Check that the `CLAUDE.md` and the `docs/` folder written today are in the repo, and commit them
- [ ] Turn on CI: `cargo fmt --check`, `cargo clippy -D warnings`, `cargo test`, `cargo-deny` for Rust licences, plus a JavaScript licence checker for npm packages
- [ ] Add the licence files, a DCO check, `CONTRIBUTING.md` and `SECURITY.md`
- [ ] Write ADR-001 to ADR-010 from Phase 3 into `docs/adr/`
- [ ] Install Braid's own Claude Code plugin in the Braid repo, so Braid records its own development from day one

**Layout spike**

- [ ] Pick three real codebases: Braid itself, one work codebase, one large open-source TypeScript repo of about 10,000 files
- [ ] Build a commit-replay harness that measures how far untouched files move
- [ ] Implement the ordered strip treemap with anchors, then circle packing and a Voronoi treemap for comparison
- [ ] Render each in a throwaway page to judge readability, not only the number
- [ ] Write ADR-003's result: which layout, and the measured stability

**Validation**

- [ ] Write the interview guide (past behaviour only, no pitching) and recruit eight agent conductors: colleagues, Claude Code and Codex communities, German developer meetups
- [ ] Run the eight interviews and note their exact words
- [ ] Decide on the Prototype Fund (below) by 25 November

## How Braid is built

Braid is built the way it wants others to work: you plan and review, agents write most of the code, and Braid watches itself being built. That makes you its first user from week one.

**The loop for every feature**

1. Write a short issue from the Phase 2 catalogue row: the job it serves, scope, acceptance criteria, and which MVP test it must pass.
2. Start the agent in its own worktree with the issue as its brief. Run at most two or three agents at once; the research in Phase 1 shows attention, not agents, is the limit.
3. Review the result through `braid diff` and, from M3, Braid's own change story. Nothing merges unread.
4. Merge, then use it for a day before starting the next feature in the same area.

**Repo rules**

- Rust: edition 2024, `cargo fmt`, `clippy -D warnings`, no `unwrap` outside tests.
- Tests: end-to-end against real git for anything touching history; recorded payload fixtures for every agent hook; the layout stability benchmark in CI from M1.
- UI: Svelte 5 and TypeScript; colours, type, spacing and radii only from the design system's tokens; component class names from its `bd-` CSS.
- Dependencies: nothing under a licence that `cargo-deny` rejects, or that the JavaScript licence checker rejects for npm packages; no network calls the user didn't configure, so fonts are bundled rather than loaded from Google Fonts.
- Every architecture change that alters an ADR gets a new ADR, not an edit to the old one.
- Commits signed off for the DCO; one change per commit, explained in the message.

**Definition of done:** the feature's acceptance criterion from Phase 2 passes, tests cover it, the budgets in Phase 3 still hold, it works in both themes and from the keyboard, and you have used it for a day.

## Open-source launch plan

The repo is public from the start but announced only with the beta, so the first public impression is a product that works, while the build itself is visible all along.

| When | What happens | Why then |
| --- | --- | --- |
| Now to March | Public repo, not announced; short build-in-public posts with screenshots and clips of the map | Visuals are Braid's best marketing; the audience grows before there is anything to install |
| Before the alpha | Trademark search for the final name, word mark filed at the DPMA or EUIPO, trademark policy, code of conduct, AI-contribution policy | Naming disputes and floods of low-quality AI PRs are known failure modes |
| From the alpha (March) | "Early" README; Discord with design partners in small waves; good first issues labelled | Real feedback without a launch-day spike the product can't hold |
| Beta launch (June) | Show HN where open-sourcing is the story; public atlas of well-known repos; VS Code Marketplace and Open VSX; Homebrew | A launch moment needs something people can try in five minutes and pictures they want to share |
| After the beta | A launch week every quarter, shipping one visible feature a day; lens and adapter API opened to contributors | Repeatable moments keep attention; plugins turn users into contributors |

Accounts that the project depends on (domain, GitHub organisation, crates.io and npm, the marketplace publisher, Discord) go under one organisation with at least two admins from the start.

## Funding and the paid tier

The one time-critical item is the Prototype Fund: its funding period (1 June to 30 November 2027) would cover the beta and the start of v2, but applications close on **30 November 2026 at 15:00 CET** ([Prototype Fund](https://www.prototypefund.de/en/application)).

| Source | Amount | Fit and timing | Next step |
| --- | --- | --- | --- |
| Prototype Fund | €47,500 for a solo applicant, 6 months | Requires residence in Germany and working self-employed, freelance or released by an employer; Braid applies as open developer infrastructure | Check eligibility this week; draft by 25 November |
| NLnet | €5,000–50,000 | Open to individuals; needs a European dimension; asks applicants to disclose AI use ([NLnet](https://www.nlnet.nl/propose)) | Watch for the next general call after the alpha |
| Sovereign Tech Fund | Larger grants | Not for user-facing apps or prototypes ([STF](https://sovereign.tech/programs/fund)) | Later, once other tools depend on Braid's core crates |
| GitHub Sponsors | Small, steady | From the public alpha | Set up with the public repo |
| Braid Team (paid) | Per active author, price after design partners | After v1, if test A7 shows demand | Three design-partner conversations before v1 |

The three grants all require the core to stay under an open-source licence, which the plan keeps.

## Execution risks

Product risks are in Phase 1; these are the ways the plan itself can fail, each with the early sign to watch and what to do when it shows.

| Risk | Early warning sign | Response |
| --- | --- | --- |
| Not enough hours: Braid competes with other projects and work | A milestone's first week ends with less than half its tasks started | Cut Should items first; move the alpha date rather than skip exit checks |
| Scope creep from the vision | Work starts on a v1 feature before M3's exit check | Park it as an issue; the catalogue's release column is the contract |
| The layout spike fails | No layout under 5% movement after two weeks | Ship the most stable one with a manual "tidy" and pinned anchors; revisit after the alpha |
| Interviews don't confirm the pain | Fewer than six of eight describe a recent incident | Re-centre the MVP on review (change story and provenance), which v0.1 already supports best |
| Agent hook changes break capture | A recorded fixture fails after an agent update | Fix the adapter first; the watcher keeps grade B coverage meanwhile |
| Agents write code nobody understands, in Braid itself | Change stories show large AI-written areas without tests | Hold merges until tests exist; use Braid's own lenses on Braid weekly |
| Burnout or isolation | Two weeks without using Braid yourself | Shrink the next milestone; ask a design partner to co-review |

## Decisions waiting on you

Five calls are yours, and the plan already follows each recommendation. Two are effectively made: Phase 1 names agent conductors as the first user, and Phase 3 and CLAUDE.md treat Svelte as decided, so they need only your yes or a change of mind. The other three are still open. The first four also sit as comments in the earlier docs.

| Decision | Recommendation | Where it's asked | Needed by |
| --- | --- | --- | --- |
| First user: agent conductors or agent-first builders | Agent conductors first | Phase 1 | Before the interviews |
| MVP centre: fleet first or map first | Fleet first, on a simple map | Phase 2 | End of M0 |
| UI framework: Svelte or React | Svelte, for small bundles across webview hosts | Phase 3 | Start of M1 |
| Apply to the Prototype Fund | Yes, if you meet the eligibility rule | Phase 1 | 25 November |
| Final name | Keep Braid if the trademark search is clear | Here | Before the alpha |

## CLAUDE.md handoff

This file sits at the root of `~/Documents/experiments/Braid`, so Claude Code knows the project, its current milestone and its rules from its first session. Markdown copies of all five planning docs and the design tokens sit beside it in `docs/`.

```markdown
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
- Works from the CLI: workspaces, worktrees and collision checks, capture from Claude Code, Codex and Cursor hooks, provenance sealed as Git AI Standard v3 notes that survive rewrites, `braid blame`, `braid diff` with export surface and cross-repo blast radius, an MCP server with 8 tools, a Claude Code plugin.
- Current milestone: M0 Foundations (2–13 Nov 2026), then M1 Map. Tasks are in `docs/plan/05-execution.md`.

## Architecture in brief

- One local daemon per machine; every surface is a thin client.
- Everything is a node or edge in the Braid Graph. Nodes have random stable IDs that survive renames; never derive an ID from a path.
- Derived data in SQLite under ~/.braid; sealed provenance in refs/notes/ai; Braid links in refs/notes/braid; intents in .braid/intents/; shared map anchors in .braid/map.json.
- Event → append-only log → incremental graph update → derived views → versioned diffs over one WebSocket.
- Containment map, anchored top tiers, ordered strip treemap inside packages; deck.gl on WebGL2.

## Rules

- Rust 2024; fmt, clippy -D warnings and tests pass before every commit; no unwrap outside tests.
- Git-history code gets end-to-end tests on real git; every hook format gets a recorded fixture.
- Hooks answer in under 20 ms. Daemon on 127.0.0.1 with Host check and token. No unconfigured network calls.
- Never answer an agent's permission prompt. Braid holds no AI API keys.
- Unknown authorship is never counted as human; every claim carries an evidence grade.
- UI values only from docs/design/tokens.json; authorship is colour plus pattern; light, dark and keyboard always.
- Apache-2.0 apps, MIT OR Apache-2.0 crates; no tldraw, Cosmograph or copyleft dependencies (cargo-deny and a JavaScript licence checker gate this in CI). Fonts are bundled; the product never loads Google Fonts.
- DCO sign-off; new ADR for each architecture change; scope follows the release column.

## Definition of done

Acceptance criterion passes, tests cover it, budgets hold, works in both themes and by keyboard, used on a real repo for a day.
```

The copy in the folder is the full version; the one above shortens the rules for reading here.

## Planning set and sources

| Phase | Artifact | In the repo |
| --- | --- | --- |
| 1 · Vision and ideation | [Doc](https://claude.ai/artifact/WMvKpKb71EKkniYqHy2kJ5) | `docs/plan/01-vision.md` |
| 2 · Product definition | [Doc](https://claude.ai/artifact/ARXGEtnr8MhKRVCNb8HcAm) | `docs/plan/02-product.md` |
| 3 · Logic and architecture | [Doc](https://claude.ai/artifact/Jwg4EG9GrZeiF3QjDvgY6o) | `docs/plan/03-architecture.md` |
| 4 · Design | [Design system](https://claude.ai/artifact/6QfVoaAyTJ9DGUZFNQqfMS), [screens](https://claude.ai/artifact/GipvFcyUhbhdMLKXUjP6fe) | `docs/plan/04-design.md`, `docs/design/` |
| 5 · Execution | This doc | `docs/plan/05-execution.md`, `CLAUDE.md` |
| Earlier | [Braid Concept Atlas](https://claude.ai/code/artifact/fc2b337b-e431-405f-af39-bb869214eaa8) | — |

Sources checked on 31 October 2026: [Prototype Fund application](https://www.prototypefund.de/en/application) · [NLnet](https://www.nlnet.nl/propose) · [Sovereign Tech Fund](https://sovereign.tech/programs/fund) · [Biome on losing project accounts](https://biomejs.dev/blog/annoucing-biome) · [Ghostty AI policy](https://github.com/ghostty-org/ghostty/blob/main/AI_POLICY.md) · [Supabase launch weeks](https://supabase.com/blog/supabase-how-we-launch). Dates in this plan are estimates under the assumptions in the summary.
