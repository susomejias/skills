# Agent Skills

A collection of agent skills that extend capabilities across debugging, testing, and development workflow.

Install an individual skill with:

```bash
npx skills@latest add susomejias/skills/<skill-folder>
```

## Skills

### Debugging & Investigation

| Skill | Description | Install |
|-------|-------------|---------|
| `systematic-debugging` | Four-phase debugging framework: root cause investigation, pattern analysis, hypothesis testing, implementation | `npx skills@latest add susomejias/skills/systematic-debugging` |
| `root-cause-tracing` | Trace bugs backward through the call stack to find the original trigger and fix the source instead of the symptom | `npx skills@latest add susomejias/skills/root-cause-tracing` |
| `find-buggy-commit` | Find the exact commit that introduced a bug using binary search. Autonomous mode (auto-testing) or interactive mode (step-by-step) | `npx skills@latest add susomejias/skills/find-buggy-commit` |

### Development Workflow

| Skill | Description | Install |
|-------|-------------|---------|
| `git-commit` | Create clean git commits with emoji conventional commit messages, staged-file awareness, and split guidance | `npx skills@latest add susomejias/skills/git-commit` |
| `testing` | Pragmatic testing guidance with integration-first bias, behavior-focused assertions, and lighter use of mocks | `npx skills@latest add susomejias/skills/testing` |

### Planning

| Skill | Description | Install |
|-------|-------------|---------|
| `write-prd` | Create a PRD through user interview, codebase exploration, and module design. Saves to `.prds/` as a dated markdown file | `npx skills@latest add susomejias/skills/write-prd` |

## Structure

Each skill lives in a top-level folder and contains a required `SKILL.md` file with optional supporting resources (reference docs, scripts).
