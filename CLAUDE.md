# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

@AGENTS.md

`AGENTS.md`, imported above, is the canonical agent guide for this repo and is
where changes belong: repository structure, the `enterprise/` directory, PR
template, conversation-state/frontend implementation details, microagents, LLM
model handling, and the lockfile/GitHub-Actions rules all live there. This file
loads it (Claude Code auto-reads `CLAUDE.md`, not `AGENTS.md`) and adds the
short orientation below.

## What This Is

**OpenHands** is an automated AI software engineer: a Python backend
(`openhands/`) plus a React/Vite frontend (`frontend/`), with a separate
`enterprise/` tree layering auth (Keycloak), billing (Stripe), integrations
(GitHub/GitLab/Jira/Linear/Slack), migrations, and telemetry on top of the
open-source core. This repo is `RJHuey73/OpenHands`, a fork.

Two things about the current shape are easy to get wrong:

- **V1 is the live application server** (`openhands/app_server/`), even though
  `make start-backend` still launches `openhands.server.listen:app` — that
  includes the V1 routes by default unless `ENABLE_V1=0`.
- **`openhands-sdk` (from the separate `software-agent-sdk` repo, pinned in
  `pyproject.toml`) owns verified models, providers, and bare-name→provider
  assignment.** The backend no longer has `openhands/cli/`, `openhands/llm/`, or
  `openhands/utils/` — that code moved out. Model additions/reclassifications
  belong in the SDK repo, not here; `.agents/skills/update-sdk/` documents
  pulling in a new SDK version.

## Before you change anything

```bash
make install-pre-commit-hooks     # ALWAYS, before the first edit
```

Then, before pushing:

```bash
# backend changes (runs on staged files)
pre-commit run --config ./dev_config/python/.pre-commit-config.yaml

# frontend changes
cd frontend && npm run lint:fix && npm run build; cd ..
```

Both must pass — they auto-fix some issues, so re-run after fixing the rest.

## Commands

```bash
make build                      # full repo setup (frontend + backend); only when needed
make run                        # full app — see AGENTS.md for the sandbox-safe invocation

poetry run pytest tests/unit/test_xxx.py      # backend tests (pytest, mirrors openhands/)
cd frontend && npm run test                    # frontend tests (vitest)
cd frontend && npm run test -- -t "TestName"   # single frontend test
cd frontend && npm run build                    # production build
cd frontend && npm run make-i18n                # regenerate i18n declarations
```

`enterprise/` has its own Poetry project, pre-commit config, and test
invocation — see `AGENTS.md` § "Enterprise Directory" rather than assuming the
root commands cover it.

## Conventions worth repeating

- **Frontend data flow is one-directional**: UI components → TanStack Query
  hooks (`frontend/src/hooks/query/`, `hooks/mutation/`) → the data-access layer
  (`frontend/src/api`) → endpoints. Never call an `api/` method straight from a
  component. Query hooks are `use[Resource]`, mutations are `use[Action]`.
- **Frontend state is Zustand** (`frontend/src/stores/`), not Redux — there is
  no `chat-slice.ts`.
- **Adding a UI action type** starts with an `ACTION_MESSAGE$ACTION_NAME` i18n
  key, which `getEventContent()` picks up automatically; only extend
  `get-action-content.ts` / `get-observation-content.ts` when the message needs
  custom formatting.
- **Pin third-party GitHub Actions to a 40-char SHA** with the version in a
  trailing comment. `actions/*`, `github/*`, and `OpenHands/*` are exempt.
- **Regenerate lockfiles with the tool version that wrote them** (the version is
  in each lockfile's header) so diffs contain dependency changes only.
- **`git add <file>`, not `git add .`** — and be careful with `git reset --hard`
  after staging.
- **PR-only artifacts (design docs, dev scripts) go in `.pr/`** at the repo root
  so they never merge to main.

## Skills in this repo

Three distinct directories, easy to confuse:

| Path | Audience |
|------|----------|
| `skills/` (formerly `microagents/`) | Public skills/microagents shipped to OpenHands **end users**. |
| `.openhands/microagents/` | Repository microagents for *this* repo's own OpenHands runs (legacy `.openhands/skills/` is still read). |
| `.agents/skills/` | This repo's **Claude Code** skills, for working on the OpenHands codebase itself. |

"Microagent" is V0 terminology and "Skill" is V1 terminology for the same
underlying files — see `skills/README.md`.
