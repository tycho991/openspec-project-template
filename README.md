# openspec-project-template

Vorlage für Projekte mit [OpenSpec](https://github.com/Fission-AI/OpenSpec) und
OpenCode 2. Die Maschinen-Seite (mise, Node, OpenCode, globale Skills) kommt
aus `ansible-terminal` (`playbooks/dev_tools.yml`). Planung und Hintergründe:
`Planung OpenSpec + OpenCode Multi-Maschinen-Setup v3.md`.

## Inhalt

| Datei | Zweck |
|---|---|
| `.mise.toml` | Node und OpenSpec exakt gepinnt, Telemetrie aus, Task `openspec-sync` |
| `opencode.json` | Sharing aus, OpenSpec-Skills (`openspec-*`) ohne Rückfrage |
| `renovate.json` | Updates per PR, 7 Tage Mindestalter, Node-Majors nur nach Freigabe |
| `.github/workflows/openspec-drift.yml` | prüft, dass die committeten Skills genau zur gepinnten Version passen |

## Neues Projekt

```bash
gh repo create tycho991/<name> --private --template tycho991/openspec-project-template --clone
cd <name>
mise trust
mise install
mise run openspec-sync
openspec init --tools opencode
git status
```

`openspec-sync` vor `init` setzt Profil und Delivery, bevor die ersten Dateien
entstehen. Danach gezielt committen:

```bash
git add openspec .opencode
git commit -m "OpenSpec einrichten"
```

`mise trust` ist pro Projekt einmal nötig. Ein pauschales Vertrauen für ganze
Verzeichnisse (`trusted_config_paths`) ist bewusst nicht eingerichtet.

## Bestehendes Projekt

Die vier Dateien aus dieser Vorlage ins Projekt kopieren (den Workflow nach
`.github/workflows/`), dann wie oben ab `mise trust`. Hat das Projekt schon
einen `openspec/`-Ordner, statt `openspec init` nur `mise run openspec-sync`.

## OpenSpec aktualisieren

Renovate öffnet einen PR, der `.mise.toml` erhöht. Im Branch dieses PRs:

```bash
mise install
mise run openspec-sync
git add -A && git commit -m "OpenSpec-Skills aktualisieren"
```

Erst dann wird die Drift-Prüfung grün. Für mehrere Projekte auf einmal:
`scripts/bump-openspec.sh` in `ansible-terminal`.

## Eigene Skills im Projekt

- Ordner `.opencode/skills/<id>/` mit genau einer `SKILL.md` direkt darin. Die
  ID ist der Ordnername: `^[a-z0-9]+(-[a-z0-9]+)*$`, nicht mit `openspec-`
  beginnend. `name` im Frontmatter ist nur Anzeigename.
- `description` im Frontmatter ist Pflicht, sonst bietet OpenCode den Skill
  nicht an.
- Keine losen `*.md` direkt in `.opencode/skills/` und keine weiteren
  `SKILL.md` in Unterordnern, OpenCode 2 machte daraus zusätzliche Skills.
  Die Drift-Prüfung meldet beides.
- Freigeben in `opencode.json` mit einer weiteren Regel:
  `{ "action": "skill", "resource": "<id>", "effect": "allow" }`. Keine
  breite Regel wie `skill *` hier eintragen: Projektregeln stehen hinter den
  globalen und würden die global freigegebenen Skills wieder auf Rückfrage
  setzen.

## Arbeitsablauf pro Änderung

1. Proposal mit OpenCode erstellen lassen (`/opsx-propose` o. ä., siehe Doku
   der gepinnten OpenSpec-Version)
2. `openspec validate <name>` und `openspec show <name>`
3. Änderung umsetzen lassen
4. `openspec archive <name> --yes`
