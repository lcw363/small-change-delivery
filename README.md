# Small Change Delivery

**English** | [简体中文](README.zh-CN.md)

A lightweight Codex skill for delivering well-defined, small-to-medium code changes with one writer in a single primary session.

It turns the engineering steps that are easiest to miss into a tight delivery loop: lock the scope and acceptance criteria, trace the impact, make the smallest production-ready change, verify according to risk, simplify the diff, and complete Review / Fix / Re-review. By default, it does not commit, push, or deploy.

## When to use it

Use this skill for:

- Small features or localized behavior changes
- Reproducible bug fixes
- Narrow changes to a single API, page, or module
- Low-to-medium-risk work that can be completed in one primary session without multi-stage decomposition
- Changes where you want to reduce missed callers, failure paths, tests, or validation without introducing a heavyweight process

Example invocation:

```text
Use $small-change-delivery to fix the broken order-list filter.

Requirements:
- Preserve the existing API contract
- Keep the change narrowly scoped
- Add a regression test
- Fix all valid P0-P3 review findings and review the new fixed point again
- Do not commit or push
```

Implicit invocation is disabled. Explicitly include `$small-change-delivery` in the request.

## When not to use it

Do not force this skill onto work that needs a different process:

| Situation | Recommended approach |
| --- | --- |
| The requirement or destination is still unclear | Discuss the design, update OpenSpec, or use `$grill-with-docs` first |
| The effort is too large or uncertain for one session | Use `$wayfinder` to build a decision map |
| Delivery spans stages or modules, or requires isolated sessions | Use `$subagent-delivery` |
| The change affects money, core authorization, security boundaries, data migration, complex concurrency, or hard-to-reverse side effects | Use a stricter domain-specific workflow and independent review |
| The request is review-only or diagnosis-only | Use the relevant review or diagnosis skill directly |

## Workflow

```text
Scope lock
  ↓
Impact analysis
  ↓
Minimal implementation
  ↓
Risk-based verification
  ↓
Code simplification
  ↓
Review → Fix → Re-review
  ↓
Report verified, unverified, and Git states separately
```

### Delivery priority

Complete the smallest production-ready loop for the confirmed requirement first. The primary flow, necessary failure paths, data state, and API behavior must work together as an acceptable outcome. Authorization, data consistency, transactions, input validation, sensitive-data protection, and failure recovery must be implemented at the same time when they are inseparable from that loop.

After the functional loop is complete, add routine security, boundary, and regression handling according to actual risk. Do not expand the scope, add abstractions without a clear payoff, exhaust low-probability theoretical cases, or extend the Review / Fix loop indefinitely in pursuit of absolute safety, total boundary coverage, or proof that no bug exists.

### 1. Lock the scope

Confirm the requirement, acceptance criteria, non-goals, risks, verification plan, and release authorization. If a key ambiguity could change a business rule, data behavior, authorization rule, API contract, or scope, pause the affected work instead of turning an assumption into production code.

### 2. Trace the impact

Follow only the layers relevant to the task: entry point, service, domain rules, mapper/SQL, DTO/VO, frontend bindings, configuration, migrations, and existing tests. For every behavior to be changed, identify its data source, callers, failure paths, compatibility requirements, and verification method.

### 3. Make the smallest production-ready change

Change only the files and behavior required by the acceptance criteria. Reuse established project patterns. Avoid unnecessary public abstractions, shared helpers, configuration, unrelated refactors, and formatting noise.

### 4. Verify according to risk

First confirm that the primary flow, necessary failure paths, data state, and API behavior form an acceptable functional loop. Then choose the smallest risk-based verification set that is sufficient for the change:

- Bug: prefer a regression test that reproduces the failure
- Business rule: cover the normal path, boundaries, illegal states, and dependency failures
- API: cover validation, HTTP status, JSON contract, authorization, and exception mapping
- SQL: cover predicates, empty results, affected-row counts, transactions, and isolation
- Frontend: cover the primary interaction, async state, error state, and critical request parameters
- Configuration or copy: run the relevant build or static checks and record reproducible manual verification

If an environment, credential, or fixture is unavailable, mark the result as `BLOCKED` or unverified. Do not present substitute evidence as live acceptance.

### 5. Simplify and review

Inspect the final diff and remove duplication, dead code, ineffective defenses, and over-abstraction introduced by the change. Review the fixed diff and fix every valid, actionable P0-P3 finding that belongs to the current scope and affects acceptance or a real risk. Rerun affected verification and review the new fixed point again. Record out-of-scope, low-probability theoretical, or no-clear-benefit suggestions as residual risk instead of expanding the implementation.

### 6. Report status precisely

Report these states separately:

- Completed behavior and key files
- Commands, test counts, and observed results
- Review fixed point, findings, and fixes
- Unverified HTTP, database, environment, or third-party checks
- Remaining risks
- Commit, push, and deploy status

Only claim delivery is complete when the code, applicable verification, and final review all support that conclusion.

## Optional companion skills

Small Change Delivery can run on its own. When the capability is available and the task warrants it, it can also use:

- `$code-simplifier` to reduce complexity introduced by the change
- `$code-review` for an independent review of medium-risk or API, SQL, transaction, and authorization changes
- `$diagnosing-bugs` for hard-to-reproduce or poorly understood failures
- `$webapp-testing` for browser behavior and interaction verification
- `$asana-openspec-java-workflow:java-test-strategy` to choose test layers for Java, Spring, and MyBatis changes

These are risk-based enhancements, not mandatory dependencies for every run.

## Installation

Clone the repository into the Codex user skill directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/lcw363/small-change-delivery.git ~/.codex/skills/small-change-delivery
```

Restart Codex or start a new task after installation so the skill list is reloaded.

### Update

```bash
git -C ~/.codex/skills/small-change-delivery pull --ff-only
```

### Uninstall

Remove `~/.codex/skills/small-change-delivery`, then restart Codex. Back up or commit local changes before removing the directory.

## Repository structure

```text
small-change-delivery/
├── SKILL.md
├── README.md
├── README.zh-CN.md
└── agents/
    └── openai.yaml
```

- `SKILL.md`: the workflow and guardrails Codex executes
- `README.md`: English installation, routing, and usage documentation
- `README.zh-CN.md`: Simplified Chinese installation, routing, and usage documentation
- `agents/openai.yaml`: the display name, summary, and default prompt shown in the skill list

## Design principles

- Keep the process proportional to the size of the change
- Establish evidence before drawing conclusions about the call chain
- Make the smallest production-ready change without expanding the requirement
- Match test depth to risk instead of running every possible test mechanically
- Bind review conclusions to the final fixed point and review again after fixes
- Report code completion, test results, live acceptance, commits, and releases as separate states

## Feedback and contributions

If a class of small-to-medium change is still easy to get wrong, open an issue and include:

- The task type and technology stack
- An example request
- The missed step or risk
- The expected completion criterion

When changing delivery behavior, update `SKILL.md` as the source of truth. Keep the README files focused on installation, routing, and usage information for people.
