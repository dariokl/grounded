# AGENTS.md

Agent instructions for this JavaScript/TypeScript web development project.

> For general project context and workflow overview, see [README.md](README.md).

## Coding Standards

### TypeScript

- Strict mode, no `any` - use `unknown` + type guards
- Explicit return types on exported functions
- Use `type` imports: `import type { X }` for type-only imports
- Prefer `interface` for object shapes, `type` for unions/intersections

### Modern Syntax (ES2022+)

- Optional chaining: `obj?.prop` instead of `obj && obj.prop`
- Nullish coalescing: `value ?? default` instead of `value || default`
- `using` declarations for resource cleanup
- Array methods: `at()`, `findLast()`, `toSorted()`, `toReversed()`
- `Object.hasOwn()` instead of `hasOwnProperty`

### Async

- Prefer `async/await` over `.then()` chains
- Always handle errors - no silent catches
- Use `Promise.all()` for parallel operations

### Naming

- Components: `PascalCase` (`UserList.tsx`)
- Functions/hooks: `camelCase` (`fetchUsers`, `useAuth`)
- Types/interfaces: `PascalCase` (`UserData`)
- Constants: `SCREAMING_SNAKE_CASE` (`API_URL`)

## Dev Environment

- This is a JS/TS web development project
- Uses custom agents defined in `.github/agents/`
- See `README.md` for workflow documentation

## Build Commands

```bash
# Install dependencies
npm install
# or
pnpm install

# Run development server
npm run dev

# Build for production
npm run build

# Type check
npx tsc --noEmit
```

## Testing

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run specific test file
npm test -- path/to/test.ts
```

## Linting

```bash
# Run linter
npm run lint

# Fix lint issues
npm run lint:fix
```

## Validation Before Commit

Always run these before committing:

1. `npm run lint` - Fix any linting errors
2. `npx tsc --noEmit` - Fix any type errors
3. `npm test` - Ensure all tests pass
4. `npm run build` - Verify build succeeds

## PR Instructions

- Title format: `[feature/fix/chore] Brief description`
- Include tests for new functionality
- Ensure CI passes before requesting review

## Project Structure

```
.github/
├── agents/          # Custom agent definitions (.agent.md)
├── skills/          # Domain-specific knowledge (SKILL.md) - add your own
src/                 # Source code
tests/
├── unit/            # Unit tests
└── integration/     # Integration tests
```

## Custom Agents

This project uses custom Copilot agents for structured workflows:

- `@planner` - **Start here** - Creates/updates `roadmap.json` and hands off to Orchestrator
- `@orchestrator` - Runs the roadmap loop, dispatches sub-agents, validates `### Orchestrator Contract` signatures, and persists learnings to `roadmap.json`
- `@research` - Gathers codebase evidence (read-only, no recommendations)
- `@architect` - Two modes: **design** (architecture decisions + ADRs) and **validation** (checks implementation drift on complex items)
- `@testing` - Writes/runs tests and reports pass/fail evidence
- `@review` - Sole verification gate: code review + lint/typecheck/build + auto-fix for trivial issues

For implementation work, the **built-in Copilot coding agent** is dispatched by Orchestrator; for test-file-focused items, Orchestrator dispatches `@testing` as the implementation step.

## Skills

Add your own skills in `.github/skills/`. See [GitHub Copilot Skills documentation](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills).

See `README.md` for detailed workflow documentation.
