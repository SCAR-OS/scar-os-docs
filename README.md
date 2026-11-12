# SCAR OS — Documentation & Research

**SCAR OS** (FYP 23F-02, Institute of Emerging Technologies, Sukkur IBA
University Khairpur Campus) is an AI-centred, developer-focused operating
environment built on Fedora Atomic/Silverblue. It lets a user say or type
what they want — in English, Urdu, or a mix — and have it carried out on
real files and projects, under rules the AI cannot change, with every
change recorded and reversible.

This repository is the project's **central documentation and research
source of truth**. Implementation code lives in
[`SCAR-OS/SCAR-OS`](https://github.com/SCAR-OS/SCAR-OS) — that repo tracks
`CLAUDE.md`/dev-setup docs for contributors, this one tracks *what* we're
building and *why*.

## How this is organized

| Directory | Contents |
|---|---|
| `MILESTONES.md` | The single source of truth for progress. If a status here disagrees with anything else, this file wins. |
| `planning/` | `PLANNING_REPORT.md` — short reading path into the rest of the documentation set. |
| `architecture/` | `INTENT_ENGINE_ARCHITECTURE.md` — system design for speech/LLM-driven intent recognition. |
| `specifications/` | `FEATURE_SPECIFICATION.md` — per-feature specs with priority, estimate, and source (SRS/defence/recommendation). |
| `decisions/` | `TECHNICAL_DECISIONS.md` — technology/approach evaluation and the resulting proposed decisions (TD-xx). |
| `testing/` | `TESTING_STRATEGY.md` — the pre-registered evaluation plan plus engineering test strategy. |

Everything above is currently **draft, pending team approval** — see each
document's own status line. Nothing in these documents is implemented yet;
implementation status is tracked separately in the `SCAR-OS` repo's README.

## Intent Engine documentation specifically

Start at `planning/PLANNING_REPORT.md` for orientation, then
`architecture/INTENT_ENGINE_ARCHITECTURE.md` for the design and
`specifications/FEATURE_SPECIFICATION.md` (prefix `IE-`) for individual
features.

## Updating documentation

1. Open an issue in `SCAR-OS/SCAR-OS` describing what needs research,
   specification, or documentation work — anyone on the team can pick it
   up, regardless of the module split described in the original proposal.
2. Update the relevant file(s) here via a pull request.
3. Once a decision or spec is approved, reference it (e.g. `TD-03`, `IE-A01`)
   from the implementation PR in `SCAR-OS/SCAR-OS` that acts on it.
4. Update `MILESTONES.md` only after the corresponding work is actually
   verified — not when it's merely started.

## Links

- Implementation: https://github.com/SCAR-OS/SCAR-OS
- Organization: https://github.com/SCAR-OS
