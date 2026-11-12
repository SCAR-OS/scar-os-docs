# SCAR OS Intent Engine: Planning Report

| | |
|---|---|
| Owner | SCAR-OS Development Team |
| Status | Summary of the planning investigation, for team approval. Nothing is implemented. |
| Date | 12 November 2026 |

This is the short reading path. Each section points to the document that holds the detail.

---

## 1. Understanding of the FYP and the Intent Engine

SCAR OS is not a new kernel and not a chatbot on a desktop. It is a layer of local services on Fedora Linux that lets a user say or type what they want, in English, Urdu or a mix, and have it carried out on real files and projects under rules the AI cannot change, with every change recorded and reversible. The proposal's research gap is architectural: making intent understanding, meaning-based access, developer context, least-privilege authorization, audit, a private area and recovery work together in one environment.

Inside that, the Intent Engine has one job: turn a request into validated candidate operations from a closed list, or ask, or decline. It is deliberately the *least trusted* part of the system. Everything after it (grounding, policy, checkpoint, sandbox, audit) exists so that a small, imperfect local model can be useful without being dangerous.

Detail: `MILESTONES.md` §1 to §4; `INTENT_ENGINE_ARCHITECTURE.md` §1 and §2.

## 2. The proposal against the technical implementation

The proposal, SRS and defence plan are consistent on the essentials: fixed schema, twelve operations, local model with constrained JSON output, deterministic policy, pre-registered targets. No recommendation in this plan conflicts with the nine objectives.

What the design phase still has to settle:

1. The SRS use cases contain spoken commands that are not among the twelve operations ("undo the last task", "switch to developer mode", "open the second one").
2. FR-17 needs a code-structure operation that is not in the list.
3. The schema cannot yet express a question, an unsupported request, or several operations.
4. The `confidence` field has no defined source, and a small model's own estimate is not a safe one.
5. FR-03 accepts Urdu script; the evaluation plan does not test it.
6. The latency target is end to end, with no share set for the engine.
7. UC-05 lets the model return an operation copied from a file and relies on policy to block it; the engine can be built so that this cannot happen at all.
8. The tech-stack review and the SRS differ on service IPC and on some names; the SRS is taken as the baseline.
9. The defence estimate of four person-weeks for the engine is well below the work described here.

Detail: `INTENT_ENGINE_ARCHITECTURE.md` §14; `PROJECT_RISKS.md` §3.4.

## 3. Recommended architecture

A pipeline of seven engine stages: receive, normalise, rule fast path, model interpretation, validate, resolve references, decide. Only one stage calls the model, once per request, with grammar-constrained output, in a separate process that has no access to files or network. Untrusted text never enters the interpreter prompt. Results are one of five statuses: proposed, clarify, unsupported, rejected, control.

Detail: `INTENT_ENGINE_ARCHITECTURE.md`.

## 4. Feature inventory

42 features in five groups, each with purpose, method, dependencies, difficulty, priority, estimate, tests and demonstration, plus ten future or excluded items with reasons. About 42 person-days for the first working version and about 123 for everything through priority P2.

Detail: `FEATURE_SPECIFICATION.md`.

## 5. Technology recommendations

Python; llama.cpp server with a small quantised open model chosen by measurement; Pydantic schemas; our own rule matcher; pytest; JSON-RPC over Unix sockets. Rejected with reasons: agent frameworks and agent loops, trained intent classifiers, symbolic planners, run-time tool discovery. Sixteen decisions are recorded, all as proposed.

Detail: `TECHNICAL_DECISIONS.md`.

## 6 and 7. Phases and weekly schedule

Ten phases over sixteen working weeks plus two buffer weeks, with a gate after each phase, a go or no-go review after the model is first measured, and three schedule variants because the available time is not recorded.

Detail: `MILESTONES.md` §6 to §9.

## 8. Testing and evaluation

The pre-registered test set and targets are used unchanged. A separate development set is used for all tuning. Three configurations are compared (rules only, model only, hybrid). Engine-level targets that have no measured basis yet are left open until the first measurements. Eleven demonstration scenarios are defined.

Detail: `TESTING_STRATEGY.md`.

## 9. Risks and unresolved questions

Nineteen risks, ten unknowns, nine questions to answer by measurement and ten design questions.

Detail: `PROJECT_RISKS.md`.

## 10. Documents created

`MILESTONES.md`, and in `docs/`: `INTENT_ENGINE_ARCHITECTURE.md`, `FEATURE_SPECIFICATION.md`, `TECHNICAL_DECISIONS.md`, `TESTING_STRATEGY.md`, `PROJECT_RISKS.md`, `PLANNING_REPORT.md`. No existing project file was changed.

## 11. Recommended first milestone

**Phase 1, week 1: repository, contracts and catalogue.** It needs no model, no Linux machine and no other component, it produces the interfaces every other team member depends on, and it turns the SRS candidate-operation format into something a test can check.

---

## Engineering assessment

**What exactly are we building?**
A local service that translates one request in English, Urdu or mixed language into operations from a closed, typed list, and that knows when to ask and when to say no. It is a constrained translator with a safety decision attached, not an autonomous agent.

**What will the first working version be able to do?**
At the end of Phase 3 (week 7): take a typed English or Roman-Urdu request for any of the twelve operations, produce a validated candidate through rules or the local model, resolve it against a fixture folder, label its risk, and print the plan without changing anything. It will ask when a required detail is missing, and it will come with measured accuracy, latency and memory on the reference laptop, compared against a rule-only baseline.

**What makes it technically meaningful?**
Four things that can be shown and measured rather than claimed. First, the separation of proposing from deciding and executing is enforced by structure: there is no path from model output to a command, and the model process cannot read files. Second, it prefers abstention to error and measures the trade-off between wrong actions and unnecessary questions. Third, it handles code-switched Urdu and English on an 8 GB machine with no network, and reports honestly how well. Fourth, it is evaluated against a frozen test set with a baseline, so the result means something whether it is good or bad.

**What is realistically achievable within the FYP?**
Typed interpretation for the twelve operations in three language forms, clarification, short multi-step requests, session references, developer requests, a service integrated with policy and task handling, engine-side security evidence, and a voice path that is solid in English and measured in Urdu. Whether the 85% and 70% targets are met cannot be known before Phase 3; the plan is built so that a miss is found in week 7, not at the final evaluation.

**What should we avoid building?**
An agent loop or general planner. A trained intent classifier. Screen or browser automation. Long-term memory. Run-time tool discovery. Model-written responses. Anything that puts file names or file content into the interpreter prompt. A framework dependency that hides the control flow the team must be able to explain.

**What is the recommended development sequence?**
Contracts and measurement first. Then a rule-based version that works end to end with no model. Then the model, measured against that baseline, with a go or no-go decision. Then language robustness and abstention. Then context and multi-step requests. Then the service, integration and security. Then voice. Then the frozen evaluation. Then integration into the image and the demonstration.
