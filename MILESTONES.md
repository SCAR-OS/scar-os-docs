# SCAR OS Intent Engine: Milestones

| | |
|---|---|
| Project | SCAR OS: An AI-Centred, Developer-Focused Operating Environment (FYP 23F-02) |
| Institution | Institute of Emerging Technologies, Sukkur IBA University Khairpur Campus |
| Supervisor | Ms. Irum Sindhu |
| Owner | SCAR-OS Development Team (Amar Jaleel, Azhar Ali, Faizan, Kamran Khan) |
| Document status | **Draft, waiting for team approval. Implementation has not started.** |
| Created | 12 November 2026 |

This file is the single source of truth for Intent Engine progress. Design detail lives in `docs/`:

| Document | Contents |
|---|---|
| [docs/INTENT_ENGINE_ARCHITECTURE.md](docs/INTENT_ENGINE_ARCHITECTURE.md) | Boundaries, pipeline, components, contracts, error handling, security |
| [docs/FEATURE_SPECIFICATION.md](docs/FEATURE_SPECIFICATION.md) | Every feature (IE-xx) with purpose, method, dependencies, difficulty, priority, estimate, tests, demonstration |
| [docs/TECHNICAL_DECISIONS.md](docs/TECHNICAL_DECISIONS.md) | Technology evaluation, decisions (TD-xx), engineering standards |
| [docs/TESTING_STRATEGY.md](docs/TESTING_STRATEGY.md) | Datasets, metrics, targets, test levels, demonstration scenarios |
| [docs/PROJECT_RISKS.md](docs/PROJECT_RISKS.md) | Risks (R-xx), unknowns (U-xx), open questions (Q-xx) |
| [docs/PLANNING_REPORT.md](docs/PLANNING_REPORT.md) | Short summary of the planning investigation and the engineering assessment |

**Rules for this file**

- A box is ticked only when the work is done **and** its listed test or check has passed. A ticked box names the commit or report that proves it.
- Dates are filled in when things happen, not predicted afterwards.
- Changes to scope, schedule or targets are recorded in the change log at the end, never made silently.
- No AI co-author credit or attribution appears in this repository, its commits, documents or pull requests.

---

## 1. Project overview

SCAR OS is a local, offline layer of services and a desktop shell on a Linux foundation. It adds voice and text control in English, Urdu and mixed Urdu-English, meaning-based access to files and software projects, a four-level risk policy for AI actions, task-level audit and rollback, and a Private Space the AI cannot reach, while the normal desktop, file manager and terminal keep working.

The **Intent Engine** is the service that turns one user request into validated candidate operations in a fixed schema, or a clarification question, or an honest "not supported". It proposes; it never executes, never resolves real paths and never decides permission. Those belong to grounding, the Task Engine and the Policy Engine.

Current state on 12 November 2026: proposal submitted (16 September), defence presented (October), SRS v1.0 written (November). No code and no repository exist according to the project record.

## 2. Baseline

### 2.1 Confirmed requirements (proposal and SRS)

| Ref | Requirement relevant to the Intent Engine |
|---|---|
| Objective 2 | Local interaction pipeline for voice, natural-language understanding, and spoken or textual responses |
| Objective 9 | Evaluate for accuracy, security, recoverability, performance |
| FR-01 | Accept typed requests and push-to-talk spoken requests |
| FR-02 | Local speech-to-text; show the transcript |
| FR-03 | Interpret English, Urdu (Roman or Urdu script) and mixed requests into one or more candidate operations in the fixed format, using a local LLM |
| FR-04 | Validate every candidate against schema and supported operation types; model output never reaches a shell |
| FR-05 | Ask a clarification question, with choices where possible, when ambiguous, low confidence or several targets match |
| FR-06 | Results shown visually and, unless muted, spoken |
| FR-16, FR-17 | Project-level and code-structure requests resolved and shown before execution |
| FR-18 | Risk level assigned by deterministic rules, not the LLM |
| FR-22 | File content, file names, READMEs and comments are data; operations from them need explicit confirmation |
| FR-37 | Simple User Mode and Developer Mode |
| NFR-01 | Typed end-to-end p50 ≤ 5 s, p95 ≤ 10 s on reference hardware, offline; voice adds ≤ 3 s at p50 |
| NFR-03 | Exact match ≥ 85% English, ≥ 70% Urdu and mixed; wrong-action rate ≤ 2% |
| NFR-05 | All SCAR processes ≤ 6 GB peak on an 8 GB machine |
| NFR-06, NFR-19 | Fully offline; no telemetry |
| NFR-11 | Fail safe: on error, timeout or uncertainty, deny or ask, never act |
| NFR-14 | The LLM only proposes structured operations |
| NFR-21 | Local IPC accepts requests only from the user's own SCAR processes |
| SRS §3.2 | Candidate operation format; twelve operation types; the set may grow in the SDS |
| SRS §2.3 | Out of scope: free clicking through applications, cloud AI, browser, defending the model against jailbreaks, perfect speech recognition |

### 2.2 Stated in the defence material (speaker script, evaluation appendix, Q&A bank)

- Grammar-constrained JSON output with llama.cpp; few-shot prompting; reasoning mode off.
- Candidate model about 4B parameters at 4 bits (about 2.5 GB); chosen by measurement in proposal Phase 3; replaceable.
- Baseline: a keyword and regex rule matcher, one rule set per operation.
- Test set: 120 typed commands (40 English, 40 Roman Urdu, 40 code-switched), 15 ambiguous, 20 developer requests, 60 spoken by 6 speakers.
- Accuracy for description arguments is judged by the resolved file, not the wording.
- Multi-step commands: a short list of operations grouped under one Task ID.
- Only the current task state is kept as context; the audit log is not fed to the model.
- No WER target for Urdu or mixed speech; judged on end-to-end intent accuracy.
- Minimum viable product named at the defence: typed English and Roman-Urdu commands, semantic search, policy with confirmations, task rollback.
- Integrity rules: test sets and targets frozen before the first measured run; every run logged; missed targets reported as missed.
- Reference machine: 8 GB RAM development laptop.

### 2.3 Technical recommendations in this plan (not yet approved)

See `docs/TECHNICAL_DECISIONS.md`. In short: Python; model in a separate sandboxed process; rules first and model second; one model call per request and no agent loop; engine-computed confidence; a result envelope around the SRS operation; templates for responses; untrusted text never in the interpreter prompt; development and test data separated from the first day; build as a portable library first and a Linux service second.

### 2.4 New possibilities that need the team's approval

| ID | Item | Why approval is needed |
|---|---|---|
| D-01 | This roadmap covers the Intent Engine workstream, with minimal stand-ins for grounding, policy and execution | Decides what the other components must plan separately |
| D-02 | Schedule variant and start date (section 6) | Time available is not recorded |
| D-03 | Rule matcher used as a fast path in the running system, not only as a baseline | FR-03 says "using a local LLM" |
| D-04 | Result envelope with `proposed`, `clarify`, `unsupported`, `rejected`, `control`; `ref` targets; `origin` field | Extends SRS §3.2 |
| D-05 | Control commands (undo, cancel, switch mode, pick result) handled by rules | Not in the twelve operations |
| D-06 | Thirteenth operation `find_symbol` | FR-17 needs it; the pre-registered test covers twelve |
| D-07 | `confidence` computed by the engine | The SRS leaves its source undefined |
| D-08 | High-risk approval by on-screen action only; no spoken "yes" | Not stated in the SRS |
| D-09 | Isolated summary call; SDS wording for UC-05 | UC-05 describes a weaker flow |
| D-10 | Extra test sets (unsupported, multi-operation, context, negation, Urdu script, control), reported without targets | Adds to the pre-registered plan |
| D-11 | JSON-RPC over Unix sockets between services | Tech-stack review said D-Bus; open for the SDS |

### 2.5 Unknowns

Listed in full in `docs/PROJECT_RISKS.md` section 3. The ones that block scheduling: final deadline and calendar (U-01), weekly capacity and who works on the engine (U-02), exact reference laptop specification (U-03), whether any code already exists (U-04), and whether a Linux environment is ready (U-07).

## 3. Confirmed objectives for the Intent Engine

Derived from section 2.1 and unchanged from the proposal:

1. Interpret typed and spoken requests in English, Urdu and mixed language into validated candidate operations, locally and offline.
2. Ask instead of guessing; keep wrong actions at or below the SRS target.
3. Never let model output become a command or a permission.
4. Meet the SRS accuracy, latency and memory targets on the reference laptop, or report clearly where it does not.
5. Deliver a reusable English, Urdu and mixed test kit with baselines.

## 4. Scope and exclusions

**In scope for this workstream:** the engine itself (normalisation, rules, model interpretation, validation, decision, session references, clarification, responses), its contracts with neighbouring components, its evaluation kit and datasets, stand-in neighbours for early demonstration, the engine's service wrapper, and its security tests.

**Out of scope for this workstream, in scope for SCAR OS:** speech-to-text and text-to-speech services, semantic index and search, developer project model, Policy Engine, Task Engine, checkpoints and rollback, audit log, Private Space enforcement, SCAR Shell, bootable image. The engine integrates with each through a contract.

**Excluded from SCAR OS (proposal §3.6, SRS §2.3) and from this plan:** agent loops and general planners, screen clicking and browser automation, cloud models, run-time tool discovery, long-term memory, wake word, model training as a planned activity.

## 5. Architecture overview

```
User -> SCAR Shell -> [Voice service] -> INTENT ENGINE -> Grounding -> Policy Engine -> Task Engine -> Linux
                                             |  ^                                           |
                                       Model runtime                                  result events
                                      (separate process,                                    |
                                       no file access)        response to Shell <-----------+
```

Inside the engine:

```
1 Receive -> 2 Normalise -> 3 Rules (fast path) --match--> 5 Validate -> 6 Resolve references -> 7 Decide
                                   |                            ^                                   |
                                   +--no match--> 4 LLM --------+               proposed / clarify / unsupported / rejected
                                                  (constrained JSON, one call)
```

Only step 4 uses the language model. Full description: `docs/INTENT_ENGINE_ARCHITECTURE.md`.

## 6. Schedule assumptions and variants

The time available is not recorded anywhere in the project documents, so the schedule is built on stated assumptions.

**Assumptions**

- A1. Implementation starts in the week after approval. Dates below assume a start on Monday 16 November 2026 and move one-for-one with the real start.
- A2. The engine must be complete, evaluated and integrated before the final FYP evaluation, assumed to fall in the spring 2027 term. The actual date is unknown (U-01).
- A3. Work estimate for P0 and P1 features is about 114 person-days, plus about 20 for datasets, recordings and evaluation runs (see the feature specification). These are unmeasured planning estimates.
- A4. Other SCAR components are built in parallel by the team against the contracts frozen in Phase 1. Until they exist, stand-ins are used.
- A5. A Linux environment is available by Phase 6.
- A6. Two buffer weeks are held for examinations and slippage; their position depends on the university calendar.

**Variants**

| Variant | Calendar | Capacity needed on this workstream | Scope |
|---|---|---|---|
| **A. Recommended** | 16 working weeks + 2 buffer (finish about 19 March 2027 if started 16 November) | About 45 to 50 person-hours per week (for example three members at 16 hours, or two at 24) | All P0 and P1 features; P2 if time allows |
| **B. Reduced capacity** | Same 16 + 2 weeks | About 25 to 30 person-hours per week | P0, plus the P1 features on the demonstration path. Cut in this order: Urdu script (IE-B02), query rewriting (IE-C08), `find_symbol` (IE-C07), correction handling (IE-C06), model unload (part of IE-B07), model sandbox enforced only in the VM demonstration (IE-E05), multi-operation limit lowered to two (IE-B06), voice evaluated in English only (IE-D04). Each cut is reported as "not evaluated". |
| **C. Compressed** | 10 working weeks + 1 buffer | About 45 to 50 person-hours per week | Phases 1 to 3 as planned (6 weeks); Phases 4 and 5 merged into 2 weeks; Phase 6 in 1 week against stand-ins; Phase 8 in 1 week; voice path demonstrated but not evaluated. Not recommended: it leaves no room for the language risk R-01. |

Variant A is used in the rest of this file.

**What can run in parallel**

| Track | Weeks | Can be done by a different member |
|---|---|---|
| T1. Engine core (critical path): contracts, rules, model interpreter, decision, dialogue, service | 1 to 13 | — |
| T2. Evaluation kit and datasets: development set, frozen test set, fixture corpus, metric scripts | 2 to 7, then 15 | Yes. The test set **must** be written by someone not on T1. |
| T3. Voice service (whisper.cpp, TTS) | 8 to 13 | Yes; only the transcript contract is shared |
| T4. Real neighbours (grounding, policy, task engine, Shell) | Any time after week 1 | Yes; against the frozen contracts |

**What must be sequential:** contracts before everything; rule baseline before the model comparison; model selection before threshold calibration; test-set freeze before any measured run; service wrapper before system and security tests; final evaluation after a feature freeze.

## 7. Relation to the nine proposal phases

| Proposal phase | Relation to this roadmap |
|---|---|
| 1 Architecture definition | Roadmap Phase 0 and the documents in `docs/` contribute to it |
| 2 Bootable image, services, IPC | Needed by roadmap Phases 6 and 9; not built here |
| 3 Local LLM, speech, structured intent | **Roadmap Phases 1 to 4 and 7 are the intent part of this phase** |
| 4 Semantic indexing and grounding | Contract in Phase 1; integration in Phase 5 |
| 5 Developer project model | Contract in Phase 1; integration in Phase 5 |
| 6 Policy, sandbox, Private Space | Integration and engine-side security in Phase 6 |
| 7 Task identity, audit, rollback | Integration in Phase 6 |
| 8 SCAR Shell, shared task state | Integration in Phase 9 |
| 9 Evaluation and final report | Roadmap Phase 8 supplies the intent results |

The brief suggested eleven phases. This plan uses ten and reorders them for three reasons: the evaluation kit comes first so that every later step is measured; a rule-based version comes before the model so there is a working system and a baseline early; and permission, execution verification and recovery are integration work for the engine, not build work, because the SRS assigns them to the Policy and Task Engines.

## 8. Phase summary

| Phase | Name | Weeks | Indicative dates (Variant A) | Gate |
|---|---|---|---|---|
| 0 | Baseline and plan | — | to approval | G0: plan approved |
| 1 | Foundations: contracts, catalogue, evaluation kit | 1–2 | 16 Nov – 27 Nov 2026 | G1: contracts v0.1 frozen |
| 2 | Deterministic baseline and walking skeleton | 3–4 | 30 Nov – 11 Dec | G2: end-to-end dry run; baseline measured |
| 3 | LLM interpreter and model selection | 5–7 | 14 Dec – 1 Jan 2027 | G3: go / no-go on model; test set frozen |
| 4 | Multilingual robustness and clarification | 8–9 | 4 Jan – 15 Jan | G4: language and abstention behaviour measured |
| 5 | Context, grounding handshake, multi-operation, developer requests | 10–11 | 18 Jan – 29 Jan | G5: two-turn and multi-step scenarios pass |
| 6 | Service, integration with policy and task engine, security | 12–13 | 1 Feb – 12 Feb | G6: security tests pass; feature freeze for P1 |
| 7 | Voice path | 14 | 15 Feb – 19 Feb | G7: spoken requests work end to end |
| 8 | Evaluation and hardening | 15 | 22 Feb – 26 Feb | G8: frozen test run reported |
| 9 | Integration, demonstration, documentation | 16 | 1 Mar – 5 Mar | G9: demonstrated on the image; v1.0 tagged |
| — | Buffer | 2 | placed around examinations | — |

---

## 9. Phases and weekly milestones

### Phase 0: Baseline and plan

- **Objective:** Agree what is being built, in what order, and how it will be measured.
- **Deliverables:** this file and the five documents in `docs/`.
- **Completion criteria:** team approval of sections 2.4 and 6; the blocking unknowns answered.
- **Risks:** R-13, R-19.

Tasks

- [x] Review the proposal, SRS, defence package, tech-stack review, research report and reference review (12 Nov 2026)
- [x] Write architecture, feature specification, technical decisions, testing strategy and risks documents (12 Nov 2026; drafts in `docs/`)
- [x] Write this milestone file (12 Nov 2026; draft)
- [ ] Team reviews the documents
- [ ] Decisions D-01 to D-11 recorded as accepted, changed or rejected
- [ ] Unknowns U-01, U-02, U-03, U-04, U-07 answered and written into section 6
- [ ] Supervisor informed of the plan
- [ ] **Gate G0:** approval to start Phase 1

---

### Phase 1: Foundations (weeks 1–2)

| | |
|---|---|
| Objective | Make every later step measurable and every interface explicit before any interpretation logic is written |
| Features | IE-A01, IE-A02, IE-A03, IE-A09, IE-A10, IE-E01, IE-E07 |
| Approach | Schemas as code (Pydantic) exported to JSON Schema; catalogue as one YAML file; evaluation kit that runs any interpreter behind one function signature; a dummy interpreter proves the kit works |
| Dependencies | G0; a repository; Python environment |
| Deliverables | Repository skeleton with CI; contracts v0.1; catalogue v0.1; evaluation kit; development set v1; fixture corpus v1; reference laptop specification |
| Testing | Unit tests for schemas, catalogue and metrics; CI green on a clean clone |
| Completion criteria | Gate G1 |
| Duration | 2 weeks |
| Risks | R-12 (contracts not agreed with other components), R-15, R-07 (test-set author must be chosen now) |

#### Week 1: Repository, contracts and catalogue

- **Objective:** A repository where the SRS candidate operation is a tested type.
- **Depends on:** G0.
- [ ] Create the repository and directory layout (`docs/TECHNICAL_DECISIONS.md` §3.1)
- [ ] Set up Python project, `ruff`, `mypy`, `pytest`, and CI running them on every push
- [ ] Record the reference laptop specification in `docs/REFERENCE_HARDWARE.md` (answers U-03)
- [ ] Write catalogue v0.1 with the twelve SRS operations: slots, required slots, mode, one-line meaning
- [ ] Implement request, candidate operation and result envelope schemas; export JSON Schema
- [ ] Unit tests: the SRS §3.2 example validates; at least 20 invalid examples are rejected with clear errors
- [ ] `scar-intent` command that validates a JSON file against the schema
- [ ] Static no-shell check in CI (IE-E01)
- [ ] Trace record format and writer (IE-A10)
- [ ] Share contracts v0.1 with the owners of grounding, policy, task engine and Shell for comment
- **Expected output:** `intent_engine` package with `contracts` and `catalogue`; CI passing.
- **Required tests:** schema and catalogue unit tests; static check.
- **Definition of Done:** a clean clone passes CI; contracts reviewed by at least one member responsible for a neighbouring component.

#### Week 2: Evaluation kit and development data

- **Objective:** Be able to score any interpreter, with intervals, before one exists.
- **Depends on:** week 1 contracts.
- [ ] Define the gold label format (JSONL) and write a one-page labelling guide (description arguments scored by resolved file)
- [ ] Build fixture corpus v1 (about 60 files, 2 small sample repositories) with stable identifiers
- [ ] Write development set v1: about 60 typed items (20 English, 20 Roman Urdu, 20 code-switched) across the twelve operations, plus 5 ambiguous
- [ ] Implement metrics: exact match, operation accuracy, argument accuracy, wrong-action rate, clarification rate, Wilson intervals
- [ ] Unit-test each metric against hand-computed cases
- [ ] Implement the runner: configuration in, per-item CSV and summary out, never overwriting earlier reports
- [ ] Run the kit with a dummy interpreter (always answers `unsupported`) and commit the report
- [ ] Name the test-set author (not working on rules or prompt) and agree the freeze date (end of week 7)
- [ ] **Gate G1:** contracts v0.1 frozen and tagged
- **Expected output:** `evalkit` with datasets, metrics, runner and a first report.
- **Required tests:** metric unit tests; runner integration test.
- **Definition of Done:** the dummy run produces the expected scores (0% exact match, 0% wrong actions); two members label five items independently and agree.

---

### Phase 2: Deterministic baseline and walking skeleton (weeks 3–4)

| | |
|---|---|
| Objective | A complete request-to-plan path that works with no model, and the first measured baseline |
| Features | IE-A04, IE-A05, IE-A11 |
| Approach | Template matcher that fires only on a full match; stand-in grounder, risk table and dry-run executor in a separate `reference` package |
| Dependencies | G1 |
| Deliverables | Normaliser; rule matcher for English and Roman Urdu; stand-ins; baseline report on the development set |
| Testing | Unit test per template; integration tests on the fixture corpus; development-set run |
| Completion criteria | Gate G2 |
| Duration | 2 weeks |
| Risks | R-14 (rules growing into a second engine: cap effort at the estimate), R-01 first evidence |

#### Week 3: Normaliser and English rules

- **Objective:** English requests for all twelve operations handled by rules where phrasing is simple.
- **Depends on:** week 1 contracts, week 2 kit.
- [ ] Normaliser: Unicode NFC, whitespace, case, script detection, language tag, length limit
- [ ] Property test: normalising twice equals normalising once
- [ ] Template format and matcher (full match only; all required slots present)
- [ ] English templates for the twelve operations
- [ ] Unit test per template, including near-miss inputs that must **not** match
- [ ] Development-set run, English subset; commit the report
- **Expected output:** `normalise` and `rules` modules; first accuracy and precision figures for rules on English.
- **Required tests:** normaliser and template unit tests; development run.
- **Definition of Done:** no partial match produces an operation in the unit tests; report committed.

#### Week 4: Roman-Urdu rules and end-to-end dry run

- **Objective:** First end-to-end demonstration, with nothing changed on disk.
- **Depends on:** week 3.
- [ ] Roman-Urdu and code-switched templates for common phrasings; first spelling-variant list
- [ ] Stand-in grounder (name and keyword match over the fixture corpus; returns resolved, ambiguous or not found)
- [ ] Stand-in risk table for the twelve operations (levels from SRS Appendix A)
- [ ] Dry-run executor that prints the plan and changes nothing
- [ ] Wire request → normalise → rules → validate → ground → risk → dry-run in `scar-intent`
- [ ] Integration tests for the three grounding outcomes
- [ ] Full development-set run with rules only; commit as the **baseline report**
- [ ] Demonstration D1 rehearsed (`docs/TESTING_STRATEGY.md` §9)
- [ ] **Gate G2:** end-to-end dry run works; baseline recorded
- **Expected output:** a typed request produces a risk-labelled plan on the fixture corpus; baseline numbers by language form.
- **Required tests:** integration tests; development run.
- **Definition of Done:** the three example requests from the SRS use cases run end to end in dry-run; baseline report committed with intervals.

---

### Phase 3: LLM interpreter and model selection (weeks 5–7)

| | |
|---|---|
| Objective | Interpret open phrasing with a local model, choose the model by measurement, and decide whether the approach holds on the reference laptop |
| Features | IE-A06, IE-A07, IE-A08, IE-B04, IE-B07 (first part), IE-E02, start of IE-B03 |
| Approach | llama.cpp server as a separate process; grammar generated from the JSON Schema; prompt generated from the catalogue; few-shot examples from the development set; temperature 0; one retry on invalid output |
| Dependencies | G2; llama.cpp built; candidate model files downloaded |
| Deliverables | `llm`, `validate`, `decide` modules; model comparison report; latency and memory measurements; three-way comparison (rules, model, hybrid); frozen test set |
| Testing | Fake-model integration tests; development-set runs per candidate; performance script |
| Completion criteria | Gate G3 |
| Duration | 3 weeks |
| Risks | R-02, R-03, R-01, R-05, R-10. This phase carries the largest technical risk in the plan, which is why it is early. |

#### Week 5: Model runtime and first constrained output

- **Objective:** A valid candidate operation from the model on the reference laptop, with timings.
- **Depends on:** week 1 schemas.
- [ ] Build or install llama.cpp; record version; write `models/MANIFEST` with file hashes
- [ ] Refresh the list of candidate small open models and their licences (TD-14); pick two or three
- [ ] Runtime client with wall-clock timeout, health check and error mapping
- [ ] Generate the grammar from the result schema; confirm output always parses
- [ ] First prompt, generated from the catalogue
- [ ] Measure tokens per second, time to first token, load time and resident memory for each candidate (answers T-01)
- [ ] Confirm the runtime can be limited to a local socket or loopback and makes no outbound connection (T-05)
- [ ] Fake model for tests: replays recorded outputs, including invalid, truncated and slow ones
- [ ] Set up a Fedora VM or other Linux environment for later phases (answers U-07)
- **Expected output:** `llm` module; a measurements table for the candidates.
- **Required tests:** client unit tests with the fake model; timeout test.
- **Definition of Done:** twenty development items produce schema-valid output from each candidate; measurements committed with the laptop specification.

#### Week 6: Prompt design, validation and model comparison

- **Objective:** Choose the model and quantisation on evidence.
- **Depends on:** week 5.
- [ ] Few-shot examples per language form from the development set; prompt version recorded
- [ ] Validator with catalogue, mode and operation-count checks; one retry; `rejected` after two failures
- [ ] Property-based tests: arbitrary JSON never passes as an operation outside the catalogue
- [ ] Prompt hygiene test (IE-E02): no untrusted string can reach the prompt builder
- [ ] Grow the development set to about 90 typed items plus 10 ambiguous and 10 developer items
- [ ] Run every candidate (model × quantisation) on the development set; record exact match, wrong-action rate, invalid-output rate, latency, memory
- [ ] Select the model; record the decision and the evidence under TD-14
- **Expected output:** `validate` module; model comparison report; a selected model.
- **Required tests:** validator unit and property tests; development runs.
- **Definition of Done:** comparison report committed; decision recorded with the numbers that justify it.

#### Week 7: Hybrid routing, basic abstention, and the go / no-go

- **Objective:** The first working version of the Intent Engine, and a decision on whether the plan stands.
- **Depends on:** weeks 4 and 6.
- [ ] Routing: rules first, model otherwise (IE-B04)
- [ ] Basic decision: missing required slot → `clarify`; no supported operation → `unsupported` (IE-A08)
- [ ] Three-way comparison on the development set: rules only, model only, hybrid
- [ ] Performance script: warm-up plus 10 runs per command for 20 development commands; engine p50 and p95 by path
- [ ] Propose engine-level targets from these measurements (`docs/TESTING_STRATEGY.md` §4.2) and record them here
- [ ] Test set complete, reviewed for leakage against the development set, and frozen by committed hash
- [ ] Demonstration D11 rehearsed (live evaluation run)
- [ ] **Gate G3: go / no-go review.** Compare development results with the SRS targets and the latency budget. If the gap is large, choose from the fallbacks under R-01 and R-02 and record the choice in the change log before continuing.
- **Expected output:** first working version: typed English and Roman-Urdu requests to validated candidates, dry-run plan, measured.
- **Required tests:** routing tests; development runs; performance script.
- **Definition of Done:** three-way report and performance report committed; test-set hash committed; G3 decision written into this file.

---

### Phase 4: Multilingual robustness and clarification (weeks 8–9)

| | |
|---|---|
| Objective | Close the gap between English and Urdu forms, and make "ask, do not guess" work in practice |
| Features | IE-B01, IE-B02, IE-B03, IE-B05, IE-B08 |
| Approach | Error analysis of development failures by category; spelling lexicon and examples for the largest categories; confidence from observable signals; thresholds from a risk-coverage curve; a small state machine for one pending question |
| Dependencies | G3 |
| Deliverables | Lexicon; calibrated thresholds; clarification dialogue; response templates; per-language report |
| Testing | Development runs by language; ambiguous items; state-machine unit tests |
| Completion criteria | Gate G4 |
| Duration | 2 weeks |
| Risks | R-01, R-04, R-06 |

#### Week 8: Roman Urdu and code-switched input

- **Objective:** Measured improvement on the Urdu forms, with causes understood.
- **Depends on:** week 7 reports.
- [ ] Categorise every development failure (spelling variant, word order, verb form, wrong operation, wrong slot, other)
- [ ] Spelling-variant lexicon and normaliser support for the top categories
- [ ] Additional few-shot examples and rules where the analysis points to them
- [ ] Urdu-script requests routed to the model path; small exploratory set written (if D-10 accepted)
- [ ] Development run by language form; compare with week 7
- [ ] Start recruiting speakers and prepare the consent form for Phase 7
- **Expected output:** lexicon; per-language report with before and after.
- **Required tests:** normaliser tests for every lexicon entry; development run.
- **Definition of Done:** every change is tied to a failure category; report committed; no English regression.

#### Week 9: Confidence, clarification dialogue and responses

- **Objective:** The engine asks the right question at the right time and answers in plain words.
- **Depends on:** week 8.
- [ ] Engine-computed confidence from the signals in architecture §8 (check T-06 for token probabilities)
- [ ] Risk-coverage curve on the development set; choose thresholds; freeze them (answers T-08)
- [ ] Clarification state machine: one pending question, choices, reply matching, one re-ask, cancel
- [ ] Response templates for every status and operation, with short speech text
- [ ] Unit tests for every state transition and template
- [ ] Development run including ambiguous items; record clarification and unnecessary-question rates
- [ ] Demonstrations D2 and D3 rehearsed
- [ ] **Gate G4**
- **Expected output:** `decide`, `dialogue`, `compose` modules; calibration report.
- **Required tests:** state-machine tests; template coverage test; development run.
- **Definition of Done:** thresholds recorded with the curve that justifies them; every status and operation has a message.

---

### Phase 5: Context, grounding handshake, multi-operation and developer requests (weeks 10–11)

| | |
|---|---|
| Objective | Handle the two-turn and multi-step requests that appear in the SRS use cases |
| Features | IE-C01, IE-C02, IE-C03, IE-B06, IE-C04, IE-C05; IE-C07 if D-06 accepted |
| Approach | Session state holds identifiers only; references are resolved by code; the grammar allows a short ordered list; developer requests carry a project description resolved by the project model or its stand-in |
| Dependencies | G4; grounding contract; sample repositories in the fixture corpus |
| Deliverables | Session and reference handling; grounding handshake; multi-operation support; developer and control commands |
| Testing | Context, multi-operation and control sets on the development side; developer items |
| Completion criteria | Gate G5 |
| Duration | 2 weeks |
| Risks | R-08 (names leaking into prompts), R-12 (real grounding not ready), R-14 |

#### Week 10: Session references and grounding handshake

- **Objective:** "Find ..., open the second one" works.
- **Depends on:** week 9 dialogue.
- [ ] Session state: pending request, last results, last target, last task, mode; expiry
- [ ] `ref` targets in the schema and grammar; resolver; "no list" case asks
- [ ] Typed context block for the prompt (IE-C02); extend the prompt hygiene test
- [ ] Grounding handshake: resolved, ambiguous (choice question), not found
- [ ] Integrate with the real semantic layer if available; otherwise the stand-in
- [ ] Development-side context set; unit and integration tests
- [ ] Demonstration D4 rehearsed
- **Expected output:** two-turn scenarios pass on the fixture corpus.
- **Required tests:** reference resolver tests; handshake integration tests.
- **Definition of Done:** hostile file names in session state do not appear in any prompt; context set report committed.

#### Week 11: Multi-operation, developer and control commands

- **Objective:** One sentence, several steps; developer requests; fixed commands.
- **Depends on:** week 10.
- [ ] Ordered operation list in grammar and validator; limit enforced
- [ ] Multi-operation development items; check order and arguments
- [ ] Developer requests: `run_tests`, `git_status`, `open_project` with project description; five sample repositories
- [ ] Control commands by rules: undo last task, cancel, switch mode, pick result, yes or no
- [ ] Mode awareness: developer operations in Simple User Mode suggest switching (IE-D05)
- [ ] `find_symbol` catalogue entry and small separate set (only if D-06 accepted)
- [ ] Development run across all sets; no regression on the twelve-operation items
- [ ] **Gate G5**
- **Expected output:** multi-step and developer requests produce correct ordered candidates.
- **Required tests:** multi-operation, developer and control unit and development tests.
- **Definition of Done:** scenarios for UC-02, UC-03 and UC-04 pass in dry-run; reports committed.

---

### Phase 6: Service, integration and security (weeks 12–13)

| | |
|---|---|
| Objective | Run the engine as a SCAR service connected to the real policy and task components, and prove its security properties |
| Features | IE-D01, IE-D02, IE-D03, IE-D06, IE-D07, IE-E03, IE-E04, IE-E05, IE-E06, rest of IE-B07 |
| Approach | JSON-RPC over a Unix socket with peer check; systemd user units for the engine and the model runtime; operating-system restrictions on the model process; isolated summary call; fault injection |
| Dependencies | G5; Linux environment; Policy Engine and Task Engine at least in minimal form (otherwise stand-ins, stated in reports) |
| Deliverables | Engine service; integration with policy, task engine and audit; security and fault-injection reports |
| Testing | Contract tests; system tests per use case; security tests; fault injection |
| Completion criteria | Gate G6 and feature freeze for P1 |
| Duration | 2 weeks |
| Risks | R-12, R-16, R-08, R-15 |

#### Week 12: Service and integration

- **Objective:** A request travels through real IPC to policy and task handling and back.
- **Depends on:** week 11; Linux environment.
- [ ] JSON-RPC server with the five methods in architecture §6.6; owner-only socket; peer credential check
- [ ] systemd user units for engine and model runtime; restart on failure; warm-up
- [ ] Policy handoff with `origin`; blocked events rendered as plain messages
- [ ] Task result events update session state and produce completion messages
- [ ] Audit events for request, candidates, validation outcome and decision
- [ ] Shell message contract agreed and covered by shared contract tests
- [ ] System tests for UC-01, UC-02, UC-03, UC-04, UC-06, UC-08 (dry-run where a real component is missing; say which)
- [ ] Demonstrations D5 and D6 rehearsed
- **Expected output:** `service` module; engine running under systemd; system test report.
- **Required tests:** contract tests per method; foreign-caller rejection; restart test.
- **Definition of Done:** each use-case scenario passes through IPC; the report states which neighbours were real and which were stand-ins.

#### Week 13: Security and fail-safe behaviour

- **Objective:** Evidence that the engine cannot be turned into a command channel and fails safely.
- **Depends on:** week 12.
- [ ] Isolated summary call with display-text-only output (IE-E03)
- [ ] Run the injected-instruction files through the engine; count operations that originate from content (target: none)
- [ ] Restrict the model runtime: no home-directory access except the model file; no network; verify by attempting both
- [ ] Fault injection: kill, hang, garbage, truncated output, trace disk full; each ends in ask or decline
- [ ] Input limits: request length, operation count, one pending question, one model call at a time
- [ ] Malformed and oversized input tests; model-output fuzzing through the fake model
- [ ] Optional model unload when idle; measure memory before and after
- [ ] Performance script re-run through the service
- [ ] Demonstrations D7, D8 and D10 rehearsed
- [ ] **Gate G6:** security report reviewed; P1 feature freeze
- **Expected output:** security and fault-injection reports.
- **Required tests:** `docs/TESTING_STRATEGY.md` §6.6 and §6.7.
- **Definition of Done:** every security test has a recorded outcome; any failure is fixed or listed as a known limitation.

---

### Phase 7: Voice path (week 14)

| | |
|---|---|
| Objective | Spoken requests reach the engine and are handled with the same safeguards as typed ones |
| Features | IE-D04 |
| Approach | The voice service supplies transcript, language and confidence; low confidence asks the user to confirm or correct the transcript before interpretation |
| Dependencies | G6; voice service (track T3); speakers and consent |
| Deliverables | Voice integration; recorded spoken test items; typed-versus-spoken development comparison |
| Testing | End-to-end spoken development items; WER by language form |
| Completion criteria | Gate G7 |
| Duration | 1 week for integration; recording may start earlier |
| Risks | R-09, R-18, R-11 |

#### Week 14: Voice integration and recordings

- **Objective:** Push-to-talk request to validated candidate, transcript visible.
- **Depends on:** week 12 service; voice service.
- [ ] Accept transcript with ASR metadata; lower confidence on poor ASR; transcript confirmation step
- [ ] Short speech text sent to the TTS service; mute respected
- [ ] Licence check for the TTS engine and voices before they go into the image
- [ ] Record the 60 spoken test items with 6 speakers, with consent; store hashes; audio stays out of the repository
- [ ] Development-side spoken items (separate from the 60) for tuning the confirmation threshold
- [ ] Typed-versus-spoken comparison on development items; WER with `jiwer`
- [ ] P2 items if time allows: correction handling (IE-C06), query rewriting (IE-C08)
- [ ] Demonstration D9 rehearsed
- [ ] **Gate G7**
- **Expected output:** working voice path; recordings ready for the frozen run.
- **Required tests:** spoken development items; transcript-confirmation tests.
- **Definition of Done:** a mixed-language spoken request completes end to end; recordings complete or the shortfall is recorded.

---

### Phase 8: Evaluation and hardening (week 15)

| | |
|---|---|
| Objective | Produce the measured results for the Intent Engine under the pre-registered rules |
| Features | IE-E08; no new features |
| Approach | Code freeze; one measured run of the frozen test set per configuration; performance and soak runs; every run logged |
| Dependencies | G7; frozen test set; reference laptop reserved |
| Deliverables | Evaluation report with targets, results, intervals and pass, miss or not evaluated for each |
| Testing | `docs/TESTING_STRATEGY.md` §2.1, §5, §6.5, §6.7 |
| Completion criteria | Gate G8 |
| Duration | 1 week |
| Risks | R-04, R-07, R-17 |

#### Week 15: Frozen run

- **Objective:** Results that can be defended.
- **Depends on:** week 14.
- [ ] Code, prompt, thresholds, catalogue and model frozen and tagged; hashes recorded
- [ ] Verify the test-set hash against the week 7 freeze
- [ ] Typed test run: rules only, model only, hybrid (120 + 15 ambiguous + 20 developer)
- [ ] Spoken test run (60 items); typed-versus-spoken comparison; WER by language form
- [ ] Additional sets (if D-10 accepted), reported separately
- [ ] Performance: 20 commands × 10 runs; engine and end-to-end p50 and p95; peak memory; one run with networking disabled
- [ ] Soak run; record errors, restarts and memory
- [ ] Write the evaluation report: every target with result, interval and status; list of failed items; limitations
- [ ] Any defect found is logged; a fix after the frozen run requires a new tagged run, reported alongside the first
- [ ] **Gate G8**
- **Expected output:** evaluation report and raw per-item results.
- **Required tests:** as listed.
- **Definition of Done:** report committed with raw data; no result altered or omitted; missed targets stated as missed.

---

### Phase 9: Integration, demonstration and documentation (week 16)

| | |
|---|---|
| Objective | The engine working inside the SCAR OS prototype, with a rehearsed demonstration and complete documents |
| Features | Integration of all P0 and P1 features with the Shell and image |
| Approach | Install the services into the prototype image; rehearse the scenarios with fallbacks; update documents to match what was built |
| Dependencies | G8; SCAR Shell; bootable image (proposal Phases 2 and 8) |
| Deliverables | Engine on the image; demonstration script; user guide section on voice and text commands; inputs for the SDS and final report; v1.0 tag |
| Testing | System tests on the image; demonstration rehearsals |
| Completion criteria | Gate G9 |
| Duration | 1 week |
| Risks | R-12 (image or Shell late) |

#### Week 16: Integration and demonstration

- **Objective:** A demonstration that works twice in a row on the target machine.
- **Depends on:** week 15; Shell and image.
- [ ] Engine and model runtime services installed and started on the prototype image
- [ ] Shell shows proposals, questions, progress and results from the engine
- [ ] System tests repeated on the image
- [ ] Demonstration script for D1 to D11 with a fallback for each live step; two full rehearsals
- [ ] Voice Command Guidelines section (SRS §5.4): example requests in three language forms, how clarification works
- [ ] Generated list of supported commands from the catalogue
- [ ] Architecture and decision documents updated to match the implementation
- [ ] Intent Engine section for the final report: design, results, limitations
- [ ] Tag v1.0
- [ ] **Gate G9**
- **Expected output:** integrated prototype; documents; tag.
- **Required tests:** system tests on the image; rehearsals.
- **Definition of Done:** two consecutive clean rehearsals; documents match the code; all boxes in this file either ticked with evidence or marked "not done" with a reason.

---

## 10. Dependencies between phases

```
G0 -> P1 -> P2 -> P3 -> P4 -> P5 -> P6 -> P7 -> P8 -> P9
             \           ^
              \--(baseline needed for model comparison)

T2 evaluation kit and data: starts in P1, test set frozen at G3, used in P8
T3 voice service:           built in parallel from week 8, joined in P7
T4 real neighbours:         built in parallel against G1 contracts, joined in P5, P6 and P9
```

External dependencies and the latest week each is needed:

| Dependency | Needed by | If late |
|---|---|---|
| Agreed contracts with neighbouring components | Week 2 | Stand-ins continue; integration risk rises |
| llama.cpp and candidate model files | Week 5 | Phase 3 slips day for day |
| Linux environment | Week 12 | Phase 6 slips; library work can continue |
| Policy Engine and Task Engine (minimal) | Week 12 | System tests run against stand-ins and say so |
| Voice service | Week 14 | Voice path demonstrated late or reported as not evaluated |
| Speakers and consent | Week 14 | Fewer speakers, reported |
| SCAR Shell and bootable image | Week 16 | Demonstration from the command line and a VM |

## 11. Deliverables

| # | Deliverable | Gate |
|---|---|---|
| 1 | Planning documents (this file and `docs/`) | G0 |
| 2 | Contracts v0.1, catalogue v0.1, evaluation kit, development set | G1 |
| 3 | Rule-based interpreter, stand-ins, baseline report | G2 |
| 4 | First working Intent Engine (typed, dry-run), model selection report, frozen test set | G3 |
| 5 | Multilingual handling, calibrated abstention, clarification dialogue | G4 |
| 6 | Session references, multi-operation and developer requests | G5 |
| 7 | Engine service, integration, security report | G6 |
| 8 | Voice path and recordings | G7 |
| 9 | Evaluation report with raw data | G8 |
| 10 | Integrated prototype, demonstration script, user and report documentation, v1.0 | G9 |

## 12. Definition of Done

**For a task (a checkbox):** code merged to `main` through review; unit tests written and passing in CI; the listed check performed; the box ticked with a commit or report reference.

**For a feature (IE-xx):** all its tasks done; the tests named in the feature specification pass; its demonstration has been shown to another team member; documentation updated; no open defect that changes its behaviour.

**For a phase (a gate):** every task ticked or explicitly deferred with a reason in the change log; the phase report committed; risks reviewed; the next phase's dependencies confirmed.

**For a measured result:** produced by the evaluation runner on the reference laptop from tagged code; raw per-item output committed; versions and hashes recorded.

A feature is never marked complete because code exists. It is complete when its test has passed and someone other than its author has seen it work.

## 13. Risks

Full register in `docs/PROJECT_RISKS.md`. The five that most threaten this plan:

| ID | Risk | First checked |
|---|---|---|
| R-02 | CPU latency over budget | Week 5 |
| R-01 | Urdu and code-switched accuracy below target | Week 7, then week 8 |
| R-12 | Neighbouring components late | Week 2, then every gate |
| R-13 | Available time unknown | Gate G0 |
| R-04 | Wrong-action rate above 2% | Week 7, then week 9 |

## 14. Progress tracking

Update this table at each gate. Status is one of: Not started, In progress, Done, Blocked, Deferred.

| Phase | Status | Started | Finished | Gate passed | Evidence (tag, report) | Notes |
|---|---|---|---|---|---|---|
| 0 Baseline and plan | In progress | 12 Nov 2026 | | | Draft documents | Waiting for approval |
| 1 Foundations | Not started | | | | | |
| 2 Baseline and skeleton | Not started | | | | | |
| 3 LLM interpreter | Not started | | | | | |
| 4 Multilingual and clarification | Not started | | | | | |
| 5 Context and multi-operation | Not started | | | | | |
| 6 Service, integration, security | Not started | | | | | |
| 7 Voice path | Not started | | | | | |
| 8 Evaluation | Not started | | | | | |
| 9 Integration and demonstration | Not started | | | | | |

Feature status (update when a feature meets its Definition of Done):

| Group | Features | Done |
|---|---|---|
| A. Essential MVP | 11 | 0 |
| B. Core advanced | 8 | 0 |
| C. Intelligence and context | 8 | 0 |
| D. OS integration | 7 | 0 |
| E. Security and reliability | 8 | 0 |

Measured results (filled only from evaluation reports):

| Target | Value required | Development set (latest) | Frozen test set |
|---|---|---|---|
| Exact match, English | ≥ 85% | not measured | not measured |
| Exact match, Urdu and mixed | ≥ 70% | not measured | not measured |
| Wrong-action rate | ≤ 2% | not measured | not measured |
| Ambiguous items that ask | ≥ 10 of 15 | not measured | not measured |
| Developer requests resolved | ≥ 16 of 20 | not measured | not measured |
| Voice end-to-end, English | within 10 points of typed | not measured | not measured |
| End-to-end latency, typed | p50 ≤ 5 s, p95 ≤ 10 s | not measured | not measured |
| Peak memory, SCAR processes | ≤ 6 GB | not measured | not measured |

Open decisions:

| ID | Decision | Status | Date | Outcome |
|---|---|---|---|---|
| D-01 | Roadmap scope: engine workstream with stand-ins | Open | | |
| D-02 | Schedule variant and start date | Open | | |
| D-03 | Rules as fast path | Open | | |
| D-04 | Result envelope, references, origin | Open | | |
| D-05 | Control commands by rules | Open | | |
| D-06 | `find_symbol` operation | Open | | |
| D-07 | Engine-computed confidence | Open | | |
| D-08 | No spoken approval for High | Open | | |
| D-09 | Isolated summary call | Open | | |
| D-10 | Extra test sets | Open | | |
| D-11 | Unix-socket JSON-RPC | Open | | |

## 15. Change log

| Date | Change | Reason | Agreed by |
|---|---|---|---|
| 12 Nov 2026 | First draft of the plan | Planning task | Pending team approval |
