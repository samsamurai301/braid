<!-- Exported from https://claude.ai/artifact/WMvKpKb71EKkniYqHy2kJ5 on 31 Oct 2026. The doc is the living version. -->

# Braid · Phase 1 — Vision & Ideation

Oct 31, 2026 · @Sam

## Summary

Braid is an open-source visual workspace for building software with AI agents: one live map of your code and your agents' work, where you understand the system, sketch what you want, send agents to build it, watch them work, and review the result as a picture before you read a single diff.

**See the system. Sketch the intent. Send the agents. Review the braid.**

This widens yesterday's Concept Atlas. The atlas framed Braid as an open alternative to GitLens and Graphite: multi-repo workspaces, worktrees, stacks and line-level AI provenance. All of that stays. It becomes the engine underneath and the Watch, Review and Remember parts of the product. What's new is the front half of the loop, the part where people think: a living map of the code, an intent canvas to plan on, and a way to hand that plan to any agent.

The bet in one sentence: in AI-era coding the scarce resource is no longer typing but human attention, and people spend attention best on pictures they can trust.

| Phase | Question it answers | Output |
| --- | --- | --- |
| 1 · Vision & ideation | What is Braid, and why will people care? | This doc |
| 2 · Product definition | For whom, for which jobs, and which features first? | Doc: personas, journeys, feature catalogue, metrics |
| 3 · Logic & architecture | How does it work inside? | Doc: data model, engine, algorithms, surfaces |
| 4 · Design | What does it look and feel like? | Design system + screen designs |
| 5 · Execution | How do we build, launch and grow it? | Doc: roadmap, OSS plan, risks, CLAUDE.md handoff |

## The problem: coding became steering, and steering is blind

A developer running three to six agents now produces more code per day than they can read, so the limit on speed has moved from writing code to understanding, directing and checking it. All four of those jobs are thinking jobs, and today's tools serve them as text: chat transcripts, terminal logs, file trees and line-by-line diffs.

People think about systems in pictures. Architects draw boxes on whiteboards and reviewers sketch call paths on paper. Those drawings go stale the day they're made, and they aren't connected to the code or to the agents changing it.

| Moment | What people do today | What goes wrong | What Braid shows instead |
| --- | --- | --- | --- |
| Understand | Read files, ask an agent to explain, open old diagrams | Mental models lag behind code that agents change daily; nobody holds the whole picture | A live map of every repo, kept current from the code itself |
| Plan | Write a prompt or a plan file in chat | The plan disappears after the session and is never linked to the code it changed | An intent canvas whose notes stay pinned to the code they're about |
| Direct | Juggle terminals, worktrees and agent tabs | Two agents edit the same file; work gets lost; context is guessed | Agents placed on the map, each in its own worktree, with overlaps flagged before they start |
| Watch | Scroll logs, poll git status | You notice problems at merge time | Live activity on the map and a shared timeline across repos |
| Review | Read diffs file by file in alphabetical order | Review fatigue; AI-written changes get rubber-stamped | The change as a story: where it landed, what kind of change, what it could break, who or what wrote it |
| Remember | Search commit messages and chats | Nobody can say why code exists or which agent wrote it | Every line linked to its intent, session and evidence |

The result is a growing comprehension debt: code that works, that nobody on the team has a mental model of, and that nobody can explain. Braid's job is to pay that debt down continuously, as a side effect of normal work.

The evidence from 2026 points the same way:

- **Review is the bottleneck.** Faros AI's study of 22,000 developers found median time in review up 441.5% and pull requests merged with no review up 31.3% as AI adoption rose ([Faros, Apr 2026](https://faros.ai/blog/ai-acceleration-whiplash-takeaways)).
- **Attention caps parallelism.** Practitioners describe two parallel agent sessions as their limit, and propose capping work in progress by review capacity ([HN thread, Mar 2026](https://hn.algolia.com/api/v1/items/47573483)).
- **Understanding lags.** Addy Osmani reports an Anthropic trial in which the AI-assisted group scored 17% lower on comprehension of the code ([Osmani](https://addyosmani.com/blog/comprehension-debt/)).
- **Trust depends on checkability.** In the Stack Overflow 2026 survey, 48.0% trust AI output only when they can easily verify it ([Stack Overflow](https://survey.stackoverflow.co/2026)).
- **Plans drift.** Spec-driven tools have no path from code back to the spec; drift detection exists only as community add-ons ([Godel](https://www.godeltech.com/change-propagation-sdds-central-unsolved-challenge/)).

## Who it's for

The first user is the **agent conductor**: an experienced developer who runs several AI agents in parallel across several repositories and has lost the overview. They feel the pain daily, they install tools from a README, and they show their setup to others. Everyone else follows from them.

| Audience | Their situation | What they want from Braid | Wave |
| --- | --- | --- | --- |
| Agent conductor (senior, staff or indie developer) | Runs Claude Code, Codex or Cursor in 3–6 worktrees across 2–10 repos | One view of every agent and repo, no collisions, fast and trustworthy review | 1 · first users |
| Tech lead or architect | Reviews more AI-written PRs than they can read; owns the system's shape | A current picture of the architecture, boundaries agents respect, review that starts with "where should I look" | 1 |
| Newcomer to a codebase | Joined a team or picked up an open-source repo | A guided map: what lives where, how a request flows, who knows what | 2 |
| Agent-first builder (designer, PM, founder who ships with agents) | Builds real software through agents but can't read every line | To see what the agents built, where, and whether it hangs together, without reading all the code | 2 · biggest growth audience |
| Open-source maintainer | Gets more AI-assisted contributions from strangers | Contributions shown as a picture with provenance; a public map of the project for contributors | 2 |
| Engineering manager or org | Needs to know how much code is AI-written, whether it lasts and whether it's reviewed | Team-level dashboards, policies and audit, self-hosted | 3 · paid |

The agent-first builder deserves a note. This group is growing fastest and gains the most from visual thinking, because the map replaces knowledge they don't have. It is also the group most likely to spread Braid beyond professional developers, which is the stated goal of open-sourcing it. Wave 1 stays with the conductor so the product is earned on hard technical ground first.

## Principles

Ten rules decide trade-offs from here on. When two features compete, the one that honours more of these wins.

1. **Picture first, text one click away.** Every view answers one question at a glance, and every mark on it opens the real code, diff or session behind it.
2. **Places don't move.** The map keeps a stable layout across days and changes, so people build spatial memory: "payments is bottom left" stays true. New code appears near its relatives instead of reshuffling the map.
3. **Derived, never drawn by hand.** Maps come from the code, git and agent events, and update live. A hand-drawn note is allowed only when it is anchored to real code, so it can't silently go stale.
4. **Show evidence, not guesses.** Every claim carries its source and strength. AI authorship has evidence grades; unknown stays unknown and is never counted as human.
5. **Local-first and private.** It runs on your machine with no account. Code never leaves unless you set up a server yourself.
6. **Bring your own agent.** Braid works with Claude Code, Codex, Cursor, Copilot and any agent that speaks MCP or ACP. It never becomes a coding agent or an editor.
7. **Meet people where they work.** The same UI appears in the browser, the desktop app, VS Code and its forks, JetBrains, and inside agent tools.
8. **Fast enough to feel like a map.** Smooth zoom and pan with 100,000 nodes, and updates within a second of a file changing. If it feels like loading a report, it has failed.
9. **Open formats, open source.** Apache-2.0 code, documented file formats, and everything exportable. Users can leave, and that's why they'll stay.
10. **Calm, not surveillance.** Show work, not people. Team-level numbers by default, individual views opt-in, prompts off unless switched on.

## The idea space

Every idea belongs to one of six moments in a loop of work, plus two concerns that run across the loop and one engine underneath. The table lists all 62 ideas generated in this phase; Phase 2 ranks them into releases.

*[Diagram: Braid loop · six moments, one graph, one engine. See it in the doc: https://claude.ai/artifact/WMvKpKb71EKkniYqHy2kJ5]*

Work goes round the loop: what you remember feeds what you see next time. Every view in every moment is a lens on the same Braid Graph, which is why a note made while planning can show up during review.

Maturity: **Core** means Braid isn't Braid without it. **Strong** means high value and likely early. **Explore** means promising but unproven.

| Moment | Idea | What it does | Maturity |
| --- | --- | --- | --- |
| See | Living map | Zoomable map of all repos down to functions, built from code and updated live | Core |
| See | Stable layout | Positions persist across changes; new code lands near its relatives | Core |
| See | Lenses | Overlays for changes, churn, AI share, ownership, tests, complexity, agent activity | Core |
| See | Cross-repo links | Packages and imports connected across repositories | Core |
| See | Fly-to search | A search result zooms the map to the place | Core |
| See | Path tracing | Shows how a request or call flows between two points | Strong |
| See | Ask the map | A question answered as a highlighted region, using the user's own agent | Strong |
| See | Guided tours | Generated walk-throughs of a feature for newcomers | Strong |
| See | Runtime overlay | Traces and errors from OpenTelemetry drawn on the map | Explore |
| Think | Intent canvas | Notes, frames, arrows and sketches above the map | Core |
| Think | Pinned notes | Notes anchored to code that follow renames and flag deletions | Core |
| Think | Sketch to plan | Turns a sketch into tasks per code area | Strong |
| Think | Acceptance criteria | Criteria on each task, reused as agent instructions and as review checks | Strong |
| Think | Boundaries | Draw allowed and forbidden dependencies, checked for agents and in CI | Strong |
| Think | Decision notes | Short decision records placed on the code they affect | Strong |
| Think | Ghost changes | Planned changes shown as faint nodes before any code exists | Explore |
| Think | Whiteboard import | Brings in Excalidraw, tldraw or Mermaid drawings and pins them to code | Explore |
| Direct | Send to agent | Drag a task onto Claude Code, Codex, Cursor or any MCP or ACP agent | Core |
| Direct | Worktree per task | Created, set up, parked and cleaned up automatically | Core |
| Direct | Agent lanes | Parallel sessions as lanes with status and elapsed time | Core |
| Direct | Approval inbox | Every permission prompt, question and finished task in one queue | Core |
| Direct | Visual context | Pick regions on the map for the agent's context, with a size estimate | Strong |
| Direct | Overlap warning | Warns before dispatch when two tasks claim the same code | Strong |
| Direct | Playbooks | Reusable multi-step flows such as plan, build, test, review | Explore |
| Watch | Agents on the map | Live markers with trails of files read and edited | Core |
| Watch | Collision radar | In-memory test merges between live worktrees | Core |
| Watch | The braid | Multi-repo timeline with change sets stitched together | Core |
| Watch | Activity stream | Meaningful events as they happen, such as "export changed" or "test added" | Strong |
| Watch | Drift tide | Branches and worktrees falling behind their base | Strong |
| Watch | Notifications | Desktop or phone alert when an agent needs a person | Strong |
| Watch | Cost meter | Time and token use per session, where the agent reports it | Explore |
| Review | Change story | A PR or change set shown as a picture first | Core |
| Review | Semantic diff (JS/TS) | Tells formatting, renames, moves, logic and API changes apart | Core |
| Review | Dependency-order diff | Files in the order the change flows, not alphabetical | Strong |
| Review | Blast radius | Callers in every repo, marked updated or not | Strong |
| Review | Criteria coverage | Which acceptance criteria each change addresses, and which none does | Strong |
| Review | Look here first | Changes ranked by risk: hotspots, AI code without tests, protected paths | Strong |
| Review | Comments to agent | Review notes placed on the map, sent back as agent instructions | Strong |
| Remember | Line-level provenance | Human, AI, mixed or unknown per line, with an evidence grade | Core |
| Remember | Why-links | The chain from a line to its session, intent and review | Core |
| Remember | Session replay | Scrub through an agent session on the map | Strong |
| Remember | Map history | A time slider showing how the system grew | Strong |
| Remember | Format interop | Reads and writes Git AI notes and Agent Trace records | Strong |
| Remember | Survival curves | How long AI-written code lasts compared with human code | Explore |
| Share | Snapshot export | Any view as an image, SVG or link | Core |
| Share | Braid cards | The change story posted to the PR as an image | Strong |
| Share | README map | A live, embeddable map of a repo | Strong |
| Share | Public atlas | Maps of well-known open-source repos as a showcase | Strong |
| Share | Live sessions | Several people on one canvas at once | Explore |
| Govern | Privacy defaults | Team-level numbers, prompts off, works-council pack | Core |
| Govern | Team dashboards | AI share, known provenance, review coverage | Strong |
| Govern | PR gate | Policy findings posted as a check on the PR | Strong |
| Govern | Boundary policy | Drawn boundaries enforced in CI | Strong |
| Govern | Org admin | SSO, roles, audit log, retention | Explore |
| Engine | braidd daemon | One local Rust process owning git, parsing, graph and storage | Core |
| Engine | Braid Graph | One graph of code, work and people under every view | Core |
| Engine | Workspace manifest | A file listing repos, groups and owners | Core |
| Engine | MCP server | Agents query the map, the fleet and provenance | Core |
| Engine | Worktrees and stacks | Worktree lifecycle, native GitHub stacks, stacks on other forges | Core |
| Engine | Language tiers | JS/TS in depth first, tree-sitter languages next, SCIP for precision | Core |
| Engine | Plugin API | New lenses, languages and agent adapters from the community | Strong |
| Engine | Desktop and IDE shells | One web UI inside Tauri, VS Code, JetBrains and the browser | Core |

## Signature concepts

Ten ideas carry the product. Each is something people will screenshot and show a colleague; together they make one connected place rather than ten features.

### 1. The living map

The whole workspace as one zoomable map. Zoomed out, repos are regions grouped by role (apps, services, libraries); zoom in and packages, modules, files and finally functions appear, each level with its own labels and connections. It is built from the code and updates as files change.

**New because** existing code diagrams are generated once per question. Braid's map is persistent, stable and live, and every other view draws on top of it.

### 2. Lenses

Every question is a lens over the same map: recent changes, churn, AI share, ownership, test coverage, complexity, live agent activity, runtime errors. You switch the lens, not the screen, so the picture of "where" stays fixed while "what" changes. Lenses are plugins, so the community can add new ones.

### 3. The intent canvas

A thinking layer above the map: notes, frames, arrows and sketches, like a whiteboard. Any note can be pinned to real code, such as a module, a function or a whole repo. Pinned notes move with the code and flag themselves when their target changes or disappears. A sketch can become a plan: Braid proposes tasks per area of code, each with acceptance criteria.

**New because** plans today live in chat and vanish with the session. Here the plan is a lasting object, sitting on the code it's about, and real changes tick it off.

### 4. Send to an agent

Drag a task onto an agent. Braid creates a worktree, packs the context (the pinned code, the intent, the acceptance criteria and the boundaries to respect) and starts the agent, or hands the brief to a session you already have open. You choose the context visually by selecting regions on the map, and see roughly how much of the agent's context window it takes.

### 5. Agents on the map

Every live agent session appears in the region it's working in, with a trail of files it read and files it edited. When two agents' territories overlap, the overlap lights up; the collision radar test-merges their worktrees in memory and warns before a conflict exists. An approval inbox gathers everything waiting on a person: permission prompts, questions, finished work.

### 6. The braid

The namesake view: every repo on one shared time axis. Changes that span repos are stitched together, worktrees run as live strands, and each commit is coloured by who or what wrote it. It answers "what happened, where, and in what order" for a whole system at once.

### 7. The change story

A pull request or change set opens as a picture before the diff: a mini-map with the touched areas lit; each change classed as formatting, rename, move, logic or API change; blast-radius rings showing what could break; a provenance ribbon; which acceptance criteria the change covers; and a "look here first" list ranked by risk. Then the diff, in dependency order instead of alphabetical order. Comments placed on the map go back to the agent as instructions.

### 8. Why-links

Click any line and see its history as a chain: who or what wrote it and how sure we are, the agent session, the intent note that asked for it, and the review that approved it. A time slider on the map shows how the system grew and which agent built what.

### 9. Shareable by design

Every view exports as a crisp image or a live link. Pull requests can get a Braid card, the change story as an image, posted automatically. Repos can embed a live map in their README. These are the things people post, which is how an open-source tool spreads.

### 10. Agents read the map too

The map isn't only for people. Through an MCP server, agents can ask Braid "where is checkout handled?", "what depends on this function?", "is anyone else editing here?" or "what was the intent behind this?". The same graph serves both audiences: agents make fewer blind edits, and Braid earns its place even when no one has the UI open.

### A morning with Braid

1. Sam opens Braid. The map shows four repos; two areas glow because agents changed them overnight.
2. He switches to the AI-share lens and sees the new express-pay module is 90% agent-written and has no tests.
3. On the intent canvas he sketches "add tests and a retry policy to express-pay", pins it to the module, and lets Braid split it into two tasks with acceptance criteria.
4. He drags one task onto Claude Code and one onto Codex. Braid creates two worktrees and warns that both will touch `useCart.ts`; he narrows one task's region and the warning clears.
5. While they work, he watches their markers move across the map and jumps from the inbox to one permission prompt, which he answers in the agent's own window.
6. When they finish, he opens the change story: both criteria covered, one export changed with two callers not updated in another repo. He comments on the map; Codex fixes it.
7. He merges. The intent note turns into a why-link on the new code, and the PR carries a Braid card for his teammate.

## Landscape and positioning

"Visual" is no longer empty ground: in 2026 Cursor, GitHub and Google all shipped canvases or artifacts for agent work. What nobody ships is a persistent, live map of the code with the agents working on it, plans that stay linked to the code afterwards, and an open, local engine that spans repos and vendors. That is Braid's position.

*[Diagram: Positioning map · what each tool shows, and how long it lasts. See it in the doc: https://claude.ai/artifact/WMvKpKb71EKkniYqHy2kJ5]*

The top-right corner is thin: the tools there map code structure but know nothing of agents or plans. Placement is a judgement from the research below, not a measurement.

| Category | Leading products, late 2026 | What they do well | What they miss |
| --- | --- | --- | --- |
| Agent vendors' canvases | [Cursor canvases](https://cursor.com/changelog/04-15-26) (Apr 2026), [Copilot app canvases](https://github.blog/changelog/2026-06-17-github-copilot-app-generally-available) (Jun 2026), [VS Code canvases](https://code.visualstudio.com/updates/v1_142) (preview, Oct 2026), [Antigravity artifacts](https://www.antigravity.google/docs/artifacts), [Claude Code artifacts](https://code.claude.com/docs/en/artifacts.md) | Interactive views agents build per task; comments flow back to the agent | Made per task and then left behind; tied to one vendor's harness; no lasting map of the code |
| Code maps | [Codemaps](https://cognition.com/blog/codemaps) (Devin Desktop), [DeepWiki](https://docs.devin.ai/work-with-devin/deepwiki), [CodeScene](https://codescene.io/docs/guides/technical/hotspots.html), [CodeViz](https://codeviz.ai), [GitDiagram](https://github.com/ahmedkhaleel2004/gitdiagram), [repowise](https://github.com/repowise-dev/repowise), [GitKraken Codemaps](https://gitkraken.com/solutions/codemaps) (still early access) | Click-through to source, semantic zoom (CodeScene), switchable colour modes | Snapshots or LLM output that varies between runs; not linked to agent work; cross-repo maps are enterprise-only; restrictive licences |
| Agent managers | [Conductor](https://www.conductor.build), [Superset](https://github.com/superset-sh/superset), [Nimbalyst](https://github.com/Nimbalyst/nimbalyst), [T3 Code](https://github.com/pingdotgg/t3code), [Claude Code agent view](https://code.claude.com/docs/en/agent-view.md), [Zed threads](https://zed.dev/docs/preview/ai/parallel-agents.html), terminal canvases such as [Cate](https://github.com/0-AI-UG/cate) | Worktree per task, inbox grouped by state, batched review comments | Show terminals and cards, not where in the code agents work; mostly one repo; ELv2, AGPL or proprietary; heavy Electron apps |
| Spec and plan tools | [Kiro](https://kiro.dev/docs/specs/index), [Spec Kit](https://github.com/github/spec-kit) (v1.0, Aug 2026), [Traycer](https://docs.traycer.ai/tasks/phases.md), [Augment Intent](https://www.augmentcode.com/blog/intent-a-workspace-for-agent-orchestration), [Plannotator](https://github.com/backnotprop/plannotator), [Builder.io visual plan](https://www.builder.io/blog/claude-code-plan) | Approve the plan first; run tasks in waves from a dependency graph; map the diff back onto the plan | Plans drift from code; nothing flows from code back to the plan ([Godel](https://www.godeltech.com/change-propagation-sdds-central-unsolved-challenge/)) |
| Git and review | GitLens and GitKraken, Graphite (owned by Cursor), GitButler (Fair Source), [CodeRabbit Change Stack](https://coderabbit.ai/blog/introducing-atlas-the-first-ai-native-code-review-interface), GitHub stacked PRs | Commit graphs, stacks, PRs read as ordered groups | Review still centres on the diff; no neutral line-level provenance |
| AI provenance | Git AI (team now at OpenAI), Agent Trace, Entire | Record which lines an AI wrote | Fragmented formats, each tied to a different camp |

Where Braid can clearly win:

1. **Agents on a map of the code, not on a grid of terminals.** Every open-source canvas tool lays out terminals; Codemaps is proprietary and static.
2. **Plans that stay linked to code.** Plan nodes tied to commits, hunks and tests, with drift flagged, stored as plain files in the repo.
3. **Review the whole batch.** One change map across many agents and repos, ranked by risk, instead of one diff per session.
4. **A permissive, embeddable core.** Apache-2.0 Rust that other tools can build on, neutral between agent vendors.
5. **Speed.** A native core and a GPU-drawn map, where users complain that Electron agent managers freeze and use gigabytes of memory.
6. **Multi-repo on your own machine.** Today multi-repo agent work is cloud-only and proprietary (Copilot projects, Cursor cloud environments).

## Open-source thesis

Open source is Braid's distribution strategy, not a side choice: the goal is for many developers to adopt a visual way of working, and a permissive licence keeps Braid embeddable, trusted and eligible for public funding.

**Licence and governance**

- Apache-2.0 for the apps, extensions and plugins; `MIT OR Apache-2.0` for the Rust crates, the Rust ecosystem's convention ([Rust API guidelines](https://rust-lang.github.io/api-guidelines/necessities.html)).
- Never relicense. The organisation server lives in a separate repo under its own licence from day one. Projects that relicensed later were forked: Terraform became [OpenTofu](https://www.theregister.com/2023/09/20/terraform_fork_opentf_opentofu/) and Redis became [Valkey](https://phoronix.com/news/Linux-Foundation-Valkey).
- Use the DCO (a sign-off on each commit), not a CLA. A CLA signals a possible relicense, as [tldraw's 2023 change](https://tldraw.dev/blog/license-update-for-the-tldraw-sdk) showed, and adds friction; [OpenInfra dropped CLAs](https://openinfra.org/dco/) for that reason.
- Register the final name as a word mark and publish a policy that lets forks exist under another name.
- Keep dependencies clean: no tldraw (its SDK needs a [production licence key](https://tldraw.dev/sdk-features/license-key)), no CC-BY-NC Cosmograph packages, EPL and MPL libraries only unmodified, and a licence check in CI.

**How Braid spreads**

| Lever | What Braid does | Precedent |
| --- | --- | --- |
| Output people want to post | Map snapshots, Braid cards on PRs, a public atlas of famous repos | tldraw's Make Real demo drew about 22M views in 30 days ([Latent Space](https://www.latent.space/p/tldraw)) |
| Living inside existing tools | VS Code Marketplace and Open VSX, MCP server, Claude Code and Codex plugins, ACP | Cline reached 5M installs, helped by Open VSX and an MCP marketplace ([Cline](https://cline.bot/blog/5m-installs-1m-open-source-grant-program)) |
| One command, no account | `brew install braid`, then `braid open` | uv and OpenCode grew on one-line installs ([uv docs](https://docs.astral.sh/uv/getting-started/installation/)) |
| Building itself in public | Braid maps and records its own development; the share of Braid written by agents is a public number | Aider reports how much of each release it wrote itself ([Aider](https://github.com/Aider-AI/aider)) |
| Launch moments | A Show HN where the open-sourcing is the story, then recurring launch weeks | Zed's open-source post reached 1,576 points ([plushcap](https://www.plushcap.com/companies/zed/hacker-news/2024/500)); Supabase's launch weeks ([blog](https://supabase.com/blog/supabase-how-we-launch)) |
| Contributor on-ramps | A lens and adapter API, good first issues, an AI-contribution policy | Excalidraw gave commit access after a first merged PR ([web.dev](https://web.dev/articles/excalidraw-and-fugu)); Ghostty's [AI policy](https://github.com/ghostty-org/ghostty/blob/main/AI_POLICY.md) |

**Funding path in Germany**

- The [Prototype Fund](https://www.prototypefund.de/en/application) pays €47,500 for a solo applicant for six months from 1 June 2027. Applications close on **30 November 2026, 15:00 CET**. It requires residence in Germany and working self-employed, freelance or released by an employer; Braid would apply as open developer infrastructure.
- [NLnet](https://www.nlnet.nl/propose) grants of €5,000–50,000 at its next general call.
- The [Sovereign Tech Fund](https://sovereign.tech/programs/fund) later, once Braid's core is a dependency of other projects; it does not fund user-facing apps or prototypes.
- GitHub Sponsors from the public alpha, and the paid organisation tier after that.
- A source-available licence on the core would rule Braid out of all of these.

**Failure modes to design against**

- A single point of failure: domains, registries, marketplace publishers and Discord sit under an organisation account with at least two admins. When Rome's company shut down, its maintainers lost all of these ([Biome](https://biomejs.dev/blog/annoucing-biome)).
- Floods of low-quality AI pull requests: a contribution policy and Braid's own provenance checks on incoming PRs.
- Maintainer burnout: funding early, scope kept small.

## Bets, risks and what Braid won't build

Braid rests on five bets. If any of them proves false, the plan changes:

1. People will keep running several agents in parallel, so the overview problem keeps growing.
2. A code map can be both stable and live at the scale of real company code, and useful every day rather than once.
3. Agent tools will keep exposing hooks and MCP, so an outside tool can see edits and feed context.
4. Reviewing a change as a picture first measurably saves time or catches more problems than reading diffs alone.
5. A neutral, open tool can win against visual features bundled into a single vendor's agent.

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| "Pretty but not daily": visual tools get admired, then abandoned | High | High | Lead with daily jobs (agent fleet, review, provenance); the map is the base of those views, not a separate place to visit. Track weekly return from day one. |
| Agent vendors ship visual features inside their own tools | High | Medium | Be the layer across vendors and repos: local, open formats, every agent. Read their data instead of fighting it. |
| Scope is far too big for a small team | High | High | Phase 2 cuts hard. The first release is JS/TS only and does a few things completely. |
| Stable layout at 100,000 nodes is hard | Medium | High | Hierarchical grouping and semantic zoom, so the screen never draws everything at once. A layout spike is the first engineering task in Phase 5. |
| Agent hook APIs keep changing | High | Medium | One adapter per agent, tested against recorded payloads; the file watcher keeps coverage when a hook breaks. |
| AI-powered features (sketch to plan, ask the map) need models and keys | Medium | Medium | The core map, fleet, provenance and review need no model at all. AI steps run through the user's own agent, so Braid holds no API keys and sends no code anywhere. |
| Monitoring concerns in Germany and the EU | Medium | Medium | Team-level by default, prompts off, self-hosting, and a ready-made works-council configuration for the paid tier. Not legal advice. |
| The name "Braid" is already taken in software | Medium | Low | Run a trademark and package-name check before the public launch; keep a second name ready. |

Braid will not build:

- An editor or IDE. It lives beside them.
- A coding agent or its own model. It directs the agents people already use.
- A git host or a CI system.
- Free-floating architecture diagrams. Every drawing is either derived from code or pinned to it.
- Classifier-based "AI detection" presented as fact. It is opt-in and always shown as the weakest evidence grade.
- Rankings of individual developers.

## Decisions and open questions

Phase 1 settles the direction; Phase 2 settles the first release.

| Decision | Choice | Why |
| --- | --- | --- |
| Scope | A visual workspace for AI-era coding; the atlas becomes its engine and its Watch, Review and Remember parts | The project's stated goal is a visual way of working, and monitoring alone is a feature others are absorbing |
| First user | The agent conductor, on JS/TS repos | Sharpest pain, most likely to install from a README and show others |
| Architecture direction | One local Rust daemon, one web UI in every shell | Keeps yesterday's v0.1 code useful and every surface consistent |
| AI inside Braid | No bundled model and no API keys; AI steps run through the user's own agent | Privacy, zero running cost, and neutrality between vendors |
| Open source | Everything a developer uses is open source; only the organisation server is paid | Adoption first; the paid line sits where companies, not individuals, feel the need |
| Name | Braid, as a working name | Check trademark and package names before launch |

Open questions for Phase 2:

- [ ] Which job sits at the centre of the first release: understanding the code (map first) or running agents (fleet first)?
- [ ] Which shell comes first: the desktop app, a browser tab on localhost, or a VS Code panel?
- [ ] Does "send to an agent" launch agents itself in v1, or hand a brief to a session the user already has open?
- [ ] Which agents are supported at launch: Claude Code, Codex and Cursor, or fewer?
- [ ] How deep does the first map go: files, or down to functions?
- [ ] Is live multiplayer on the canvas part of the open-source core, and when?
- [ ] Do agent-first builders get a simpler mode, or the same interface with better defaults?
- [ ] Which single number tells us Braid is working in its first six months?

## Sources

Research checked on 31 October 2026. Earlier market facts (GitHub stacked PRs, the Cursor–Graphite and SpaceX–Cursor deals, provenance formats) come from the [Braid Concept Atlas](https://claude.ai/code/artifact/fc2b337b-e431-405f-af39-bb869214eaa8) of 30 October and its source list.

Problem and practice

- [Faros AI: AI acceleration whiplash](https://faros.ai/blog/ai-acceleration-whiplash-takeaways)
- [Addy Osmani: Comprehension debt](https://addyosmani.com/blog/comprehension-debt/)
- [Stack Overflow Developer Survey 2026](https://survey.stackoverflow.co/2026)
- [Godel: change propagation in spec-driven development](https://www.godeltech.com/change-propagation-sdds-central-unsolved-challenge/)

Visual and agent tools

- [Cognition: Codemaps](https://cognition.com/blog/codemaps) · [DeepWiki docs](https://docs.devin.ai/work-with-devin/deepwiki)
- [Cursor 3.1 changelog: canvases](https://cursor.com/changelog/04-15-26)
- [GitHub Copilot app GA](https://github.blog/changelog/2026-06-17-github-copilot-app-generally-available)
- [VS Code 1.142 release notes](https://code.visualstudio.com/updates/v1_142)
- [Antigravity artifacts](https://www.antigravity.google/docs/artifacts)
- [Claude Code artifacts](https://code.claude.com/docs/en/artifacts.md) · [agent view](https://code.claude.com/docs/en/agent-view.md)
- [CodeScene hotspots](https://codescene.io/docs/guides/technical/hotspots.html) · [CodeViz](https://codeviz.ai) · [GitDiagram](https://github.com/ahmedkhaleel2004/gitdiagram) · [repowise](https://github.com/repowise-dev/repowise)
- [GitKraken Codemaps](https://gitkraken.com/solutions/codemaps)
- [Conductor](https://www.conductor.build) · [Superset](https://github.com/superset-sh/superset) · [Nimbalyst](https://github.com/Nimbalyst/nimbalyst) · [T3 Code](https://github.com/pingdotgg/t3code) · [Cate](https://github.com/0-AI-UG/cate)
- [Kiro specs](https://kiro.dev/docs/specs/index) · [GitHub Spec Kit](https://github.com/github/spec-kit) · [Traycer phases](https://docs.traycer.ai/tasks/phases.md) · [Augment Intent](https://www.augmentcode.com/blog/intent-a-workspace-for-agent-orchestration) · [Plannotator](https://github.com/backnotprop/plannotator) · [Builder.io visual plan](https://www.builder.io/blog/claude-code-plan)
- [CodeRabbit Change Stack](https://coderabbit.ai/blog/introducing-atlas-the-first-ai-native-code-review-interface)

Open source and funding

- [Rust API guidelines: licensing](https://rust-lang.github.io/api-guidelines/necessities.html)
- [tldraw licence update](https://tldraw.dev/blog/license-update-for-the-tldraw-sdk) · [tldraw licence key](https://tldraw.dev/sdk-features/license-key)
- [OpenInfra: moving to the DCO](https://openinfra.org/dco/)
- [The Register: OpenTofu](https://www.theregister.com/2023/09/20/terraform_fork_opentf_opentofu/) · [Phoronix: Valkey](https://phoronix.com/news/Linux-Foundation-Valkey)
- [Latent Space: tldraw](https://www.latent.space/p/tldraw) · [Cline: 5M installs](https://cline.bot/blog/5m-installs-1m-open-source-grant-program) · [Aider](https://github.com/Aider-AI/aider) · [Supabase launch weeks](https://supabase.com/blog/supabase-how-we-launch) · [web.dev: Excalidraw](https://web.dev/articles/excalidraw-and-fugu)
- [Biome announcement](https://biomejs.dev/blog/annoucing-biome) · [Ghostty AI policy](https://github.com/ghostty-org/ghostty/blob/main/AI_POLICY.md)
- [Prototype Fund application](https://www.prototypefund.de/en/application) · [NLnet](https://www.nlnet.nl/propose) · [Sovereign Tech Fund](https://sovereign.tech/programs/fund)
