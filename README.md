# Agent Skills

A collection of agent skills that extend capabilities across debugging, testing, and development workflow.

Install an individual skill with:

```bash
npx skills@latest add <github-owner>/skills/<skill-folder>
```

Replace `<github-owner>` with your GitHub username or organization.

## Debugging & Investigation

These skills help you investigate failures before reaching for fixes.

- **systematic-debugging** — Apply a four-phase debugging framework that enforces root cause investigation, pattern analysis, hypothesis testing, and implementation.

  ```bash
  npx skills@latest add <github-owner>/skills/systematic-debugging
  ```

- **root-cause-tracing** — Trace bugs backward through the call stack to find the original trigger, add instrumentation, and fix the source instead of the symptom.

  ```bash
  npx skills@latest add <github-owner>/skills/root-cause-tracing
  ```

## Development Workflow

These skills help you work in smaller, safer, and more reviewable steps.

- **testing** — Apply pragmatic testing guidance with an integration-first bias, behavior-focused assertions, and lighter use of mocks.

  ```bash
  npx skills@latest add <github-owner>/skills/testing
  ```

- **git-commit** — Create clean git commits with emoji conventional commit messages, staged-file awareness, and guidance for splitting mixed changes.

  ```bash
  npx skills@latest add <github-owner>/skills/git-commit
  ```

## Structure

Each skill lives in a top-level folder so it can be installed directly by path:

- `git-commit/`
- `root-cause-tracing/`
- `systematic-debugging/`
- `testing/`

Each folder contains a required `SKILL.md` file and may include supporting resources such as reference docs or scripts.
