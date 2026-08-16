# Contributing to ELIZA

Thanks for your interest in contributing. ELIZA is a personal infrastructure project evolving toward a general-purpose open-source platform, so expect some rough edges — and expect the maintainer to be opinionated about architectural consistency (see [docs/DECISIONS.md](docs/DECISIONS.md)).

## Development Workflow

1. **Check existing issues/discussions** before starting significant work, to avoid duplicate effort.
2. **Fork the repository** and create a feature branch from `main`.
3. **Set up your dev environment** per the [README's Development Setup](README.md#development-setup) section.
4. **Write tests** for new functionality — unit tests for logic, integration tests for tool/agent interactions.
5. **Update documentation** relevant to your change (architecture, agents, requirements — whichever apply). Documentation is not optional for non-trivial changes.
6. **Record architecture decisions** in [docs/DECISIONS.md](docs/DECISIONS.md) for any change that affects system design, technology choice, or agent behavior.
7. **Open a pull request** against `main` following the process below.

## Branching Strategy

- `main` — always deployable; protected branch requiring PR review
- `feature/<short-description>` — new functionality
- `fix/<short-description>` — bug fixes
- `docs/<short-description>` — documentation-only changes
- `refactor/<short-description>` — non-functional code changes

Branch names should be short and descriptive: `feature/home-assistant-integration`, `fix/memory-agent-race-condition`.

## Commit Conventions

This project follows [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short summary>

[optional body]

[optional footer(s)]
```

**Types:** `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `ci`

**Examples:**
```
feat(agents): add Home Agent device control support

fix(memory): resolve race condition in fact conflict resolution

docs(architecture): add sequence diagram for voice round-trip
```

Scope should reference the affected component (`agents`, `memory`, `tools`, `voice`, `api`, `deploy`, etc.).

## Pull Request Process

1. Ensure all tests pass locally (`pytest`) and linting/type checks are clean (`ruff check .`, `mypy eliza/`).
2. Fill out the PR template, including:
   - What the change does and why
   - Which requirements/roadmap items it addresses (if applicable)
   - Any new dependencies introduced and why
   - Screenshots/traces for behavior changes where relevant (e.g., a Langfuse trace showing new agent behavior)
3. Keep PRs focused — one logical change per PR. Large architectural changes should be discussed via an issue/ADR draft before implementation.
4. At least one maintainer review is required before merge.
5. CI must pass (tests, lint, type check) before merge.
6. Squash-merge is preferred to keep `main` history clean; write a good final commit message summarizing the change.

## Code Style

- Python: formatted and linted via `ruff`, type-checked via `mypy`. Run `ruff format .` before committing.
- Prefer explicit over clever — this codebase optimizes for long-term maintainability over cleverness, consistent with the project's [core philosophy](docs/VISION.md#core-philosophy).
- New tool integrations should follow the existing tool interface pattern (see [docs/AGENTS.md](docs/AGENTS.md)) — don't introduce a one-off calling convention.

## Adding a New Integration

New integrations are one of the most valuable contributions. To add one:

1. Prefer building it as an MCP server if a reasonable one doesn't already exist for the target service (see [docs/DECISIONS.md](docs/DECISIONS.md) ADR-0004).
2. Document the integration's capabilities, required credentials/scopes, and confirmation requirements (does it need human-in-the-loop per [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) FR-4?).
3. Add integration tests covering both success and failure/unavailable paths — the Tool Agent must degrade gracefully if your integration is down.
4. Update [docs/AGENTS.md](docs/AGENTS.md) and [README.md](README.md) feature lists.

## Questions

Open a GitHub Discussion for design questions or proposals before investing significant implementation time — this helps avoid wasted work on directions that don't fit the project's architecture or philosophy.
