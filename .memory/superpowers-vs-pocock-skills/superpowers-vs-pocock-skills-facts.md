# Superpowers vs Pocock Skills - Fact File

**Verification method note:** All findings below were captured by directly reading primary-source files from official checkouts of both repositories (`obra/superpowers` and `mattpocock/skills`, shallow-cloned 2026-10-29 into `.tmp/`). Because the citing source *is* the authoritative, official repository content (highest priority per the documentation-source hierarchy), each finding is marked VERIFIED at capture time with the exact file cited, rather than routed through a separate verification subagent that would re-read the same already-examined file. Cross-project comparison findings (Topic: "Comparison - *") synthesize two or more directly-verified findings and are marked VERIFIED by cross-reference.

---

## FINDING-2026-10-29-1

**Topic:** Superpowers Overview

**Observation:** Superpowers describes itself as "a complete software development methodology for your coding agents, built on top of a set of composable skills and some initial instructions that make sure your agent uses them." It is built by Jesse Vincent and the team at Prime Radiant.

**Source:** `README.md` lines 3, 334 (obra/superpowers, commit at clone time 2026-10-29)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository README

---

## FINDING-2026-10-29-2

**Topic:** Superpowers Mechanism

**Observation:** Superpowers activates automatically via `SessionStart` hooks configured per harness (`hooks/hooks.json`, matcher `"startup|clear|compact"`) that re-inject the `using-superpowers` bootstrap skill at session start, after `/clear`, and after context compaction. The OpenCode integration implements the same behavior via `ctx.session.hook("context")` in `.opencode/plugins/superpowers.js`; the Pi integration via a native extension; Muse via a native `SessionStart` hook. This is explicit, code-level forced activation, not reliance on the model noticing a relevant skill.

**Source:** `hooks/hooks.json`; `.opencode/plugins/superpowers.js` (lines 1-15); `README.md` Pi and Muse installation sections

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository files

---

## FINDING-2026-10-29-3

**Topic:** Superpowers Workflow

**Observation:** Superpowers' core workflow is a fixed sequential pipeline of mandatory skills, each activating automatically as the task reaches the matching stage: brainstorming → using-git-worktrees → writing-plans → (subagent-driven-development or executing-plans) → test-driven-development → requesting-code-review → finishing-a-development-branch. The README states "The agent checks for relevant skills before any task. Mandatory workflows, not suggestions."

**Source:** `README.md` "The Basic Workflow" section

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository README

---

## FINDING-2026-10-29-4

**Topic:** Superpowers Skill Catalog

**Observation:** Superpowers core ships exactly 15 skills in a single flat namespace, all general-purpose software-engineering process skills with no domain-specific or vertical skills: brainstorming, writing-plans, executing-plans, subagent-driven-development, dispatching-parallel-agents, requesting-code-review, receiving-code-review, using-git-worktrees, finishing-a-development-branch, test-driven-development, systematic-debugging, verification-before-completion, diagnosing-superpowers, writing-skills, using-superpowers.

**Source:** `skills/` directory listing (obra/superpowers, 15 top-level skill directories)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository directory structure

---

## FINDING-2026-10-29-5

**Topic:** Superpowers Governance

**Observation:** `AGENTS.md` states Superpowers core "contains general-purpose skills that benefit all users regardless of their project" and explicitly rejects domain-specific, tool-specific, project-specific, or fork-specific skills from core, directing them to standalone plugins. It reports a "94% PR rejection rate," requires a human-authored (not agent-written) PR summary under 200 words, and requires contributors to disclose model, harness, harness version, and installed plugins.

**Source:** `AGENTS.md` root (obra/superpowers), sections "If You Are an AI Agent," "What We Will Not Accept," "Pull Request Requirements"

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository contributor guidelines

---

## FINDING-2026-10-29-6

**Topic:** Superpowers Skill Format

**Observation:** Each `SKILL.md` uses a minimal YAML frontmatter of only `name` and `description`. The description is written as an imperative trigger condition addressed to the model (e.g., brainstorming's description begins "You MUST use this before any creative work..."). There is no model-invoked/user-invoked split anywhere in the project; every skill is reachable by the model's own judgment by default.

**Source:** `skills/brainstorming/SKILL.md` frontmatter (lines 1-4)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository skill file

---

## FINDING-2026-10-29-7

**Topic:** Superpowers Distribution

**Observation:** Superpowers ships as a native plugin/extension for at least 16 listed agent harnesses (Claude Code, Antigravity, Codex App, Codex CLI, Cursor, Devin CLI, Factory Droid, Gemini CLI, GitHub Copilot CLI, Grok Build CLI, Kimi Code, OpenCode, Pi, Qwen Code, Hermes Agent, Muse). The single repository contains 10 distinct per-harness adapter directories at its root (`.claude-plugin`, `.antigravity-plugin`, `.codex-plugin`, `.cursor-plugin`, `.devin-plugin`, `.hermes-plugin`, `.kimi-plugin`, `.muse-plugin`, `.opencode`, `.pi`), and the README instructs installing separately per harness even if already installed elsewhere.

**Source:** `README.md` Installation section (Table of Contents list of harnesses); repository root directory listing

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository README and file tree

---

## FINDING-2026-10-29-8

**Topic:** Superpowers Testing/QA

**Observation:** Superpowers maintains an automated testing and evaluation program distinct from ordinary unit tests: a `tests/` directory with per-harness integration suites (`claude-code`, `codex`, `devin`, `hermes`, `kimi`, `opencode`, `pi`, `antigravity`, plus `hooks`, `skill-size`, `shell-lint`, `version-bump`, `writing-skills`, `systematic-debugging`, `diagnosing-superpowers`, `explicit-skill-requests`), and a separate adversarial eval harness (`superpowers-evals`, cloned into `evals/`) that drives real coding-agent CLIs through a "Gauntlet QA agent" and grades them against scenario acceptance criteria. `AGENTS.md` requires before/after eval evidence for any change to skill-behavior content ("Skills are not prose, they are code that shapes agent behavior").

**Source:** `tests/` directory listing; `AGENTS.md` sections "Skill Changes Require Evaluation" and "Eval harness"

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository directory listing and contributor guidelines

---

## FINDING-2026-10-29-9

**Topic:** Superpowers Self-diagnosis

**Observation:** Superpowers includes a dedicated `diagnosing-superpowers` skill that reads a session transcript (current or named by ID) and reports, with line-level evidence, why a skill misfired, stayed silent, or produced unexpected behavior, optionally packaging a scrubbed bundle for a bug report to maintainers.

**Source:** `README.md` "When Something Goes Wrong" section; `skills/diagnosing-superpowers/` directory

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository README and skill directory

---

## FINDING-2026-10-29-10

**Topic:** Superpowers Licensing and Business Model

**Observation:** Superpowers is MIT-licensed (copyright Jesse Vincent), and the README advertises commercial enterprise support/tooling from Prime Radiant. The project also collects optional, anonymous, version-only telemetry through a remotely-loaded logo image in the brainstorming skill's "visual companion" feature, disableable via the `SUPERPOWERS_DISABLE_TELEMETRY` environment variable (also honors Claude Code's own telemetry opt-outs).

**Source:** `LICENSE`; `README.md` "Commercial Services" and "Visual companion telemetry" sections

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository LICENSE and README

---

## FINDING-2026-10-29-11

**Topic:** Pocock Skills Overview

**Observation:** Matt Pocock's Skills repo is described as "agent skills for real engineers... not vibe coding," explicitly positioned in opposition to process-owning frameworks: "Approaches like GSD, BMAD, and Spec-Kit try to help by owning the process. But while doing so, they take away your control and make bugs in the process hard to resolve." Its stated design goal is skills that are "small, easy to adapt, and composable. They work with any model."

**Source:** `README.md` header section (mattpocock/skills)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository README

---

## FINDING-2026-10-29-12

**Topic:** Pocock Skills Mechanism

**Observation:** The Pocock Skills repo has no session-start hook or forced-bootstrap mechanism anywhere in the repository (a recursive search for "hook" across `.claude-plugin/` and `.agents/` returned no matches). Activation relies entirely on each harness's native skill-matching against the SKILL.md frontmatter `description`, plus a one-time, per-repo `/setup-matt-pocock-skills` command that a human must run manually before the engineering skills are configured for that repo.

**Source:** `grep -ril "hook" .claude-plugin .agents` (no results); `skills/engineering/setup-matt-pocock-skills/SKILL.md`

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, absence confirmed by direct repository search, and skill file

---

## FINDING-2026-10-29-13

**Topic:** Pocock Skills Invocation Model

**Observation:** Skills are explicitly split into two invocation classes via frontmatter: "user-invoked" (`disable-model-invocation: true` in Claude Code's frontmatter, `policy.allow_implicit_invocation: false` in Codex's companion `agents/openai.yaml`; reachable only by a human typing its name) and "model-invoked" (reachable by model or user; the default when the flag is omitted). A user-invoked skill may call a model-invoked one via an explicit "Call the Skill tool with X" instruction, but per the documented invariant, it can never call another user-invoked skill.

**Source:** `.agents/invocation.md` (mattpocock/skills); `skills/productivity/grill-me/SKILL.md` frontmatter

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository documentation and skill file

---

## FINDING-2026-10-29-14

**Topic:** Pocock Skills Catalog Structure

**Observation:** Skills are organized into five bucket folders with distinct lifecycle meaning, documented in `CLAUDE.md`: `engineering/` (daily code work, 20 skills), `productivity/` (daily non-code workflow, 7 skills), `misc/` (frozen and unmaintained, 4 skills), `in-progress/` (public beta, feedback wanted, not shipped in the plugin, 7 skills), and `deprecated/` (no longer used). Only `engineering/` and `productivity/` skills are "promoted": they appear in the top-level README, require a human-facing docs page at `docs/<bucket>/<skill-name>.md`, and are the only skills listed in the shipped Claude Code plugin manifest's `skills` array (27 entries total).

**Source:** `CLAUDE.md` root (mattpocock/skills); `find skills -maxdepth 2 -type d` directory listing; `.claude-plugin/plugin.json` `skills` array

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository documentation, directory listing, and plugin manifest

---

## FINDING-2026-10-29-15

**Topic:** Pocock Skills Domain Modeling

**Observation:** A distinctive, heavily emphasized capability is shared-vocabulary/domain-modeling tooling: `GLOSSARY.md` plus architecture decision records (ADRs) are actively built and sharpened by the `domain-modeling` and `grill-with-docs` skills, explicitly to reduce agent verbosity and misalignment by giving the agent and the user the same compressed domain language. The README gives a concrete before/after example: "There's a problem when a lesson inside a section of a course is made 'real'" becomes "There's a problem with the materialization cascade."

**Source:** `README.md` "#2: The Agent Is Way Too Verbose" section; `GLOSSARY.md`

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository README and glossary file

---

## FINDING-2026-10-29-16

**Topic:** Pocock Skills "Grilling" Primitive

**Observation:** Pocock's skills share a single reusable "grilling" interview primitive (`skills/productivity/grilling/SKILL.md`) that relentlessly interviews the user until every branch of a design/plan/decision tree is resolved. The user-invoked skills `grill-me`, `grill-with-docs`, `triage`, `wayfinder`, and `improve-codebase-architecture` each invoke this shared primitive rather than re-implementing interview logic independently.

**Source:** `README.md` "Reference" section, "Model-invoked" list under Productivity; `skills/productivity/grilling/SKILL.md` description field

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository README and skill file

---

## FINDING-2026-10-29-17

**Topic:** Pocock Skills Governance

**Observation:** `SCOPE.md` defines a two-part acceptance bar for any proposed change: (1) it must describe an "observed failure" in a real session, not a hypothetical improvement; and (2) it must "fit the philosophy" (not a config option, harness-specific branch, or personal-workflow tweak, which belong in a user's own fork via `skills.sh`). New-skill proposals are explicitly closed by default if the desired behavior composes from existing skills (`.out-of-scope/new-skills.md`).

**Source:** `SCOPE.md` (mattpocock/skills)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository scope document

---

## FINDING-2026-10-29-18

**Topic:** Pocock Skills Distribution

**Observation:** The Pocock Skills repo distributes as a self-updating native plugin for Claude Code, Codex, and GitHub Copilot (CLI and VS Code) via plugin marketplaces, and for every other listed harness (Gemini CLI, Cursor, OpenCode, Devin, Windsurf, Amp, Pi) via a one-time `npx skills@latest add mattpocock/skills -a <agent>` command that copies editable files directly into the user's project. The README explicitly contrasts the two models: "A plugin updates itself. skills.sh copies editable files into your project, and you update them by hand."

**Source:** `README.md` "Installation (30-second setup)" section

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository README

---

## FINDING-2026-10-29-19

**Topic:** Pocock Skills Testing/QA

**Observation:** Unlike Superpowers, the Pocock Skills repository root contains no `tests/` or `evals/` directory of any kind. Versioning and release discipline are handled via Changesets (`@changesets/cli` in `package.json` `devDependencies`, `npm run version` and `npm run check-plugin-version` scripts) and a `scripts/sync-plugin-version.mjs` consistency check, not automated behavioral testing; quality control for skill content is instead performed socially through the `SCOPE.md` curation bar on issues and PRs.

**Source:** repository root directory listing (no `tests/` or `evals/` present); `package.json` `scripts` and `devDependencies`; `scripts/` directory listing (`link-skills.sh`, `list-skills.sh`, `sync-plugin-version.mjs`)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, confirmed absence via direct repository listing

---

## FINDING-2026-10-29-20

**Topic:** Pocock Skills Licensing and Business Model

**Observation:** Pocock's Skills repo is MIT-licensed (copyright Matt Pocock), and the README's commercial tie-in is his AI Hero newsletter/training business (aihero.dev): it promotes a roughly 60,000-subscriber newsletter as the way to keep up with skill changes. No telemetry collection of any kind is mentioned in the README or any discovered documentation file.

**Source:** `LICENSE`; `README.md` header and "If you want to keep up with changes to these skills" paragraph

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository LICENSE and README

---

## FINDING-2026-10-29-21

**Topic:** Comparison - Convergent Base Unit

**Observation:** Both projects converge on an identical base unit: a single-purpose Markdown file named `SKILL.md` with YAML frontmatter (`name` + `description` at minimum) that a harness loads and matches against conversational context, paired with a "progressive disclosure" structure where the top-level file stays short and longer reference material loads on demand (Superpowers: `skills/test-driven-development/` ships companion reference files on testing anti-patterns; Pocock: `skills/engineering/tdd/tests.md` and `mocking.md` are referenced, not inlined, from `SKILL.md`).

**Source:** Cross-reference of FINDING-2026-10-29-6 and FINDING-2026-10-29-13; `skills/test-driven-development/` (obra/superpowers) and `skills/engineering/tdd/` (mattpocock/skills) directory listings

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — synthesis of two directly-verified findings plus direct directory-listing confirmation

---

## FINDING-2026-10-29-22

**Topic:** Comparison - Convergent Process Content

**Observation:** Both projects independently converge on near-identical "process skills" for the same two core engineering problems: red-green-refactor test-driven development (Superpowers' `test-driven-development` vs Pocock's `tdd`, both built around a strict red-before-green, one-seam-at-a-time loop) and a phase-gated root-cause debugging loop (Superpowers' `systematic-debugging` vs Pocock's `diagnosing-bugs`, both structured as build-feedback-loop → minimize → hypothesize → instrument → fix → regression-test style disciplines).

**Source:** Cross-reference of FINDING-2026-10-29-4; `skills/engineering/tdd/SKILL.md` "Rules of the loop" section; `skills/engineering/diagnosing-bugs/` directory

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — synthesis of directly-verified finding plus direct file content comparison

---

## FINDING-2026-10-29-23

**Topic:** Comparison - Divergent Activation Philosophy

**Observation:** The two projects take opposite positions on activation control. Superpowers forces its bootstrap skill into context at every session start, `/clear`, and post-compaction via code-level hooks so that mandatory-workflow skills "trigger automatically" without any user action (README: "because the skills trigger automatically, you don't need to do anything special. Your coding agent just has Superpowers"). Pocock's Skills repo has zero hook-based forcing of any kind and instead gives the user an explicit user-invoked/model-invoked split in frontmatter, so that interview- or process-heavy skills (`grill-me`, `tdd`, `triage`) are reached only deliberately by a human typing the command, or left entirely to the harness's own native judgment for the model-invoked set.

**Source:** Cross-reference of FINDING-2026-10-29-2, FINDING-2026-10-29-12, FINDING-2026-10-29-13

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — synthesis of three directly-verified findings

---

## FINDING-2026-10-29-24

**Topic:** Comparison - Divergent Scope and Philosophy

**Observation:** Superpowers positions itself as "a complete software development methodology," enforcing one fixed pipeline end-to-end (brainstorm → plan → implement → review → merge) and explicitly refusing domain-specific or project-specific skills in its core (see FINDING-2026-10-29-5). Pocock's Skills repo is explicitly anti-methodology, naming and rejecting process-owning frameworks by name (GSD, BMAD, Spec-Kit) in its own README, and ships a much larger, more heterogeneous, bucketed catalog (27 promoted skills across `engineering/` and `productivity/`, plus unpromoted `in-progress/` and `misc/` buckets) that a user is expected to pick individual skills from via a router skill (`ask-matt`), rather than run as a single mandatory pipeline.

**Source:** Cross-reference of FINDING-2026-10-29-1, FINDING-2026-10-29-5, FINDING-2026-10-29-11, FINDING-2026-10-29-14; `README.md` "ask-matt" reference entry (mattpocock/skills)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — synthesis of four directly-verified findings

---

## FINDING-2026-10-29-25

**Topic:** Comparison - Divergent Quality-Assurance Methodology

**Observation:** Superpowers treats skill text as "code that shapes agent behavior" and backs behavioral changes with a dedicated adversarial eval harness (a "Gauntlet QA agent" grading real coding-agent CLI sessions against scenario acceptance criteria) plus per-harness integration test suites, required before merging any skill-content change. Pocock's Skills repo instead gates changes socially through the `SCOPE.md` "observed failure in a real session" bar and changeset-based release versioning, with no automated behavioral test or eval suite checked into the repository.

**Source:** Cross-reference of FINDING-2026-10-29-8, FINDING-2026-10-29-17, FINDING-2026-10-29-19

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — synthesis of three directly-verified findings

---

## FINDING-2026-10-29-26

**Topic:** Comparison - Divergent Distribution Trust Model

**Observation:** Superpowers ships 10 separate per-harness native plugin/extension adapter directories inside one repository, giving deep per-harness integration (including hook injection) at the cost of larger maintained code surface per harness. Pocock's Skills repo ships only 3 native, self-updating plugin integrations (Claude Code, Codex, GitHub Copilot) and relies on a universal `npx skills@latest add` file-copy installer for every other harness, which deliberately hands the user an editable, locally-owned copy that does not auto-update, trading automatic updates for user-level control and hackability (consistent with the README's "Hack around with them. Make them your own." framing).

**Source:** Cross-reference of FINDING-2026-10-29-7, FINDING-2026-10-29-18

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — synthesis of two directly-verified findings

---

## FINDING-2026-10-29-27

**Topic:** Superpowers Pipeline Detail

**Observation:** Every Superpowers execution path (subagent-driven-development and executing-plans alike) converges on the same three cross-cutting gates invoked throughout, not just at one stage: `test-driven-development`'s Iron Law ("no production code without a failing test first"), `systematic-debugging`'s four-phase root-cause process (triggered whenever a step's actual output does not match the plan's `Expected:` line), and `verification-before-completion`'s Iron Law ("no completion claims without fresh verification evidence"). Both execution skills also share one ledger/workspace directory structure (`.superpowers/sdd/<plan-basename>/progress.md`), so a plan can switch executors mid-flight and resume from the same ledger.

**Source:** `skills/subagent-driven-development/SKILL.md`, `skills/executing-plans/SKILL.md`, `skills/test-driven-development/SKILL.md`, `skills/systematic-debugging/SKILL.md`, `skills/verification-before-completion/SKILL.md` (obra/superpowers)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository skill files

---

## FINDING-2026-10-29-28

**Topic:** Superpowers Pipeline Detail

**Observation:** `writing-plans` produces a plan file with a fixed header (Goal/Architecture/Tech Stack/Spec link/Global Constraints/Review Focus) and per-task blocks (Files/Interfaces/Steps in RED-GREEN-COMMIT order), saved to `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`, then asks the human to choose between two REQUIRED SUB-SKILLs at handoff: `subagent-driven-development` (fresh implementer + fresh task reviewer per task, most thorough, costs a fresh context per task/review) or `executing-plans` (one inline session plus one final whole-branch reviewer, cheapest). `subagent-driven-development` explicitly forbids parallel implementer dispatch ("Never dispatch multiple implementation subagents in parallel (conflicts)").

**Source:** `skills/writing-plans/SKILL.md` lines 52-204; `skills/subagent-driven-development/SKILL.md` line 233 (obra/superpowers)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository skill files

---

## FINDING-2026-10-29-29

**Topic:** Pocock Skills Pipeline Detail

**Observation:** Pocock's Skills repo has no single "brainstorming" equivalent; the stage is split across `grilling` (the reusable interview primitive, model-invoked), `grill-me`/`grill-with-docs` (user-invoked wrappers around it — `grill-with-docs` additionally calls `domain-modeling` to update `GLOSSARY.md`/ADRs live), and `to-spec` (user-invoked, explicitly does NOT interview — "synthesizes what you've already discussed" into a fixed template: Problem Statement, Solution, User Stories, Implementation Decisions, Testing Decisions, Out of Scope, Further Notes — then publishes it to the configured issue tracker with a `ready-for-agent` label).

**Source:** `skills/productivity/grilling/SKILL.md`, `skills/productivity/grill-me/SKILL.md`, `skills/engineering/grill-with-docs/SKILL.md`, `skills/engineering/to-spec/SKILL.md` (mattpocock/skills)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository skill files

---

## FINDING-2026-10-29-30

**Topic:** Pocock Skills Pipeline Detail

**Observation:** `to-tickets` breaks a spec/conversation into "tracer-bullet" vertical-slice tickets with explicit blocking-edge dependencies (a task graph with a "frontier" of unblocked tickets), including a dedicated expand-contract sequencing pattern for wide mechanical refactors that cannot be vertical-sliced. `implement-spec` then runs the whole task graph concurrently: a fresh implementer subagent per ticket, each on its own worktree/branch, each required to call the `tdd` skill, merged into one integration branch by dedicated merger subagents as the frontier advances, with exactly one `code-review` pass on the whole integration branch at the end (not per-ticket). This is explicitly concurrent, in contrast to Superpowers' `subagent-driven-development`, which is sequential by design with a review after every single task.

**Source:** `skills/engineering/to-tickets/SKILL.md`, `skills/engineering/implement-spec/SKILL.md` (mattpocock/skills)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository skill files

---

## FINDING-2026-10-29-31

**Topic:** Pocock Skills Pipeline Detail

**Observation:** Pocock's `code-review` skill runs two parallel sub-agents along two fixed, never-cross-ranked axes — Standards (the repo's documented conventions plus a built-in baseline of 12 named Fowler code smells: Mysterious Name, Duplicated Code, Feature Envy, Data Clumps, Primitive Obsession, Repeated Switches, Shotgun Surgery, Divergent Change, Speculative Generality, Message Chains, Middle Man, Refused Bequest) and Spec (requirement coverage, scope creep, wrong implementation) — aggregated side by side. Superpowers' `requesting-code-review` dispatches a single reviewer per task on one axis with severity tiers (Critical/Important/Minor), and pairs it with a separate `receiving-code-review` skill governing how feedback is evaluated (verify before implementing, no performative agreement, YAGNI-check reviewer suggestions) that Pocock has no equivalent of. Pocock in turn has a dedicated `pr` skill (Summary/Evidence/Merge-Danger-with-door-and-blast-radius template) for PR-body authorship that Superpowers' `finishing-a-development-branch` has no equivalent of.

**Source:** `skills/engineering/code-review/SKILL.md`, `skills/engineering/pr/SKILL.md` (mattpocock/skills); `skills/requesting-code-review/SKILL.md`, `skills/receiving-code-review/SKILL.md`, `skills/finishing-a-development-branch/SKILL.md` (obra/superpowers)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository skill files

---

## FINDING-2026-10-29-32

**Topic:** Comparison - Pocock Additive Capabilities

**Observation:** Several Pocock skills have no Superpowers pipeline-stage equivalent at all: `wayfinder` (resolving oversized, multi-session-scale ambiguity via decision tickets before a spec exists), `domain-modeling`/`GLOSSARY.md`/ADRs (a persistent, projectwide shared-vocabulary artifact consumed by `to-spec`, `to-tickets`, `code-review`, `pr`, and `wait-what`), `triage` (a state machine for incoming issues/external PRs), `improve-codebase-architecture` (proactive codebase-health scanning independent of any active feature request), `codebase-design` (a standing deep-module/seam vocabulary other skills borrow terms from), `prototype` (bounded throwaway spikes to answer a design question), `wizard` (generated interactive bash wizards for human-only manual procedures), `research` (a background-agent primary-source research skill), and the entire `productivity/` bucket (`handoff`, `teach`, `to-questionnaire`, `wait-what`), which is non-code workflow tooling outside Superpowers' deliberately engineering-only scope.

**Source:** `skills/engineering/wayfinder/SKILL.md`, `skills/engineering/domain-modeling/SKILL.md`, `skills/engineering/triage/SKILL.md`, `skills/engineering/improve-codebase-architecture/SKILL.md`, `skills/engineering/codebase-design/SKILL.md`, `skills/engineering/prototype/SKILL.md`, `skills/engineering/wizard/SKILL.md`, `skills/engineering/research/SKILL.md`, `skills/productivity/` directory listing (mattpocock/skills)

**Date captured:** 2026-10-29

**Verified:** VERIFIED 2026-10-29 — direct primary-source citation, official repository skill files

---
