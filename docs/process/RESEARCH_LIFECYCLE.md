# IDEA / RESEARCH / FUTURE LIFECYCLE

Status: CANONICAL_PROCESS
Scope: repository documentation and project decision flow

## Lifecycle

```text
IDEA
  ↓
RESEARCH
  ↓
RESEARCH_DONE
  ↓
DECISION
  ├─ REJECT
  ├─ ADOPT
  └─ FUTURE
        ↓
      PLANNED
        ↓
   IMPLEMENTATION
        ↓
       DONE
```

## Repository Structure

```text
docs/
├─ ideas/
├─ research/
├─ futures/
├─ plans/
└─ reports/
```

## Rules

- `ideas/` contains unvalidated concepts and observations.
- `research/` contains active investigations and completed research evidence. Completed research stays here with `Status: RESEARCH_DONE`.
- After research, disposition must be explicit: `REJECT`, `ADOPT`, or `FUTURE`.
- `futures/` contains intentionally deferred candidates worth preserving for later. A Future should reference its research evidence when available.
- `plans/` contains approved implementation plans / PTs. `reports/` contains audits, closure reports, implementation evidence, and final status.

## Promotion Rule

Research does not become canon automatically. Promotion requires an explicit decision. Keep original research evidence immutable where practical and materialize downstream decisions as separate artifacts.

## Branch Rule

Ideas, research, futures, process docs, and cross-cutting project documentation belong on the repository's canonical documentation/development branch. Do not park general research on unrelated feature branches. Feature branches are for concrete implementation work unless a repository explicitly defines otherwise.
