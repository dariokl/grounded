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
│  - Hands off to Orchestrator when roadmap is ready                   │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      ORCHESTRATOR AGENT                              │
│  - Loads roadmap.json as source of truth                             │
│  - Selects one item per iteration (priority → id order)              │
│  - Dispatches to sub-agents based on complexity                      │
│  - Updates roadmap state and loops until complete                    │
└─────────────────────────────────────────────────────────────────────┘
                                   │
        ┌──────────────────────────┼──────────────────────────┐
        │ simple                   │ medium/complex           │
        ▼                          ▼                          │
┌─────────────────┐    ┌─────────────────┐                    │
│ ⚡ COPILOT AGENT │    │ 🔍 RESEARCH     │                    │
│ - Implement     │    │ - Gather facts  │                    │
└────────┬────────┘    └────────┬────────┘                    │
         │                      ▼                             │
         │             ┌─────────────────┐                    │
         │             │ 🏗️ ARCHITECT    │                    │
         │             │ - Make decision │                    │
         │             └────────┬────────┘                    │
         │                      ▼                             │
         │             ┌─────────────────┐                    │
         │             │ ⚡ COPILOT AGENT │                    │
         │             │ - Implement     │                    │
         │             └────────┬────────┘                    │
         └──────────────────────┼─────────────────────────────┘
                                ▼
                    ┌──────────────────────────┐
                    │      🧪 TESTING AGENT    │
                    │  - Write/run tests       │
                    │  - Report evidence       │
                    └──────────────────────────┘
                                ▼
                    ┌──────────────────────────┐
                    │      🔍 REVIEW AGENT     │
                    │  - Code quality          │
                    │  - Security review       │
                    │  - Run verification      │
                    └──────────────────────────┘
                                ▼
                    ┌──────────────────────────┐
                    │   UPDATE roadmap.json    │
                    │   Loop to next item      │
                    └──────────────────────────┘
```

**Note:** Orchestrator dispatches agents sequentially per item. Simple items skip Research/Architect. All paths end with Testing → Review.

### Agent Overview

| Agent            | Purpose                                          | Tools           | Sub-agents / Handoffs                |
| ---------------- | ------------------------------------------------ | --------------- | ------------------------------------ |
| **Planner**      | Creates/updates `roadmap.json`                   | read + write    | Research, Orchestrator               |
| **Orchestrator** | Runs roadmap loop, dispatches sub-agents         | read + terminal | Research, Architect, Testing, Review |
| **Research**     | Gathers evidence (read-only, no recommendations) | read-only       | —                                    |
| **Architect**    | Makes architecture decisions, writes ADRs        | read + write    | Research (sub-agent)                 |
| **Testing**      | Writes and runs tests, reports evidence          | read + terminal | —                                    |
| **Review**       | Code review + verification gates                 | read + terminal | —                                    |

### Tool Restrictions

Agents have intentionally restricted tool access:

- **Read-only agents** (Research): Can search and analyze but NOT modify files
- **Write agents** (Planner, Architect): Can create and edit files
- **Terminal agents** (Orchestrator, Testing, Review): Can run commands

### Dispatch Flow

Orchestrator dispatches based on item complexity in `roadmap.json`:

| Complexity  | Dispatch Path                                           |
| ----------- | ------------------------------------------------------- |
| **simple**  | Copilot Agent → Testing → Review                        |
| **medium**  | Research → Architect → Copilot Agent → Testing → Review |
| **complex** | Research → Architect → Copilot Agent → Testing → Review |

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
.github/
├── agents/
│   ├── planner.agent.md        # Creates/updates roadmap.json
│   ├── orchestrator.agent.md   # Runs roadmap loop, dispatches sub-agents
│   ├── research.agent.md       # Gathers evidence (read-only)
│   ├── architect.agent.md      # Makes architecture decisions
│   ├── testing.agent.md        # Writes and runs tests
│   └── review.agent.md         # Code review + verification
├── skills/                     # Add your own skills here (gitignored)
└── AGENTS.md                   # Coding standards & build commands
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
