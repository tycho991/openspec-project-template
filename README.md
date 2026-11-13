# openspec-project-template

Template for projects using [OpenSpec](https://github.com/Fission-AI/OpenSpec)
with Claude Code in VS Code (extension `anthropic.claude-code`) or, as an
option, [OpenCode](https://opencode.ai) in the terminal. OpenCode runs on the
Claude subscription through its provider "Claude Agent", which starts the local
Claude Code; Anthropic has not approved that route publicly. The machine side
(mise, Node, Claude Code and OpenCode settings, global skills, the `pj` command) comes from
`ansible-terminal` (`playbooks/dev_tools.yml`). Background and decisions:
[docs/dev-tools.md in ansible-terminal](https://github.com/tycho991/ansible-terminal/blob/main/docs/dev-tools.md).

## Contents

| File | Purpose |
|---|---|
| `.mise.toml` | Node and OpenSpec pinned exactly, telemetry off, tasks `openspec-profile` and `openspec-sync` |
| `.gitignore` | keeps `.claude/settings.local.json` (personal approvals) and openrig's `.openrig/` local |
| `renovate.json` | updates by PR, 7-day minimum age, Node majors only after approval |
| `.github/workflows/openspec-drift.yml` | checks that the committed skills and commands match the pinned version exactly |

## New project

```bash
pj new <name>
```

`pj` creates `tycho991/<name>` as a private repo from this template, clones it to
`~/Downloads/Code/<name>`, sets up mise and OpenSpec (Node and OpenSpec pinned,
`openspec init --tools claude,opencode`), commits, pushes and opens the
project in VS Code. There, in the Claude Code panel: `/opsx:propose "…"`.
`pj new <name> --opencode` starts OpenCode in the terminal instead (there the
command is `/opsx-propose "…"`), `pj new <name> --claude` Claude Code in the
terminal.

By hand, without `pj`:

```bash
gh repo create tycho991/<name> --private --template tycho991/openspec-project-template --clone
cd <name> && mise trust && mise install && mise run openspec-profile && openspec init --tools claude,opencode
git add openspec .claude .opencode && git commit -m "Set up OpenSpec" && git push
```

`openspec-profile` sets the profile and delivery before the first files are
generated; `openspec-sync` calls it itself afterwards every time. OpenSpec 1.14.3
creates:

- `openspec/` with specs, changes and `config.yaml`
- `.claude/commands/opsx/<id>.md`, available in Claude Code as `/opsx:<id>`
- `.claude/skills/openspec-*/` for the skills
- `.opencode/commands/opsx-<id>.md`, available in OpenCode as `/opsx-<id>`
- `.opencode/skills/openspec-*/`, the same skills for OpenCode

`openspec-sync` keeps both sets in line with the pinned version.

`mise trust` is needed once per project. Blanket trust for whole directories
(`trusted_config_paths`) is deliberately not set up.

## Open an existing project

```bash
pj open                      # pick from local and GitHub projects
pj open frame                # or with a search term
pj open frame --opencode     # OpenCode in this terminal instead of VS Code
pj open frame --claude       # Claude Code in this terminal
```

Clones if needed, updates (`git pull --ff-only`), sets up mise and opens
VS Code (default, `--code`), OpenCode or Claude Code. If the project does not use OpenSpec yet, run
`pj openspec` in the project folder: it adds the files from this template and
commits them on the branch `openspec-setup` (not pushed). If it uses OpenSpec
but has no `.opencode/` yet, `pj openspec` adds the OpenCode files instead and
commits them on the branch `openspec-opencode` (not pushed).

## Adding the template files to an existing project by hand

Copy `.mise.toml`, `.gitignore`, `renovate.json` and the workflow (to
`.github/workflows/`) into the project, then continue as above from
`mise trust`. If the project already has an `openspec/` folder for both tools,
run just `mise run openspec-sync` instead of `openspec init`; if it only has
the Claude Code files, `openspec init --tools claude,opencode` adds
`.opencode/` and leaves the rest alone.

## Updating OpenSpec

Renovate opens a PR that raises the version in `.mise.toml`. On that PR's
branch:

```bash
mise install
mise run openspec-sync
git add -A -- .mise.toml openspec .claude/commands '.claude/skills/openspec-*' .opencode
git status --short    # anything left over is not from OpenSpec
git commit -m "Update OpenSpec skills"
```

Only then does the drift check turn green. The explicit paths keep files from
other tools out of the commit, such as the skills and `CLAUDE.md` guidance an
[openrig](https://github.com/mvschwarz/openrig) team puts into the project.
The quotes matter: git expands `openspec-*` itself and so also stages skills
that the new version removed. In a project without `.opencode/`, leave that
path out, otherwise git stops with "pathspec did not match". For several projects at once, use
`scripts/bump-openspec.sh` in `ansible-terminal`.

## Your own skills in a project

- Folder `.claude/skills/<name>/` with a `SKILL.md`. The folder name is the
  skill name: `^[a-z0-9]+(-[a-z0-9]+)*$`, not starting with `openspec-`.
- `description` in the frontmatter is required, otherwise Claude does not pick
  the skill on its own. The drift check reports both.
- Personal skills (`~/.claude/skills/`) take precedence over project skills of
  the same name.
- OpenCode loads `.claude/skills/` and `~/.claude/skills/` as well, so a skill
  there serves both tools. OpenCode-only skills go to `.opencode/skills/<name>/`
  under the same rules; the drift check covers both folders.

## Workflow per change

1. `/opsx:propose` in the Claude Code panel (proposal, tasks and spec deltas
   under `openspec/changes/<name>/`)
2. `openspec validate <name>` and `openspec show <name>`
3. `/opsx:apply` to implement
4. `/opsx:archive`, or `openspec archive <name> --yes`

In OpenCode the commands are `/opsx-propose`, `/opsx-apply` and
`/opsx-archive`. The exact commands are in `.claude/commands/opsx/` and
`.opencode/commands/` and depend on the pinned OpenSpec version.

With an [openrig](https://github.com/mvschwarz/openrig) team, step 3 can go to
the team instead:
`rig send dev-build@starter 'Implement openspec/changes/<name> with /opsx:apply, then have dev-review check it.'`
The team members are ordinary Claude Code sessions in the project and see the
same `/opsx:` commands. Setup and rules: the "openrig" section in
[ansible-terminal's docs/dev-tools.md](https://github.com/tycho991/ansible-terminal/blob/main/docs/dev-tools.md#openrig).
