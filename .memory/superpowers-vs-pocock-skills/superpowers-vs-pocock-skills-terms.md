# Superpowers vs Pocock Skills Terms

**Last Updated:** 2026-10-29 00:00
**Verified Terms:** 5
**Pending Terms:** 0

---

## Table of Contents

- [Agent Skill](#agent-skill)
- [SKILL.md](#skillmd)
- [Progressive Disclosure](#progressive-disclosure)
- [Model-invoked Skill](#model-invoked-skill)
- [User-invoked Skill](#user-invoked-skill)
- [SessionStart Hook](#sessionstart-hook)

---

## Agent Skill

**Captured:** 2026-10-29 00:00

A packaged, reusable unit of instructions, context, and optionally scripts/resources that an AI coding agent can load and follow to perform a specific task or enforce a specific process, distributed as a folder containing a `SKILL.md` file.

### Sources

| Source | Status | Verification |
|--------|--------|---------------|
| [obra/superpowers skills/ directory](https://github.com/obra/superpowers/tree/main/skills) | VERIFIED | Verified 2026-10-29 by direct repository read: 15 skill folders, each one `SKILL.md`-based unit |
| [mattpocock/skills skills/ directory](https://github.com/mattpocock/skills/tree/main/skills) | VERIFIED | Verified 2026-10-29 by direct repository read: bucketed skill folders, each one `SKILL.md`-based unit |

### Description

Both projects use "skill" to mean the same structural concept: a directory whose entry point is `SKILL.md`, loaded by a harness (Claude Code, OpenCode, Codex, etc.) and matched against conversational context via its frontmatter `description`. A skill may bundle companion reference files that are not loaded until the skill itself is active (see Progressive Disclosure). This is the base unit both repositories are built from, consistent with Claude Code's own "Agent Skills" naming for the feature.

### See Also

- [SKILL.md](#skillmd)
- [Progressive Disclosure](#progressive-disclosure)

### Referenced By

- [FINDING-2026-10-29-6](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-6) - Superpowers Skill Format
- [FINDING-2026-10-29-21](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-21) - Comparison - Convergent Base Unit

---

## SKILL.md

**Captured:** 2026-10-29 00:00

The required entry-point filename for an Agent Skill: a Markdown file with YAML frontmatter (at minimum `name` and `description`) followed by the skill's instructions body.

### Sources

| Source | Status | Verification |
|--------|--------|---------------|
| [obra/superpowers skills/brainstorming/SKILL.md](https://github.com/obra/superpowers/blob/main/skills/brainstorming/SKILL.md) | VERIFIED | Verified 2026-10-29 against frontmatter lines 1-4 |
| [mattpocock/skills skills/engineering/tdd/SKILL.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/tdd/SKILL.md) | VERIFIED | Verified 2026-10-29 against frontmatter lines 1-4 |

### Description

Both repositories use the identical filename and minimum frontmatter shape, but diverge on optional fields: Superpowers' frontmatter never includes anything beyond `name`/`description`; Pocock's frontmatter adds `disable-model-invocation: true` for user-invoked skills (see Model-invoked Skill / User-invoked Skill).

### See Also

- [Agent Skill](#agent-skill)
- [Model-invoked Skill](#model-invoked-skill)
- [User-invoked Skill](#user-invoked-skill)

### Referenced By

- [FINDING-2026-10-29-6](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-6) - Superpowers Skill Format
- [FINDING-2026-10-29-13](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-13) - Pocock Skills Invocation Model (via Agent Skill)
- [FINDING-2026-10-29-21](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-21) - Comparison - Convergent Base Unit

---

## Progressive Disclosure

**Captured:** 2026-10-29 00:00

A skill-authoring pattern where the top-level `SKILL.md` stays short and defers detail to companion reference files that are read only when the skill's specific sub-topic is reached, keeping the always-loaded context small while allowing arbitrarily deep supporting material.

### Sources

| Source | Status | Verification |
|--------|--------|---------------|
| [obra/superpowers skills/test-driven-development/](https://github.com/obra/superpowers/tree/main/skills/test-driven-development) | VERIFIED | Verified 2026-10-29: SKILL.md links out to a testing-anti-patterns reference file rather than inlining it |
| [mattpocock/skills skills/engineering/tdd/](https://github.com/mattpocock/skills/tree/main/skills/engineering/tdd) | VERIFIED | Verified 2026-10-29: SKILL.md links out to tests.md and mocking.md rather than inlining them |

### Description

Both projects independently apply this same pattern for their TDD skills (and others): the `SKILL.md` body states the rules of the loop, then says "see X.md for examples" rather than inlining the examples. This keeps the model-visible context minimal until the specific reference is actually needed, which is the generally-documented rationale for splitting Agent Skills into a short entry file plus companion resources.

### See Also

- [Agent Skill](#agent-skill)
- [SKILL.md](#skillmd)

### Referenced By

- [FINDING-2026-10-29-21](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-21) - Comparison - Convergent Base Unit

---

## Model-invoked Skill

**Captured:** 2026-10-29 00:00

A skill reachable both by the AI model's own judgment (automatic, context-based matching against its `description`) and by a human explicitly naming it; the default invocation class when no restricting frontmatter flag is present.

### Sources

| Source | Status | Verification |
|--------|--------|---------------|
| [mattpocock/skills .agents/invocation.md](https://github.com/mattpocock/skills/blob/main/.agents/invocation.md) | VERIFIED | Verified 2026-10-29 — this document defines the term exactly and is the canonical, project-internal source |

### Description

This is an internal/project-specific term coined by Matt Pocock's Skills repo to label one half of its explicit invocation-class split (the other half is User-invoked Skill). Superpowers has no equivalent term or concept: every Superpowers skill behaves as "model-invoked" by this definition, but the project does not name or document the distinction because it never restricts invocation.

### See Also

- [User-invoked Skill](#user-invoked-skill)
- [SKILL.md](#skillmd)

### Referenced By

- [FINDING-2026-10-29-13](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-13) - Pocock Skills Invocation Model
- [FINDING-2026-10-29-23](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-23) - Comparison - Divergent Activation Philosophy

---

## User-invoked Skill

**Captured:** 2026-10-29 00:00

A skill reachable only by a human explicitly typing its name (e.g., as a slash command); marked by `disable-model-invocation: true` in frontmatter (Claude Code) and `policy.allow_implicit_invocation: false` in a companion `agents/openai.yaml` (Codex). A user-invoked skill may call a model-invoked skill, but never another user-invoked skill.

### Sources

| Source | Status | Verification |
|--------|--------|---------------|
| [mattpocock/skills .agents/invocation.md](https://github.com/mattpocock/skills/blob/main/.agents/invocation.md) | VERIFIED | Verified 2026-10-29 — this document defines the term exactly and is the canonical, project-internal source |
| [mattpocock/skills skills/productivity/grill-me/SKILL.md](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) | VERIFIED | Verified 2026-10-29 — concrete example of the `disable-model-invocation: true` flag in use |

### Description

This is an internal/project-specific term, the counterpart to Model-invoked Skill. It is used for orchestration skills like `grill-me`, `tdd`, and `setup-matt-pocock-skills` where the project wants a human decision gate rather than letting the model decide when to run a heavyweight interview or setup flow. Superpowers has no equivalent concept.

### See Also

- [Model-invoked Skill](#model-invoked-skill)
- [SKILL.md](#skillmd)

### Referenced By

- [FINDING-2026-10-29-13](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-13) - Pocock Skills Invocation Model
- [FINDING-2026-10-29-23](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-23) - Comparison - Divergent Activation Philosophy

---

## SessionStart Hook

**Captured:** 2026-10-29 00:00

A harness-level lifecycle event (fired at session start, after `/clear`, and/or after context compaction) that a plugin can register a handler against to inject content or run code before the user's first message is processed.

### Sources

| Source | Status | Verification |
|--------|--------|---------------|
| [obra/superpowers hooks/hooks.json](https://github.com/obra/superpowers/blob/main/hooks/hooks.json) | VERIFIED | Verified 2026-10-29 — concrete Claude Code hook registration with matcher `"startup\|clear\|compact"` |
| [obra/superpowers .opencode/plugins/superpowers.js](https://github.com/obra/superpowers/blob/main/.opencode/plugins/superpowers.js) | VERIFIED | Verified 2026-10-29 — OpenCode V2 equivalent via `ctx.session.hook("context")` |

### Description

Superpowers is built around registering a handler on this hook (under whatever name each harness gives it) in nearly every one of its 10 harness adapters, specifically to force its `using-superpowers` bootstrap skill into context before the model can act. A recursive search of Pocock's Skills repository found no hook registration of any kind, confirming the two projects take opposite positions on using this mechanism (see FINDING-2026-10-29-23).

### See Also

- [Agent Skill](#agent-skill)

### Referenced By

- [FINDING-2026-10-29-2](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-2) - Superpowers Mechanism
- [FINDING-2026-10-29-23](superpowers-vs-pocock-skills-facts.md#finding-2026-10-29-23) - Comparison - Divergent Activation Philosophy

---
