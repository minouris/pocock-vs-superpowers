# Pocock vs Superpowers

A comparative research project examining two "agent skills" frameworks for AI coding agents: [Obra's Superpowers](https://github.com/obra/superpowers) and [Matt Pocock's Skills](https://github.com/mattpocock/skills).

## Purpose

Both projects package reusable, markdown-based instructions ("skills") that guide an AI coding agent through a software development lifecycle, but they take different approaches to activation, scope, and governance. This repository captures a systematic comparison of the two: what each project is, where their capabilities overlap, where they diverge, how their respective project lifecycles map onto each other stage by stage, and how a user could run both together in one project.

## Main Report

**[docs/superpowers_pockock_comparison.md](docs/superpowers_pockock_comparison.md)** is the primary deliverable. It covers:

- What each project is, and its stated philosophy
- Where the two frameworks overlap and where they diverge
- Advantages and disadvantages of each approach
- A stage-by-stage breakdown of Superpowers' enforced pipeline, and how the same lifecycle maps onto Pocock's Skills
- Capabilities Pocock's Skills offers that Superpowers has no equivalent for
- Practical guidance for using both toolkits together in the same project

## Other Contents

- **[docs/dual-session-skill-experiment-plan.md](docs/dual-session-skill-experiment-plan.md)** — a design for a controlled experiment comparing the two frameworks by running isolated OpenCode sessions through a shared sample project. Planning only; not yet executed.
- **[.memory/](.memory/superpowers-vs-pocock-skills/superpowers-vs-pocock-skills-index.md)** — the underlying research knowledge base: every finding and term behind the report, each with an exact file and line citation back to the two source repositories.

## Sources

Research was conducted by directly examining the official repositories of both projects rather than relying on secondary summaries. See the main report's "Research Trail" section for full traceability.
