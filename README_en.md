<div align="center">

![Outsource Logo](docs/outsource-logo.png)

# Outsource.skill

### Delegate mechanical work to low-cost sessions, keep the main session focused on judgment and code

[![Outsource](https://img.shields.io/badge/Outsource-skill-6B7280.svg?labelColor=111827)](SKILL.md)
[![Agent](https://img.shields.io/badge/Agent-skill-6B7280.svg?labelColor=0099FF)](#quick-start)
[![Templates](https://img.shields.io/badge/Templates-6%20scenarios-6B7280.svg?labelColor=7C3AED)](references/scenarios)
[![License](https://img.shields.io/badge/License-MIT-6B7280.svg?labelColor=16A34A)](LICENSE)

**Delegate the noise. Keep the judgment.**

English · [简体中文](README.md)

[Quick Start](#quick-start) · [Core Ideas](#core-ideas) · [Workflow](#workflow) · [Scenario Templates](#scenario-templates) · [File Protocol](#file-protocol) · [Output Example](#output-example)

</div>

---

## Introduction

Outsource is a workflow skill for agents.

It is not a build tool or an automation script collection. It is a mechanical-task delegation protocol: when the main session hits noisy execution work such as builds, tests, version checks, FQN resolution, log grep, environment probing, or data collection, the main session does not run those commands directly. Instead, it produces a self-contained delegation prompt for a low-cost model session or subagent to execute.

The main session only consumes the structured report returned by that execution session, keeping its context for work that actually needs judgment: architecture, decisions, code changes, and tradeoffs.

Good fits:

- Build logs are long and would pollute the main context.
- Dependency versions, class names, and method signatures need to be checked without burning high-value tokens.
- Logs, tests, and service reachability checks only need facts and evidence.
- Repetitive queries can be handled by a cheaper model while the stronger model keeps the global picture.

---

## Core Ideas

### Keep Noise Out Of The Main Session

A failed build, a large log grep, or a jar symbol scan can produce hundreds or thousands of lines. Outsource keeps that raw noise in the low-cost session and sends the main session only a short structured report.

This saves tokens, but more importantly, it protects the main context window.

### Strong Models Judge, Cheap Models Execute

Code writing, architecture, and risk judgment need a strong model with full context. Running commands, checking versions, and pasting exact errors only require a cheaper model to follow instructions and report facts.

Outsource separates those jobs: the main session defines the task, the execution session runs it, and the main session continues from the evidence.

### The Prompt Is The Protocol

Every delegation must be a five-part prompt:

| Section | Purpose |
|---|---|
| Role | Position the target session as a domain expert and state the task |
| Background | Provide path, stack, version, and intent so the target session is not blind |
| Steps | Give executable commands, purposes, and fallbacks where needed |
| Rules | Enforce read-only behavior, verbatim evidence, no fabrication, and uncertainty labels |
| Output | Embed the report format so the user can paste the prompt once |

---

## Workflow

| Stage | Main session | Target session | Output |
|---|---|---|---|
| 1. Classify | Decide whether the task is mechanical, verification, or exploration work | - | Delegate or handle inline |
| 2. Write prompt | Produce a self-contained five-part delegation prompt | - | A prompt ready to paste |
| 3. Execute | Let the user choose subagent or new session | Run commands, collect data, preserve evidence | Structured report |
| 4. Consume report | Continue coding or deciding from evidence | - | PASS / FAIL / next delegation |

Outsource uses three task sizes:

| Tier | Fit | Handling |
|---|---|---|
| Tier 0 | Tiny and clean, such as reading one short file or one small value | Main session handles it inline |
| Tier 1 | Small to medium, mechanical, may be noisy or multi-step | Delegate, usually subagent is recommended |
| Tier 2 | Large, repetitive, very noisy, needs human review, or should use separate billing | Delegate, usually a new session is recommended |

---

## Quick Start

When using this repository as an Agent skill, the main entry point is:

```text
SKILL.md
```

In an Agent environment that supports skills, requests like these should trigger Outsource:

```text
Delegate a build verification run for me.
```

```text
Generate a delegation prompt to check dependency versions and FQNs.
```

```text
Delegate the log investigation to a low-cost session and require the report format.
```

The main session will produce a complete prompt. You can send it to:

| Mode | Best for |
|---|---|
| subagent | The environment supports agent tools, the task is medium-sized, and you want the report returned automatically |
| new session | You want separate low-cost quota, the task is larger, or you want to review execution manually |

Outsource does not provide a CLI, install dependencies, or modify target projects by itself. Its output is an executable delegation prompt and a reporting protocol.

---

## Scenario Templates

Outsource currently includes 6 scenario templates:

| Scenario | File | Summary |
|---|---|---|
| Build / compile verification | [`build-compile.md`](references/scenarios/build-compile.md) | Run a build and report PASS/FAIL with exact errors |
| Dependency / version / FQN resolution | [`dependency-symbol.md`](references/scenarios/dependency-symbol.md) | Check versions, artifact existence, fully qualified class names, and signatures |
| Data query / collection | [`data-query.md`](references/scenarios/data-query.md) | Read-only querying from DBs, APIs, CLIs, or files with original values |
| Log / error investigation | [`log-debug.md`](references/scenarios/log-debug.md) | Grep logs by pattern and report only matching evidence |
| Environment / connectivity probe | [`env-probe.md`](references/scenarios/env-probe.md) | Probe ports, services, APIs, and health endpoints |
| Deep learning / dataset inspection | [`dataset-inspect.md`](references/scenarios/dataset-inspect.md) | Inspect size, schema, label distribution, bad samples, and leakage |

The generic skeleton is in:

```text
references/templates.md
```

The authoring flow for new scenarios is in:

```text
references/authoring.md
```

---

## File Protocol

```text
outsource/
├── SKILL.md                         # skill entry: triggers, workflow, execution modes, antipatterns
├── LICENSE                          # MIT License
└── references/
    ├── templates.md                 # five-part delegation prompt skeleton
    ├── authoring.md                 # authoring flow for custom scenarios
    └── scenarios/
        ├── index.md                 # scenario index
        ├── build-compile.md         # build / compile verification
        ├── dependency-symbol.md     # dependency / version / FQN resolution
        ├── data-query.md            # data query / collection
        ├── log-debug.md             # log / error investigation
        ├── env-probe.md             # environment / connectivity probe
        └── dataset-inspect.md       # dataset inspection
```

---

## Delegation Rules

Every delegation prompt should preserve these rules:

- Read-only by default: do not edit files or commit code unless the task explicitly allows writes.
- Return key facts verbatim: versions, FQNs, errors, matching log lines, and signatures.
- If something cannot be found, say it cannot be found. Do not invent plausible answers.
- Mark uncertainty or ambiguity explicitly.
- Report facts and evidence only. Do not make subjective root-cause conclusions.
- Report blockers such as network issues, auth failures, missing tools, or timeouts without repeatedly forcing retries.

---

## Design Boundaries

Outsource is responsible for:

- Defining the delegation methodology
- Providing a five-part prompt skeleton
- Providing common engineering scenario templates
- Constraining the report format and evidence requirements for low-cost sessions

It is not responsible for:

- Automatically running build, test, or query commands
- Installing dependencies or managing runtime environments
- Choosing subagent vs new session without user approval
- Inferring root causes without evidence
- Handling irreversible side effects

---

## Output Example

Below is the shape of a delegation prompt for build verification.

```text
# Role
You are a build and compilation expert. Your task is to verify whether the current project can compile and whether dependencies can be resolved, then report the result faithfully.

# Background
- Target: /path/to/project, stack: maven
- Why: dependency versions were just changed and compilation needs to be verified

# Steps
1. cd /path/to/project
2. Run the build: mvn -q -DskipTests compile. Purpose: check whether compilation passes
3. If it fails, preserve the full error output, especially the first error and dependency resolution errors

# Rules
- Read-only. Do not edit any files or change pom/config files.
- Paste error messages verbatim. Do not summarize them as "there is a compile error".
- Distinguish code compilation errors from dependency resolution errors and report each as facts.
- Report facts only. Do not make subjective conclusions.
- Report blockers honestly and do not repeatedly retry.

# Output
## Build Verification Report
1. Compile result: PASS / FAIL
2. Key error text, if FAIL:
   ```
   <paste>
   ```
3. Dependency resolution issue: <yes/no; which dependency and what error>
4. Blockers: <write "none" if none>
```

After the target session returns the report, the main session continues from the evidence: fix code, adjust dependencies, or issue a narrower follow-up delegation.

---

## License

MIT License. See [`LICENSE`](LICENSE).
