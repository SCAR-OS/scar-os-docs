# SCAR OS Intent Engine: Technical Decisions

| | |
|---|---|
| Owner | SCAR-OS Development Team |
| Status | Every decision below is **Proposed**. None is accepted until the team approves it. |
| Date | 12 November 2026 |

Part 1 evaluates the existing technologies and approaches that were considered. Part 2 lists the decisions (TD-xx) that follow. Part 3 gives the engineering standards.

Licence entries marked *(verify)* were not re-checked against the upstream repository for this document and must be confirmed before the component is adopted. Hardware figures are estimates from the project documents, not measurements.

---

## Part 1. Technology and approach evaluation

### 1.1 How other systems turn language into actions

| System or approach | What it does and how | Fit for SCAR OS | Use |
|---|---|---|---|
| **Android Intents, Apple App Intents** | Applications declare typed actions with typed parameters; the assistant can only call what was declared. | The right mental model: a closed, typed catalogue instead of free control. Not usable code for Linux. | Adopt the idea (operation catalogue). |
| **Home Assistant Assist (hassil templates)** | Sentence templates matched locally in milliseconds; a language model is used only when no template matches. Open source. | Same hybrid shape we need, proven on low-power hardware. Its templates are for home devices and European languages, so the content does not transfer. | Adopt the design; write our own matcher. |
| **Rasa, Snips NLU, Mycroft Adapt/Padatious (classifier plus slot filling)** | A trained intent classifier and an entity tagger. Fast and small. | Needs labelled training data per language. We have none for Roman Urdu or code-switched commands, and arguments here are open descriptions ("the slides about Btrfs from week 3"), which taggers handle poorly. Snips and Mycroft are no longer maintained. [DEF Q&A 1.2] | Reject as main interpreter. Our rule matcher covers the "fast and small" role. |
| **Embedding nearest-neighbour intent classification** | Embed the request, compare with example requests per intent. | Could pick the operation cheaply, but does not extract arguments, so a second step is still needed. | Hold as a fallback if the model is too slow (risk R-02). |
| **LLM function calling / tool calling** | The model is given tool schemas and emits a structured call. | This is what stage 4 does. The difference is that our output is a *proposal* that passes policy, not a call that runs. | Adopt, with constrained decoding. |
| **Grammar-constrained decoding (llama.cpp GBNF, JSON Schema to grammar)** | Token sampling is restricted so output always parses against a grammar. | Removes malformed JSON, the main failure of small models, at no extra dependency. Already the stated plan. [DEF Q&A 3.2] | Adopt. |
| **Outlines, Guidance, Instructor (structured-output libraries)** | Python libraries that constrain or validate model output. | Duplicate what llama.cpp already does natively; add dependencies. | Reject unless the native grammar proves insufficient. |
| **Agent loops (ReAct and similar), LangChain, LangGraph, AutoGen** | The model alternates reasoning, tool calls and reading results until done. Frameworks package this. | Slow on CPU, unreliable with small models, and the loop is the injection route. Frameworks hide the control flow that we must be able to defend line by line in a viva. | Reject for the core. See architecture §7.4. |
| **Classical planners (PDDL, LLM+P)** | A symbolic planner searches for an action sequence from a formal domain model. | Our requests are one to five steps. A planner adds a formal domain model to maintain for no gain at this size. | Reject. |
| **AIOS (agiresearch)** | A Python framework that mediates agents' access to model, memory, storage and tools through "system calls". Research code. | Confirms the architecture (the OS layer mediates, the agent does not act directly). It serves agent developers and needs extra services; it is not a desktop intent service. [Proposal §2.1] | Reference only. |
| **ThorOS (YggdrasilAI)** | Voice-first Debian system: Goal, Planner, Agents, Permission Manager; faster-whisper, Ollama, Piper. Single maintainer. | Closest working relative. Shows the pipeline is buildable by a small team. No semantic index, rollback or developer model. [Research report §5] | Reference only; do not fork. |
| **CaMeL (Debenedetti et al., 2025, arXiv 2503.18813)** | Separates control flow (from the trusted user request) from data flow (untrusted content) so injected text cannot change which actions run. | The full system is heavy, but its central rule fits a small engine exactly: fix the plan from the user's words; treat everything read later as data. | Adopt the principle (architecture §7.3, §11). |
| **Progent (Shi et al., 2025)** | Declarative allow and forbid rules on tool calls. | Belongs to the Policy Engine, not the Intent Engine. Listed here because the engine must emit fields policy can reason about. | Policy Engine reference. |
| **Model Context Protocol (MCP)** | A standard way to describe tools to a model. | Useful later for interoperability. Run-time discovery of unknown tools conflicts with the closed list. [Tech-stack item 42] | Defer. Keep catalogue fields compatible. |

### 1.2 Components to integrate

| Component | Role | Hardware need | Complexity for us | Licence | Build or integrate |
|---|---|---|---|---|---|
| **llama.cpp** | Runs the quantised model on CPU, optional GPU offload; native grammar constraint; `llama-bench` for measurement | CPU only is enough; RAM = model size plus context | Low: run its server, call it locally | MIT | Integrate |
| **Ollama** | Wrapper around llama.cpp with model management | Same | Lower at first, but hides settings we need to pin (grammar, seed, quantisation) and runs a network-style daemon | MIT | Alternative only |
| **Qwen3 ~4B, 4-bit** | Candidate model | About 2.5 GB RAM (project estimate) [DEF Q&A 1.3] | None beyond prompt design | Apache-2.0 | Integrate; choose by measurement |
| **Smaller Qwen3 sizes (~1.7B)** | Faster fallback | About 1.2 to 1.5 GB (estimate) | None | Apache-2.0 | Candidate in the Phase 3 comparison |
| **Gemma family (~4B)** | Alternative model named in the tech-stack review | Similar | None | Gemma terms of use *(verify current version)* | Candidate in the Phase 3 comparison |
| **whisper.cpp** | Speech-to-text (voice service, outside the engine) | Under 0.5 GB for a small model (project estimate) | Low | MIT | Integrate |
| **Piper** | Text-to-speech (voice service) | Small | Low | Original repository MIT; active development moved to a GPL-3.0 repository (OHF-Voice/piper1-gpl); voices carry their own licences *(verify before bundling in the image)* | Integrate, after licence check |
| **Pydantic v2** | Schemas, validation, JSON Schema export | None | Low | MIT | Integrate |
| **pytest, Hypothesis** | Tests; property-based tests for the validator | None | Low | MIT / MPL-2.0 *(verify)* | Integrate |
| **jiwer** | Word error rate for the voice evaluation [DEF B3] | None | Low | Apache-2.0 *(verify)* | Integrate |
| **RapidFuzz** | Fuzzy matching for Roman-Urdu spelling variants | None | Low | MIT *(verify)* | Integrate if needed |
| **SQLite** | Local storage for traces if files prove unwieldy | None | Low | Public domain | Integrate if needed |

The model list must be refreshed at the start of Phase 3. Small open models change every few months, and the right choice is whichever performs best on our own development set on our own laptop, not whichever is named here.

### 1.3 What we build ourselves

The operation catalogue, the schemas, the normaliser and Roman-Urdu lexicon, the rule matcher, the prompt and grammar generation, the validator, confidence and abstention logic, session and reference handling, the clarification dialogue, the response templates, the service wrapper, and the evaluation kit with its English, Urdu and mixed datasets. This is the original engineering content of the Intent Engine. The model, the speech engines and the runtime are parts; the engine is the logic that makes a weak model safe and useful.

---

## Part 2. Decisions

Each decision has a status of **Proposed**. "Needs approval" means it changes or extends something already written in the proposal, SRS or defence material.

### TD-01 The engine outputs proposals in a closed schema and nothing else
- **Basis:** SRS FR-04, NFR-14. Confirmed requirement, recorded here for completeness.
- **Consequence:** No shell, no code generation, no free-form tool names.

### TD-02 Implementation language: Python 3.11 or newer
- **Reasons:** The evaluation plan already names Python tools (pytest, jiwer, psutil). Model inference dominates latency by three orders of magnitude, so interpreter speed is irrelevant here. Lowest learning cost for the team (assumed; team skills are not recorded).
- **Alternative:** Rust for the service. Rejected for now: no measurable benefit for this component, slower development.
- **Needs approval:** Yes, the SRS does not fix a language.

### TD-03 The model runs in a separate process (llama.cpp server), not inside the engine
- **Reasons:** The process can be denied all file and network access by the operating system, which turns "the model cannot read your files" from a promise into an enforced property. The model stays loaded between requests. A crash or hang does not take the engine down. The model can be swapped without code changes.
- **Alternative:** In-process bindings (llama-cpp-python). Simpler to start; acceptable for the first two weeks of Phase 3 if the server causes setup problems.
- **To verify in Phase 3:** that the server can listen on a Unix socket or loopback only, and that no outbound connection is ever made (NFR-19).

### TD-04 Grammar-constrained JSON, temperature 0, fixed seed, reasoning mode off
- **Reasons:** Format guaranteed; runs are repeatable, which the evaluation integrity rules need; shorter output means lower CPU latency. [DEF Q&A 3.2, 3.3, evaluation Part A]

### TD-05 Hybrid interpreter: deterministic rules first, model second
- **Reasons:** Fixed commands must be instant and never misread. The rule matcher has to be built anyway as the pre-registered baseline. No model call is spent where none is needed.
- **Risk:** A loose rule could fire wrongly. Mitigation: full-template match only, and the rule path is evaluated on its own for precision.
- **Needs approval:** Yes. SRS FR-03 says interpretation uses "a local LLM". The hybrid still uses it for everything the rules do not match with certainty, and the evaluation will report three configurations (rules only, model only, hybrid) so nothing is hidden.

### TD-06 One interpretation call per request; no agent loop; multi-step as one validated list (limit five)
- **Reasons:** Architecture §7.4.

### TD-07 Confidence is computed by the engine, not written by the model
- **Reasons:** Architecture §8. Self-reported confidence from a small model is not a dependable basis for a safety decision.
- **Needs approval:** Yes, it defines a field the SRS left undefined.

### TD-08 IPC: JSON-RPC 2.0 over Unix domain sockets between SCAR services
- **Reasons:** It is the SRS reference design; it is trivial to test from a script; peer credentials are available on the socket. D-Bus remains for desktop integration (notifications), as the SRS says.
- **Conflict:** The tech-stack review recommended D-Bus for service calls. The SRS memory note lists this as open for the SDS.
- **Needs approval:** Yes (SDS decision).

### TD-09 Untrusted text is never placed in the interpreter prompt; `summarise_file` is an isolated call
- **Reasons:** Architecture §7.3 and §11. Makes FR-22 hold by construction for the interpreter.
- **Needs approval:** Yes, because UC-05 describes a weaker flow.

### TD-10 Session state is short-lived and typed; no long-term memory; no conversation history in the prompt
- **Basis:** DEF Q&A 3.7.

### TD-11 Operation catalogue is a single versioned file; prompt, grammar, validator and documentation are generated from it
- **Reasons:** One source of truth. Adding an operation becomes: add an entry, add examples, add the tool, add tests.

### TD-12 Responses and questions come from templates, not from the model
- **Reasons:** Deterministic, instant, testable, and no second model call. English first; Roman-Urdu templates later.

### TD-13 Control commands are handled by rules only
- **Scope:** undo last task, cancel, switch mode, pick result N, yes/no to a pending question.
- **Reasons:** Closed phrasing; must not depend on a model.
- **Needs approval:** Yes, with gap 1 and Q-A3 in the architecture document.

### TD-14 Model and quantisation are chosen by measurement in Phase 3
- **Method:** Two or three candidate models at two quantisation levels, on the development set and the reference laptop; record exact match, wrong-action rate, latency, memory. Keep the smallest that meets the targets. [DEF Q&A 1.3]
- **No fine-tuning** unless code-switched accuracy stays under the target after prompt work. [DEF Q&A 1.8]

### TD-15 Development set and test set are separate from day one
- **Reasons:** Few-shot examples, thresholds and rules are all tuned on data. If that data is the test set, the result is worthless. Test items are written by a member who does not build the interpreter and are frozen by hash before the first measured run. [DEF B8]

### TD-16 The engine is developed as a portable library first, a Linux service second
- **Reasons:** Most of the engine (schemas, rules, model client, evaluation) runs on any operating system, so work does not wait for the bootable image. Only sandboxing, sockets and systemd need Linux. The team's working folder is on Windows, so this ordering avoids an early blocker. A Fedora VM, WSL2 or dual boot is needed from Phase 6 at the latest.

---

## Part 3. Engineering standards

Kept deliberately small. Each item has a reason.

### 3.1 Repository layout (proposed)

```
scar-os/
  MILESTONES.md                 single source of truth for progress
  docs/                         architecture, features, decisions, testing, risks
  intent_engine/
    pyproject.toml
    src/scar_intent/
      contracts/                schemas, JSON Schema export
      catalogue/                operations.yaml + loader
      normalise/                unicode, script, Roman-Urdu lexicon
      rules/                    templates and matcher
      llm/                      runtime client, prompt builder, grammar
      validate/                 schema and catalogue checks
      decide/                   confidence, thresholds
      dialogue/                 session, references, clarification
      compose/                  response templates
      service/                  JSON-RPC server (Phase 6)
      cli.py                    scar-intent command
    tests/
      unit/  integration/  security/
  reference/                    stand-in grounder, policy table, dry-run executor
  evalkit/
    datasets/dev/               tunable
    datasets/test/              frozen, hash recorded
    fixtures/                   small file corpus and sample repositories
    metrics.py  run.py
    reports/                    every run, including failures
  models/                       not in version control; MANIFEST with file hashes
```

One package per SCAR component, so other components can be added beside `intent_engine/` without restructuring.

### 3.2 Module boundaries

- `contracts` depends on nothing. Every other module depends on it.
- `rules`, `llm`, `validate`, `decide`, `dialogue`, `compose` do not import each other's internals; they exchange contract types.
- Only `llm` talks to the model runtime. Only `service` opens sockets.
- Nothing in `scar_intent` may spawn a process or touch user files. A test enforces this.

### 3.3 Interface contracts

Schemas are code (Pydantic), exported as JSON Schema files that other components consume. A schema change bumps `schema_version` and is recorded in `TECHNICAL_DECISIONS.md`. Breaking changes need agreement from the owners of the affected component.

### 3.4 Logging and debugging

Structured JSON lines; one trace record per request (architecture §13). No raw audio, no file contents. Log level set by environment variable. A `scar-intent --explain` flag prints the path taken, the prompt version and the raw output.

### 3.5 Testing

| Level | What | When it runs |
|---|---|---|
| Unit | Schemas, normaliser, rule templates, validator, metrics, reference resolution | Every commit, in CI, no model needed |
| Integration | Interpreter with a recorded-response fake model; engine with stand-in grounding and policy | Every commit, in CI |
| Model-in-the-loop | Development-set evaluation with the real model | Locally on the reference laptop, at least weekly and before each phase gate |
| System | Typed and spoken requests through Shell, engine, policy, task engine | From Phase 6 |
| Performance | Latency and memory on the reference laptop | Phase 3 onward, same script each time |
| Security | Injection files, malformed input, fail-safe, no-shell check, socket peer check | Phase 6 onward; static checks from Phase 1 |

Details and datasets are in `TESTING_STRATEGY.md`.

### 3.6 Documentation

`MILESTONES.md` for progress. The five documents in `docs/` for design. A short README per package saying how to run and test it. The catalogue generates the user-facing list of supported commands. No documentation is duplicated; documents link to each other.

### 3.7 Version control

Git. `main` is always passing. Short-lived branches, one reviewer from the team per merge. Commit messages state what and why. Tags at each phase gate (`ie-phase-3`, ...). Model files and recordings stay out of the repository; their hashes go in. No AI co-author lines or attribution in commits, files or pull requests.

### 3.8 Continuous integration

Lint (`ruff`), type check (`mypy`), unit and integration tests on every push. CI does not run the real model: hosted runners differ from the reference laptop, so model accuracy and latency are measured only on the reference machine and the reports are committed. Hosting (GitHub or other) is not decided; no repository exists yet.

### 3.9 Environments

| Environment | Purpose |
|---|---|
| Developer machine (any OS) | Library, rules, tests, evaluation harness |
| Reference laptop (8 GB RAM per project documents; exact specification to be recorded) | Every reported latency, memory and accuracy number |
| Fedora VM or installed Fedora Atomic | Service, sockets, sandboxing, voice, Shell |
| Disposable VM with dummy home directory | Injection and policy tests [DEF B4] |
