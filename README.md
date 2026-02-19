# Grounded

My personal agentic workflow **GitHub Copilot Custom Agents**.

> **Note:** This is an experimental setup I use to improve my Copilot workflow. It is a work in progress, and the agents usually need tweaking per project and stack.

### Prerequisites

- [VS Code](https://code.visualstudio.com)
- [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) extension
- GitHub Copilot subscription

### Installation

1. **Copy the `.github` folder and `AGENTS.md`** to your project root:

   ```bash
   # Clone this repo
   # Copy to your project
   cp -r grounded/.github grounded/AGENTS.md your-project/
   ```

2. **Customize for your project:**
   - Edit `AGENTS.md` with your coding standards and build commands
   - This repo's `AGENTS.md` is JS/TS-oriented; for non-TypeScript projects, create your own `AGENTS.md` instead of copying it as-is
   - Add skills in `.github/skills/` for your tech stack

3. **Open VS Code** in your project and start using agents.

### What You Get

```
your-project/
├── .github/
│   ├── agents/           # 6 specialized agents
│   └── skills/           # Add your own skills here
└── AGENTS.md             # Coding standards & build commands
```

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USER REQUIREMENT                             │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        PLANNER AGENT                                 │
│  - Creates/updates roadmap.json                                      │
│  - Defines items with complexity, acceptance criteria, verification  │
│  - Persists Research findings into planningResearch field            │
│  - Hands off to Orchestrator when roadmap is ready                   │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      ORCHESTRATOR AGENT                              │
│  - Loads roadmap.json as source of truth                             │
│  - Selects one item per iteration (priority → id order)              │
│  - Dispatches to sub-agents based on complexity                      │
│  - Evaluates sub-agent verdicts (does NOT run checks directly)       │
│  - Blocks failed items and reports to the user                       │
│  - Updates roadmap state and loops until complete                    │
└─────────────────────────────────────────────────────────────────────┘
                                   │
        ┌──────────────────────────┼──────────────────────────┐
        │ simple                   │ medium        complex    │
        ▼                          ▼               ▼          │
┌─────────────────┐    ┌─────────────────┐ ┌──────────────┐   │
│ ⚡ COPILOT AGENT │    │ 🔍 RESEARCH     │ │ 🔍 RESEARCH  │   │
│ - Implement     │    │ - If needed     │ │ - Always     │   │
└────────┬────────┘    └────────┬────────┘ └──────┬───────┘   │
         │                      ▼                  ▼          │
         │             ┌─────────────────────────────────┐     │
         │             │ 🏗️ ARCHITECT (design mode)      │     │
         │             │ - Make decision + ADR            │     │
         │             └────────────┬────────────────────┘     │
         │                          ▼                          │
         │             ┌─────────────────┐                     │
         │             │ ⚡ COPILOT AGENT │                     │
         │             │ - Implement     │                     │
         │             └────────┬────────┘                     │
         │                      │                              │
         │                      ▼ (complex only)               │
         │             ┌──────────────────────────┐            │
         │             │ 🏗️ ARCHITECT (validate)  │            │
         │             │ - Check design alignment  │            │
         │             └────────┬─────────────────┘            │
         └──────────────────────┼──────────────────────────────┘
                                ▼
                    ┌──────────────────────────┐
                    │      🧪 TESTING AGENT    │
                    │  - Write/run tests       │
                    │  - Report pass/fail      │
                    └──────────────────────────┘
                                ▼
                    ┌──────────────────────────┐
                    │      🔍 REVIEW AGENT     │
                    │  - Code quality          │
                    │  - Sole verification     │
                    │    gate (lint/types/build)│
                    └──────────────────────────┘
                                ▼
                    ┌──────────────────────────┐
                    │   Orchestrator evaluates │
                    │   verdicts → pass/  │
                    │   block → next item      │
                    └──────────────────────────┘
```

**Note:** Orchestrator dispatches agents sequentially per item. Simple items skip Research/Architect. Medium items skip Research if planning already gathered findings. Complex items always run Research and add an Architect validation step after implementation.

### Agent Overview

| Agent            | Purpose                                                                                                          | Tools                            | Sub-agents / Handoffs                |
| ---------------- | ---------------------------------------------------------------------------------------------------------------- | -------------------------------- | ------------------------------------ |
| **Planner**      | Creates/updates `roadmap.json`, persists Research findings                                                       | read + search + write            | Research, Orchestrator               |
| **Orchestrator** | Runs roadmap loop, dispatches sub-agents, evaluates verdicts                                                     | read + search + write            | Research, Architect, Testing, Review |
| **Research**     | Gathers evidence (read-only, no recommendations)                                                                 | read + search                    | —                                    |
| **Architect**    | Design mode: architecture decisions + ADRs. Validation mode: checks implementation against design (complex only) | read + search + write            | —                                    |
| **Testing**      | Writes and runs tests (unit + integration), reports pass/fail evidence                                           | read + search + write + terminal | —                                    |
| **Review**       | Sole verification gate: code review + lint/typecheck/build + auto-fix                                            | read + search + write + terminal | —                                    |

### Tool Restrictions

Agents have intentionally restricted tool access:

- **Read-only agents** (Research): Can search and analyze but NOT modify files
- **Write agents** (Planner, Architect): Can create and edit files
- **Write + verify agents** (Review): Can review, auto-fix trivial lint issues, and run verification commands
- **Terminal agents** (Testing, Review): Can run commands

### Dispatch Flow

Orchestrator dispatches based on item complexity in `roadmap.json`:

| Complexity     | Dispatch Path                                                                           |
| -------------- | --------------------------------------------------------------------------------------- |
| **simple**     | Copilot Agent → Testing → Review                                                        |
| **medium**     | (Research if needed) → Architect → Copilot Agent → Testing → Review                     |
| **complex**    | Research (always) → Architect → Copilot Agent → Architect Validation → Testing → Review |
| **greenfield** | Architect (design) → Scaffold from ADR → then normal complexity-based items             |

Special routing rule: if an item primarily creates or updates test files, Orchestrator dispatches `@testing` for the implementation step instead of the regular Copilot coding agent.

### Key Design Decisions

- **Single verification owner:** Review is the sole gate for lint, typecheck, and build. Testing owns test execution. Orchestrator reads their verdicts — it never runs checks directly.
- **Standardized output contracts:** All sub-agents (including the implementation step) return a structured `### Orchestrator Contract` section with `Status`, `Evidence`, and agent-specific fields. Orchestrator parses this section to extract verdicts — no freeform interpretation.
- **Research deduplication:** Planner persists Research findings into `planningResearch` on each roadmap item. Orchestrator skips Research for medium items when findings already exist; complex items always run fresh Research.
- **Verification noise control:** Planner prefers behavior-level verification evidence for implementation items and avoids repeating global commands across non-final items; broad checks are concentrated in a final gate item.
- **Review auto-fix:** Review has limited write access for deterministic, trivial fixes (formatting, unused imports). It cannot make logic changes or fixes requiring judgment.
- **Failure policy:** When Testing or Review fails, Orchestrator marks the item as blocked and reports the failure evidence to the user for manual intervention.
- **Structured context compression:** Sub-agent output is not forwarded as raw prose. Orchestrator extracts only the `### Orchestrator Contract` section and builds a typed summary per agent before forwarding to the next step.
- **Inter-item learnings:** Orchestrator persists learnings from each completed item into a top-level `learnings` array in `roadmap.json`, which is included in all subsequent dispatches so later items benefit from earlier discoveries.
- **Architect validation:** Complex items get an extra Architect pass after implementation to catch design drift before tests run.
- **Greenfield projects:** When Research shows no existing patterns (fresh project), Planner creates an `Architect (design)` → `Scaffold project from ADR` sequence before feature items. Orchestrator handles these specially — no Testing or Review required for design/scaffold steps.

## Usage

> **Current experience note:** Copilot CLI currently has better Skills support, while VS Code Agent Mode currently provides the smoother handoff-button workflow.

### Workflow Example

```
@planner Add user authentication with login/logout
```

The Planner will:

1. Create or update `roadmap.json` with items tagged by complexity
2. Show the `🎯 Start Orchestration` handoff button
3. You click the button to start the autonomous loop

Orchestrator then:

1. Loads `roadmap.json` and picks the next ready item
2. Dispatches sub-agents based on complexity (simple skips Research/Architect)
3. Updates roadmap state and loops until complete

### Direct Agent Usage

```
@planner Create a plan for adding dark mode support
@research Find all authentication-related code in this project
@architect Design the data model for user subscriptions
@testing Write tests for the auth service
@review Check the UserService for security issues
```

## Handoffs

In **VS Code Agent Mode**, agents use handoffs to guide you through the workflow. The main handoff:

- `🎯 Start Orchestration` — Planner → Orchestrator (starts the autonomous loop)

Orchestrator dispatches sub-agents internally via `runSubagent()` so the loop stays in control.

In **Copilot CLI**, handoff buttons are not currently supported, so you need to call the next agent manually.

## Directory Structure

```
your-project/
├── .github/
│   ├── agents/
│   │   ├── planner.agent.md        # Creates/updates roadmap.json
│   │   ├── orchestrator.agent.md   # Runs roadmap loop, dispatches sub-agents
│   │   ├── research.agent.md       # Gathers evidence (read-only)
│   │   ├── architect.agent.md      # Makes architecture decisions (design + validation modes)
│   │   ├── testing.agent.md        # Writes and runs tests
│   │   └── review.agent.md         # Code review + sole verification gate
│   └── skills/                     # Add your own skills here (gitignored)
└── AGENTS.md                       # Coding standards & build commands
```

## Skills

Skills are domain-specific knowledge modules that agents invoke automatically. Add your own skills based on your tech stack in `.github/skills/`.

Skill files are gitignored so each project can have its own.

### Platform Compatibility

- Skills support currently works with Copilot coding agent, GitHub Copilot CLI, and Agent Mode in VS Code Insiders. VS Code stable support is still rolling out. See [GitHub Copilot Skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills).
- Handoffs are not currently supported in Copilot CLI Agent Mode (tracked in [github/copilot-cli#561](https://github.com/github/copilot-cli/issues/561)).
- If your environment does not reliably pick up Skills, put critical stack-specific guidance directly inside each `.agent.md` prompt as a fallback.
- AGENTS.md limitations: [VS Code Custom Instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)

## License

[MIT](LICENSE)
