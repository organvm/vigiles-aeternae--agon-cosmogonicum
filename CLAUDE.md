# CLAUDE.md — Vigiles Aeternae — Agon Cosmogonicum

## What This Is

Vigiles Aeternae is a **mythological governance simulation engine** for the ORGANVM system. It maps 22 regimes drawn from world mythologies and fiction to concrete audit operations against live system state (registry, seeds, metrics). Each regime embodies a distinct governance philosophy and produces findings in its own narrative voice. Eight Watcher Orders provide domain-specific audit perspectives that cut across regime boundaries.

The central concept is the **Agon** — a competitive deliberative cycle where multiple regimes audit the same system state, producing divergent findings. Where all regimes agree, a Constitutional law candidate emerges. Where they conflict, the conflict is recorded as research. The Colosseum (`colosseum/`) documents the rules of engagement for this process.

Package name: `vigiles-engine`. CLI entry point: `vigiles`.

## Commands

```bash
# Install for development
cd /Users/4jp/Workspace/meta-organvm/vigiles-aeternae--agon-cosmogonicum
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

# Run tests
pytest tests/ -v
pytest tests/test_auditor.py::test_name -v   # single test

# Lint
ruff check src/

# CLI — run a single regime audit
vigiles colosseum run --regime malazan

# CLI — compare two or more regimes
vigiles colosseum compare --regimes malazan,tolkien,norse

# CLI — identify constitutional law candidates (full consensus)
vigiles colosseum consensus

# CLI — view audit chronicle by date
vigiles colosseum chronicle [--date YYYY-MM-DD]

# CLI — list all Watcher Orders
vigiles orders list

# CLI — inspect a specific Order
vigiles orders show architectorum
```

## Architecture

### `auditor.py` — Core audit engine

- `Finding` dataclass: a single audit finding (`rule_check`, `severity`, `target`, `description`, `regime`, `details`)
- `AuditReport` dataclass: complete report from one regime's Agon cycle — holds a list of `Finding` objects and auto-computes a severity summary
- `CHECK_REGISTRY`: global `dict[str, callable]` mapping check IDs to implementation functions
- `@register_check(check_id)`: decorator that registers a check function into `CHECK_REGISTRY`
- `run_audit(regime, registry, workspace)`: main entry point — iterates a regime's `audit_rules`, looks up each `rule.check` in `CHECK_REGISTRY`, calls the function, and collects all findings
- Built-in checks: `seed_mandate_alignment`, `promotion_readiness_genuine`, `constraint_respect`, `stale_status_without_proof`, `pressure_accumulation_unresolved`, `single_point_of_failure`, `role_definition_exists`, `documentation_honesty`, `knowledge_distribution`, `irf_completion_rate`

### `regime.py` — Regime loader

- `AuditRule` dataclass: `check` (str ID into `CHECK_REGISTRY`), `description`, `severity` (`critical|high|medium|low`)
- `Phaethon` dataclass: a regime's declared blind spot — `risk`, `symptom`, `corrective_regime`
- `HealthModel` dataclass: regime's definition of `healthy`, `stressed`, and `critical` system states
- `Narrative` dataclass: a regime's aesthetic identity — `voice`, `aesthetic`, `soundtrack`, `character_archetype`
- `Regime` dataclass: the full governance philosophy — holds all above plus `priorities` (weighted values), `violations` (named anti-patterns), `metalaws_acknowledged`, `forbidden_fusions`
- `Regime.from_yaml(path)`: classmethod loader from YAML
- `load_all_regimes(regimes_dir)`: loads all regime YAMLs from a directory

### `orders.py` — Watcher Order loader

- `WatcherOrder` dataclass: loaded from order YAML — `number`, `name`, `latin`, `symbol`, `domain`, `power`, `constraint`, `failure_mode`, `system_functions`
- Imports `AuditReport`, `Finding`, `_iter_repos`, `_load_registry` from `auditor.py` to run Order-perspective audits

### `consensus.py` — Constitutional law engine

- `ConstitutionalCandidate` dataclass: a finding confirmed by all regimes across at least two Agon cycles — holds `target`, `description`, `severity_ceiling`, `regimes_confirming`, `cycles_confirmed`, `status` (`candidate|enacted|rejected`)
- `find_consensus(chronicles_dir)`: scans all chronicle entries looking for findings where every regime that ran confirmed the same issue

### `divergence.py` — Cross-regime comparison

Compares findings from multiple regime audits and classifies divergences into:
- *Perspective divergence* — same fact, different interpretation (both preserved)
- *Priority conflict* — same fact, different severity (highest surfaced as ceiling)
- *Structural contradiction* — mutually exclusive claims (escalated to Cosmogonists)

### `chronicle.py` — Append-only audit history

Immutable JSONL record of every Agon cycle. The Witnesses' function in code. `record_cycle()` appends; `read_chronicle()` replays. Never edited, only extended.

### `phaethon.py` — Blind spot detection

Every regime declares its own weakness in its `phaethon` field. This module monitors audit findings against those declarations and raises `PhaethonWarning` when a regime's philosophy is producing the very harm it predicted. `detect_phaethon(reports, regimes)` returns a list of `PhaethonWarning` objects.

### `reporter.py` — Narrative report generator

`render_report(regime, report)`: renders an `AuditReport` in the regime's narrative voice, aesthetic, and character archetype. `render_divergence_report(result)`: renders a `DivergenceResult` as markdown.

### `cli.py` — CLI entry points

Two top-level command groups: `vigiles colosseum` (run, compare, consensus, chronicle) and `vigiles orders` (list, show). Wires together `run_audit`, `find_consensus`, `analyze_divergence`, `detect_phaethon`, `record_cycle`, `render_report`, and `render_divergence_report`.

## The 22 Regimes

| File | Regime Name | Cosmology Source |
|------|-------------|-----------------|
| `abrahamic--covenantal-hierarchy.yaml` | Covenantal Hierarchy | Abrahamic traditions |
| `african--distributed-divination.yaml` | Distributed Divination | African / Yoruba cosmology |
| `daoist--effortless-alignment.yaml` | Effortless Alignment | Daoism |
| `discworld--narrative-causality.yaml` | Narrative Causality | Terry Pratchett — Discworld |
| `dreamtime--participatory-creation.yaml` | Participatory Creation | Indigenous Australian — Dreamtime |
| `dune--prophylactic-tyranny.yaml` | Prophylactic Tyranny | Frank Herbert — Dune |
| `egyptian--ritual-maintenance.yaml` | Ritual Maintenance | Ancient Egyptian cosmology |
| `foundation--stochastic-governance.yaml` | Stochastic Governance | Isaac Asimov — Foundation |
| `greek-roman--succession-governance.yaml` | Succession Governance | Greek/Roman mythology |
| `hindu-buddhist--dharmic-cycle.yaml` | Dharmic Cycle | Hindu/Buddhist cosmology |
| `jojo--willpower-succession.yaml` | Willpower Succession | Hirohiko Araki — JoJo's Bizarre Adventure |
| `lost--custodial-selection.yaml` | Custodial Selection | J.J. Abrams — Lost |
| `lynch--permeability-collapse.yaml` | Permeability Collapse | David Lynch (Twin Peaks / Mulholland Drive) |
| `malazan--pressure-ascendancy.yaml` | Pressure Ascendancy | Steven Erikson — Malazan Book of the Fallen |
| `marvel-cosmic--hierarchical-jurisdiction.yaml` | Hierarchical Jurisdiction | Marvel Comics cosmic pantheon |
| `matrix--program-hierarchy.yaml` | Program Hierarchy | The Wachowskis — The Matrix |
| `mesoamerican--metabolic-exchange.yaml` | Metabolic Exchange | Mesoamerican (Aztec/Maya) cosmology |
| `norse--defiant-stewardship.yaml` | Defiant Stewardship | Norse mythology |
| `one-piece--distributed-truth.yaml` | Distributed Truth | Eiichiro Oda — One Piece |
| `tolkien--delegated-authority.yaml` | Delegated Authority | J.R.R. Tolkien |
| `wot--balance-and-taint.yaml` | Balance and Taint | Robert Jordan — Wheel of Time |
| `zelda--recursive-triadic.yaml` | Recursive Triadic | Nintendo — The Legend of Zelda |

## The 8 Watcher Orders

| Order | Name | Latin | Symbol | Domain | Power |
|-------|------|-------|--------|--------|-------|
| I | The Architects | Ordo Architectorum | △ | Design law — what structures must exist before anything is built | Define what IS: schemas, naming conventions, directory structures, immutable creation standards |
| II | The Oracles | Ordo Sibyllarum | ◉ | Predictive monitoring — pressure cartography, sensing where the system will break | Read system state and forecast; not prophecy but pressure mapping |
| III | The Seraphim | Ordo Seraphorum | ⛨ | Active protection — guarding core truths, the firewall function | Preserve what must not be corrupted; strength is refusal |
| IV | The Psychopomps | Ordo Psychopomporum | ⚷ | Transition — mediating movement between states, the gate function | Control passage: promotion, demotion, graduation, archival |
| V | The Smiths | Ordo Fabrorum Recursivorum | ∞ | Adversarial recursion — stress testing through replication, contradiction scanning, systemic infection | Find weaknesses by becoming the weakness |
| VI | The Witnesses | Ordo Testium Aeternum | ◎ | Immutable recording — append-only truth, the event log that cannot be edited | See everything and record faithfully; the ground truth |
| VII | The Cosmogonists | Ordo Demiurgorum | ☉ | Meta-systemic coherence — the system watching itself, the source code of reality | Perceive the whole; see the Colosseum from outside the Colosseum |
| VIII | The Nataraja | Ordo Nataraja | 🜂 | Sacred dissolution — necessary ending, renewal through elimination, graceful archival | End things that must end; every ending carries the seed of what comes next |

## Key Patterns

### Adding a new audit check

1. In `auditor.py`, define a function decorated with `@register_check("your_check_id")`:
   ```python
   @register_check("your_check_id")
   def check_your_thing(rule: AuditRule, registry: dict, workspace: Path, **_kwargs) -> list[Finding]:
       findings = []
       for organ_key, repo in _iter_repos(registry):
           if <condition>:
               findings.append(Finding(
                   rule_check=rule.check,
                   severity=rule.severity,
                   target=f"{organ_key}/{repo['name']}",
                   description="Human-readable description",
                   regime="",  # filled by run_audit caller
               ))
       return findings
   ```
2. The check is now available to any regime — wire it in a regime YAML (see below).

### Adding a new regime

1. Create `regimes/<cosmology>--<slug>.yaml` following the schema in `colosseum/regime-schema.yaml`.
2. Required top-level keys under `regime:`: `name`, `latin`, `cosmology`, `version`, `philosophy`, `priorities`, `violations`, `health_model`, `phaethon`, `audit_rules`, `narrative`, `metalaws_acknowledged`.
3. Each entry in `audit_rules` must name an existing check ID registered in `CHECK_REGISTRY`.

### Wiring a check into a regime YAML

In the regime's `audit_rules` block, add:
```yaml
audit_rules:
  - check: your_check_id
    description: "What this check means in this regime's philosophical terms"
    severity: critical   # critical | high | medium | low
```

The `check` value must exactly match a key registered via `@register_check` in `auditor.py`. The `severity` in the regime YAML overrides the check's default — the same check can be `critical` in one regime and `medium` in another.

### The Agon loop (how it all fits together)

```
vigiles colosseum run --regime <name>
  → load_all_regimes()        # regime.py: parse YAML into Regime dataclasses
  → run_audit()               # auditor.py: look up each rule.check in CHECK_REGISTRY, collect Findings
  → render_report()           # reporter.py: markdown in the regime's narrative voice
  → record_cycle()            # chronicle.py: append-only JSONL record

vigiles colosseum compare --regimes a,b,c
  → run_audit() × N          # one AuditReport per regime
  → analyze_divergence()     # divergence.py: classify agreements and conflicts
  → detect_phaethon()        # phaethon.py: flag blind spot manifestation
  → render_divergence_report()

vigiles colosseum consensus
  → read_chronicle()         # chronicle.py: replay all past cycles
  → find_consensus()         # consensus.py: ConstitutionalCandidates where all regimes agree
```

<!-- ORGANVM:AUTO:START -->
## System Context (auto-generated — do not edit)

**Organ:** META-ORGANVM (Meta) | **Tier:** standard | **Status:** LOCAL
**Org:** `meta-organvm` | **Repo:** `vigiles-aeternae--agon-cosmogonicum`

### Edges
- *No inter-repo edges declared in seed.yaml*

### Siblings in Meta
`.github`, `organvm-corpvs-testamentvm`, `alchemia-ingestvm`, `schema-definitions`, `organvm-engine`, `system-dashboard`, `organvm-mcp-server`, `praxis-perpetua`, `stakeholder-portal`, `materia-collider`, `organvm-ontologia`, `cvrsvs-honorvm`, `custodia-securitatis`

### Governance
- *Standard ORGANVM governance applies*

*Last synced: 2026-05-23T00:26:31Z*

## Active Handoff Protocol

If `.conductor/active-handoff.md` exists, **READ IT FIRST** before doing any work.
It contains constraints, locked files, conventions, and completed work from the
originating agent. You MUST honor all constraints listed there.

If the handoff says "CROSS-VERIFICATION REQUIRED", your self-assessment will
NOT be trusted. A different agent will verify your output against these constraints.

## Session Review Protocol

At the end of each session that produces or modifies files:
1. Run `organvm session review --latest` to get a session summary
2. Check for unimplemented plans: `organvm session plans --project .`
3. Export significant sessions: `organvm session export <id> --slug <slug>`
4. Run `organvm prompts distill --dry-run` to detect uncovered operational patterns

Transcripts are on-demand (never committed):
- `organvm session transcript <id>` — conversation summary
- `organvm session transcript <id> --unabridged` — full audit trail
- `organvm session prompts <id>` — human prompts only


## System Library

Plans: 269 indexed | Chains: 5 available | SOPs: 8 active
Discover: `organvm plans search <query>` | `organvm chains list` | `organvm sop lifecycle`
Library: `/Users/4jp/Code/organvm/praxis-perpetua/library`


## Active Directives

| Scope | Phase | Name | Description |
|-------|-------|------|-------------|
| system | any | atomic-clock | The Atomic Clock |
| system | any | execution-sequence | Execution Sequence |
| system | any | multi-agent-dispatch | Multi-Agent Dispatch |
| system | any | session-handoff-avalanche | Session Handoff Avalanche |
| system | any | system-loops | System Loops |
| system | any | prompting-standards | Prompting Standards |
| system | foundation | agent-seeding-and-workforce-planning | agent-seeding-and-workforce-planning |
| system | foundation | architecture-decision-records | architecture-decision-records |
| system | any | background-task-resilience | background-task-resilience |
| system | any | context-window-conservation | context-window-conservation |
| system | foundation | legal-compliance-matrix | legal-compliance-matrix |
| system | foundation | ontological-renaming | ontological-renaming |
| system | foundation | readme-and-documentation | readme-and-documentation |
| system | any | session-self-critique | session-self-critique |
| system | any | the-descent-protocol | the-descent-protocol |
| system | any | the-membrane-protocol | the-membrane-protocol |
| system | any | theory-to-concrete-gate | theory-to-concrete-gate |
| system | any | triangulation-protocol | triangulation-protocol |

Linked skills: SOP-TRIADIC-REVIEW-PROTOCOL, api-design-patterns, cicd-resilience-and-recovery, coding-standards-enforcer, continuous-learning-agent, contract-risk-analyzer, cross-agent-handoff, evaluation-to-growth, gdpr-compliance-check, genesis-dna, multi-agent-workforce-planner, planning-and-roadmapping, promotion-and-state-transitions, quality-gate-baseline-calibration, repo-onboarding-and-habitat-creation, security-threat-modeler, session-self-critique, structural-integrity-audit, the-membrane-protocol, triple-reference


**Prompting (Anthropic)**: context 200K tokens, format: XML tags, thinking: extended thinking (budget_tokens)


## Task Queue (from pipeline)

**1** pending tasks | Last pipeline: unknown

- `cdcdd1b9e395` Plan: SEC Domain Global Architecture + Carry-Forward Verification [gcp, vercel]

Cross-organ links: 168 | Top tags: `mcp`, `python`, `rollup`, `chezmoi`, `bash`

Run: `organvm atoms pipeline --write && organvm atoms fanout --write`


## System Density (auto-generated)

AMMOI: 25% | Edges: 0 | Tensions: 0 | Clusters: 0 | Adv: 27 | Events(24h): 37975
Structure: 8 organs / 148 repos / 1654 components (depth 17) | Inference: 0% | Organs: META-ORGANVM:63%, ORGAN-I:53%, ORGAN-II:48%, ORGAN-III:54% +5 more
Last pulse: 2026-05-23T00:26:28 | Δ24h: n/a | Δ7d: n/a


## Dialect Identity (Trivium)

**Dialect:** SELF_WITNESSING | **Classical Parallel:** The Eighth Art | **Translation Role:** The Witness — proves all translations compose without loss

Strongest translations: I (formal), IV (structural), V (analogical)

Scan: `organvm trivium scan META <OTHER>` | Matrix: `organvm trivium matrix` | Synthesize: `organvm trivium synthesize`


## Logos Documentation Layer

**Status:** ACTIVE | **Symmetry:** 0.5 (DREAM)

Nature demands a documentation counterpart. This formation maintains its narrative record in `docs/logos/`.

### The Tetradic Counterpart
- **[Telos (Idealized Form)](../docs/logos/telos.md)** — The dream and theoretical grounding.
- **[Pragma (Concrete State)](../docs/logos/pragma.md)** — The honest account of what exists.
- **[Praxis (Remediation Plan)](../docs/logos/praxis.md)** — The attack vectors for evolution.
- **[Receptio (Reception)](../docs/logos/receptio.md)** — The account of the constructed polis.

### Alchemical I/O
- **[Source & Transmutation](../docs/logos/alchemical-io.md)** — Narrative of inputs, process, and returns.



*Compliance: Record exists without implementation.*

<!-- ORGANVM:AUTO:END -->
