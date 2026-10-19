# openspec-project-template

Template for projects using [OpenSpec](https://github.com/Fission-AI/OpenSpec)
and Claude Code in VS Code (extension `anthropic.claude-code`). The machine side
(mise, Node, Claude Code settings, global skills, the `pj` command) comes from
`ansible-terminal` (`playbooks/dev_tools.yml`). Background:
`Planung OpenSpec + Claude Code Multi-Maschinen-Setup v4.md`.

## Contents

| File | Purpose |
|---|---|
| `.mise.toml` | Node and OpenSpec pinned exactly, telemetry off, tasks `openspec-profile` and `openspec-sync` |
| `.gitignore` | keeps `.claude/settings.local.json` (personal approvals) local |
| `renovate.json` | updates by PR, 7-day minimum age, Node majors only after approval |
| `.github/workflows/openspec-drift.yml` | checks that the committed skills and commands match the pinned version exactly |

## New project

```bash
pj new <name>
```

`pj` creates `tycho991/<name>` as a private repo from this template, clones it to
`~/Downloads/Code/<name>`, sets up mise and OpenSpec (Node and OpenSpec pinned,
`openspec init --tools claude`), commits, pushes and opens the project in
VS Code. There, in the Claude Code panel: `/opsx:propose "…"`.

By hand, without `pj`:

```bash
gh repo create tycho991/<name> --private --template tycho991/openspec-project-template --clone
cd <name> && mise trust && mise install && mise run openspec-profile && openspec init --tools claude
git add openspec .claude && git commit -m "Set up OpenSpec" && git push
```

`openspec-profile` sets the profile and delivery before the first files are
generated; `openspec-sync` calls it itself afterwards every time. OpenSpec 1.14.3
creates:

- `openspec/` with specs, changes and `config.yaml`
- `.claude/commands/opsx/<id>.md`, available in Claude Code as `/opsx:<id>`
- `.claude/skills/openspec-*/` for the skills

`mise trust` is needed once per project. Blanket trust for whole directories
(`trusted_config_paths`) is deliberately not set up.

## Open an existing project

```bash
pj open            # pick from local and GitHub projects
pj open frame      # or with a search term
```

Clones if needed, updates (`git pull --ff-only`), sets up mise and opens
VS Code. If the project does not use OpenSpec yet, run `pj openspec` in the
project folder: it adds the files from this template and commits them on the
branch `openspec-setup` (not pushed).

## Adding the template files to an existing project by hand

Copy `.mise.toml`, `.gitignore`, `renovate.json` and the workflow (to
`.github/workflows/`) into the project, then continue as above from
`mise trust`. If the project already has an `openspec/` folder, run just
`mise run openspec-sync` instead of `openspec init`.

## Updating OpenSpec

Renovate opens a PR that raises the version in `.mise.toml`. On that PR's
branch:

```bash
mise install
mise run openspec-sync
git add -A && git commit -m "Update OpenSpec skills"
```

Only then does the drift check turn green. For several projects at once, use
`scripts/bump-openspec.sh` in `ansible-terminal`.

## Your own skills in a project

- Folder `.claude/skills/<name>/` with a `SKILL.md`. The folder name is the
  skill name: `^[a-z0-9]+(-[a-z0-9]+)*$`, not starting with `openspec-`.
- `description` in the frontmatter is required, otherwise Claude does not pick
  the skill on its own. The drift check reports both.
- Personal skills (`~/.claude/skills/`) take precedence over project skills of
  the same name.

## Workflow per change

1. `/opsx:propose` in the Claude Code panel (proposal, tasks and spec deltas
   under `openspec/changes/<name>/`)
2. `openspec validate <name>` and `openspec show <name>`
3. `/opsx:apply` to implement
4. `/opsx:archive`, or `openspec archive <name> --yes`

The exact commands are in `.claude/commands/opsx/` and depend on the pinned
OpenSpec version.
