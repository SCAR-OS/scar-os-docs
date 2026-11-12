# SCAR OS Intent Engine: Feature Specification

| | |
|---|---|
| Owner | SCAR-OS Development Team |
| Status | Draft for team approval. No feature below is implemented. |
| Date | 12 November 2026 |

Conventions:

- **Difficulty:** Low, Medium, High, relative to a final-year team.
- **Priority:** P0 needed for the first working version; P1 needed for the full FYP prototype; P2 desirable; P3 future or excluded.
- **Estimate:** person-days of focused work (about six productive hours). These are planning estimates with no measured basis yet; revise them after Phase 1.
- **Phase:** phase in `MILESTONES.md`.
- **Source:** SRS requirement, defence material (DEF), or recommendation (REC).

The list is intentionally short. A feature is included only if it is required by the SRS, needed to measure an SRS target, or needed to make the engine safe to connect to real files.

---

## A. Essential MVP features

| ID | Feature and purpose | How it works | Depends on | Diff. | Pri. | Est. | Tests | Demonstration |
|---|---|---|---|---|---|---|---|---|
| IE-A01 | **Operation catalogue.** One definition of what the engine may propose. (SRS §3.2) | A versioned YAML file: operation name, slots, required slots, mode, one-line meaning, examples per language. Loaded and validated at start. | None | Low | P0 | 2 | Loader unit tests; every SRS operation present; duplicate and unknown fields rejected | Show the file; show the generated command list |
| IE-A02 | **Contracts.** Typed request, candidate operation and result envelope. (SRS §3.2, §3.3) | Pydantic models, JSON Schema export, `schema_version`. | IE-A01 | Low | P0 | 2 | SRS example validates; table of invalid examples rejected; round-trip | Validate a sample by hand on the command line |
| IE-A03 | **Command-line interface.** Lets the engine be used and tested without the Shell. | `scar-intent "<text>"` prints the result; `--explain` prints the trace. | IE-A02 | Low | P0 | 1 | CLI smoke tests | Used in every early demo |
| IE-A04 | **Input normaliser.** Makes equal requests look equal before interpretation. | Unicode NFC, whitespace, case folding for Latin text, script detection, language tag (`en`, `ur-Latn`, `ur-Arab`, `ur-en`), length limit. | IE-A02 | Low | P0 | 2 | Table-driven unit tests; property test: normalising twice changes nothing | Trace shows normalised text and tag |
| IE-A05 | **Rule matcher.** The pre-registered baseline and the fast path. (DEF B3; REC TD-05) | Templates per operation and language form; fires only on a full match with all required slots. | IE-A01, IE-A04 | Medium | P0 | 6 | Unit test per template; precision measured on the development set; no partial-match firing | "list recent files" answered with no model loaded |
| IE-A06 | **LLM interpreter.** Handles open phrasing in three language forms. (SRS FR-03) | One grammar-constrained call to the local model with catalogue, few-shot examples and typed context. | IE-A01, IE-A02, model runtime | High | P0 | 10 | Fake-model integration tests; development-set evaluation; timeout test | Mixed-language move request becomes a valid candidate |
| IE-A07 | **Validator.** Nothing unvalidated leaves the engine. (SRS FR-04) | Schema check, catalogue check, mode check, operation-count limit; one retry, then `rejected`. | IE-A02 | Low | P0 | 2 | Unit tests; property-based tests with random JSON; retry path | Feed a deliberately invalid output through the fake model |
| IE-A08 | **Ask or decline (basic).** Prefer asking over acting wrongly. (SRS FR-05) | Missing required slot gives `clarify`; no matching operation gives `unsupported`. | IE-A06, IE-A07 | Medium | P0 | 3 | Ambiguous and out-of-scope development items | "delete the old ones" produces a question, not a deletion |
| IE-A09 | **Evaluation kit.** Proves, with numbers, what the engine does. (Objective 9; DEF B3) | Gold files in JSONL, metric functions, runner, CSV and summary reports with confidence intervals. | IE-A02 | Medium | P0 | 6 | Metric unit tests against hand-computed cases | Run the kit live; show the report |
| IE-A10 | **Trace logging.** Makes results reproducible and errors explainable. | One structured record per request (architecture §13). | IE-A02 | Low | P0 | 1 | Trace schema test; no file content or audio in traces | `--explain` output |
| IE-A11 | **Stand-in neighbours.** Allows an end-to-end demonstration before other components exist. | Minimal grounder over a fixture folder (name and keyword match), a risk table for the twelve operations, a dry-run executor that prints the plan and changes nothing. Marked clearly as stand-ins. | IE-A02 | Low | P0 | 4 | Integration tests on the fixture corpus | Typed request to printed, risk-labelled plan |

MVP total: about 39 person-days.

## B. Core advanced features

| ID | Feature and purpose | How it works | Depends on | Diff. | Pri. | Est. | Tests | Demonstration |
|---|---|---|---|---|---|---|---|---|
| IE-B01 | **Roman-Urdu and code-switch handling.** The project's language claim. (SRS FR-03, §2.7) | Lexicon of spelling variants ("main/mein/mai", "karo/kro/kardo"), verb and postposition list, few-shot examples per form; optional fuzzy match. | IE-A04, IE-A05, IE-A06 | High | P1 | 6 | Per-language accuracy on the development set; error analysis by category | Same request typed three ways gives the same candidate |
| IE-B02 | **Urdu-script input (exploratory).** FR-03 accepts it; the evaluation plan does not test it. | Script detection routes to the model path; no rules initially. | IE-A06 | Medium | P2 | 2 | Small exploratory set, reported with no target | One Urdu-script request |
| IE-B03 | **Engine-computed confidence and thresholds.** Controls the wrong-action rate. (SRS NFR-03; REC TD-07) | Signals in architecture §8; thresholds tuned on the development set, then frozen. | IE-A05, IE-A06 | Medium | P1 | 4 | Risk-coverage curve on the development set; threshold regression test | Show a low-confidence request turning into a question |
| IE-B04 | **Hybrid routing.** Uses the model only when needed. (REC TD-05) | Rule path first; model path otherwise; disagreement lowers confidence. | IE-A05, IE-A06 | Low | P1 | 2 | Three-way comparison: rules, model, hybrid | Latency difference between a fixed command and an open request |
| IE-B05 | **Clarification dialogue.** Completes a request in one extra turn. (SRS FR-05) | State machine with one pending question, choices, reply matching (number, name, yes/no), one re-ask, then cancel. | IE-A08 | Medium | P1 | 5 | State-machine unit tests; ambiguous set | "Which project?" with two choices, answered by voice or click |
| IE-B06 | **Multi-operation requests.** One request, several steps, one task. (SRS FR-03; DEF Q&A 3.5) | The grammar allows an ordered list (limit five); validated as a whole; passed on together so the Task Engine groups them under one Task ID. | IE-A06, IE-A07 | Medium | P1 | 4 | Multi-step development items; order preserved; limit enforced | "Create a folder called submission and move my FYP documents into it" |
| IE-B07 | **Model service management.** Keeps the engine responsive inside 8 GB. (SRS NFR-05) | Warm-up at start, wall-clock timeout, health check, restart on failure, optional unload when idle. | Model runtime | Medium | P1 | 4 | Kill and timeout tests; memory sampling | Kill the model process; engine reports and recovers |
| IE-B08 | **Response composer.** Short, plain, consistent messages. (SRS FR-06, §2.7 user constraints) | Templates keyed by result status and operation; separate short speech text. | IE-A02 | Low | P1 | 3 | Template coverage test: every status and operation has a message | Read-out of a result |

## C. Intelligence and context features

| ID | Feature and purpose | How it works | Depends on | Diff. | Pri. | Est. | Tests | Demonstration |
|---|---|---|---|---|---|---|---|---|
| IE-C01 | **Session references.** "Open the second one", "move it". (SRS UC-02) | Session keeps identifiers of the last result list, last target and last task; the model or rule emits a `ref`; ordinary code resolves it. | IE-A02, IE-B05 | Medium | P1 | 4 | Context-dependent set; expiry; missing-list case asks | Find, then "open the second one" |
| IE-C02 | **Typed context for the interpreter.** Better interpretation without leaking data. | Mode, result-list length, pending-task flag are given to the model; no names or content. | IE-A06 | Low | P1 | 1 | Prompt-content test: no untrusted strings | Trace shows the context block |
| IE-C03 | **Grounding handshake.** Connects descriptions to real resources. (SRS §3.2, UC-02, UC-04) | Contract in architecture §6.4; ambiguity becomes a choice question. | IE-B05, grounding service or stand-in | Medium | P1 | 4 | Resolved, ambiguous and not-found cases on the fixture corpus | Two matching files produce a choice |
| IE-C04 | **Developer requests.** Objective 4 through the engine. (SRS FR-16, UC-04) | `run_tests`, `git_status`, `open_project` with a `project` description; Developer Project Model resolves repository and command. | IE-A06, Developer Project Model or stand-in | Medium | P1 | 4 | Twenty developer items over five sample repositories [DEF B3] | "Run the backend tests of my FYP project" shows repository and command before approval |
| IE-C05 | **Control commands.** Undo, cancel, switch mode, pick result. (SRS UC-02, UC-06, UC-08; REC TD-13) | Rules only; routed as `control` results. | IE-A05 | Low | P1 | 2 | Unit tests per command and language form | "Undo the last task" |
| IE-C06 | **Correction and negation.** "Don't delete it, move it." (DEF Q&A 3.6) | A correction while a proposal is pending replaces it; negation examples in few-shot; delete stays High so the user still confirms. | IE-B05, IE-C01 | High | P2 | 3 | Negation set (proposed addition) | Correct a pending proposal by voice |
| IE-C07 | **Symbol query operation `find_symbol`.** FR-17 needs an operation. | Thirteenth catalogue entry; grounding is the symbol index. | IE-A01, Developer Project Model | Low | P2 | 2 | Separate small set; not part of the twelve-type test | "Show the function that stores employee passwords" |
| IE-C08 | **Query rewriting for retrieval.** Better search for mixed-language descriptions. (DEF Q&A 1.5, marked new there) | The interpreter also emits an English rewrite of the description for the embedding search. | IE-A06 | Medium | P2 | 2 | Retrieval comparison with and without rewrite | Optional |

## D. OS integration features

| ID | Feature and purpose | How it works | Depends on | Diff. | Pri. | Est. | Tests | Demonstration |
|---|---|---|---|---|---|---|---|---|
| IE-D01 | **IPC service.** The engine as a SCAR background service. (SRS §3.3, NFR-21) | JSON-RPC 2.0 on a Unix socket, owner-only, peer credential check, systemd user unit. | IE-A02; Linux | Medium | P1 | 5 | Contract tests per method; foreign-user request rejected; restart test | `systemctl --user status`; request sent from a script |
| IE-D02 | **Policy Engine handoff.** (SRS FR-18, FR-22) | Candidate plus `origin` and trace id passed on; `blocked` events turned into plain messages. | IE-A02; Policy Engine | Low | P1 | 2 | Integration test with real or stand-in policy | A forbidden request shows the block reason |
| IE-D03 | **Task Engine events.** (SRS FR-24, FR-35) | Result events update session state and produce the completion message. | IE-B08; Task Engine | Low | P1 | 2 | Event contract tests | Completion message with Undo |
| IE-D04 | **Voice path.** (SRS FR-01, FR-02, UC-01) | Accepts transcript with ASR confidence and language; low confidence asks the user to confirm or correct the transcript first. | IE-B03; voice service | Medium | P1 | 4 | Sixty spoken items; typed-versus-spoken comparison [DEF B3] | Push-to-talk mixed-language request |
| IE-D05 | **Mode awareness.** (SRS FR-37, UC-08 1a) | Developer operations are not proposed in Simple User Mode; the engine suggests switching. | IE-A01 | Low | P1 | 1 | Unit tests per operation and mode | Developer request in Simple mode |
| IE-D06 | **Audit events.** (SRS FR-27) | Request text, candidates, validation outcome and decision sent to the audit subsystem. | Audit subsystem | Low | P1 | 2 | Completeness check: one event chain per request | Audit viewer shows the request |
| IE-D07 | **Shell contract.** (SRS FR-34, FR-35) | Stable message shapes for proposal, question, progress and result so the Shell can render them. | IE-A02 | Low | P1 | 2 | Contract tests shared with the Shell | Task panel |

## E. Security and reliability features

| ID | Feature and purpose | How it works | Depends on | Diff. | Pri. | Est. | Tests | Demonstration |
|---|---|---|---|---|---|---|---|---|
| IE-E01 | **No-shell invariant.** (SRS NFR-14) | The engine package contains no process-spawning or file-writing calls outside its own trace directory; a static test fails the build otherwise. | None | Low | P0 | 1 | Static check in CI | Show the test |
| IE-E02 | **Prompt hygiene.** (SRS FR-22; REC TD-09) | The prompt builder accepts only typed, trusted fields; a test asserts no untrusted string can reach it. | IE-A06 | Low | P0 | 1 | Unit test with hostile file names in session state | Trace shows prompt |
| IE-E03 | **Isolated summary call.** (SRS UC-05; REC TD-09) | `summarise_file` content goes to a separate model call whose only output type is display text. | IE-A06; Task Engine reads the file | Medium | P1 | 3 | Forty injected files [DEF B4]: zero operations originate from content | Summarise a README with a hidden instruction |
| IE-E04 | **Fail-safe behaviour.** (SRS NFR-11) | Timeouts, model failure and invalid output all end in ask or decline. | IE-B07 | Medium | P1 | 3 | Fault-injection cases: kill, hang, garbage, truncated output | Kill the model mid-request |
| IE-E05 | **Model process sandbox.** (SRS NFR-15, NFR-17, NFR-19) | The model runtime runs with no home-directory access except the model file and no network. | Linux; Landlock or systemd options | Medium | P1 | 3 | Attempted file read and network connection from the runtime are denied | Show the denial in a test run |
| IE-E06 | **Input limits.** | Maximum request length, maximum operations, one pending question, one in-flight model call. | IE-A04, IE-A07 | Low | P1 | 1 | Boundary tests | None |
| IE-E07 | **Reproducible runs.** (DEF B8) | Pinned model file hash, prompt version, seed, temperature 0; recorded in every trace and report. | IE-A10 | Low | P0 | 1 | Same input twice gives identical output | Re-run one evaluation item |
| IE-E08 | **Soak stability.** | Long run of mixed requests; watch errors and memory. | Phase 8 | Low | P1 | 1 | See `TESTING_STRATEGY.md` | Report |

## F. Experimental and future features

| ID | Feature | Why not now | Pri. |
|---|---|---|---|
| IE-F01 | **LoRA fine-tuning** of the interpreter model | Only if code-switched accuracy stays below target after prompt work; needs a separate synthetic training set and GPU time. [DEF Q&A 1.8] | P3, conditional |
| IE-F02 | **Learning from corrections** | Online learning would make results unrepeatable. A safe version is to log corrections locally and use them to grow the development set by hand. | P3 |
| IE-F03 | **MCP export of the catalogue** | Interoperability only; no FYP requirement. Keep fields compatible. [Tech-stack item 42] | P3 |
| IE-F04 | **Long-term memory and personalisation** | Privacy and evaluation cost; contradicts the stated "current task state only". [DEF Q&A 3.7] | P3 |
| IE-F05 | **General planner or agent loop** | Architecture §7.4. | Excluded |
| IE-F06 | **Whisper prompt biasing with index names** | Belongs to the voice service; also places file names in a model prompt, which needs its own security review. [DEF Q&A 2.x] | P3, voice service |
| IE-F07 | **Urdu spoken responses** | Urdu text generation and TTS quality are untested. English responses meet FR-06. | P3 |
| IE-F08 | **Screen clicking, browser automation** | Out of scope in the proposal (§3.6) and SRS (§2.3). | Excluded |
| IE-F09 | **Wake word, always-on listening** | The SRS requires push-to-talk. | Excluded |
| IE-F10 | **Emotion or frustration detection** | No requirement; noted as out of scope in the defence Q&A. | Excluded |

## Summary by priority

| Priority | Features | Estimate (person-days) |
|---|---|---|
| P0 | IE-A01 to IE-A11, IE-E01, IE-E02, IE-E07 | about 42 |
| P1 | IE-B01, B03 to B08, IE-C01 to C05, IE-D01 to D07, IE-E03 to E06, IE-E08 | about 72 |
| P2 | IE-B02, IE-C06, IE-C07, IE-C08 | about 9 |
| Total P0 to P2 | | about 123 person-days |

The estimate excludes writing the frozen test set, recording speakers, and the final evaluation runs, which are counted in the roadmap. It also excludes the real grounding, policy, task, voice and Shell components, which are separate work.
