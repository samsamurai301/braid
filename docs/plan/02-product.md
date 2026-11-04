<!-- Exported from https://claude.ai/artifact/ARXGEtnr8MhKRVCNb8HcAm on 31 Oct 2026. The doc is the living version. -->

# Braid · Phase 2 — Product Definition

Oct 31, 2026 · @Sam

## Summary

The first release, **Braid MVP "Map & Fleet"**, gives an agent conductor one live map of their JS/TS repos with every running agent placed on it, collisions flagged before they happen, line-level AI provenance, and a first picture-before-diff review. Thinking tools (the intent canvas and send-to-agent with criteria) arrive in v1; sharing, boundaries and other languages in v2; organisation features in a separate paid tier.

This doc builds on [Phase 1 — Vision & Ideation](https://claude.ai/artifact/WMvKpKb71EKkniYqHy2kJ5). It answers Phase 1's open questions with a recommendation each; any of them can be overruled before Phase 3 starts.

| Phase 1 question | Recommendation | Reason |
| --- | --- | --- |
| Map first or fleet first? | Fleet first, drawn on a simple map | Running agents is the daily pain; a map people visit daily beats a map they admire once |
| Which shell first? | A local web UI served by the daemon, wrapped in a thin desktop app; VS Code panel in v1 | One UI to build, no marketplace review in the critical path |
| Does send-to-agent launch agents? | MVP launches an agent's CLI in a new worktree with a brief; v1 adds criteria and visual context | Real dispatch without building a terminal emulator |
| Which agents at launch? | Claude Code and Codex; Cursor in v1 | Both have stable edit hooks and v0.1 already captures Claude Code |
| How deep does the first map go? | Down to files; functions in v1 | File level needs no semantic parser and still shows where agents work |
| Multiplayer canvas? | Open-source, but not before v2 | Single-player has to be loved first |
| Simpler mode for agent-first builders? | Same interface with friendlier defaults, from v2 | One product, tuned later from real use |
| Which number shows it works? | Weekly active workspaces that complete the loop | Counts real use, not installs or stars |

## Personas

Four people anchor every product decision; Lena is primary until v1 ships. These are composites built from Phase 1's research, not interviews yet, and the validation plan below replaces guesses with real people.

### Lena, the agent conductor (primary)

- **Setup:** staff engineer at a 120-person SaaS company. Seven repos: Next.js storefront, NestJS API, shared SDK, design system, React Native app, Terraform, docs. Runs Claude Code in three worktrees and Codex in another, and reviews about fifteen PRs a day, most written by agents.
- **Wants:** to keep four agents busy without them colliding, to know every morning what changed where, and to merge with confidence.
- **Frustrations:** juggling terminals; conflicts discovered at rebase time; after a squash merge nobody can tell which lines an agent wrote; reviewing diffs file by file until her attention runs out.
- **In her words:** "I'm not short of code. I'm short of overview."
- **Braid works for her when:** it's the first window she opens each morning, conflicts stop surprising her, and a change set takes less time to review.

### Marco, the tech lead

- **Setup:** leads eight developers on a payments platform and owns its architecture. His team uses whichever agents they like.
- **Wants:** the architecture to stay intact while output rises; to know where risk is concentrated; rules that agents respect without being reminded.
- **Frustrations:** architecture diagrams are stale within weeks; boundary violations surface in review or production; he can't see how much of the payments code an AI wrote.
- **In his words:** "Every agent is a new junior who never read the architecture doc."
- **Braid works for him when:** he checks the map's lenses weekly, and boundaries he drew once are checked on every change.

### Aisha, the agent-first builder

- **Setup:** product designer turned founder, building her SaaS with Claude Code. Reads some code, not all of it.
- **Wants:** to understand what has been built and where it lives, so she can ask agents for precise changes and avoid breaking things.
- **Frustrations:** the codebase is a black box; agents rebuild things that already exist; she can't tell which changes are risky.
- **In her words:** "I can ship features. I can't tell you how my app works."
- **Braid works for her when:** she points agents at the right place on the map instead of describing it in words.

### Jonas, the open-source maintainer

- **Setup:** maintains a popular TypeScript library with about forty regular contributors and a growing stream of AI-assisted pull requests from strangers.
- **Wants:** to review outside contributions fast and to show new contributors where to start.
- **Frustrations:** large AI-written PRs nobody tested; explaining the project's structure again and again.
- **In his words:** "Half my pull requests are written by agents for people who never ran the tests."
- **Braid works for him when:** every PR arrives with a Braid card showing where it lands and who or what wrote it, and the README carries a live map.

### Not for, yet

- Managers looking to rank individual developers. Braid will never offer it.
- Teams that don't use coding agents. The map alone exists elsewhere.
- Codebases without JS or TS until v2 adds tree-sitter languages.

## Jobs to be done

Twelve jobs, written as "when …, I want to …, so I can …". The MVP serves the first five completely; the rest arrive in v1 and v2. A feature that serves none of these jobs doesn't get built.

| # | Job | Who | Moment | Served from |
| --- | --- | --- | --- | --- |
| J1 | When I start my day, I want to see what changed across all my repos and who or what changed it, so I can decide where to look first | Lena, Marco | See, Watch | MVP |
| J2 | When several agents are running, I want to see where each one is working, so I can keep them out of each other's way | Lena | Watch | MVP |
| J3 | When two pieces of work are about to touch the same code, I want to know before they conflict, so I can re-scope or sequence them | Lena | Watch, Direct | MVP |
| J4 | When an agent needs me, I want one place that shows every waiting request, so I don't leave agents idle | Lena | Direct | MVP |
| J5 | When I review agent work, I want to see where it landed and which lines the agent wrote before reading the diff, so I can spend attention where the risk is | Lena, Marco, Jonas | Review, Remember | MVP (first version) |
| J6 | When I have a feature in mind, I want to sketch it on the code it touches and turn it into tasks, so agents start with the right scope | Lena, Aisha | Think | v1 |
| J7 | When I hand work to an agent, I want to give it exactly the right context and acceptance criteria, so it gets it right the first time | Lena, Aisha | Direct | v1 |
| J8 | When a change alters an export, I want to see every caller in every repo, so nothing breaks downstream | Lena, Marco | Review | v1 |
| J9 | When I find code I don't understand, I want to see why it exists and who or what wrote it, so I can change it safely | everyone | Remember | v1 |
| J10 | When I join or inherit a codebase, I want a guided picture of how it fits together, so I'm productive in days, not weeks | Aisha, newcomers | See | v2 |
| J11 | When agents work on critical code, I want rules about what may depend on what, enforced automatically, so the architecture holds | Marco | Think, Govern | v2 |
| J12 | When I show my work to others, I want a picture of the change that explains itself, so reviewers and readers get it at a glance | Jonas, everyone | Share | v1 |

## Core journey

The core journey is the loop from Phase 1 played by Lena with two agents. The person decides and judges; Braid carries everything in between; the agents only build.

*[Diagram: Core journey · plan, dispatch, watch, review · 13 steps across three lanes. See it in the doc: https://claude.ai/artifact/ARXGEtnr8MhKRVCNb8HcAm]*

The person appears at five points and never has to visit a terminal: to state the intent, hand out work, answer requests, review and merge. In the MVP, Lena types the task instead of sketching it, and Braid starts from step 3.

## Other journeys

Four more journeys cover the first minutes, the daily habit, review and learning a codebase. Each has one moment of truth: the step that decides whether the person comes back.

### First run: a map of my repos in under five minutes, no account

1. Install with one command: `brew install braid`, or the curl or cargo equivalents.
2. Run `braid init` in a folder of repos. Braid finds the git repos, groups them (apps, services, libraries) from `package.json` and folder names, and writes `braid.toml`.
3. Run `braid install`. It shows exactly which agent hooks, git hooks and notes settings it will add, per repo, and asks once.
4. Run `braid open`. The desktop app opens and the map fills in as the daemon indexes: repos first, then folders, then files.
5. An empty state offers two paths: start an agent in a fresh worktree from Braid, or keep working as usual and let Braid notice.

**Moment of truth:** seeing your own repos drawn as a map within a minute of `braid open`.

### Morning check: what happened, and what needs me

1. Open Braid. A banner sums up since yesterday: change sets, finished agent sessions, items waiting.
2. The map opens on the "changed since yesterday" lens; touched areas glow, coloured by human or AI.
3. Selecting a glowing area shows its commits, sessions and AI share in a side panel.
4. Finished agent work opens as a change story; approve it or send comments back.
5. The inbox reaches zero.

**Moment of truth:** knowing in two minutes where to look, without opening a terminal.

### Reviewing a change set across repos (v1 in full)

1. A change set called checkout-v2 spans three repos and five pull requests.
2. Its change story shows the touched areas on a mini-map, each change classed (formatting, rename, move, logic, API), the provenance ribbon, and blast-radius warnings.
3. A "look here first" list ranks the risky parts: an export with two callers not updated, AI-written logic without tests.
4. Comments placed on the map go back to the agents that wrote the code.
5. Braid suggests a merge order from the dependency map; merge.

**Moment of truth:** a warning about a broken caller in another repo, before merge rather than after.

### Learning an unfamiliar codebase (v2)

1. Open a repo; the map starts at package level and offers a guided tour.
2. Search "checkout"; the map flies there.
3. Trace the path from an API route to the database call.
4. Pin a note explaining what you learned, so the next person finds it on the map.

**Moment of truth:** being able to say "checkout works like this" after one session.

## Product structure

The map is home. Every other view is either a panel beside it or a lens on top of it, so people never lose their sense of place. A command palette (⌘K) reaches everything, and every action has a keyboard shortcut.

**Views**

| View | Answers | Shown as | Release |
| --- | --- | --- | --- |
| Map | Where is what, and what's happening there? | Full canvas, the home view | MVP |
| Lens bar | What am I looking at? | Switcher over the map: changes, AI share, agent activity; more in v1 | MVP |
| Inbox | What's waiting on me? | Panel: permission prompts, questions, finished work | MVP |
| Fleet | What is each agent doing? | Lanes panel, plus markers on the map | MVP |
| Change story | What does this change mean? | Full view opened from the inbox, the map or a PR | MVP (first version), full in v1 |
| Line view | Who or what wrote this, and why? | Code panel with provenance gutter; why-links in v1 | MVP |
| Braid timeline | What happened when, across repos? | Full view with a shared time axis | v1 |
| Intent canvas | What are we trying to do, and where? | A drawing layer on the map | v1 |
| Workspace settings | Which repos, agents and privacy settings? | Settings page | MVP |

**Objects people work with**

| Object | What it is | Created by |
| --- | --- | --- |
| Workspace | A set of repos treated as one, described in `braid.toml` | The person, via `braid init` |
| Repo | One git repository and its worktrees | Discovered |
| Code node | A repo, package, folder, file, or (v1) function or class | Derived from code |
| Lens | A way of colouring the map with one measure | Built in; plugins in v2 |
| Worktree | A separate checkout for one task or agent | Braid or the agent tool |
| Agent session | One run of one agent in one worktree, with its edits | Captured from hooks |
| Task | A unit of work with a scope on the map and, from v1, acceptance criteria | The person, or sketch-to-plan |
| Intent note | A note, frame or arrow on the canvas, optionally pinned to code nodes | The person (v1) |
| Change set | Related branches and PRs across repos that ship together | The person, or Braid suggests |
| Provenance record | Who or what wrote each line, with an evidence grade | Captured, then sealed at commit |
| Snapshot | An exported image or link of any view | The person |

Every object can be reached from the map, and every object shows where it sits on the map. That two-way rule is what keeps Braid one place instead of nine tools.

## Releases

Three open-source releases and one paid tier. A release starts only when the one before it passes its usage gate, so the plan follows real adoption instead of a calendar.

*[Diagram: Release ladder · three open releases, one paid tier, three gates. See it in the doc: https://claude.ai/artifact/ARXGEtnr8MhKRVCNb8HcAm]*

Gates are counted in weekly active workspaces completing the loop (see Success metrics). Dates come in Phase 5, once the layout spike has measured how hard the map really is.

## Feature catalogue

All 62 ideas from Phase 1, placed in a release as 66 features: 21 in the MVP, 23 in v1, 14 in v2, 3 in the paid tier and 5 later (some ideas split into more than one feature). Effort is a first guess for one engineer working with agents: **S** about a week, **M** two to three weeks, **L** four to six. Phase 5 turns this into a schedule.

For MVP rows, priority says what happens if time runs short: **Must** items define the release; **Should** items slip to v1 rather than delay it.

**Head start from v0.1.** Yesterday's Rust spike (about 7,500 lines, 52 tests) already works from the command line for workspaces, worktrees and collision checks, capture from Claude Code, Codex and Cursor, sealed provenance that survives rewrites, an Oxc-based semantic diff with cross-repo blast radius, an MCP server with 8 tools and a local daemon with a hook receiver. For those rows the remaining effort is mostly UI and hardening, so the estimates below are upper bounds.

| Feature | Moment | Release | MVP priority | Effort | Depends on |
| --- | --- | --- | --- | --- | --- |
| braidd daemon and Braid Graph | Engine | MVP | Must | L | — (part built in v0.1) |
| Workspace manifest and `braid init` | Engine | MVP | Must | S | daemon (part built) |
| One-command install and hook setup | Engine | MVP | Must | M | daemon |
| Local web UI and desktop shell | Engine | MVP | Must | M | daemon |
| Living map, down to files | See | MVP | Must | M | Braid Graph |
| Stable layout | See | MVP | Must | M | living map |
| Lenses: changes, AI share, agent activity | See | MVP | Must | M | map, provenance |
| Line-level provenance and blame | Remember | MVP | Must | M | capture (built in v0.1) |
| Worktree per task | Direct | MVP | Must | M | daemon (part built) |
| Agent lanes | Direct | MVP | Must | S | session capture |
| Approval inbox | Direct | MVP | Must | M | session capture |
| Agents on the map | Watch | MVP | Must | M | map, sessions |
| Collision radar | Watch | MVP | Must | M | worktrees |
| Change story, first version | Review | MVP | Must | M | map, provenance |
| Privacy defaults | Govern | MVP | Must | S | — |
| Cross-repo package links | See | MVP | Should | M | living map |
| Fly-to search | See | MVP | Should | S | map |
| Launch an agent with a brief | Direct | MVP | Should | M | worktrees |
| Activity stream | Watch | MVP | Should | S | Braid Graph |
| Snapshot export | Share | MVP | Should | S | UI |
| MCP server, read tools | Engine | MVP | Should | S | Braid Graph (part built) |
| Function-level map (Oxc) | See | v1 | — | L | JS/TS parser |
| More lenses: churn, ownership, tests, complexity | See | v1 | — | M | map |
| Intent canvas | Think | v1 | — | L | map |
| Pinned notes | Think | v1 | — | M | canvas, stable node IDs |
| Acceptance criteria | Think | v1 | — | S | tasks |
| Sketch to plan, through the user's agent | Think | v1 | — | M | canvas, MCP |
| Send to agent with visual context | Direct | v1 | — | M | launch with brief |
| Overlap warning before dispatch | Direct | v1 | — | S | collision radar |
| Cursor support | Direct | v1 | — | M | capture adapters |
| Braid timeline | Watch | v1 | — | L | Braid Graph |
| Drift tide | Watch | v1 | — | S | worktrees |
| Notifications | Watch | v1 | — | S | inbox |
| Semantic diff (JS/TS) | Review | v1 | — | L | JS/TS parser |
| Dependency-order diff | Review | v1 | — | M | function-level map |
| Blast radius | Review | v1 | — | L | function-level map, resolver |
| Criteria coverage | Review | v1 | — | M | acceptance criteria |
| Look here first | Review | v1 | — | M | semantic diff, provenance |
| Comments to agent | Review | v1 | — | M | change story |
| Why-links | Remember | v1 | — | M | intents, provenance |
| Format interop (Git AI, Agent Trace) | Remember | v1 | — | M | provenance |
| Braid cards on PRs | Share | v1 | — | M | change story |
| Public atlas of well-known repos | Share | v1 | — | M | snapshot export |
| VS Code extension | Engine | v1 | — | M | web UI |
| Path tracing | See | v2 | — | M | function-level map |
| Ask the map, through the user's agent | See | v2 | — | M | MCP |
| Guided tours | See | v2 | — | M | path tracing |
| Simple-mode defaults for agent-first builders | See | v2 | — | S | usage data |
| Boundaries, drawn and checked | Think | v2 | — | M | canvas, resolver |
| Decision notes | Think | v2 | — | S | pinned notes |
| Session replay | Remember | v2 | — | M | session capture |
| Map history | Remember | v2 | — | L | stable layout |
| README map embed | Share | v2 | — | M | snapshot export |
| Live multiplayer canvas | Share | v2 | — | L | intent canvas |
| Stacks on GitLab, Bitbucket and Gerrit | Engine | v2 | — | L | forge adapters |
| Tree-sitter languages | Engine | v2 | — | L | language tiers |
| JetBrains plugin | Engine | v2 | — | M | web UI |
| Plugin API for lenses, languages, adapters | Engine | v2 | — | M | stable internals |
| Team server and dashboards | Govern | Paid | — | L | provenance, notes sync |
| PR gate with policies | Govern | Paid | — | M | team server |
| SSO, roles, audit log, retention | Govern | Paid | — | M | team server |
| Runtime overlay (OpenTelemetry) | See | Later | — | L | map |
| Ghost changes | Think | Later | — | M | canvas |
| Whiteboard import | Think | Later | — | M | pinned notes |
| Playbooks | Direct | Later | — | M | send to agent |
| Cost meter | Watch | Later | — | S | session capture |

## MVP definition

The MVP is done when an agent conductor can run two or more agents across two or more JS/TS repos for a full week using Braid as their overview, without opening a terminal just to find out what's going on. It runs on macOS and Linux; Windows follows in v1.

Each Must feature has a test it has to pass. The numbers are proposed targets, to be confirmed by the layout spike and the first benchmarks in Phase 5.

| Feature | Done when |
| --- | --- |
| Daemon and graph | Indexes a 7-repo, 10,000-file workspace in under 60 seconds cold; reflects a saved file within 1 second; restarts without a full re-index |
| Workspace manifest | braid init finds every git repo in a folder, groups them and writes braid.toml, and the map shows those groups. It replaces v0.1's braid ws init, which stays as an alias |
| Install and hooks | `braid install` shows every config change before making it; `braid uninstall` restores the originals exactly |
| Web UI and desktop shell | The same UI runs in a browser tab on localhost and in the desktop app on macOS and Linux |
| Living map | Every repo, folder and file is on the map; zooming from workspace to file stays smooth (60 fps) on a 2021 laptop |
| Stable layout | After 100 recorded commits of adds, deletes and renames, untouched files move less than 5% of the view width on average |
| Lenses | Switching lens takes under 100 ms; every lens shows a legend and how it is measured |
| Provenance and blame | Attribution survives rebase, amend, squash and cherry-pick on a recorded test corpus, for Claude Code and Codex sessions |
| Worktree per task | Create, list, park and remove from the UI and the CLI; setup scripts run; removal leaves no orphaned worktree or branch |
| Agent lanes | A Claude Code or Codex session started anywhere in the workspace appears within 2 seconds |
| Approval inbox | Permission prompts and "waiting for input" states appear with a one-click jump to the session. Answering always happens in the agent's own window; Braid never answers for the person |
| Agents on the map | A marker reaches the edited file's region within 1 second of the hook event |
| Collision radar | Overlapping edits in two live worktrees raise a warning within 5 seconds; false alarms are measured on the corpus |
| Change story, first version | For any branch: touched areas on a mini-map, files grouped by area, a provenance ribbon per file, whether tests changed, then the diff |
| Privacy defaults | No network calls the user didn't configure; telemetry off by default; prompts never stored unless switched on |

Out of the MVP, on purpose: the intent canvas, semantic diff, blast radius, Cursor, the VS Code extension, anything multi-user, deep support for languages other than JS/TS (other files still appear on the map), and forge APIs beyond what the `gh` CLI already gives locally.

## Success metrics

The north star is **weekly active workspaces that complete the loop**: in a given week, Braid saw at least one agent session in the workspace and someone opened at least one change story there. It counts real use of the core loop, not installs or stars.

Braid is local-first, so measurement is opt-in. A `braid telemetry on` switch sends anonymous counts only: no code, paths, prompts or names. Everything else comes from design-partner check-ins, release downloads and the community.

| Metric | Job | How it's measured | Proposed target, MVP alpha | Proposed target, v1 beta |
| --- | --- | --- | --- | --- |
| Workspaces completing the loop weekly (north star) | J1–J5 | Opt-in telemetry, partner check-ins | 10 | 500 |
| Activation: map shown within 5 minutes of install | First run | Opt-in telemetry | 70% | 80% |
| Week-2 return of activated workspaces | J1 | Opt-in telemetry | 50% | 40% |
| Agents connected per active workspace | J2 | Opt-in telemetry | 2 or more | 2 or more |
| Collisions flagged before a real conflict | J3 | Daemon counter, with whether the person acted | Tracked, no target yet | Tracked, no target yet |
| Change stories opened per active workspace per week | J5 | Opt-in telemetry | 5 | 10 |
| Share of merged lines with known provenance (grades A–C) | J5, J9 | Local report; the atlas's original north star | 80% | 90% |
| Snapshots and Braid cards shared | J12 | Export counter | Tracked | 1 per active user per week |

Guardrails, which must hold whatever else improves:

- Agent hook round trip under 20 ms, since agents wait on it.
- Daemon memory under 400 MB with 50 repos indexed, and near-zero CPU when idle.
- Crash-free sessions above 99%.
- Uninstalls tracked and asked about: a short optional "why are you leaving?" prompt.

The targets are starting points to argue with, not forecasts; the validation plan below says how the first ones get tested.

## Packaging

Everything one person uses on their own machine is free and open source, forever. Money comes only from what an organisation needs across many people and repos.

|  | Braid (open source) | Braid Team (paid, from after v1) |
| --- | --- | --- |
| Licence | Apache-2.0, crates `MIT OR Apache-2.0` | Separate repo and licence |
| Runs | On each developer's machine | Self-hosted or Braid cloud |
| Includes | Map, lenses, canvas, fleet, inbox, change story, provenance, timeline, snapshots, MCP server, IDE and agent plugins | Org-wide dashboards (AI share, survival, review coverage), PR gate with policies, SSO and SCIM, roles, audit log, retention, works-council configuration, support |
| Account | None | Organisation login |

Pricing is set after talking to design partners. A useful reference: CodeScene charges €18 or €27 per active author per month ([CodeScene pricing](https://codescene.com/pricing)). Until then, GitHub Sponsors and public grants fund the open-source work.

## Validation plan

Seven assumptions could sink Braid; each gets a cheap test before or during the MVP build. Run them in this order, because the first two decide whether the rest is worth building.

| # | Assumption | Test | Go ahead if | When |
| --- | --- | --- | --- | --- |
| A1 | Agent conductors hit the overview problem every week | Eight 30-minute interviews with developers running two or more agents; ask about the last time they lost track or hit a conflict, not about Braid | At least six of eight describe a concrete incident from the past two weeks | By 20 Nov 2026, in M1's first week |
| A2 | A file-level map is useful daily, not just impressive | Use a clickable prototype from the Phase 4 designs, then the first build, on Braid's own repo and one work codebase for two weeks | You open it daily without forcing it; three of five alpha users return in week 2 | First month of the build |
| A3 | A stable, live layout is achievable at real scale | Layout spike on three real codebases, the largest about 10,000 files | Passes the stable-layout test in the MVP table | Week 1–2 of the build |
| A4 | Picture-first review saves time or catches more | Review ten real PRs twice, once diff-only and once change-story first, with a colleague | Change story first is at least 20% faster or finds more issues | When the change story works |
| A5 | Agent hooks give reliable session data | Recorded payload corpus for Claude Code and Codex, plus a week of real use | At least 95% of agent edits captured at grade A or B | Alongside the build |
| A6 | People share Braid pictures | Snapshot export in the alpha, with a share counter | Alpha users share without being asked | Alpha |
| A7 | Self-hosting and neutrality matter to companies | Three conversations with engineering leads at German companies about self-hosting, provenance and works councils | Two of three would join a pilot of the paid tier | Before v1 |

Interview tips, since this is the first round: ask about past behaviour ("tell me about the last time…"), never about the future ("would you use…"); don't show Braid until the end; and write down their exact words, which become the copy on the website later.

## Decisions and questions for Phase 3

Phase 2 fixes the first release as Map & Fleet for agent conductors on JS/TS, with Lena as the primary persona, the loop as the north star and Must features as the line that can't move.

Phase 3 has to answer these engineering questions:

- [ ] Map shape for the MVP: nested regions (a containment map, stable by construction) or a node-and-link graph? The stable-layout test favours nested regions with links drawn on demand.
- [ ] How do code nodes keep their identity across renames and moves, so pinned notes and positions survive?
- [ ] Which hook events reliably signal "waiting for a person" in Claude Code and Codex, for the inbox?
- [ ] What does the Braid Graph store, where (SQLite, git notes, plain files in the repo), and what is derived on the fly?
- [ ] How does the daemon stream a live graph to the UI cheaply: snapshots, diffs or both?
- [ ] How expensive is the collision radar's in-memory merge with four live worktrees, and how often can it run?
- [ ] Where do intent notes and acceptance criteria live so they travel with the repo and diff cleanly in review?
- [ ] How does opt-in telemetry work without ever touching code, paths or prompts?
