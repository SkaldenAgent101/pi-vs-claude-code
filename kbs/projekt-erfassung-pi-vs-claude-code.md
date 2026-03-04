# Projekt-Erfassung: `pi-vs-claude-code`

**Stand:** 2026-03-04  
**Fork (origin):** `git@github.com:SkaldenAgent101/pi-vs-claude-code.git`  
**Branch/Commit-Snapshot:** `main` @ `083e188decee62b6050ba5ee55cca8f0801678f8`

---

## 1) Kurzüberblick

`pi-vs-claude-code` ist ein Playground für **Pi Coding Agent Extensions**. Das Repo demonstriert, wie weit sich Pi in Richtung:

- UI/UX-Anpassung (Footer, Widgets, Overlays, Theme-Wechsel)
- Sicherheits-Gates (Damage Control)
- Agenten-Orchestrierung (Subagents, Teams, Chains, Meta-Agent)
- Prompt-/System-Persona-Steuerung

ausbauen lässt.

**Stack:** Bun + just + pi  
**Entry-Pattern:** `pi -e extensions/<name>.ts` oder `just ext-...`

---

## 2) Extension-Katalog (pro Datei)

> Fokus: Zweck, registrierte Tools/Commands/Shortcuts, zentrale Events.

| Extension | Zweck | Registrierungen | Wichtige Events/Mechanik | Quelle |
|---|---|---|---|---|
| `pure-focus.ts` | Entfernt Footer/Statusline für „distraction-free mode“ | – | setzt leeren Footer bei Session-Start | `extensions/pure-focus.ts` |
| `minimal.ts` | Kompakter Footer mit Modell + Context-Bar `[###-------]` | – | custom Footer auf `session_start` | `extensions/minimal.ts` |
| `purpose-gate.ts` | Erzwingt Session-Zweck vor erster Arbeit | – | `session_start`: Prompt nach Zweck, `input`: blockiert ohne Zweck, `before_agent_start`: injiziert Zweck in System-Prompt | `extensions/purpose-gate.ts` |
| `tool-counter.ts` | 2-zeiliger Footer mit Modell/Context, Tokens/Kosten, CWD/Branch, Tool-Tally | – | `tool_execution_end` zählt Tools, `session_start` setzt Footer mit Branch-Updates | `extensions/tool-counter.ts` |
| `tool-counter-widget.ts` | Live-Widget mit farbigen Tool-Call-Kacheln | – | `tool_execution_end` erhöht Zähler, `session_start` setzt Widget | `extensions/tool-counter-widget.ts` |
| `theme-cycler.ts` | Theme-Wechsel per Keys + Command + kurzer Swatch-Preview | `/theme`, Shortcuts: `ctrl+x`, `ctrl+q` | `session_start` Status, `session_shutdown` Timer-Cleanup | `extensions/theme-cycler.ts` |
| `system-select.ts` | Auswahl eines Agent-Persona-Prompts aus `.pi/.claude/.gemini/.codex` (lokal+global) | `/system` | `session_start` scannt Agents + default Tools, `before_agent_start` prepended Persona-Prompt, optional Tool-Restriktion via Frontmatter | `extensions/system-select.ts` |
| `session-replay.ts` | Timeline-Overlay der Session-Historie mit Navigation/Expand | `/replay` | custom Overlay UI, `session_start` nur Defaults | `extensions/session-replay.ts` |
| `damage-control.ts` | Safety-Interception für Tools via YAML-Regeln | – | `session_start` lädt `.pi/damage-control-rules.yaml`, `tool_call` blockt/confirmt gefährliche Calls + Audit-Log | `extensions/damage-control.ts`, `.pi/damage-control-rules.yaml` |
| `cross-agent.ts` | Discovery/Bridge für `.claude/.gemini/.codex` Commands/Skills/Agents | dynamisch: `/cmd`, `/skill:<name>` | Registriert Commands **synchron beim Laden**; Startup-Panel zeigt gefundene Assets | `extensions/cross-agent.ts` |
| `subagent-widget.ts` | Hintergrund-Subagents mit persistenten Sessions + Live-Widgets | Tools: `subagent_create`, `subagent_continue`, `subagent_remove`, `subagent_list`; Commands: `/sub`, `/subcont`, `/subrm`, `/subclear` | startet `pi`-Subprozesse (`--mode json`), streamt Fortschritt, sendet Result als Follow-up | `extensions/subagent-widget.ts` |
| `tilldone.ts` | Task-Disziplin: erst planen, dann arbeiten; erzwingt Task-Statusfluss | Tool: `tilldone` (`new-list/add/toggle/remove/update/list/clear`), Command: `/tilldone` | `tool_call` blockiert alle anderen Tools ohne aktive Tasks; Footer+Widget+Nudge auf `agent_end` | `extensions/tilldone.ts` |
| `agent-team.ts` | Dispatcher-Orchestrator: Primär-Agent delegiert alles an Spezialisten | Tool: `dispatch_agent`; Commands: `/agents-team`, `/agents-list`, `/agents-grid` | Teamauswahl aus `teams.yaml`, Grid-Widget, Subprozess pro Agent mit Session-Fortsetzung | `extensions/agent-team.ts`, `.pi/agents/teams.yaml` |
| `agent-chain.ts` | Sequenzielle Pipelines mit `$INPUT`/`$ORIGINAL` über mehrere Agents | Tool: `run_chain`; Commands: `/chain`, `/chain-list` | lädt Chains aus YAML, führt Schritte nacheinander aus, visualisiert Step-Status im Widget | `extensions/agent-chain.ts`, `.pi/agents/agent-chain.yaml` |
| `pi-pi.ts` | Meta-Agent: parallele Expertenrecherche zum Bauen neuer Pi-Artefakte | Tool: `query_experts`; Commands: `/experts`, `/experts-grid` | startet Experten parallel als Subprozesse, baut System-Prompt aus `pi-orchestrator.md` | `extensions/pi-pi.ts`, `.pi/agents/pi-pi/*.md` |
| `themeMap.ts` | Shared Helper: Extension→Theme Mapping + Titelsetzung | – | `applyExtensionDefaults(import.meta.url, ctx)` in fast allen Extensions | `extensions/themeMap.ts` |

---

## 3) Multi-Agent-Konfiguration im Repo

### Teams (`.pi/agents/teams.yaml`)

Definiert Team-Sets für `agent-team`:

- `full`: scout, planner, builder, reviewer, documenter, red-team
- `plan-build`: planner, builder, reviewer
- `info`: scout, documenter, reviewer
- `frontend`: planner, builder, bowser
- `pi-pi`: ext/theme/skill/config/tui/prompt/agent-expert

### Chains (`.pi/agents/agent-chain.yaml`)

Definiert Pipeline-Workflows für `agent-chain`:

- `plan-build-review`
- `plan-build`
- `scout-flow`
- `plan-review-plan`
- `full-review`

---

## 4) Bedienung / Einstieg

### Direkt

```bash
pi -e extensions/minimal.ts
pi -e extensions/agent-team.ts
pi -e extensions/pi-pi.ts
```

### Über `just`

```bash
just
just ext-minimal
just ext-agent-team
just ext-agent-chain
just ext-pi-pi
```

---

## 5) Quellen (mit Herkunft)

### A) Repo-Fork / Remote

- `git remote -v` → `origin = git@github.com:SkaldenAgent101/pi-vs-claude-code.git`
- Snapshot-Commit: `083e188decee62b6050ba5ee55cca8f0801678f8`

### B) Ausgewertete Projektdokumente

- `README.md`
- `justfile`
- `CLAUDE.md`
- `THEME.md`
- `TOOLS.md`
- `RESERVED_KEYS.md`
- `specs/agent-forge.md`
- `specs/agent-workflow.md`
- `specs/damage-control.md`
- `specs/pi-pi.md`
- `.claude/commands/prime.md`

### C) Ausgewertete Codequellen

- `extensions/*.ts` (alle 16 Dateien)
- `.pi/settings.json`
- `.pi/agents/teams.yaml`
- `.pi/agents/agent-chain.yaml`
- `.pi/damage-control-rules.yaml`
- `.pi/agents/pi-pi/*.md` (indirekt über Extension-Logik/Struktur)

---

## 6) Einordnung

Dieses Fork-Repo ist ein gutes **Referenzlabor für Pi-Extension-Muster**:

- klein → groß (UI-Feinschliff bis Agent-Orchestrierung)
- synchron registrierte Commands/Tools als Best Practice
- starke Kombination aus **Governance (damage-control, tilldone)** und **Autonomie (subagent, team, chain, pi-pi)**

Damit eignet es sich sehr gut als Basis, um eigene produktive Pi-Agenten aufzubauen.
