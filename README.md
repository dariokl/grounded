# Grounded

My personal agentic workflow setup for JS/TS web development using **GitHub Copilot Custom Agents**.

> **Note:** This is an experimental setup I use to improve my Copilot workflow. It is a work in progress, and the agents usually need tweaking per project and stack.

### Prerequisites

- [VS Code](https://code.visualstudio.com)
- [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) extension
- GitHub Copilot subscription

### Installation

1. **Copy the `.github` folder and `AGENTS.md`** to your project root:

   ```bash
   # Clone this repo
   git clone https://github.com/YOUR_USERNAME/grounded.git

   # Copy to your project
   cp -r grounded/.github grounded/AGENTS.md your-project/
   ```

2. **Customize for your project:**
   - Edit `AGENTS.md` with your coding standards and build commands
   - This repo's `AGENTS.md` is JS/TS-oriented; for non-TypeScript projects, create your own `AGENTS.md` instead of copying it as-is
   - Add skills in `.github/skills/` for your tech stack

3. **Open VS Code** in your project and start using agents with `@planner`

### What You Get

```
your-project/
├── .github/
│   ├── agents/           # 7 specialized agents
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
│  - Triages request (simple/medium/complex)                           │
│  - Creates implementation plans                                      │
│  - Routes to appropriate agents                                      │
└─────────────────────────────────────────────────────────────────────┘
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
         ▼                         ▼                         ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ 🔍 RESEARCH     │    │ 🏗️ ARCHITECT    │    │ ⚡ IMPLEMENT    │
│ (read-only)     │    │ (creates types) │    │ (full access)   │
│ - Find patterns │    │ - Design arch   │    │ - Write code    │
│ - Analyze code  │    │ - ADRs          │    │ - Create files  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                         │                         │
         └─────────────────────────┼─────────────────────────┘
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      IMPLEMENT AGENT                                 │
│  - Writes production code                                            │
│  - Creates files and components                                      │
│  - Has terminal access                                               │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    ▼                              ▼
┌──────────────────────────┐      ┌──────────────────────────┐
│      🧪 TESTS AGENT      │      │      🔍 REVIEW AGENT     │
│  - Unit tests            │      │  - Code quality          │
│  - Integration tests     │      │  - Security review       │
│  - Runs test suite       │      │  - Best practices        │
└──────────────────────────┘      │  - Type check & lint     │
                    │             │  - Build verification    │
                    └─────────────┴──────────────────────────┘
```

## Custom Agents

All agents are defined in `.github/agents/` using the `.agent.md` format with YAML frontmatter.

### Agent Overview

| Agent            | Purpose                                   | Tools           | Handoffs To                    |
| ---------------- | ----------------------------------------- | --------------- | ------------------------------ |
| **Orchestrator** | Entry point for complex multi-agent tasks | read + agent    | Planner, Research              |
| **Planner**      | Triage, implementation plans              | read-only       | Research, Architect, Implement |
| **Research**     | Finds codebase patterns                   | read-only       | Architect, Planner             |
| **Architect**    | System design, ADRs, trade-offs           | read + write    | Implement, Research            |
| **Implement**    | Writes code                               | full access     | Review, Tests                  |
| **Tests**        | Writes tests                              | full access     | Review                         |
| **Review**       | Code review + verification                | read + terminal | Implement, Planner             |

### Tool Restrictions

Agents have intentionally restricted tool access:

- **Read-only agents** (Planner, Research, Review): Can search and analyze but NOT modify files
- **Write agents** (Architect, Implement, Tests): Can create and edit files
- **Terminal agents** (Implement, Tests, Integration): Can run commands

### Triage Flow

The Planner agent automatically routes based on complexity:

| Complexity  | Criteria                         | Action                               |
| ----------- | -------------------------------- | ------------------------------------ |
| **Simple**  | Single file, clear change        | ⚡ Quick Implement                   |
| **Medium**  | Multiple files, needs context    | Create plan → Implement              |
| **Complex** | New feature, architecture needed | 🔍 Research or 🏗️ Architecture first |

## Usage

### Select an Agent in VS Code

1. Open Copilot Chat
2. Click the agent dropdown (or type `@`)
3. Select an agent (e.g., `@planner`)
4. Type your request

### Workflow Example

```
@planner Implement a user authentication feature with login/logout
```

The Planner will:

1. Assess complexity (simple/medium/complex)
2. For complex tasks: Show handoff buttons like "🔍 Research First" or "🏗️ Start Architecture"
3. For simple tasks: Skip directly to "⚡ Quick Implement"
4. You click the appropriate button to continue the workflow

### Direct Agent Usage

```
@planner Create a plan for adding dark mode support
@research Find all authentication-related code in this project
@architect Design the data model for user subscriptions
@implement Create a Button component following existing patterns
@review Check the UserService for security issues
```

## Handoffs

Agents use **handoffs** to guide you through the workflow. After each response, you'll see buttons like:

- � Research First
- 🏗️ Start Architecture
- ⚡ Quick Implement
- 🧪 Write Tests
- 🔍 Review Code
- 📋 Back to Planner

Click a button to transition to the next agent with context preserved.

## Directory Structure

```
.github/
├── agents/
│   ├── orchestrator.agent.md   # Complex multi-agent coordination
│   ├── planner.agent.md        # Entry point - triage & planning
│   ├── research.agent.md       # Codebase analysis
│   ├── architect.agent.md      # System design
│   ├── implement.agent.md      # Code writing
│   ├── tests.agent.md          # Test writing
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
- AGENTS.md limititations [VS Code Custom Instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)

## License

[MIT](LICENSE)
