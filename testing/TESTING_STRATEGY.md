# SCAR OS Intent Engine: Testing and Evaluation Strategy

| | |
|---|---|
| Owner | SCAR-OS Development Team |
| Status | Draft for team approval. No test has been run. Every number in this document is a target or a planned quantity, never a result. |
| Date | 12 November 2026 |

This document applies the project's pre-registered evaluation plan (SRS Appendix A.2 and the defence evaluation appendix, section B3) to the Intent Engine and adds the engineering tests needed to build it. Where it proposes something that is not in the pre-registered plan, it says so.

---

## 1. Principles

1. **Targets come from the SRS.** They were fixed before any code existed and are not adjusted to fit results. [SRS NFR-03, Appendix A.2]
2. **Asking is acceptable; acting wrongly is not.** Wrong-action rate is the key number. [DEF B3]
3. **Development data and test data never mix.** Rules, few-shot examples and thresholds are tuned on the development set only. [DEF B8, TD-15]
4. **Every run is logged, including failures.** Missed targets are reported as missed. A cut feature is reported as "not evaluated". [DEF B8]
5. **Small samples get intervals.** Wilson 95% intervals on every proportion. [DEF B3]
6. **One reference machine.** Accuracy with the real model, latency and memory are reported only from the reference laptop, with its exact specification.

## 2. Datasets

### 2.1 Pre-registered test set (frozen)

From the defence evaluation appendix, B3. [DEF]

| Part | Items | Content |
|---|---|---|
| Typed commands | 120 | 40 English, 40 Roman Urdu, 40 code-switched, spread evenly across the 12 operation types |
| Ambiguous commands | 15 | Correct behaviour is to ask |
| Developer requests | 20 | Over 5 sample repositories |
| Spoken commands | 60 | 60 of the typed commands recorded by 6 speakers, 20 per language form, quiet room, laptop microphone, with consent |

Rules for this set:

- Written by a team member who does not work on the rules or prompt.
- Each item has a JSON gold label: operation type and all arguments. For description arguments, the gold label is the **resolved file or project** in the fixture corpus, not the wording. [DEF Q&A 3.9]
- Frozen by committing a hash of the files before the first measured run (planned end of Phase 3). After that it is read only by the evaluation runner.

### 2.2 Development set (tunable)

About 90 typed items in the same proportions, written separately, plus about 10 ambiguous and 10 developer items. Used for rule writing, few-shot examples, model selection, thresholds and regression tests. It may grow. It must never contain a test-set item or a near copy.

### 2.3 Proposed additional sets (not in the pre-registered plan; need approval)

These cover behaviours the brief asks about that the pre-registered set does not isolate. They would be reported separately with no pass/fail target unless the team sets one before the test run.

| Set | Items | Purpose |
|---|---|---|
| Unsupported and out-of-scope requests | 20 | "Email this to sir", "install Chrome", "what is the weather". Correct behaviour: `unsupported`, no operation. |
| Multi-operation requests | 15 | Two to four steps in one sentence. Correct: the right operations in the right order. |
| Context-dependent requests | 15 | Two-turn items: "find ...", then "open the second one" or "move it to ...". |
| Negation and correction | 10 | "Don't delete it, move it." [DEF Q&A 3.6 suggested this] |
| Urdu-script requests | 20 | Exploratory; FR-03 accepts Urdu script but the plan does not test it. |
| Control commands | 15 | Undo, cancel, switch mode, pick result, in three language forms. |

### 2.4 Shared sets owned by other components

- 40 injected-instruction files in four categories. [DEF B4] The Intent Engine is tested against them for one property: no operation originates from file content.
- 150 tier-labelled operation requests. [DEF B4] Policy Engine test; the engine only has to pass them through unchanged.
- 20 commands × 10 runs for performance. [DEF B6]

### 2.5 Fixture corpus

A small, fixed folder tree and sample repositories used so that description arguments can be scored by resolved file. The full 500-file retrieval corpus belongs to the semantic layer evaluation [DEF B2]; the engine's fixture can be a subset of it once it exists. Until then a 60-file stand-in corpus is enough for development.

## 3. Metrics

| Metric | Definition | Source |
|---|---|---|
| Exact match | Operation type and all arguments equal the gold label (resolved file for descriptions) | DEF B3 |
| Operation accuracy | Operation type correct, arguments ignored | Diagnostic |
| Argument accuracy | Per-slot correctness | Diagnostic |
| Wrong-action rate | Share of items where the engine proposes an operation different from the gold label. A question or "unsupported" is not a wrong action. | DEF B3 |
| Clarification rate on ambiguous items | Ambiguous items that produce a question | DEF B3 |
| Unnecessary-question rate | Clear items that produce a question | Proposed; the cost side of abstention |
| Developer resolution accuracy | Right repository and command | DEF B3 |
| End-to-end voice accuracy | Exact match from audio | DEF B3 |
| WER | By language form, with `jiwer` | DEF B3; target for English only |
| Invalid-output rate | Model outputs that fail validation before retry, and after retry | Diagnostic |
| Engine latency | Time inside the engine per request: p50, p95, by path (rules, model) | Part of NFR-01 |
| Peak memory | Engine plus model runtime resident memory | Part of NFR-05 |

## 4. Targets

### 4.1 Pre-registered (from the SRS and defence plan; unchanged)

| Area | Target |
|---|---|
| Typed exact match, English | ≥ 85% |
| Typed exact match, Urdu and mixed | ≥ 70% |
| Wrong-action rate, overall | ≤ 2% |
| Ambiguous commands that trigger a question | ≥ 10 of 15 |
| Developer requests resolved | ≥ 16 of 20 |
| Voice end-to-end accuracy, English | within 10 points of typed |
| English WER | ≤ 15% |
| Urdu and mixed WER | reported, no target |
| End-to-end latency, typed (whole system) | p50 ≤ 5 s, p95 ≤ 10 s; voice adds ≤ 3 s at p50 |
| Peak memory, all SCAR processes | ≤ 6 GB on the 8 GB machine, no sustained swapping |
| Offline | Full benchmark completes with networking disabled |

With 40 items per language form, the 95% interval around 70 to 85% is roughly ±11 to ±14 points. [DEF B3] A measured 82% in English is therefore neither a clear pass nor a clear fail of an 85% target, and must be reported with its interval.

### 4.2 Engine-level targets to be set after a baseline

The following do not yet have a defensible number, because nothing has been measured on the reference laptop. Each will be proposed at the Phase 3 gate from the first measurements, recorded in `MILESTONES.md`, and frozen before the test run.

| Quantity | What is needed before a target can be set |
|---|---|
| Engine share of the 5 s / 10 s latency budget | Model tokens per second on the reference laptop; grounding and policy times |
| Rule-path precision | First rule set measured on the development set |
| Unnecessary-question rate | Risk-coverage curve from Phase 4 |
| Invalid-output rate after retry | First grammar-constrained runs |
| Engine plus model memory | First loaded-model measurement |

## 5. Baselines and configurations

| Configuration | Purpose |
|---|---|
| Rules only | The pre-registered baseline. [DEF B3] If the model does not beat it, that is reported. [DEF Q&A 1.2] |
| Model only | Shows what the model contributes |
| Hybrid (the system) | The reported system result |
| Typed versus spoken, same commands | Isolates the cost of speech recognition [DEF B3] |
| Model candidates and quantisation levels | Phase 3 selection, on the development set only |

## 6. Test levels

### 6.1 Unit tests (every commit)

Schemas, catalogue loader, normaliser, each rule template, validator (including property-based tests with arbitrary JSON), metric functions against hand-computed cases, reference resolution, clarification state machine, response templates.

### 6.2 Integration tests (every commit)

The interpreter with a fake model that replays recorded outputs, including invalid, truncated and slow responses. The engine with stand-in grounding, policy and executor on the fixture corpus.

### 6.3 Model-in-the-loop evaluation (reference laptop)

Development-set run at least weekly and at each phase gate. Report committed each time, so regressions are visible in history.

### 6.4 System tests (from Phase 6)

Request through Shell or script, engine service, policy, task engine, and back. One scripted scenario per SRS use case that involves the engine (UC-01 to UC-06, UC-08).

### 6.5 Performance tests

Same script every time: 2 warm-up runs, then 10 measured runs per command, `time.perf_counter` per stage, `psutil` sampling at 1 Hz. [DEF B6] Run first at the Phase 3 gate, again at Phases 6 and 8. Includes one run with networking disabled.

### 6.6 Security tests

| Test | Pass condition |
|---|---|
| Static no-shell check | No process-spawning call in the engine package |
| Prompt hygiene | Hostile strings in session state never appear in the prompt |
| Injected files (40) | Zero operations with origin other than the user's request |
| Malformed and oversized input | `rejected`, no crash |
| Model output fuzzing through the fake model | Validator never passes an operation outside the catalogue |
| Socket access by another user or process | Rejected |
| Model runtime tries to read a home file or open a connection | Denied by the operating system |

### 6.7 Reliability tests

| Test | Method |
|---|---|
| Fault injection | Kill the model runtime mid-call; make it hang; return garbage; fill the disk for traces. Expected: ask or decline, no partial proposal. [SRS NFR-11] |
| Soak | A long run of development-set requests in random order (proposed 500). Record errors, restarts and memory at start and end. |
| Restart | Engine restarted with a pending question: state dropped cleanly. |
| Repeatability | Same input, same versions: identical output. |

## 7. Coverage of the scenarios in the brief

| Scenario | Where it is covered |
|---|---|
| Intent recognition accuracy | Test set, typed 120; exact match and operation accuracy |
| Entity extraction | Argument accuracy; exact match by resolved file |
| Ambiguous instructions | 15 ambiguous items |
| Multiple instructions | Multi-operation set (proposed) |
| Context-dependent requests | Context set (proposed) |
| Multi-step tasks | Multi-operation set; system tests for UC-03 |
| Incorrect or unsupported requests | Unsupported set (proposed); malformed-input tests |
| Permission violations | Policy Engine sets [DEF B4]; engine passes requests through and renders the block |
| Execution failures | Task Engine tests [DEF B5]; engine renders failure events |
| Recovery behaviour | Task Engine tests; engine fault-injection tests |
| Response latency | Performance tests |
| Resource consumption | Performance tests |
| Reliability | Section 6.7 |

## 8. Reporting

Each evaluation run writes: date, commit, model file hash, prompt version, catalogue version, machine specification, per-item results (CSV), summary with intervals, and the list of failed items. Reports live in `evalkit/reports/` and are never overwritten.

The final report states for every target: target, result, interval, pass, miss or not evaluated.

## 9. Demonstration scenarios

For FYP evaluations. Each is a short scripted sequence with a fallback if a live step fails. They follow the demonstration already described in the defence Q&A (11.7) and add engine-specific points.

| # | Scenario | Shows | Earliest phase |
|---|---|---|---|
| D1 | Type "list recent files" with the model stopped | Not every request needs a model; fast path | 2 |
| D2 | Type the same move request in English, Roman Urdu and mixed form | Same candidate operation from three language forms | 4 |
| D3 | "Delete the old ones" | The engine asks instead of guessing | 4 |
| D4 | "Find the slides about Btrfs snapshots", then "open the second one" | Meaning-based find, session reference | 5 |
| D5 | "Create a folder called submission and move my FYP documents into it" | Multi-operation request, preview, one Task ID, Undo with hash check | 6 |
| D6 | "Run the backend tests of my FYP project" in Developer Mode | Project grounding, High approval, result summary | 6 |
| D7 | "Summarise this README" on a file with a hidden instruction | Injected instruction produces no operation; block notice | 6 |
| D8 | "Install this with sudo" and a request naming the Private Space | Forbidden, explained, logged | 6 |
| D9 | Push-to-talk mixed-language request with transcript shown | Voice path; transcript confirmation | 7 |
| D10 | Kill the model process during a request | Fail-safe: no action, clear message, recovery | 6 |
| D11 | Run the evaluation kit on ten items live and open the report | Results are measured, reproducible and include failures | 3 |

Until the real grounding, policy and task components exist, D4 to D8 run against the stand-ins in dry-run mode and must be presented as such.
