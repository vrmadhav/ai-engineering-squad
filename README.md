# ai-engineering-squad

My personal engineering team: a reusable, Codex-native squad for building robust,
production-ready applications with clear ownership and independent verification.

This repository contains instructions, agent configuration, and document
templates. It has no application, selected stack, hosting/database provider,
framework dependencies, or deployment pipeline yet.

## The squad

The main Codex chat acts as **Engineering Lead / Orchestrator**. It plans work,
delegates tasks, manages file ownership, integrates changes, and reports evidence.

| Agent | Responsibility |
| --- | --- |
| [`architect`](.codex/agents/architect.toml) | System architecture, technical decisions, contracts, data models, security, scalability |
| [`frontend`](.codex/agents/frontend.toml) | UI, reusable components, responsive design, accessibility, performance, frontend tests |
| [`backend`](.codex/agents/backend.toml) | APIs, databases, authentication/authorization, integrations, migrations, reliability, backend tests |
| [`qa`](.codex/agents/qa.toml) | Independent execution of checks, defect/security/regression review, release-readiness recommendation |

All roles follow [AGENTS.md](AGENTS.md). QA can recommend blocking a release.
Only you can approve production deployment.

## Start a new product

1. Copy this repository into a new product repository, or copy `AGENTS.md`,
   `.codex/`, and `templates/` into an existing one. Inspect and merge existing
   instructions/configuration instead of overwriting them. Preserve applicable
   license notices when copying. Keep this squad repository reusable.
2. Open the product repository as the project root in a current local Codex
   desktop app, CLI, or IDE extension. Review and trust the project through your
   client's trust controls: project `.codex/` configuration loads only for trusted
   projects. Start a fresh chat/session after adding or changing configuration.
3. Describe the product, intended users, constraints, and success criteria using
   the kickoff prompt below. The Lead captures requirements and delegates design.
4. Save filled template copies under `docs/` (for example, `docs/requirements/`,
   `docs/architecture/`, `docs/contracts/`, `docs/plans/`, and `docs/qa/`). Keep the
   reusable originals in `templates/` unchanged.
5. Agree on architecture and API contracts before coding. Once a stack is chosen,
   establish real lint, type-check, test, and build commands in the plan and
   project scripts. Add equivalent CI checks as part of product setup.
6. Implement, integrate, and obtain independent QA evidence. Review the delivery
   report and explicitly approve any later production deployment.

### Product kickoff prompt

```text
Act as Engineering Lead and use this repository's AI engineering squad.
The product is: <idea>.
Users and critical journeys: <details>.
Constraints and success criteria: <details>.
Inspect the repository and follow AGENTS.md. Use architect to establish the
requirements, architecture, data model, and API contracts. Record agreement with
the Lead and affected implementers before coding. Then delegate independent
frontend and backend tasks with exclusive file ownership, integrate their work,
and have qa independently run checks and verify acceptance criteria.
Report actual test evidence and blockers. Do not deploy without my approval.
```

For design only, end the prompt with: `Stop after requirements, architecture,
contracts, and the implementation plan; do not implement yet.`

## Trigger agents

Use direct natural-language delegation requests in the main Codex chat:

```text
Delegate to architect to review this feature's design and API contract.
Return decisions, risks, and open questions before implementation.

Use frontend and backend for the independent tasks in the agreed plan.
Assign exclusive writable paths, wait for both, and integrate their results.

Have qa independently verify the integrated change against the acceptance
criteria. Run the required checks and return evidence and a release verdict.
```

These are prompts, not custom slash commands. Current Codex discovers standalone
TOML files under `.codex/agents/`; each file's `name` is its agent identifier.
No manual `[agents.architect]` registration is needed in current releases.
The files provide the required `name`, `description`, and `developer_instructions`.
Models and reasoning inherit from the parent; no model subscription is hardcoded.

The [project configuration](.codex/config.toml) enables agents and caps concurrent
spawned threads at three, excluding the Lead. All four roles are available but
run in stages. A stricter runtime limit takes precedence. The parent uses
workspace-write sandboxing and on-request approvals; subagents inherit actual
session permissions. Agent instructions do not provide separate sandbox boundaries.

In the desktop app, inspect subagent activity from the main chat. Current CLI
sessions provide `/subagents` to inspect/switch between agent threads. Ask the
Lead to steer or stop agents. Subagents consume additional model usage.

## Orchestration and quality gates

1. **Discover:** Lead inspects existing work and records measurable requirements.
2. **Design:** Architect proposes decisions and contracts; Lead and affected
   implementers acknowledge the accepted revisions. Resolve blockers first.
3. **Plan:** Lead defines dependencies, exclusive file ownership, verification
   commands, and acceptance criteria. Shared files have one named writer.
4. **Implement:** Frontend and Backend work in parallel only where independent.
   Contract changes return to design. No concurrent edits to the same file.
5. **Integrate:** Lead reconciles changes and runs lint, type checks, tests, and build.
6. **Verify:** A separate QA agent inspects the final state, executes checks, and
   returns evidence. Owners fix defects; QA independently retests.
7. **Report:** Lead reports results, risks, and readiness. Failed checks, missing
   required evidence, or critical/high defects block readiness. Deployment needs
   your explicit approval for the exact release and target.

These are engineering instructions, not a workflow engine or technical deployment
lock. When a product gains CI and production access, enforce the gates with
required checks, protected deployment environments, and appropriate credentials.
Never disable a failing check to obtain a green result.

## Reusable templates

| Template | Purpose |
| --- | --- |
| [Product requirements](templates/product-requirements.md) | Scope, users, acceptance criteria, quality targets, open questions |
| [Architecture decision](templates/architecture-decision.md) | Alternatives, decision, security, data, failures, agreement |
| [API contract](templates/api-contract.md) | Typed inputs/outputs, validation, authorization, errors, compatibility |
| [Implementation plan](templates/implementation-plan.md) | Dependencies, file ownership, delegation packets, verification commands |
| [QA report](templates/qa-report.md) | Independent checks, acceptance traceability, defects, evidence, verdict |
| [Release readiness](templates/release-readiness.md) | Final gates, rollout/rollback, risks, explicit deployment approval |

## Validate and troubleshoot

From the repository root, inspect your installed version and configuration:

```sh
codex --version
codex --strict-config doctor --summary
```

`doctor` also reports local environment/auth/runtime health. Read each result;
its overall outcome is not an agent-discovery or product-test guarantee. Never
publish raw diagnostic output without checking it for private machine details.

In a fresh trusted project session, use this smoke-check prompt:

```text
Validate squad discovery without changing files. List the loaded instruction
sources and available custom agent names. Spawn architect, frontend, backend,
and qa sequentially, each only reporting its role and the applicable engineering
rules. Wait for their results. Distinguish actual custom-agent selection from
a generic agent given role instructions. Do not run application or deployment work.
```

If discovery fails, confirm the project root and trust status, inspect TOML errors
and `name` fields, and check for global/nested `AGENTS.override.md` or configuration
overrides. Restart the session after fixes. Use a current Codex release supporting
standalone custom agent files; older releases may use different syntax. If the
client cannot select custom agents, report it instead of claiming they loaded.

Bootstrap verification evidence and known limitations are recorded in
[docs/setup-validation.md](docs/setup-validation.md). There are no application
commands to run until a product and stack exist; that is not a passing application
test suite or a production-readiness claim.

## Configuration sources

Checked against official OpenAI documentation on October 23, 2026:

- [Subagents and standalone custom agent schema](https://developers.openai.com/codex/multi-agent)
- [Configuration precedence and project trust](https://learn.chatgpt.com/docs/config-file/config-basic)
- [AGENTS.md discovery](https://learn.chatgpt.com/docs/agent-configuration/agents-md)

MIT licensed; see [LICENSE](LICENSE).
