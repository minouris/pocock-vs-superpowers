# Dual-session experiment plan: Superpowers vs Matt Pocock's Skills on OpenCode

## Status

Planning document only. No trial session has been run. Every command below has
been verified against the OpenCode v2.0.18 binary installed in this sandbox
(`--help` output, `opencode debug paths`, `opencode debug config`,
`opencode session export`, and the bundled `@opencode-ai/sdk` TypeScript type
declarations) and against the primary-source files of both skill repositories
(`.tmp/superpowers`, `.tmp/skills`). Nothing in this document was executed
against a live trial; the verification commands used to confirm CLI behaviour
were run read-only, against this environment's own pre-existing session or
against disposable scratch directories under `/tmp/opencode`, never against a
newly spun-up trial session.

A note on sourcing: `opencode.ai/v2/docs/*` returned HTTP 403 to `webfetch` in
this sandbox for every page tried (`/docs/cli`, `/docs/config`,
`/v2/llms.txt`), and no desktop browser was connected to fetch them another
way. Official documentation for OpenCode's CLI and config reference could
therefore not be consulted directly for this plan. In its place, this plan
relies on the installed OpenCode v2.0.18 binary's own `--help` text, its
`debug` introspection subcommands, and the TypeScript type declarations
shipped in `@opencode-ai/sdk`, all of which are first-party, versioned
artefacts of the exact binary a human would run this experiment against, and
are cited by command/file throughout. Where a claim could not be verified this
way, it is marked **ASSUMPTION** or **NEEDS VERIFICATION** rather than stated
as settled fact.

## Scope and non-goals

This document designs a reproducible experiment. It does not run one. It does
not install Superpowers or Pocock's Skills anywhere, real or scratch. It does
not create the sample project. Every section ends where execution would
begin; the final section is a runbook for a human (or an agent explicitly
instructed to execute it) to follow later, after reviewing this plan.

## Assumptions log (collect-and-flag)

These are the places a human partner's input was required but unavailable in
this subagent context. Each is restated inline where it arises; they are
collected here for quick review:

1. **Sample project brief content and fixed tech stack** (Python + FastAPI +
   SQLite + pytest) — a product decision normally made during brainstorming
   itself. See "Sample project brief."
2. **Hybrid fairness mechanism** (detailed upfront brief + virtual-stakeholder
   fallback script, rather than pure interview-replay or pure brief-only) —
   a tradeoff call between realism and determinism. See "Fairness mechanism."
3. **Exact model/provider pinning** — this sandbox's configured model
   (`anthropic-azure-msd/mnorr001-claude-sonnet-5`) is an internally routed
   alias, not something a real comparison should use unexamined. See "Risks."
4. **How skills.sh wires Pocock's user-invoked skills for the OpenCode
   target** (native `command` vs `skill` registration) — inferred from
   OpenCode's API surface, not empirically confirmed, because running the
   installer was out of scope for a plan-only task. See "Installation
   procedure" and "Non-interactive driving mechanism."
5. **Budget and hard-cap numbers** (turns, wall-clock, spend) are rough,
   first-pass estimates, explicitly not verified against real provider
   pricing for the pinned model. See "Risks, costs, and guardrails."
6. **Cost field semantics** — in this sandbox, `AssistantMessage.cost` reads
   `0` for every message (an internally routed, no-metered-cost provider), so
   this plan's cost-capture mechanism has been verified structurally (the
   field exists, aggregates correctly) but not verified to produce a
   non-zero, meaningful dollar figure against a real metered provider. See
   "Instrumentation plan."

---

## 1. Sample project brief

**ASSUMPTION, needs human sign-off before use.** Brainstorming/grilling would
normally negotiate this with a human; here it is pre-written so both trial
sessions receive an identical, fixed starting point.

### Brief text (verbatim, to be pasted into both sessions)

> Build a **Bookmark Manager API**: a small HTTP service for saving and
> organising web bookmarks, for personal/local use (single user, no auth, no
> deployment target — it only needs to run on localhost for development and
> testing).
>
> **Tech stack (fixed, not open for discussion):** Python 3.11+, FastAPI,
> SQLite (via the standard library `sqlite3` module or `sqlalchemy`, your
> choice), `pytest` for tests. Do not propose an alternative stack; this
> choice has already been made.
>
> **Required endpoints:**
> - `POST /bookmarks` — create a bookmark. Body: `url` (required, must be a
>   valid absolute HTTP/HTTPS URL), `title` (required, non-empty string),
>   `tags` (optional list of strings). Returns the created bookmark with a
>   generated `id` and `created_at`/`updated_at` timestamps.
> - `GET /bookmarks` — list bookmarks, newest first. Supports an optional
>   `tag` query parameter (filter to bookmarks carrying that tag) and an
>   optional `q` query parameter (case-insensitive substring match against
>   `title` or `url`). `tag` and `q` may be combined.
> - `GET /bookmarks/{id}` — fetch one bookmark by id, 404 if absent.
> - `PATCH /bookmarks/{id}` — partially update `url`, `title`, and/or `tags`.
> - `DELETE /bookmarks/{id}` — delete a bookmark, 404 if absent.
> - `GET /tags` — list every distinct tag currently in use, each with a count
>   of how many bookmarks carry it, sorted by count descending.
>
> **Data rules:**
> - Tags are case-insensitive and stored normalised (lower-cased, trimmed).
>   A bookmark cannot carry the same tag twice.
> - Persistence is a SQLite file on disk (not in-memory), so data survives a
>   process restart.
>
> **Two decisions are deliberately left to you to resolve** (ask if you need
> to, or state your assumption plainly if you proceed without asking):
> 1. What should happen if `POST /bookmarks` is called with a `url` that
>    already exists in the database?
> 2. Should `GET /bookmarks` be paginated, and if so, how?
>
> **Definition of done:** the service starts with a single documented command,
> every endpoint above behaves as specified, there is an automated test suite
> that exercises the HTTP layer (not just internal functions) and passes, and
> the work is committed to git with a history that shows how it was built.
> Out of scope: authentication, multi-user support, a UI, deployment
> configuration, rate limiting. If you find yourself about to build any of
> those, stop and flag it instead.

### Why this brief

- Fixing the tech stack removes a real but irrelevant degree of freedom: we
  want to compare *process*, not compare whether one session happened to pick
  a stack the model is more fluent in. Both frameworks' interview skills
  (`brainstorming`'s "how it gets made" question; Pocock's `grilling`) would
  normally negotiate stack choice with a human — pre-deciding it is a
  deliberate, flagged tradeoff against realism.
- The two deliberately-ambiguous decisions (duplicate URL handling,
  pagination) exist on purpose: they are exactly the kind of thing a
  real human stakeholder would be asked about, and they let the rubric
  measure "how much did it ask vs assume" (section 5) on identical ground
  across both sessions, using the fairness mechanism in section 3.
- Scope is bounded to fit a single bounded trial (one resource, six
  endpoints, no auth) while still exercising every lifecycle stage both
  frameworks care about: an interview/spec stage, a planning stage, a
  TDD implementation stage, and a review/completion stage.

---

## 2. Isolation mechanism

### 2.1 What was verified about OpenCode's config resolution

Four environment variables fully redirect OpenCode's per-user state, verified
empirically against the installed v2.0.18 binary:

```
XDG_CONFIG_HOME   -> <root>/config   (global opencode.jsonc, discovered plugins)
XDG_DATA_HOME     -> <root>/data     (session database, logs, cloned plugin repos)
XDG_CACHE_HOME    -> <root>/cache    (downloaded/built plugin packages, binaries)
XDG_STATE_HOME    -> <root>/state    (misc runtime state)
```

Verified with `opencode debug paths` run under two different sets of these
four variables pointed at disposable scratch directories (`/tmp/opencode/...`):
each run reported a fully distinct `data`, `cache`, `config`, `state`, `log`,
`repos`, and `db` path, all rooted under the scratch directory supplied, with
no path falling back to the real `$HOME`. The `home` row itself only follows
`OPENCODE_TEST_HOME`, a separate variable that does **not** redirect
config/data/cache/state (verified: setting only `OPENCODE_TEST_HOME` left
`data`/`cache`/`config`/`state` pointed at the real user home) — the four XDG
variables are the isolation primitive to use, not `OPENCODE_TEST_HOME`.

`opencode debug config` lists every configuration document OpenCode actually
merged for the current invocation, in resolution order. Run from this
project's directory with the real environment, it returned exactly two
documents: `~/.config/opencode/opencode.jsonc` (global) and
`/workspaces/pocock-vs-superpowers/opencode.json` (this repo's own
project-level config — which, notably, loads a third-party
`opencode-rules@next` plugin that is the actual source of every "injected
rule" system-message seen in this very task). This is the concrete,
demonstrated mechanism by which an ancestor directory's config can silently
attach to a session: OpenCode walks up from the launch directory looking for
`opencode.json(c)`/`.opencode/opencode.json(c)` at every level.

**Consequence for isolation:** the two trial project directories must not
share any filesystem ancestor that itself carries an `opencode.json(c)` or
`.opencode/opencode.json(c)` — including, as this task's own session just
demonstrated, this very repository. They must live outside
`/workspaces/pocock-vs-superpowers` entirely.

Several other `OPENCODE_*` environment variables exist in the installed
binary (`OPENCODE_CONFIG_DIR`, `OPENCODE_CONFIG`, `OPENCODE_CONFIG_CONTENT`,
`OPENCODE_CONFIG_PROJECT_DISABLE`, `OPENCODE_DISABLE_PROJECT_CONFIG`,
`OPENCODE_DB`, `OPENCODE_DISABLE_AUTOUPDATE`, among others) found via
`strings` on the compiled binary. Their exact semantics were not independently
confirmed (no working path to the official docs in this sandbox, and testing
every one was out of scope for a plan-only pass). **NEEDS VERIFICATION**
before the real run: re-test `opencode debug config`/`opencode debug paths`
with each candidate variable set, to decide whether any of them should be
used as an additional belt-and-braces guard (e.g. a project-config-disable
variable as a second line of defence against ancestor-directory bleed, on
top of the directory-placement rule above).

### 2.2 Directory layout

All of the below lives outside this repository, under the pre-approved
scratch area:

```
/tmp/opencode/experiment-<date>/
├── shared/
│   ├── brief.md                      (section 1, identical for both trials)
│   └── virtual-stakeholder-qa.md     (section 3, identical for both trials)
├── trial-a-superpowers/
│   ├── home/
│   │   ├── config/        <- XDG_CONFIG_HOME
│   │   ├── data/          <- XDG_DATA_HOME
│   │   ├── cache/         <- XDG_CACHE_HOME
│   │   └── state/         <- XDG_STATE_HOME
│   └── project/           <- the sample project, fresh `git init`
└── trial-b-pocock/
    ├── home/
    │   ├── config/
    │   ├── data/
    │   ├── cache/
    │   └── state/
    └── project/
```

Before anything else, verify `/tmp` and `/tmp/opencode` themselves carry no
`opencode.json(c)` (`find /tmp -maxdepth 2 -iname 'opencode.json*'` should
return nothing). This is the directory-ancestor guard described above,
checked mechanically rather than assumed.

Each trial is driven with all four XDG variables exported to that trial's
`home/` subtree and the working directory set to that trial's `project/`
subtree, for every single `opencode` invocation across the trial's entire
lifetime (setup, every prompt, every export). Nothing is ever invoked with
the ambient/default environment once a trial starts.

### 2.3 Verification step (run once per trial, before any real prompt)

```bash
cd /tmp/opencode/experiment-<date>/trial-a-superpowers/project
XDG_CONFIG_HOME=../home/config XDG_DATA_HOME=../home/data \
XDG_CACHE_HOME=../home/cache XDG_STATE_HOME=../home/state \
opencode debug config
```

Confirm the returned document list contains **exactly** two entries: the
trial's own isolated global config and the trial's own project-level config
(absent, until step 4 creates one), and nothing pointing outside the trial's
`home/`/`project/` subtree. Repeat for trial B. Confirm the two trials' `db`
paths (`opencode debug paths`) differ. This is the acceptance test for
isolation — do not proceed past it until it passes for both trials.

---

## 3. Installation procedure

Both installs happen once per trial, inside that trial's isolated
environment, before any sample-project prompt is sent.

### 3.1 Trial A: Superpowers

Per `.tmp/superpowers/.opencode/INSTALL.md`, OpenCode V2 (2.0.4+, this sandbox
runs 2.0.18) uses a `plugins` array entry with a git-backed package spec. The
installed binary's own `opencode plugin add` subcommand ("Install a plugin
and add it to the global configuration") is the documented way to do this
without hand-editing JSON:

```bash
cd /tmp/opencode/experiment-<date>/trial-a-superpowers/project
export XDG_CONFIG_HOME=../home/config XDG_DATA_HOME=../home/data
export XDG_CACHE_HOME=../home/cache XDG_STATE_HOME=../home/state
opencode plugin add "superpowers@git+https://github.com/obra/superpowers.git#<pinned-tag>"
```

**Pin an exact tag or commit, not a floating branch.** INSTALL.md's own
"Updating" section warns that OpenCode/Bun can pin a resolved git dependency
in a lockfile/cache and not always pick up new commits on a re-run — for a
reproducible comparison, a moving `main`/`dev` reference is a non-determinism
source in its own right (see section 6). Pick the latest tagged release at
plan-approval time and record the exact tag/commit in the results.

**NEEDS VERIFICATION before running for real:** this sandbox's own
`opencode.json` uses the singular keys `"plugin"` / `"agent"` (confirmed by
reading the file directly), while `.tmp/superpowers/.opencode/INSTALL.md`
documents the plural `"plugins"` key for "OpenCode V2 (2.0.4 or later)" —
the exact version installed here. This is a real, observed discrepancy
between what the Superpowers project's own install doc says V2 uses and what
this installed v2.0.18 binary's own live config actually contains. Do not
hand-edit the config key name; use `opencode plugin add`, which writes
whichever key this specific binary build expects, and confirm with
`opencode debug config` that the entry landed and was recognised (plugin
shows up in `opencode plugin list`).

After install, confirm activation per the INSTALL.md's own documented
acceptance test: ask "Tell me about your superpowers" in a scratch prompt and
confirm the bootstrap fired, and separately confirm `opencode plugin list`
shows `superpowers` loaded with no errors in
`<XDG_DATA_HOME>/log/opencode.log` (filtering `role=server`, per
INSTALL.md's troubleshooting section).

### 3.2 Trial B: Pocock's Skills

Per the repo's own `README.md` ("Any other agent, or editable files"
section), the universal installer is:

```bash
cd /tmp/opencode/experiment-<date>/trial-b-pocock/project
export XDG_CONFIG_HOME=../home/config XDG_DATA_HOME=../home/data
export XDG_CACHE_HOME=../home/cache XDG_STATE_HOME=../home/state
npx skills@latest add mattpocock/skills -a opencode -y --copy \
  -s "setup-matt-pocock-skills,ask-matt,grill-with-docs,grill-me,grilling,to-spec,to-tickets,implement,implement-spec,tdd,code-review,pr,diagnosing-bugs,domain-modeling,research,codebase-design"
```

Flags confirmed against the installed `skills` CLI's own `--help` output
(run read-only during planning, no skills actually added): `-a opencode`
targets OpenCode specifically; `-y` skips interactive confirmation (required
for a non-interactive trial); `--copy` copies files directly into the
project's agent directory rather than symlinking into `node_modules`,
producing a self-contained, inspectable trial directory; `-s` restricts the
install to the promoted skills relevant to an engineering lifecycle (omitting
the `productivity/` bucket members not already pulled in as dependencies,
and the `misc/`/`in-progress/` buckets entirely, consistent with
`CLAUDE.md`'s own description of what is "promoted"). `README.md` explicitly
says to include `setup-matt-pocock-skills` in the installed set, which the
list above does. Confirm success with `npx skills@latest list` inside the
trial's isolated environment.

**ASSUMPTION / NEEDS VERIFICATION, flagged because the installer was not run
for real during planning:** `CLAUDE.md` (`.agents/invocation.md`) documents
that user-invoked skills are gated per-harness (Claude Code:
`disable-model-invocation: true` frontmatter; Codex: a companion
`agents/openai.yaml` policy file). Neither source states explicitly what the
skills.sh `-a opencode` target writes for a user-invoked skill. OpenCode's
own API exposes two structurally different invocation surfaces — a `skill`
tool (model-invoked, autonomous) and a `POST /session/{id}/command` endpoint
(explicit, slash-command-style invocation, confirmed in
`@opencode-ai/sdk`'s `SessionCommandData` type) — and OpenCode's documented
config schema includes a `commands` field. The most likely mapping is that
skills.sh emits an OpenCode custom command (e.g.
`.opencode/command/to-spec.md`) for each user-invoked skill, invokable via
the `command` API, and a plain skill registration for model-invoked ones.
**Before running Trial B for real**, install once in a disposable sandbox,
inspect exactly what files land under the trial project's `.opencode/`
directory, and confirm which of the two invocation surfaces each relevant
skill (`to-spec`, `to-tickets`, `implement-spec`, `grill-with-docs`,
`setup-matt-pocock-skills`) actually uses, then adjust the orchestrator
script in section 4 accordingly. This plan's orchestrator design in section
4.3 assumes the command-API mapping and flags every place that assumption
bites.

---

## 4. Non-interactive driving mechanism

### 4.1 Options considered

| Option | Mechanism | Verdict |
|---|---|---|
| `opencode run <message> --standalone` per prompt | Each call starts a fresh private server, processes one message, exits | Simplest to script; pays a cold-start cost per prompt; fine for a handful of prompts, wasteful for a long multi-turn pipeline |
| `opencode serve` long-lived + SDK/API | One private server per trial, driven by repeated `POST /session/{id}/message` or `/command` calls | Keeps the agent/session warm across the whole pipeline; matches how the SDK and the `@opencode-ai/sdk` client guide expect automation to work; recommended |
| `opencode mini` | Minimal *interactive* TUI, not scriptable for unattended input | Rejected: still interactive |
| `opencode acp` | Agent Client Protocol server (for editor integrations, e.g. Zed) | Rejected: wrong protocol for a scripted driver, no advantage over the plain HTTP API here |

**Recommendation: `opencode serve` + scripted HTTP/SDK driving**, confirmed
against the installed binary:

- `opencode serve --hostname 127.0.0.1 --port <unique-port>` starts the v2
  API/web server (confirmed via `opencode serve --help`); run once per trial,
  in the background, with that trial's XDG variables exported and its
  working directory set to that trial's `project/`.
- `opencode session create` is not a separate CLI subcommand; it is reached
  via the generic passthrough: `opencode api post /session --data
  '{"title":"trial-a-run-1"}' --server http://127.0.0.1:<port>` (confirmed:
  `SessionCreateData`'s body takes `{parentID?, title?}` and a `directory`
  query parameter; `opencode api` passthrough syntax is documented in the
  `opencode` skill and matches `--help` for other subcommands).
- Each scripted prompt is `opencode api post /session/{id}/message --data
  '{"parts":[{"type":"text","text":"<prompt text>"}]}' --server
  http://127.0.0.1:<port>` (confirmed against `SessionPromptData`'s body
  shape: `{model?, agent?, system?, tools?, parts: [...]}`).
- Each scripted **user-invoked Pocock skill** ("virtual slash command") is
  `opencode api post /session/{id}/command --data
  '{"command":"to-spec","arguments":"..."}'` (confirmed against
  `SessionCommandData`'s body shape: `{command, arguments, agent?, model?,
  messageID?}`) — see the flagged assumption in section 3.2 about whether
  this is in fact how skills.sh wires user-invoked skills for OpenCode.
- `--auto` (confirmed flag on `run`/`mini`/top-level `opencode`) auto-approves
  permissions not explicitly denied. There is no equivalent flag documented
  for `serve`; **NEEDS VERIFICATION** whether `serve`-driven sessions need an
  agent/permission config set to auto-approve instead (likely via the
  `permission` config block, same mechanism used by this very environment's
  `opencode.jsonc`, which already sets explicit `allow`/`ask`/`deny` rules per
  action). Before the real run, configure each trial's isolated global config
  with permissive-but-scoped permissions (shell/edit allowed inside the
  trial's own project directory, `external_directory` and any destructive
  action denied) so the pipeline does not stall on an interactive approval
  prompt that nothing is present to answer.

### 4.2 Why not `opencode run --standalone` per turn

It was considered as the simpler fallback and is documented here for
completeness, since the task calls for both options. A fully scripted
sequence of `opencode run --standalone --session <id> --format json
"<prompt>"` calls (confirmed flags: `--session`/`-s` to continue a session,
`--format json` for machine-parseable output, `--standalone` for a private
server) achieves the same effect with less infrastructure (no background
`serve` process to manage), at the cost of a fresh server cold-start on every
single prompt — for a pipeline with dozens of scripted turns (brainstorming
Q&A, plan review, per-task implementation checkpoints, review, finish), this
adds material, repeated overhead. **Use this fallback only if `opencode
serve` proves awkward to manage in the execution environment** (e.g. no
reliable way to background a long-lived process and query its port).

### 4.3 Orchestrator design

A single orchestrating script (not part of this plan to write in full code,
per the no-implementation-code constraint on this planning pass, but
specified precisely enough to write later) performs, per trial:

1. Export XDG variables, start `opencode serve` in the background, poll
   `opencode api get /api/info --server http://127.0.0.1:<port>` until it
   responds.
2. Create a session scoped to the trial's `project/` directory.
3. Send the brief (section 1) as the first message.
4. Loop: read the assistant's latest message/parts (via
   `GET /session/{id}/message` or by polling `session export`); if a
   `TextPart` contains a question addressed to the user, look it up against
   the virtual-stakeholder script (section 5) and send the matched answer as
   the next message; if the session appears to have produced a design
   doc/plan/ticket and is waiting for approval, send a fixed, identical
   "Approved, please proceed" turn (logged as an orchestrator decision, not
   invented per-trial); if a scripted user-invoked command applies at this
   point in Pocock's pipeline (e.g. after enough has been discussed, issue
   the `to-spec` command), issue it via the `/command` endpoint.
5. Continue until the session reaches a terminal state (the brief's
   "definition of done" is met, or a hard cap from section 6 is hit).
6. Export the session (`opencode session export <id>` — unsanitised, kept
   only in the results directory; a `--sanitize` copy additionally produced
   for anything shared outside the trusted analysis environment), capture
   `opencode stats --project . --cost --json --full`, stop `serve`.

This loop is necessarily semi-automatic in one respect: step 4's
question-detection and brief-matching is the one place genuine judgement is
required to keep both trials' virtual-stakeholder answers identical and
correctly matched. Section 5 specifies exactly how that judgement is kept
deterministic.

---

## 5. Fairness mechanism for the interactive stages

### The problem

Both frameworks open with (or strongly tend toward) an interview stage with
a human in the loop: Superpowers' `brainstorming` is a **hard gate** — per
`skills/brainstorming/SKILL.md`, nothing gets built until a human approves a
design doc. Pocock's equivalent (`grill-with-docs`/`grilling`) is not a hard
gate in the same way (it is reachable by model judgement, and the brief could
also legitimately route straight to the no-interview `to-spec` skill, whose
own frontmatter says "no interview, just synthesis of what you've already
discussed") — this asymmetry is real and is itself part of what the
comparison should surface, not something to paper over.

### Chosen approach: detailed brief as primary mechanism, scripted fallback for anything it misses

**ASSUMPTION, flagged for human review — this is a real tradeoff, not a
clearly correct call:**

1. **Primary:** the brief in section 1 is written to be exhaustive enough
   that most clarifying questions converge to an answer already stated in
   it. This maximises determinism (nothing to look up or match at runtime)
   at a real cost to ecological validity: an exhaustive upfront brief is not
   how a real human actually behaves, and it narrows the gap between
   frameworks' interview quality precisely where the rubric wants to measure
   it.
2. **Fallback, for anything the brief does not cover:** a fixed table of
   question-patterns to answers, below, used identically by the
   orchestrator for both trials. Any question not matched by a pattern in
   the table gets the same fixed fallback line for both trials: *"Use your
   own best judgement, proceed, and note the assumption explicitly in
   whatever spec or plan document you produce."*

| If the agent asks about (pattern) | Give this answer |
|---|---|
| duplicate URL on create | "Reject it: return 409 Conflict with a message naming the existing bookmark's id." |
| pagination for `GET /bookmarks` | "Yes, paginate: `limit` (default 20, max 100) and `offset` query params, with a response that also reports the total count." |
| authentication / multi-user | "No auth, single user, out of scope — restated from the brief." |
| deployment target / hosting | "Local development only, no deployment config needed — restated from the brief." |
| which test framework | "pytest, as stated in the brief." |
| ORM vs raw SQL | "Your choice; the brief already says sqlite3 or sqlalchemy, either is fine." |
| what counts as done / acceptance criteria | Restate the brief's "Definition of done" paragraph verbatim. |
| whether to use git branches/worktrees | "Use whatever your own methodology normally does here; no additional constraint from me." |
| anything about committing, PR process, review process | "Follow your own normal process; I have no opinion beyond the definition of done." |
| anything not matched above | "Use your own best judgement, proceed, and note the assumption explicitly in whatever spec or plan document you produce." |

The last two rows are deliberately hands-off: the experiment wants to observe
each framework's *own* process defaults (worktree use, review process,
commit granularity), not have the orchestrator accidentally steer one
session's methodology more than the other's by answering process questions
differently. The rubric (section 7) explicitly scores "how much the agent
asked vs assumed," so under-specifying on purpose, for anything that is
process rather than product, is intentional.

### Tradeoffs, stated plainly

- **Realism vs determinism:** an exhaustive brief plus a scripted fallback
  table is maximally reproducible (any third party re-running this plan gets
  the same answers) but is not how a real stakeholder behaves, and may
  suppress genuine interview-quality differences between the two frameworks.
- **Matching risk:** the orchestrator (a human or an LLM judge following the
  table mechanically) must map a free-form question to the nearest pattern
  row. If matched inconsistently between trials, that itself becomes a
  hidden source of unfairness. **Mitigation:** use the identical
  matching process (ideally the identical judge/script, run once per
  question, logged) for both trials, and log every match decision in the
  results so a third party can audit it.
- **An alternative considered and rejected for this plan:** a full
  "virtual stakeholder" persona playing the role interactively and
  improvising answers. Rejected because improvisation cannot be guaranteed
  identical across two separate trial sessions asking questions in different
  order or phrasing — it trades determinism for realism in a way this
  specific comparison (process and methodology, not interview-quality
  benchmarking) does not need.
- **Recommended before committing to a full run:** a cheap pilot — run just
  the interview/spec stage of each framework once, read the transcript, and
  confirm the brief is not so exhaustive that neither framework asks
  anything at all. If both sessions sail straight through with zero
  questions, that is itself useful evidence but means the "how much did it
  ask vs assume" axis of the rubric is uninformative for this run, and the
  brief should be deliberately under-specified further before the real
  trials.

---

## 6. Instrumentation plan

Every mechanism below was confirmed against this sandbox's own, real,
pre-existing session (read-only operations: `opencode session list`,
`opencode session export`), not against a newly created trial session.

### 6.1 Full transcript

`opencode session export <id>` (confirmed CLI subcommand, `--sanitize` flag
available for redacted copies). Verified JSON shape:

```jsonc
{
  "info": {
    "id": "...", "projectID": "...", "agent": "build",
    "model": { "id": "...", "providerID": "...", "variant": "default" },
    "cost": 0,                         // session-level rollup, confirmed present
    "tokens": {                        // session-level rollup, confirmed present
      "input": 0, "output": 0, "reasoning": 0,
      "cache": { "read": 0, "write": 0 }
    },
    "outcome": "succeeded",
    "time": { "created": 0, "updated": 0, "idle": 0, "viewed": 0 },
    "title": "...", "location": { "directory": "..." }
  },
  "messages": [
    {
      "id": "...", "type": "user" | "assistant" | "synthetic",
      "time": { "created": 0, "streamed": 0, "completed": 0 },
      "content": [
        { "type": "text", "text": "..." },
        { "type": "reasoning", "text": "...", "time": {...} },
        {
          "type": "tool", "id": "...", "name": "skill",
          "state": { "status": "completed", "input": { "id": "brainstorming" }, "content": [...] }
        }
      ]
    }
  ]
}
```

Confirmed concretely by exporting this very session: a `tool` part with
`name == "skill"` carries the invoked skill's identifier at
`state.input.id` (verified values `"analysis"` and
`"dispatching-parallel-agents"` were correctly extracted from this session's
own real transcript this way). This is the mechanical basis for the
lifecycle checklist in section 6.4 — skill invocations are detected by
scanning for `{"type":"tool","name":"skill"}` parts and reading
`state.input.id`, not by reading prose and guessing.

Keep the unsanitised export as the primary analysis artefact (full fidelity:
file contents, exact prompts); keep a `--sanitize`d copy for anything shared
outside a trusted analysis environment, per OpenCode's own stated purpose for
that flag ("Redact sensitive transcript and file data").

### 6.2 Token usage and cost

- **Session-level rollup:** `info.cost` and `info.tokens` in the export, as
  above — confirmed present and structurally correct, but in this sandbox
  `cost` reads `0` throughout (an internally routed, non-metered provider).
  **Before the real run, confirm the pinned provider/model (section 7)
  actually reports a non-zero `cost` figure** — if it does not, fall back to
  computing cost from `tokens` x the provider's published per-token rate,
  computed by the orchestrator rather than read from the field.
- **Per-message granularity:** every `AssistantMessage` in `messages[]`
  independently carries its own `cost`/`tokens` (confirmed in
  `@opencode-ai/sdk`'s `types.gen.d.ts`, `AssistantMessage` type,
  `tokens: {input, output, reasoning, cache: {read, write}}`), and every
  `StepFinishPart` inside a message carries the same shape at step
  granularity. This is the basis for a per-lifecycle-stage cost breakdown:
  sum the tokens/cost of every message between one skill-invocation tool
  part and the next, attributing that span to the stage the earlier skill
  invocation opened.
- **Cross-check:** `opencode stats --project . --cost --json --full`
  (confirmed subcommand/flags) aggregates cost/tokens/model/tool-reliability
  for a project independently of a single session's export; run it per trial
  (filtering by that trial's isolated `--project .`, since each trial's
  isolated `XDG_DATA_HOME` means its database only ever contains that
  trial's own sessions) as a second, independently-computed source that the
  manual per-message sum should agree with.

### 6.3 Wall-clock time and turn count

- **Wall-clock:** the orchestrator script timestamps immediately before and
  after each `/session/{id}/message` or `/command` call it issues, plus an
  overall start/stop around the whole trial. Cross-check against
  `info.time.created`/`updated` in the export and each message's own
  `time.created`/`streamed`/`completed`.
- **Turn count, two senses, both recorded:** (a) orchestrator-issued turns —
  the number of scripted prompts/commands actually sent (this is the
  controlled, comparable number); (b) internal agent steps — the number of
  `StepStartPart`/`StepFinishPart` pairs and `subagent`/`task` tool
  dispatches found in the export (this is "how much work did the framework's
  own pipeline fan out into," a direct proxy for Superpowers'
  subagent-driven-development fan-out vs Pocock's narrower code-review-only
  fan-out, called out as a likely real difference in section 8).

### 6.4 Lifecycle checklist

Scored mechanically by scanning the export for `{"type":"tool","name":"skill"}`
parts (model-invoked) and, pending the section 3.2 verification, equivalent
`{"type":"tool","name":"command"? }`-shaped or command-response entries for
user-invoked Pocock skills, plus corroborating shell/file-write evidence
(e.g. a `git worktree add` shell call, a file write under
`docs/superpowers/plans/`).

**Superpowers checklist** (derived from `README.md`'s "Basic Workflow" and
the skill catalogue, section-cited in the fact file,
`FINDING-2026-10-29-3`/`-4`):

| # | Stage | Detected by | Reached? |
|---|---|---|---|
| 1 | Bootstrap injected | first user message contains the `EXTREMELY_IMPORTANT`/`using-superpowers` marker text (structural, not a skill-tool call — injected by the plugin itself per `.opencode/plugins/superpowers.js`) | |
| 2 | `brainstorming` invoked | skill tool, `input.id == "brainstorming"` | |
| 3 | Design doc written and approved | file write under `docs/superpowers/specs/*.md` + a subsequent user-turn approving it | |
| 4 | `using-git-worktrees` invoked | skill tool, `input.id == "using-git-worktrees"` + a `git worktree add` shell call | |
| 5 | `writing-plans` invoked | skill tool, `input.id == "writing-plans"` + file write under `docs/superpowers/plans/*.md` | |
| 6 | Execution method chosen | skill tool, `input.id` is `subagent-driven-development` or `executing-plans` | |
| 7 | `test-driven-development` followed | skill tool, `input.id == "test-driven-development"` + evidence of a failing-then-passing test run (two `shell`/test-runner tool calls for the same test, first non-zero exit then zero) | |
| 8 | `requesting-code-review` invoked | skill tool, `input.id == "requesting-code-review"` | |
| 9 | `verification-before-completion` invoked | skill tool, `input.id == "verification-before-completion"` | |
| 10 | `finishing-a-development-branch` invoked | skill tool, `input.id == "finishing-a-development-branch"` + final `git log`/merge evidence | |

**Pocock's Skills checklist** (derived from `README.md`'s "Why These Skills
Exist" walkthrough and each cited `SKILL.md`'s own frontmatter):

| # | Stage | Detected by | Reached? |
|---|---|---|---|
| 1 | `setup-matt-pocock-skills` run | command invocation, `command == "setup-matt-pocock-skills"` (one-time, expected during environment setup rather than inside the timed trial) | |
| 2 | Interview/spec-sharpening stage | command invocation `grill-with-docs` (or skill `grilling`/`grill-me`), or model-invoked judgement skips straight to `to-spec` per that skill's own "no interview, just synthesis" frontmatter — record which path was taken, it is itself a finding | |
| 3 | `to-spec` invoked | command invocation, `command == "to-spec"` + a written spec artefact | |
| 4 | `to-tickets` invoked | command invocation, `command == "to-tickets"` + ticket files/entries with declared blocking edges | |
| 5 | Implementation stage | command invocation `implement-spec` or `implement`, with a nested skill tool `input.id == "tdd"` + failing-then-passing test-run evidence | |
| 6 | `code-review` invoked | skill tool, `input.id == "code-review"` + two parallel `subagent`/`task` dispatches (Standards axis, Spec axis — this skill's own description states it runs both in parallel sub-agents) | |
| 7 | `pr` invoked | skill tool, `input.id == "pr"` + a produced PR-body text/file | |
| 8 | `diagnosing-bugs` invoked (conditional) | skill tool, `input.id == "diagnosing-bugs"`, only expected if a bug actually surfaced during implementation | |

### 6.5 Final artefact quality

Run independently of either agent's own self-report, by the human/analysis
environment, not by a prompt asking the agent to confirm its own success
(per the `verification-before-completion` skill's own stated philosophy —
evidence before assertions, applied here to the *experiment's* verification,
not just each trial's internal one):

1. `pytest` (or whichever runner the trial actually used) run fresh, in the
   trial's own project directory, confirming the suite passes with no
   modification.
2. A fixed smoke script (identical for both trials, written once by the
   human reviewer, not by either agent) that starts the service and curls
   every endpoint in the brief, checked for correct status codes and basic
   shape.
3. `git log --stat` / `git branch -a` read directly, to confirm (or refute)
   that the methodology each framework claims to follow actually left the
   evidence it should: worktree and branch structure for Superpowers; commit
   granularity matching whatever each framework's own TDD skill specifies
   (one commit per red-green-refactor cycle is the common convergent
   content both frameworks describe, per `FINDING-2026-10-29-22`).

---

## 7. Comparison rubric

| Axis | What is scored | Evidence source |
|---|---|---|
| **Experience: friction** | Count of orchestrator-answered clarifying questions; count of permission/approval stalls; any point the pipeline needed a human (or the scripted fallback) to unstick it | transcript + orchestrator's own interaction log |
| **Experience: clarity** | Whether the agent's running narration (plan documents, spec documents, PR body) would let a human unfamiliar with the session understand what was built and why, read cold | human read of the produced spec/plan/PR-body artefacts, scored independently for each trial by the same reader |
| **Experience: asked vs assumed** | For each of the brief's two deliberately-ambiguous decisions (section 1) and any other decision point that arose, did the agent ask (matched against section 5's table) or assume (and if it assumed, did it say so)? | transcript scan + section 5's logged match decisions |
| **Success: task completion** | Did the brief's "definition of done" get met? (binary, then qualified by what was missing if not) | section 6.5 |
| **Success: methodology fidelity** | Checklist completion percentage from section 6.4, per framework's own checklist (not cross-compared stage-for-stage, since the two pipelines are not the same shape — compared as "did each framework do what it says it does," not "did both do the same thing") | section 6.4 |
| **Success: failure modes** | Any detected loop (the same tool call with materially the same input repeated 3+ times with no state change), stall (no progress for N turns), or hard-cap trip (section 8) | transcript scan, orchestrator's cap-tripped log |
| **Token usage: total** | Session-level `tokens`/`cost` rollup (section 6.2) | export `info` block, cross-checked against `opencode stats` |
| **Token usage: by stage** | Per-stage sums attributed via the method in section 6.2 | export `messages[]`, segmented at each checklist-stage boundary |
| **Token usage: fan-out overhead** | Tokens attributable to subagent/task dispatches specifically (Superpowers' per-task subagent + reviewer; Pocock's code-review's two parallel subagents; Pocock's `research` skill's background agent, if invoked) vs tokens attributable to the top-level driving session | export, segmented by `subagent`/`task` tool part `sessionID` vs the top-level trial session's own id |

The writeup should present each axis for both trials side by side, and
should not force an overall "winner" score — the two frameworks have
different design philosophies (forced single pipeline vs anti-methodology,
user-selected skills, per `FINDING-2026-10-29-23`/`-24`) and a fair
comparison reports how each performs against *its own* stated goals as well
as against the shared task.

---

## 8. Risks, costs, and guardrails

### 8.1 Budget estimate

**Rough, explicitly unverified against real provider pricing for a pinned
model (see 8.3) — order-of-magnitude only, to be replaced with real numbers
once the model is pinned and a pilot run exists:**

- Superpowers' pipeline dispatches a fresh subagent *and* a fresh reviewer
  per plan task (`subagent-driven-development`), plus whole-branch review at
  the end (`requesting-code-review`). For a brief this size (roughly 6-10
  plan tasks: data model, each endpoint or small endpoint group, tag
  listing, validation, test suite), that is plausibly 12-20 subagent
  sessions, each a handful of turns, on top of the controller session's own
  turns.
- Pocock's pipeline (`to-spec` → `to-tickets` → `implement-spec` →
  `code-review` → `pr`) runs mostly in one continuous session, with fan-out
  limited to `code-review`'s two parallel axes (and optionally `research`'s
  background agent, not expected to trigger for a brief this well-specified).
- **First-pass estimate:** Trial A (Superpowers) 1.5-3x the token/cost
  footprint of Trial B (Pocock), driven by the subagent fan-out difference
  itself — which is also exactly the kind of finding this experiment exists
  to surface, not a flaw in the design.
- In absolute terms, assuming a mid-tier frontier model's typical per-token
  pricing, a plausible range is **single-digit to low-double-digit USD for
  Trial B, and up to roughly 2-3x that for Trial A**, for one complete run
  of each. **This number has not been checked against the pinned model's
  actual published rates and should be treated as a placeholder the human
  reviewer replaces before approving spend.**

### 8.2 Hard caps

Enforced by the orchestrator script, not left to either agent's own
judgement:

- **Max orchestrator-issued turns per trial:** 40. If hit, abort the trial,
  record it as a cap-trip (a result in its own right, not a discarded run),
  and still run sections 6.5/7 against whatever state exists.
- **Max wall-clock per trial:** 90 minutes from first prompt to last.
- **Max spend per trial:** a hard figure set by the human reviewer once 8.1
  is replaced with real numbers; checked by polling `opencode stats --cost`
  at intervals, not only at the end, so a runaway trial can be aborted
  mid-flight rather than discovered after the fact.
- **Max subagent/task fan-out per trial:** 20 dispatches. Guards
  specifically against a recursive or looping dispatch pattern (e.g. a
  reviewer repeatedly spawning another reviewer) rather than legitimate,
  bounded fan-out from the pipeline's own design.

### 8.3 Sources of non-determinism, and mitigations

- **Model/provider version drift.** This sandbox's configured model
  (`anthropic-azure-msd/mnorr001-claude-sonnet-5`) is an internally routed
  alias; routed aliases can point at a different underlying model snapshot
  over time without the alias name changing. **Mitigation:** before the real
  run, pin an exact, directly-addressed model (provider + model id +
  variant, in the `provider/model#variant` form `opencode run --model`
  itself accepts) for both trials, and record the exact string used in the
  results. Run both trials back-to-back (same day, same pinned string) to
  further reduce the window for silent upstream model updates.
- **Temperature/sampling.** Not independently confirmed whether every
  provider this binary supports exposes a configurable temperature through
  OpenCode's agent config. **Mitigation:** check the pinned provider's
  config surface once the model is chosen; if temperature is configurable,
  pin it identically for both trials and record the value; if it is not
  configurable for the chosen provider, record that fact rather than assume
  determinism it cannot actually provide.
- **Skill auto-invocation flakiness.** Superpowers removes this source only
  for its own first-turn bootstrap (forced via a session-context hook, not
  model judgement, per `.opencode/plugins/superpowers.js`); every subsequent
  skill in its own pipeline (`brainstorming` choosing to hand off to
  `writing-plans`, etc.) is still reached by the model's own judgement
  reading the bootstrap text, same as every one of Pocock's model-invoked
  skills. **This is not a confound to mitigate away — it is a real property
  of both frameworks and should be reported as such**, including the
  possibility that a re-run of the identical trial produces a different
  checklist result purely from this source. **Mitigation, to the extent one
  exists:** run each trial once for the primary comparison, but budget
  (section 8.1/8.2 permitting) for a second repetition of at least one trial
  to get a feel for how much the checklist outcome actually varies run to
  run, and report that variance honestly rather than present a single run
  as definitive.
- **Network/install non-determinism.** Superpowers installs from a
  git-backed spec; pin an exact tag/commit (section 3.1) rather than a
  floating branch. Pocock's installer (`npx skills@latest`) resolves
  whatever `skills@latest` the npm registry serves at install time; **record
  the resolved `skills` package version** (printed by the installer, or
  checkable via `npx skills@latest --version` immediately before the real
  install) alongside the exact commit of `mattpocock/skills` it pulled
  skills from, since both can drift between a planning pass and the actual
  run.
- **Time-of-day / provider-side changes between the two trials.** Run both
  trials in the same sitting, back-to-back, rather than days apart, to
  minimise the chance that a provider-side model or pricing change affects
  one trial and not the other.

### 8.4 Required human approval before any execution

This plan assumes it will be reviewed by a human partner before a single
real command runs. Specifically requiring sign-off, beyond the assumptions
already flagged inline:

- The exact pinned model/provider string and its approximate per-token cost
  (section 8.1/8.3).
- The final hard-cap numbers (section 8.2), particularly the spend cap.
- The sample project brief (section 1) and the virtual-stakeholder table
  (section 5), since both stand in for a human partner's own judgement.
- Running `opencode plugin add` and `npx skills@latest add` at all — both
  fetch and execute third-party code from the network
  (`github.com/obra/superpowers`, `github.com/mattpocock/skills`, and
  whatever `skills@latest` itself resolves to on npm) into a real, if
  isolated, environment.
- Any actual spend-incurring API call — nothing in this plan should run
  against a real, metered provider account without that account's owner
  explicitly approving it.

---

## 9. How to actually run this

A step-by-step procedure for a human (or an agent explicitly instructed to
execute, after human approval of this plan) to follow. Each step references
the section above it implements.

1. **Resolve the open assumptions.** Get explicit sign-off from a human
   partner on: the sample project brief (section 1), the hybrid fairness
   mechanism and its Q&A table (section 5), the pinned model/provider
   (section 8.3), and the hard-cap numbers (section 8.2). Do not proceed
   past this step without it.
2. **Prepare the scratch area.** Create
   `/tmp/opencode/experiment-<date>/` with the layout in section 2.2.
   Verify `/tmp` and `/tmp/opencode` carry no ancestor `opencode.json(c)`.
3. **Pilot the fairness mechanism.** Run only the interview/spec stage of
   each framework once (a short, capped trial), read the transcripts, and
   confirm the brief neither over-specifies (zero questions from either
   side) nor under-specifies (questions the fallback table can't match) to
   the point the comparison would be uninformative. Adjust the brief/table
   if needed and re-confirm with the human partner before the full run.
4. **Verify isolation.** Run the section 2.3 acceptance test for both
   trial directories. Do not proceed until both pass.
5. **Install Superpowers into Trial A** (section 3.1), pinned to an exact
   tag/commit, and confirm activation per INSTALL.md's own acceptance test.
6. **Install Pocock's Skills into Trial B** (section 3.2). Immediately
   after, inspect exactly what landed under the trial's `.opencode/`
   directory and resolve the section 3.2 flagged assumption (command vs
   skill registration for user-invoked skills) before writing the final
   orchestrator script.
7. **Stand up the driving mechanism** (section 4): `opencode serve` per
   trial, confirm `opencode api get /api/info` responds against each
   trial's server, and configure each trial's isolated global config with
   the permission rules needed for unattended operation (section 4.1's
   flagged `--auto`-equivalent-for-`serve` question).
8. **Run Trial A to completion or a hard cap**, per the orchestrator design
   in section 4.3, driven by the brief and fairness table. Export the
   session (section 6.1), capture stats (section 6.2), stop `serve`.
9. **Run Trial B to completion or a hard cap**, identically.
10. **Run the artefact-quality checks** (section 6.5) for both trials,
    independently, by the human/analysis environment — not by asking either
    agent to self-report.
11. **Score both trials against the lifecycle checklists** (section 6.4)
    and the comparison rubric (section 7), by reading each export
    mechanically (skill/command tool-part scan) before reading it for
    prose/narrative quality.
12. **Write up the comparison**, reporting both frameworks against their own
    stated goals as well as the shared task, explicitly naming every
    non-determinism source from section 8.3 that could plausibly have
    affected the specific result observed, and including the raw exports
    and stats as an appendix/attachment rather than only summarised figures.
13. **Decide on repetition.** If budget (section 8.1/8.2) allows, repeat at
    least one trial once more to sanity-check how much the checklist/rubric
    outcome varies run to run (section 8.3's skill-flakiness source), and
    report that variance alongside the primary result.
