# The Merge Works

**An interactive machine that explains git merges, parallel agent work, and the AI-Native SDLC — built as a reusable mental model, with a deliberate safety/security lens for regulated (e.g. healthcare) engineering.**

> **Open [`index.html`](./index.html) (live at [jagnyesh.github.io/merge-works](https://jagnyesh.github.io/merge-works/))** in any browser. No build step, no dependencies, no network calls — a single self-contained file. Works in light and dark, keyboard-navigable, respects reduced-motion.

---

## Why this exists

Two things are genuinely hard to *picture* at the same time:

1. **What a merged PR actually is** — the moment a reversible proposal becomes irreversible shared history.
2. **What running several AI coding agents in parallel does to the shape of the work** — and why "more agents" does not mean "more shipped."

Rather than draw another boxes-and-arrows diagram, this takes the approach of the [**"anatomy" agent skill**](https://github.com/wheresryan22/anatomy): *don't diagram the concept — invent a physical machine whose working **is** the concept*, drawn as an interactive isometric figure where one underlying model drives both the geometry and the live gauges.

The result is **The Merge Works**: a running machine where each part is a real step in the workflow, and the instrument panel reads live metrics off the simulation.

---

## The method: explain by building the mechanism

The core move is to pick a physical metaphor that is *load-bearing*, not decorative — every part must do the job its concept does:

| Machine part | What it really is |
|---|---|
| **Trunk line** (front rail) | `main`. It advances one notch per merge. |
| **A lane + its car** | one branch / one PR / one issue, flowing left→right through stations. |
| **Car colour** | the change's **risk tier** (high / medium / low). |
| **Build lamp** (red→green) | CI. Red while building/failing, green when it passes. |
| **Eval sub-check** | a continuous-eval gate that only fires on LLM-touching changes. |
| **Reviewer beam** (cyan) | fresh-context code review. It can only reach a fixed number of cars at once. |
| **The fitter** (amber) | `git rebase` onto the *current* trunk tip. |
| **Crane** | the merge itself — lifts a car and locks it onto the trunk. |
| **Plan hopper** (upstream) | the planning step: stamps an `intent.md`, sized by tier, into a queue. |
| **Maintain monitor + return-pipe** | post-merge signals (CI, eval, cost, trace) that re-file a new intent. |

Two ideas the machine is built to make *obvious*:

- **A merge is the lock-on.** Until the crane lifts it, a car is a reversible proposal on its own lane. The crane is the one irreversible step. (This is what "a merged PR" *means*.)
- **The trunk moves while you work.** Every merge forces the other in-flight cars back through the fitter (rebase) before their own lock-on. That reconciliation is the tax parallel work pays — and it's the part that's hardest to visualize until you watch it happen.

---

## Mapping to the Anthropic *AI-Native SDLC Playbook*

The machine traces all six stages — simplified — of Anthropic's [AI-Native SDLC Playbook](https://claude.com/resources/articles/the-ai-native-sdlc-playbook) (Louis Claxton, Aug 2026):

| Stage | In the machine |
|---|---|
| **1 · Plan** | the hopper stamps an `intent.md` and feeds the queue |
| **2 · Design** | `intent.md` expands to `spec.md`; planning weight scales with risk tier (local) |
| **3 · Build** | plan mode → committed `plan.md` → diff + tests on a branch (`CLAUDE.md`, skills, hooks named but not drawn) |
| **4 · Test** | build goes green; changes to `CLAUDE.md`/skills/hooks run the eval suite; each fixed breach adds a regression eval |
| **5 · Deploy** | PR review → rebase → merge, human approval for high/medium; no release gate yet |
| **6 · Maintain** | post-merge signals against control bands: 1σ log, 2σ read-only diagnosis, 3σ → fresh `intent.md` at the front of the queue |

### Where it aligns

The single most faithful element is the **review-bandwidth ceiling**. The playbook warns that under human-speed controls "either the review queue builds or code ships under-reviewed," and caps parallel sessions by "how many streams one person can review properly." The machine dramatizes exactly this: one reviewer, a fixed **WIP of 2**, and a **review-debt** gauge that climbs the moment you add lanes faster than the beam can cover them. Parallel sessions, selective human review ("layers of agentic review with human review reserved for regulated and critical code"), continuous merge, and CI feedback all map cleanly.

### Where it honestly diverges (read these twice)

- **Risk tiers are an extension, not a playbook construct.** The playbook prescribes `intent.md` + plan-mode-as-default; the high/medium/low *classifier* that routes planning weight and merge authority is a local adaptation for a blast-radius-sensitive codebase. Aligned in spirit with "human review reserved for regulated and critical code" — but don't claim the playbook mandates it.
- **There's a merge gate but no release gate.** The playbook gates at both: code-owner approval at merge, and an approval hook (with dev/staging/prod autonomy tiers) at release. The machine stops at merge because there's no production deploy yet. The Maintain loop is likewise *adapted*: it draws from post-merge signals (nightly CI, eval regressions, cost drift, traces), not a live prod environment.
- **Rebase is an extension.** The playbook doesn't cover rebasing or merge conflicts; its answer to parallel sessions colliding is one worktree per session. The fitter is here because the trunk-moves-while-you-work cost is real git behavior worth seeing, not because the playbook describes it.
- **Simplified, not complete.** The artifact chain (`intent.md → spec.md → plan.md → diff+tests → PR → incident`) appears only as labels. `CLAUDE.md`, skills, hooks-as-gates, `REVIEW.md`, managed settings and the 2σ/3σ split between "open a PR" and "run a runbook" aren't modeled. "Front of the queue" for breaches is a local priority rule.
- **Low-tier self-merge is a local deviation.** The playbook keeps code-owner approval on every PR ("branch protection still requires approval from a code owner"; "the agent that wrote the code has no way to approve it"). The machine lets low-tier cars merge without the reviewer beam; the attested-SHA rule below narrows that risk but doesn't match the playbook.
- **Don't let the picture oversell automation.** If Plan-feeds-queue and Maintain-auto-refiles are still design intent rather than shipped machinery, mark them as such. A diagram that shows everything running reads as "we do all this" when you don't yet.

---

## The safety & security lens

This is the part that matters most in regulated engineering, where a wrong change isn't just a bug — it's an auditable, sometimes reportable, event. The practices below are what make faster AI-assisted development *safer*, not just quicker:

- **Plan weight by blast radius, not by autonomy duration.** A change touching a PHI boundary, auth, or cohort/phenotype semantics earns a full design-and-ADR chain; a docs-only change earns the issue body. Spend rigor where a mistake is expensive and auditable — nowhere else.
- **Never review in the authoring session.** A session grading its own homework assumes its own intent. Review runs in a fresh context, and the pipeline enforces it mechanically rather than by good intentions.
- **A human merges anything behavior- or PHI-touching.** Low-risk, well-bounded classes (docs, harness-internal) may self-merge under a standing rule — but only against a review attestation bound to the exact reviewed commit SHA, so a post-review push can't slip through.
- **Lock test/golden/eval fixtures before implementation.** "Regenerate the expected outputs to match" is exactly how a silent correctness bug sails through a green build. Commit tests first, then freeze them.
- **Vet every tool and skill before it runs, cold, after the task — never mid-task on an agent's suggestion.** Read the whole `SKILL.md` *and* its bundled scripts (not the README); grep for network calls, `curl | bash`, credential/env access, and package installs; require a named maintainer and a pinned version/commit; trial anything that executes commands in a throwaway sandbox first. Prefer the lightest mechanism that works — every gate you add is itself attack surface and maintenance.
- **Verified merges per week is the metric — not raw merges.** A merge without evidence (test output, an exercised end-to-end path, a consolidated review verdict) is inventory of future rework. Faster models accrue that inventory *faster*, so the discipline matters more as capability rises, not less.
- **Empirical confirmation over static analysis at interface boundaries.** For bugs at wrappers, third-party APIs, async layers, or caches, read the wire / dump runtime state before scoping a fix. Tests that assert at the wrapper layer (not the wire) have let real bugs run silently for months.

### A worked example of the discipline (this repo)

This repository is itself a small case study. The original request was to publish *the whole conversation* that produced the artifact. The assistant flagged that the session context contained a live-looking API-key fragment, descriptions of unfixed vulnerabilities in a healthcare system, and private file paths — and **refused to publish the raw transcript**, proposing a sanitized, author-curated writeup instead. Only the generalized artifact and this README were made public (via GitHub Pages). That is the safety lens applied to the act of sharing itself: *sending content to an external service publishes it; it may be cached or indexed even after deletion.* When in doubt, sanitize and scope down.

---

## Using the artifact

Open `index.html` and:

- **Run / Pause / Step** — drive the simulation (Step is there for reduced-motion).
- **Lanes (1–5)** — the key experiment. Push to 4–5 and watch `open→green` stay flat while `green→merge` balloons and **review debt** climbs red. That's the ceiling: the reviewer, not the machine.
- **Fire 3σ breach** — manually trigger the Maintain loop; the breach travels the return-pipe back to the hopper and re-files a new intent. Background signals also land at 1σ (logged) and 2σ (read-only diagnosis), which don't re-file.

The gauges — queue depth, open→green, green→merge, reviewer WIP, review debt, verified merges, 3σ breaches re-filed — are all read live off the same model that draws the parts.

---

## Provenance & sanitization

- The **README is generalized** — no product names, issue numbers, file paths, or internal specifics — so it's safe to read on its own terms.
- The **artifact** is generalized the same way — no internal project names or workflow-doc references — and contains no secrets, data, or vulnerability details. It is published publicly via GitHub Pages.
- Nothing here contains credentials, PHI, or security-sensitive internals. If you spot something that looks otherwise, open an issue and it gets scrubbed.

## Credits

- Visual method: the [**anatomy** agent skill](https://github.com/wheresryan22/anatomy) — interactive isometric technical illustration.
- Framework: Anthropic's [**AI-Native SDLC Playbook**](https://claude.com/resources/articles/the-ai-native-sdlc-playbook).
- Built collaboratively with Claude Code (Opus 4.8).
