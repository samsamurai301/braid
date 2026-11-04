<!-- Exported from https://claude.ai/artifact/Jwg4EG9GrZeiF3QjDvgY6o on 31 Oct 2026. The doc is the living version. -->

# Braid · Phase 3 — Logic & Architecture

Oct 31, 2026 · @Sam

## Summary

Braid is one local Rust daemon, `braidd`, that turns code, git history and agent activity into a single live graph, plus one Svelte web UI that draws that graph as a map in every shell: desktop app, browser tab, VS Code and later JetBrains. Agents feed the daemon through their hooks and read it through MCP. Nothing leaves the machine unless the person sets up the paid team server.

This doc extends yesterday's v0.1 code (six crates including `braid-git`, `braid-prov`, `braid-ws`, `braid-cli` and `braidd`; about 7,500 lines and 52 tests) rather than replacing it. It covers the MVP in depth and v1 in outline, following [Phase 2](https://claude.ai/artifact/ARXGEtnr8MhKRVCNb8HcAm).

| Decision | Choice | Main reason |
| --- | --- | --- |
| Process model | One daemon per machine, thin clients everywhere | Heavy work (git, parsing, provenance) happens once; every surface shows the same truth |
| Core data model | The Braid Graph: typed nodes and edges for code, work and people | Every view is a query or lens on one model |
| Map shape | A containment map (nested regions) with links drawn on demand | Stable by construction, readable at every zoom, cheap to draw |
| Identity | Stable node IDs that survive renames and moves | Positions, pinned notes and history must not break when files move |
| Storage | SQLite for the graph, git notes for provenance, plain files in the repo for intents | Each kind of data lives where it is shared naturally |
| Live updates | Versioned graph diffs over a WebSocket | Cheap to stream 100,000 nodes; any client can resync from a version |
| UI | Svelte 5 + TypeScript, WebGL2 rendering through deck.gl, DOM cards up close | Small bundles for four webview hosts; WebGL2 works everywhere |
| Canvas state | Loro CRDT, in Rust and in WASM | One format on both sides, ready for multiplayer in v2 |
| AI work | Delegated to the user's own agent through MCP or its CLI | No keys, no model costs, no code leaving the machine |

## System architecture

Everything heavy happens once, inside `braidd`; everything people see is a thin client on top of it; agents and repos sit outside and talk to it through hooks, MCP and git.

*[Diagram: System architecture · surfaces, the braidd daemon, and what it talks to. See it in the doc: https://claude.ai/artifact/Jwg4EG9GrZeiF3QjDvgY6o]*

The Braid Graph is the centre: workspaces, provenance and intents write into it; the map, change stories and every interface read from it. The team server is the only part that runs elsewhere, and it reads git notes from the forge rather than talking to anyone's daemon.

## The Braid Graph

One typed graph holds three families of nodes: code (what exists), work (what is happening and why) and people and tools (who or what did it). Every view is a query on this graph and every lens is an attribute on its nodes, which is what lets a note made while planning appear during review.

*[Diagram: Braid Graph · three families of nodes and the edges between them. See it in the doc: https://claude.ai/artifact/Jwg4EG9GrZeiF3QjDvgY6o]*

The agent session is the hub: it edits code, runs in a worktree, belongs to a task and is run by a tool, which is how one click on a line can reach the intent behind it.

| Family | Node kind | Key attributes | Comes from |
| --- | --- | --- | --- |
| Code | Workspace, group | Name, repos, owners | `braid.toml` |
| Code | Repo | Path, default branch, remotes | Discovery |
| Code | Package | Name, version, manifest path | `package.json`, workspace protocols |
| Code | Folder, file | Path, language, lines, churn, owners, AI share | Git and the file watcher |
| Code | Symbol (v1) | Kind, qualified name, exported?, signature | Oxc |
| Work | Intent note (v1) | Text, anchors, status | `.braid/intents/*.md` |
| Work | Task | Title, scope (code nodes), acceptance criteria, status, agent | Person or sketch-to-plan |
| Work | Agent session | Tool, model, worktree, state, task link | Agent hooks |
| Work | Worktree, branch | Path, base, ahead/behind, change footprint | Git |
| Work | Commit | SHA, message, sealed provenance | Git and notes |
| Work | Change set, PR | Member branches and PRs, merge order | Person or forge |
| Work | Review | Verdict, comments with map anchors | Braid or forge |
| People and tools | Person | Git identity (hashed for any shared report) | Git |
| People and tools | Agent tool | Tool name, model | Hook payloads |

Edges, all directed: **contains** (code hierarchy), **imports** (file to file, package to package), **calls** (symbol to symbol, v1), **edits** (session to file or symbol), **touches** (commit to file), **runs in** (session to worktree), **part of** (commit or PR to change set), **pinned to** (intent to any code node), **implements** (task to intent), **covers** (hunk to criterion, v1), **written by** (line range to session or person), **reviewed in** (change set to review).

**Identity rules.** Positions on the map, pinned notes and history all hang on node identity, so identity must survive the things code goes through.

1. Every node gets a random, sortable ID (a ULID) the first time Braid sees it. The ID never encodes a path; a separate table maps it to its current location.
2. A file renamed or moved keeps its ID when git's rename detection matches it (similarity of 50% or more), or when the watcher sees a delete and a create with 60% or more content similarity within a few seconds.
3. A symbol (v1) is identified by its file, kind and qualified name; when renamed, it keeps its ID if its body's token hash stays similar.
4. When a file splits, the part with the largest overlap keeps the ID; the rest are new, and any pinned note on them is flagged for a look.
5. Packages in different repos are linked by package name and version range, and imports are resolved with `oxc_resolver`, so the graph spans repos without a monorepo.

## Storage

Each kind of data lives where it is naturally shared: derived data stays in a local database that can always be rebuilt, authorship travels with commits in git notes, and plans travel with the code as plain files in the repo.

| Data | Where | Format | Shared how | Why there |
| --- | --- | --- | --- | --- |
| Braid Graph: structure, metrics, node IDs | `~/.braid/workspaces/<id>/graph.db` | SQLite | Not shared; rebuilt from git if deleted | Derived, fast to query |
| Map anchors: positions of groups, repos and packages | `.braid/map.json` next to `braid.toml` | JSON | Through git, if `braid.toml` lives in a shared meta-repo | So a whole team sees the same map, and "payments is bottom left" is true for everyone |
| Positions below package level | Local cache in `graph.db` | SQLite | Not shared; recomputed deterministically | The ordered layout gives the same result on every machine |
| Pending provenance (before commit) | Ledger per repo (v0.1) | SQLite | Local | Short-lived; sealed at commit |
| Sealed provenance | `refs/notes/ai` | Git AI Standard v3 | Push and fetch with refspecs Braid sets up | Travels with commits; readable by other tools |
| Braid links: line to session to task to intent | `refs/notes/braid` | JSON note per commit | Same refspecs | Keeps the Git AI note clean and compatible |
| Agent sessions and events | `events.db` | SQLite, append-only | Local; opt-in export | Activity stream, replay, inbox history |
| Intent notes, tasks, criteria (v1) | `.braid/intents/<id>.md` in the repo concerned, or the meta-repo for cross-repo work | Markdown with YAML front matter | Git; reviewed in pull requests like code | Plans travel with the code and diff cleanly |
| Canvas geometry (v1) | Loro document in the daemon; readable `.braid/canvas.json` written on save | Loro, JSON snapshot | Git now; live sync in v2 | The CRDT merges concurrent edits after a pull |
| Search index | `~/.braid/.../search/` | tantivy | Local | Fly-to search |
| Settings | `~/.config/braid/config.toml`; tokens in the OS keychain | TOML | Local | Privacy |

Nothing in `~/.braid` is hard to replace except unsealed provenance and local session history: deleting it costs a re-index and that history, never code, sealed authorship or plans, which all live in git.

## Event pipeline

Every change, whatever its source, takes the same path: it is written to an append-only log, the graph recomputes only what that change invalidates, and the views that depend on it stream a diff to the UI.

*[Diagram: Event pipeline · four sources into one log, one graph update, one stream. See it in the doc: https://claude.ai/artifact/Jwg4EG9GrZeiF3QjDvgY6o]*

Hooks are answered as soon as their event is in the log, which keeps them under 20 ms; everything to the right of the log runs afterwards. The log also makes the system debuggable: replaying it rebuilds any state, which is how the session replay in v2 works.

## The living map

The map is a containment map: every code node is a region inside its parent (workspace, group, repo, package, folder, file), and links between regions are drawn only when asked for. Nested regions stay stable when code changes, read well at every zoom level, and are cheap to draw. Node-and-link graphs, which reshuffle on every change, are kept for detail views such as path tracing in v2.

### Layout in two tiers

1. **Top tiers: groups, repos and packages are anchored.** The first layout orders siblings so that packages which import each other sit side by side (a one-dimensional ordering of the dependency graph, its Fiedler vector), then lays them out as an ordered treemap. The resulting rectangles are saved as anchors in `.braid/map.json`. After that they change only when a new top-level node arrives, which takes space from its most related sibling, or when someone presses "tidy".
2. **Inside a package: ordered strip treemap.** Folders and files are laid out in path order in strips, the layout from Bederson, Shneiderman and Wattenberg's work on ordered treemaps. Path order means a new file lands between its alphabetical neighbours and nothing else moves far. Only the container that changed is recomputed.

**Size.** A file's area is proportional to the square root of its line count, with a minimum, so a 5,000-line file doesn't swallow its folder and a 20-line file stays visible. A region resizes only when its weight changes by more than 10%, so the map doesn't breathe with every edit.

**Stability test.** Replay 100 real commits and measure how far untouched files move. The MVP target is an average under 5% of the view width (Phase 2). The layout spike in Phase 5 tests three alternatives against this number: the ordered strip treemap above, circle packing as in CodeScene, and a Voronoi treemap.

### Semantic zoom

| Level | What is drawn | Labels | Agents shown as |
| --- | --- | --- | --- |
| Workspace | Groups and repos | Repo names | A count per repo |
| Repo | Packages and top folders | Package and folder names | A count per package |
| Package | Folders and files as tiles | File names when a tile is wider than about 48 px | A marker on the file |
| File | The file as a card: outline (v1 symbols), provenance strip, recent changes | Everything | Marker plus the session's latest action |

Labels are culled by available space and ordered by the active lens, so the most important names win when space is short. Links (imports, blast radius, overlaps) are bundled at the current zoom level: two repos show one thick arc rather than three hundred thin ones.

### Lenses

A lens is a function from a node to a value, plus a legend and a note on how it is measured. The daemon computes it with a query over the graph, aggregates it upwards (sum, mean or maximum, set per lens) and sends a compact array keyed by node index. The UI colours regions with the design system's scales. Switching lens reuses the geometry, which is why it can take under 100 ms.

### Rendering

The map is drawn with deck.gl on WebGL2: polygon layers for regions, a signed-distance-field text layer for labels, icons for agents and arc layers for links. At file level a Svelte card appears in the DOM above the canvas. The UI asks the daemon only for the containers in view at the current level, like map tiles, so a 100,000-file workspace never ships 100,000 rectangles at once. WebGPU is a later upgrade: it's missing on Linux webviews, so WebGL2 stays the baseline.

## Agents

Braid watches agents through their hooks, places each session on the map, and tracks one state per session; the inbox is simply the list of sessions in a state that waits on a person.

*[Diagram: Agent session states · the two accented ones fill the inbox. See it in the doc: https://claude.ai/artifact/Jwg4EG9GrZeiF3QjDvgY6o]*

A session moves right as work finishes and loops back when review asks for changes. Everything that waits on a person is in the inbox; everything else only shows on the map and in the fleet.

**Session model.** A session starts at the first hook event carrying a new session ID. Braid links it to a worktree by its working directory, to a task through the `BRAID_TASK_ID` environment variable when Braid launched it (or through the worktree's task otherwise), and to an agent tool and model from the payload or, as v0.1 does for Claude Code, from the transcript.

**Signals per agent.** Claude Code's hooks cover the whole life of a session: start, each tool use, notifications when it needs permission or has waited for input, the end of each turn, and the end of the session. Codex reports edits through `PostToolUse` on `apply_patch`; its signals for "waiting on a person" still have to be checked against its hook documentation. Where an agent sends no such signal, Braid falls back to a heuristic: the process is alive, no event for 30 seconds, last event was a question. Heuristic states are labelled as guesses in the inbox.

**Launching (send to agent).** Two modes, both in the MVP:

1. **Interactive:** Braid creates the worktree and opens the person's own terminal app there, running the agent with the brief as its first prompt. The person stays in the agent's normal interface.
2. **Unattended:** Braid runs the agent's headless mode (`claude -p`, `codex exec`) with the brief and streams its output into the session panel. Permissions follow the agent's own settings; Braid never grants anything extra and never answers a permission prompt by itself.

The brief is a Markdown file written into the worktree (and ignored by git): the intent, the task and its scope as paths. v1 adds acceptance criteria and a request to report progress through Braid's MCP tools; v2 adds the boundaries to respect.

**Worktree lifecycle.** Worktrees go to `<repo>.worktrees/<name>` on a `braid/<name>` branch, the convention from v0.1. A setup script from `braid.toml` runs after creation (installing dependencies, for example). Parked worktrees stay on disk but leave the map. Removal is refused while there are unmerged changes, and a weekly sweep offers to remove worktrees whose branches were merged.

**MCP tools.** v0.1 ships eight tools. The target set:

| Tool | What an agent gets | Release |
| --- | --- | --- |
| `braid_status` | Repos, branches, worktrees and live sessions in the workspace | MVP |
| `braid_where` | Code nodes matching a concept or name, with paths | MVP |
| `braid_neighbors` | What a file or symbol imports and what depends on it, across repos | MVP |
| `braid_who_wrote` | Provenance for a line range, with evidence grade | MVP |
| `braid_overlap` | Whether another live session is editing the same code | MVP |
| `braid_task` | The current task's intent, scope, criteria and boundaries | v1 |
| `braid_report_progress` | Lets the agent say which criteria a change addresses | v1 |
| `braid_change_story` | A structured summary of the agent's own change, for self-review | v1 |
| `braid_create_worktree` | Asks Braid to create a worktree for a sub-task | v1 |

## Collision radar

The radar escalates in three levels, from cheap to certain, and only runs the expensive check on the few pairs of worktrees that actually share files. v0.1's `braid wt collisions` already does the final level with `git merge-tree`.

1. **Footprints.** For every live worktree, the daemon keeps a footprint: per file, the changed line ranges against the worktree's base, including uncommitted edits. It updates on each save, debounced by half a second.
2. **Candidate pairs.** An index from file to the worktrees touching it gives the candidate pairs. Pairs that share no file are never checked, so four agents usually produce zero to two candidates.
3. **Level 1, same file.** Two worktrees change the same file in different places. Shown as a quiet badge.
4. **Level 2, overlapping lines.** Their line ranges intersect, after mapping both onto a common base when their bases differ. Shown as a warning on the map, at the overlap.
5. **Level 3, real conflict.** Braid runs an in-memory three-way merge: `git merge-tree --write-tree` for committed work, with uncommitted changes first written as temporary tree objects. A conflict marks the exact hunks and lights the inbox.
6. **Before dispatch (v1).** A new task's scope on the map is compared with live footprints, so overlap is flagged before an agent even starts.

**Cost.** Footprint updates are proportional to the changed hunks. A merge check takes tens of milliseconds per pair and runs at most every five seconds per pair, and only when a footprint changed. The budget is under 50 ms per pair and near-zero CPU when nothing changes.

## Provenance and why-links

Provenance works as the atlas designed and v0.1 built it: record authorship when code is written, never guess it later. Why-links add one thing on top: a chain from each line back to the reason it exists.

**The provenance pipeline (built in v0.1)**

1. **Capture.** An agent hook posts each edit to the daemon with path, patch, session, tool and model. The hook gets its reply in a few milliseconds (median 7.6 ms measured in v0.1).
2. **Ledger.** Pending hunks are stored per worktree and file, keyed by a whitespace-insensitive token hash so a formatter run doesn't erase authorship.
3. **Fill gaps.** The watcher catches edits no hook reported, such as shell scripts run by an agent, as grade B.
4. **Seal.** At commit, pending hunks are matched to committed lines and written as a Git AI–compatible note.
5. **Follow.** Rebase, amend, squash and cherry-pick carry attribution forward; `braid carry` restores it after a forge squash-merge.

Evidence grades: **A** captured from a hook, **B** observed during a live session, **C** imported from another tool, **D** a weak signal such as a commit trailer, **U** unknown. Unknown is never counted as human.

**Why-links (v1).** Each sealed commit also gets a small JSON note under `refs/notes/braid` that links its sessions to their task and intent IDs. Following the chain for any line:

1. The line's provenance gives its session (Git AI note).
2. The session gives its task, if Braid launched it or the person linked it (Braid note).
3. The task gives its intent note and acceptance criteria (`.braid/intents/`).
4. The change set gives the review that approved it.

Sessions started outside Braid have no task link until someone adds one; the line view shows the gap rather than hiding it. Keeping Braid's links in a separate notes ref means the Git AI note stays readable by any tool that understands the standard.

## Change story

A change story is built by one pipeline from any range (a branch against its base, a PR, or a change set across repos) into a structured object that the UI draws, the MCP tool returns and the Braid card renders. The MVP runs steps 1, 2, 4 and 5 at file level; v1 adds the rest.

1. **Diff** each repo in the range, with rename detection.
2. **Map hunks to code nodes**: files in the MVP, symbols in v1 using Oxc spans.
3. **Classify each hunk** as formatting only (same syntax tree after normalising), rename (symbol renamed and references updated), move (token hashes match code removed elsewhere), logic, or API (an exported signature changed). v0.1's `braid diff` already separates formatting from logic and lists export changes.
4. **Overlay provenance**: the AI, human and mixed share per hunk, from sealed notes or the ledger for uncommitted work.
5. **Find tests**: test files touched, and tests that import the changed modules, giving "tests touched for this area: yes or no".
6. **Blast radius**: follow changed exports through the reverse import graph across repos; mark each caller updated or not.
7. **Score risk** per group of hunks (below) and build "look here first".
8. **Order** files topologically by imports, foundations before consumers; formatting-only groups collapse to the end.
9. **Match criteria** (v1): each acceptance criterion is matched to hunks inside the task's scope, to tests that mention it, and to what the agent reported through `braid_report_progress`. Criteria with no match show in red; hunks outside the scope show as "outside the plan".

**Risk score, proposed weights** (to be tuned on real reviews in validation test A4):

| Signal | Points |
| --- | --- |
| Changed export with callers not updated | +5 |
| Logic change at least 80% AI-written, with no tests touched | +4 |
| Protected path (policy file or CODEOWNERS) | +3 |
| Hotspot: top 10% of files by churn times complexity | +2 |
| Any logic change | +1 |
| Formatting only | −2 |

The score orders attention; it never blocks a merge. Each item in "look here first" says which signals put it there.

**Braid cards.** The daemon renders the change story's summary as SVG from the same data, then rasterises it, so a GitHub Action can run `braid card` in CI and post the image to the pull request without a browser or a server.

## Intent canvas and plans (v1)

Intents and tasks are plain Markdown files in the repo, so plans are reviewed in pull requests like code and never depend on Braid's database; the canvas only adds geometry on top.

**File format.** One file per intent under `.braid/intents/`:

```markdown
---
id: int_01JC8Z…
title: Add tests and a retry policy to express-pay
status: in-progress        # open | in-progress | done | dropped
anchors:
  - node: n_01JB2K…        # stable node ID
    hint: web-storefront/src/pay/ExpressPay.tsx
tasks:
  - id: tsk_01JC90…
    title: Add a retry policy
    scope: [n_01JB2K…]
    criteria:
      - Retries network errors up to 3 times with backoff
      - Never retries a 4xx response
    agent: claude-code
    status: in-review
---
Free notes, decisions and links go here.
```

**Geometry.** A note's position is stored relative to the region it is pinned to, so notes move with the map. Unpinned notes sit in free space on the canvas.

**Anchoring rules.**

- A pinned node that moves or is renamed carries the note with it (identity rules above).
- A pinned node that is deleted turns the note **orphaned**: it stays where it was, marked, until someone re-pins or drops it.
- A pinned node that splits re-pins to the largest successor and flags the note for a look.
- The `hint` path lets a person, or Braid after a fresh clone, re-find the node even without the database.

**Sketch to plan.** Braid builds a prompt from the intent text, the anchored nodes, their neighbours (imports, dependents, owners) and the boundaries, then sends it to the person's chosen agent through its headless mode, asking for tasks as JSON against a published schema. The answer is validated and shown as draft task cards to edit, accept or discard. Braid stores no API key and pays for no tokens.

**Acceptance criteria** are short, testable sentences. They go into the agent's brief, come back through `braid_report_progress`, and are matched in the change story.

## Interfaces

The daemon has four doors, all local: an HTTP and WebSocket API for UIs, a local socket for the CLI, hook endpoints for agents, and an MCP server for agents to read from.

| Interface | Transport | Used by | Notes |
| --- | --- | --- | --- |
| HTTP API | JSON over HTTP on `127.0.0.1:7471` (v0.1) | Every UI, scripts | Versioned under `/v1/`; v0.1 already serves status, sessions, activity, blame, collisions and diff |
| Live stream | WebSocket at `/v1/stream`, MessagePack frames | Every UI | Subscriptions and versioned diffs (below) |
| CLI socket | JSON-RPC over a Unix socket (named pipe on Windows) | `braid` CLI | Many commands also work without the daemon, as in v0.1 |
| Hook endpoints | HTTP POST per agent (`/hooks/claude-code`, `/hooks/codex`, `/hooks/cursor`, generic) | Agent hooks | Must answer within 20 ms; heavy work runs after the reply |
| MCP server | stdio and streamable HTTP, built on `rmcp` | Any MCP client | Tools listed under Agents |
| Plugin API (v2) | WebAssembly modules run in a sandbox | Community lenses, language and agent adapters | A lens plugin is a pure function from node data to a value |

**Streaming protocol.** A client subscribes to what it shows: map containers in view at the current zoom, the active lens, the fleet, the inbox. The daemon sends a snapshot tagged with a graph version, then a stream of diffs, each listing node, edge and attribute additions, updates and removals since the last version. The client acknowledges versions; after a gap (sleep, reconnect) it asks for a resync from its last version and gets either the missing diffs or a fresh snapshot. The same protocol carries agent events and inbox changes, so there is one source of truth for every view.

## Delivery surfaces

One web bundle, many thin shells: each shell only finds the daemon, passes it a token and hosts the same Svelte UI, so a feature built once appears everywhere.

| Surface | Release | How it connects | Notes |
| --- | --- | --- | --- |
| CLI (`braid`) | v0.1, MVP | Local socket, or direct | Scriptable; works over SSH |
| Browser tab | MVP | Daemon serves the UI at `localhost:7471/app`; WebSocket | Zero install beyond the daemon |
| Desktop app (Tauri 2) | MVP (macOS, Linux); Windows in v1 | Bundles the daemon as a sidecar; webview loads the UI | Native menus, notifications, opening terminals |
| Claude Code plugin | v0.1, MVP | HTTP hooks to the daemon; MCP server | Already passes `claude plugin validate` |
| Codex | MVP | `PostToolUse` hook on `apply_patch`; MCP | Hook trust must be confirmed by the user |
| Cursor | v1 (capture works in v0.1) | `afterFileEdit` and `afterTabFileEdit` hooks; MCP; the VS Code extension also runs |  |
| VS Code extension | v1 | Webview hosting the UI; editor decorations for the provenance gutter | Published to both the VS Marketplace and Open VSX |
| MCP App view | v1, optional | A `ui://` resource rendered by the host | Only local hosts such as VS Code, Cursor and Goose can reach a local daemon; Claude and ChatGPT need a remote server ([MCP Apps spec](https://github.com/modelcontextprotocol/ext-apps/blob/main/specification/2026-01-26/apps.mdx)) |
| GitHub Action | v1 | Runs `braid card` in CI and posts the image to the PR | No server needed |
| JetBrains plugin | v2 | JCEF webview | WebGPU inside JCEF is unverified, which doesn't matter with WebGL2 |
| Team server | Paid | Reads sealed notes from forges | Never needs full source code |

## Technology choices

Every dependency below is compatible with an Apache-2.0 project; versions were checked on 31 October 2026 against npm and crates.io.

| Layer | Choice | Version, licence | Why, and the catch |
| --- | --- | --- | --- |
| Daemon runtime | Rust 2024, tokio, axum | From v0.1 | Proven in the spike |
| Git | gitoxide for reads, git CLI for writes | From v0.1 | gix has no push, rebase or worktree removal yet |
| JS/TS analysis | Oxc parser, semantic, resolver; salsa for incremental work | From v0.1 | Fast; no type information, so types stay optional |
| Storage | SQLite (rusqlite), tantivy | From v0.1 | Local and embedded |
| Graph algorithms | petgraph | 0.8.3, MIT or Apache-2.0 | Orderings, topological sort, reachability |
| Map layout | Braid's own Rust code, also compiled to WebAssembly | — | No library offers anchored, ordered treemaps; same code in daemon and browser |
| Detail graph layout (v2) | elkjs in a web worker | 0.12.1, EPL-2.0 or GPL (use EPL, unmodified) | Only option with ports, edge routing and nesting |
| UI framework | Svelte 5 with TypeScript and Vite | 5.57.3, MIT | About 17 KB gzipped against React's 69 KB in a measured counter app; matters across four webview hosts |
| Map rendering | deck.gl on WebGL2 | 9.4.1, MIT | Polygon, text, icon and arc layers; a WebGPU path already exists for later |
| Node-link views (v2) | sigma.js | 4.0.4, MIT | WebGL2 labels and picking for 10,000–100,000-node graphs |
| Canvas CRDT | Loro, in Rust and WebAssembly | 1.17.1, MIT | One format in daemon and UI; movable trees suit frames and groups |
| Desktop shell | Tauri | 2.12, MIT or Apache-2.0 | Small, Rust-native; GitButler shows the Tauri plus Svelte plus Rust stack works for a git tool |
| MCP | rmcp (server), ext-apps SDK (MCP App view) | From v0.1; ext-apps 2.0.4 | MCP Apps spec 2026-01-26 is stable |

**Ruled out:** tldraw (production needs a licence key and its terms conflict with Apache-2.0), the `@cosmograph/*` packages (CC-BY-NC, a non-commercial licence), the `forceatlas2` crate (AGPL-3.0-only) and copying GitButler code (Fair Source). EPL and MPL libraries are used only unmodified. `cargo-deny` and a JavaScript licence checker run in CI so a banned licence can't slip in.

Sources for versions and licences: npm registry and crates.io, plus the project pages listed under Sources.

## Performance budgets

The budgets that agents depend on are already met by v0.1; the open risk is the map, which has no measurements yet.

| What | Budget | v0.1 measured (release build, 2-vCPU VM) |
| --- | --- | --- |
| Agent hook round trip | Under 20 ms | Median 7.6 ms, p95 12.7 ms |
| `git commit` with provenance sealing | Under 100 ms for 1,000 lines | 29 ms |
| Status across 50 repos, daemon warm | Under 300 ms | 204–228 ms; 344–350 ms with AI share |
| Cold index of a 10,000-file workspace | Under 60 s | Not measured |
| Saved file to map update | Under 1 s | Not measured |
| Map pan and zoom, 100,000-file workspace | 60 fps, at most about 20,000 tiles drawn at once | Not measured |
| Lens switch | Under 100 ms | Not measured |
| Collision check per pair | Under 50 ms | Not measured separately |
| Daemon memory with 50 repos | Under 400 MB; idle CPU near zero | Not measured |
| Initial UI download | Under 500 KB gzipped | Not built |

Status with AI share is above budget today; caching the share per commit in `graph.db` should fix it, since a commit's sealed note rarely changes once written.

## Security and privacy

A local daemon that sees all of someone's code is a target, so it trusts nothing that hasn't proven it is local and authorised.

| Threat | Defence |
| --- | --- |
| A web page in the browser calls the daemon (DNS rebinding, cross-site requests) | Binds to 127.0.0.1 only; checks the Host header (in v0.1); requires a random per-install token on every API and WebSocket call; CORS limited to Braid's own origins |
| Another local process reads the token | Token file readable only by the user, or the OS keychain where available |
| A malicious hook payload | Strict schemas; payloads are parsed, never executed; size limits; fuzzed in CI |
| Secrets or prompts leaking into storage | Session references and hashes by default; transcripts only when switched on, with secret redaction |
| Braid approving something on an agent's behalf | Never: the inbox routes requests, the agent's own permission system decides |
| A compromised release | Signed releases, a software bill of materials, `cargo-deny` and licence checks in CI |
| Telemetry revealing code | Opt-in, counts only, published schema, and `braid telemetry show` to see exactly what would be sent |
| The team server seeing source code | It reads sealed notes and diff metadata; full source only if an admin configures it |

## Testing strategy

Braid's riskiest promises are about trust (authorship is right, the map doesn't jump), so the test suite is built around recorded reality rather than mocks.

| Layer | What it proves | How |
| --- | --- | --- |
| End-to-end on real git | Capture, seal, rewrite and blame work | v0.1's 52 tests drive the real binary and git; extended per feature |
| Hook payload corpus | Every supported agent version still parses | Recorded payloads per agent and version as fixtures; a new agent release adds a fixture |
| Rewrite corpus | Authorship survives rebase, amend, squash, cherry-pick and forge squash-merge | Scripted histories with expected blame |
| Layout stability benchmark | The map stays put | Replays 100 commits from three open-source repos; fails CI if average displacement passes 5% |
| Performance benchmarks | Budgets hold | Criterion benchmarks with thresholds in CI |
| UI components | Views behave | Vitest for logic; Playwright screenshots of the map in light and dark themes |
| Fuzzing | Bad input can't crash the daemon | `cargo-fuzz` on hook payloads and notes parsing |
| Dogfooding | It's useful | Braid records its own development, and the Braid repo's AI share is a public number |

## Decisions and open questions

Ten architecture decisions come out of this phase. Two are already proven by v0.1; the rest are proposed and become accepted once the first MVP milestone confirms them. Each will get a short decision record in `docs/adr/`.

| # | Decision | Status |
| --- | --- | --- |
| ADR-001 | One local daemon per machine; every surface is a thin client | Accepted (v0.1) |
| ADR-002 | Provenance in Git AI–compatible notes; Braid's own links in `refs/notes/braid` | Accepted for provenance (v0.1); proposed for links |
| ADR-003 | Containment map with anchored top tiers and ordered strip treemaps inside | Proposed; the layout spike decides |
| ADR-004 | Random stable node IDs with rename tracking | Proposed |
| ADR-005 | Intents and tasks as Markdown files in the repo | Proposed |
| ADR-006 | Svelte 5 UI with deck.gl on WebGL2 | Proposed |
| ADR-007 | Versioned graph diffs over one WebSocket stream | Proposed |
| ADR-008 | AI work delegated to the user's own agent; no keys in Braid | Proposed |
| ADR-009 | Loro for canvas state | Proposed |
| ADR-010 | Apache-2.0 apps, `MIT OR Apache-2.0` crates, DCO, licence gate in CI | Proposed |

New code joins the v0.1 workspace as these crates and packages: `braid-graph` (the Braid Graph and identity), `braid-map` (layout and lenses, also built for WebAssembly), `braid-story` (change stories and cards), `braid-intent` (intents and tasks, v1), `braid-stream` (the WebSocket protocol), plus `ui/` (the Svelte app) and `desktop/` (the Tauri shell).

Open questions:

- [x] Interactive launch: settled. Braid opens the person's own terminal app (see Launching); no embedded terminal.
- [ ] Which hook events signal "waiting on a person" in Codex and Cursor? Check against their current hook documentation.
- [ ] Ordered strip treemap, circle packing or Voronoi treemap? The layout spike decides on the stability test.
- [ ] Where do `braid.toml` and the shared map anchors live for a team without a meta-repo?
- [x] Windows: settled. The MVP runs on macOS and Linux; Windows arrives in v1 (Phase 2).
- [ ] Where does opt-in telemetry go: a small self-hosted collector, or a hosted service?

## Sources

Checked on 31 October 2026. The v0.1 facts (crates, tests, measurements) come from the build report of 30 October; earlier engine choices come from the [Braid Concept Atlas](https://claude.ai/code/artifact/fc2b337b-e431-405f-af39-bb869214eaa8).

- [deck.gl 9.4.0 release (WebGPU port)](https://github.com/visgl/deck.gl/releases/tag/v9.4.0)
- [sigma.js 4.0.0 release](https://github.com/jacomyal/sigma.js/releases/tag/sigma@4.0.0) · [sigma.js performance notes](https://www.sigmajs.org/how-to/technical/performance/)
- [cosmos.gl repository](https://github.com/cosmosgl/graph)
- [WebGPU implementation status](https://github.com/gpuweb/gpuweb/wiki/Implementation-Status) · [Tauri webview versions](https://v2.tauri.app/reference/webview-versions/) · [Tauri issue on WebGPU in WebKitGTK](https://github.com/tauri-apps/tauri/issues/6381)
- [elkjs](https://github.com/kieler/elkjs) · [React Flow on layouting](https://reactflow.dev/learn/layouting/layouting)
- [Loro](https://github.com/loro-dev/loro) · [Automerge 3](https://automerge.org/blog/automerge-3/)
- [Vello](https://github.com/linebender/vello) · [Linebender, April 2026](https://linebender.org/blog/tmil-25/)
- [MCP Apps specification 2026-01-26](https://github.com/modelcontextprotocol/ext-apps/blob/main/specification/2026-01-26/apps.mdx) · [MCP Apps host compatibility](https://www.mcpbundles.com/guides/mcp-apps-host-compatibility)
- [tldraw licence](https://tldraw.dev/legal/tldraw-license) · [Apache licence policy](https://apache.org/legal/resolved)
- [GitButler repository](https://github.com/gitbutlerapp/gitbutler)
- [CodeScene hotspots](https://codescene.io/docs/guides/technical/hotspots.html)
- [Git AI Standard v3.0.0](https://github.com/git-ai-project/git-ai/blob/main/specs/git_ai_standard_v3.0.0.md)
- Ordered treemap layouts: Bederson, Shneiderman and Wattenberg, "Ordered and Quantum Treemaps", ACM Transactions on Graphics, 2002 (from memory, not re-checked).
