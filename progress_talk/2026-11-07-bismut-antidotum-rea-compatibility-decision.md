# BismutAntidotum · REA Semantic Diff & Compatibility Decision Engine

**Typ:** progress_talk / Architekturidee und Forschungsplan  
**Stand:** 2026-11-07  
**Status:** PROPOSED — kein implementierter oder abgenommener Vertrag  
**Baseline:** Vintage Story 1.22.7 / Bismut Game→Video Boundary  
**Geltung:** DEV-Werkzeuge, Release-Runtime und Update-/Migrationsprozess  
**PC1:** einzige aktuelle Hardware-Autorität; **PC2 bleibt PARKED**.

## 1. Ziel

**Vintage Story soll seine Engine ändern können, ohne automatisch Bismuts Rust Core oder native DX12-Video-Engine umzubauen.**

BismutAntidotum ist die möglichst dünne, versionsbewusste Hook-/Compatibility-Schicht zwischen Vintage Story und Bismuts stabiler Game→Video-Bridge. **Bismut REA** untersucht neue Spielversionen, erkennt relevante Änderungen, ordnet deren Auswirkungen Bismut-Abhängigkeiten zu und empfiehlt die kleinste erforderliche Aktion.

Leitfrage beim Update:

> „Was wurde zwischen Version A und B verändert, was davon betrifft Bismut, funktioniert unsere bisherige Lösung unverändert, müssen wir einen Adapter patchen oder lohnt die Übernahme einer neuen Funktion?“

**Nicht jedes Vintage-Story-Major-Update ist ein Bismut-Major-Update.**

## 2. Architektur und Verantwortung

```text
Vintage Story (bekannte Version / neue pre, rc, stable)
  └─ BismutAntidotum.Runtime
       ├─ Version/Assembly-/Capability-Check
       ├─ freigegebene Harmony Hooks + VS-Version-Adapter
       ├─ minimaler Contract/Lifecycle Check
       └─ Bismut Game→Video Bridge ABI (versioniert)
             ├─ Bismut Rust Core: Meshing / Scheduling
             └─ Bismut DX12 Video Engine: Residency / GPU Rendering

DEV only:
Vintage Story DLLs → Bismut REA Templates
  → IL-/Signature-/Call-Graph-/Dataflow-Diff
  → Semantic Hook Candidate Discovery
  → Bismut Dependency / Impact Graph
  → Compatibility Decision Engine
  → validierter Adapter / freigegebenes Manifest
```

Die native Engine darf **keine Vintage-Story-Typen** als externen Contract verlangen. Externe Verträge bestehen aus Bismut-eigenen, versionierten Datenstrukturen, explizitem Ownership und Lifecycle. Die erste konkrete Referenz ist `ChunkRenderer.AddTesselatedChunk(TesselatedChunk, ClientChunk)`, unser Terrain-Übergabepunkt **vor OpenGL-Verarbeitung** in der derzeit geprüften VS-Referenz. Dieser *semantische Übergabezeitpunkt* ist voraussichtlich beständiger als sein Klassen- oder Methodenname, aber **nicht garantiert stabil**.

**Standalone bedeutet vorerst unabhängige Module/Tests, nicht zwingend separate Betriebssystemprozesse.** Die native DX12-DLL darf im Game-Prozess laufen. Ein separater Renderprozess wäre eine neue Architekturentscheidung mit Window-/IPC-/Synchronisationskosten.

## 3. DEV vs. Benutzer-RELEASE

| Fähigkeit | DEV (intern) | RELEASE (Spieler) |
|---|---|---|
| REA Templates / Version-Diffs | volle Schemas, Diff-Reports, Suchprofile | nicht erforderlich |
| ILSpy/REA, semantische Kandidatensuche | Signatur-, IL-, Call-Graph-, Dataflow-Analyse | **keine** automatische Hook-Suche |
| Harmony | Instrumentierung, Probes, Hook-Tests | **nur freigegebene** Hooks |
| Hook-Fingerprints | Analyse, neue Version prüfen/freigeben | Manifest-/Vertragsprüfung |
| Game→Video Bridge | Capture, ABI-Parität, Ownership-/Reset-Tests | schlanker Binär-Transport |
| Rust und DX12 | Benchmark-/Profiler-/Debug-Build | optimierte Release-Binaries |
| Diagnostik | PIX/Nsight/ETW, ausführliche Dumps, Metrics | minimale Logs und optionale Diagnose |
| Versionskonflikt | Kandidaten prüfen, Adapter entwickeln | verständliche Meldung; **vor Übernahme** fail-closed |
| Fehler nach Renderübernahme | Live-Analyse | sauber abschalten, ggf. Neustart erforderlich |

**Ein gemeinsam genutzter Core**, aber getrennte DEV-Tools und Release-Verpackung; keine zwei auseinanderlaufenden Hook-Implementierungen.

## 4. Bismut REA Template: semantischer Hook-Vertrag

Jeder Hook besitzt eine maschinenlesbare Beschreibung dessen, **was** er garantiert, getrennt von der versionsspezifischen Information, **wo** er im Spiel sitzt.

```yaml
# Konzept / Beispielschema, noch nicht validiert
id: BISMUT-REA-G2V-001
schema_version: 1
purpose: completed_chunk_handoff

reference:
  game_version: "1.22.7"
  assembly: "VintagestoryLib.dll"
  type: "ChunkRenderer"
  method: "AddTesselatedChunk"
  parameters: ["TesselatedChunk", "ClientChunk"]
  assembly_sha256: "TO_BE_CAPTURED"

semantic_anchors:
  stage: before_opengl_resource_upload
  required: [chunk_identity, vertices, indices, removal_lifecycle]
  invariants:
    - payload_valid_during_capture
    - ownership_explicit
    - removal_and_world_reset_handled

discovery:
  - signature
  - normalized_il_fingerprint
  - call_graph
  - dataflow
  - lifecycle

validation:
  - static_contract
  - runtime_probe
  - payload_parity
  - unload_reset
  - live_regression

release_policy:
  unknown_build: fail_closed
  unknown_fingerprint: quarantine_for_review
  automatic_hook_migration: false
  approval_required: true
```

**Wichtig:** Statische Ähnlichkeit ist nur ein Kandidat, kein Beweis für identische Semantik. Runtime-Hook-Aktivierung erst nach qualifizierter Versions- und Contract-Prüfung. Keine selbstheilende Bytecode-Manipulation beim Spieler.

## 5. Vintage Story A → B: Semantic Version Diff

**Eingaben:**
- Referenz-Assemblies A (SHA256, Versionsnummer, Herkunft) und neue Assemblies B.
- Öffentliche API-Historie, Release Notes, vsapi-Commits, ggf. Hook-relevante private IL-Strukturen.
- Bismut REA Templates, verifizierte Hook-Manifeste und **Engine Cartography Dependency Graph**.
- Optional: reproduzierbare Runtime-Probes in isolierter Testinstallation.

**Erkannte Änderungstypen:**

| Typ | Signal | Beispielwirkung |
|---|---|---|
| API_BREAK | Typ/Signatur entfernt, verändert, verschoben | Hook-Aufruf/Kompatibilität |
| SEMANTIC_CHANGE | IL-Datenfluss oder Verhalten bei gleicher Signatur verändert | falsche Daten / Zeitpunkt |
| LIFECYCLE_CHANGE | Queue-Reihenfolge, Disposal, Thread/Ownership geändert | Race/Use-after-free |
| PERFORMANCE_CHANGE | neue Locks, Allocations, Main-Thread-Arbeit | mögliche Regression, live messen |
| ADDED_CAPABILITY | neue API/Funktion ohne bestehenden Bedarf | optionaler Backlog-Kandidat |
| NO_RELEVANT_CHANGE | keine Abhängigkeiten betroffen | keine Umsetzung |

**Nicht aus dem öffentlichen API-Repository allein ableitbar:** interne Harmony-Hook-Änderungen, tatsächliche Queue-Semantik, Lifetime und GPU-/Renderer-Verhalten. Diese erfordern Assembly-Diff und gegebenenfalls echte Laufzeittests.

## 6. Compatibility Decision Engine

Für **jede** Änderung beantwortet der Agent:
1. Ist eine Bismut-Abhängigkeit direkt oder transitiv betroffen?
2. Ist unser bestehender Contract inklusive Semantik und Lifecycle nachweislich weiterhin gültig?
3. Ist die neue VS-Funktion technisch nötig, optional nützlich oder irrelevant?
4. Welche **kleinste Änderung** reicht, um volle Kompatibilität zu behalten?
5. Welche Evidenz und Tests benötigt die Freigabe?

| Entscheidung | Bedeutung | Konsequenz |
|---|---|---|
| `NO_IMPACT` | außerhalb unseres Dependency Graph | ignorieren |
| `COMPATIBLE` | Contract weiter gültig | neue VS-Version qualifizieren, keine Engine-Änderung |
| `SHIM_REQUIRED` | kleine Signatur-/Layout-/Adapterabweichung | Antidotum-Kompatibilitätspatch |
| `OPTIONAL_FEATURE` | neue API, unsere Lösung funktioniert | Future-Backlog, separat benchmarken |
| `BREAKING` | Hook/Bridge/Lifecycle nachweislich unvereinbar | isolierte Adapterreparatur oder bewusste ABI-Migration |
| `UNKNOWN` | Evidenz reicht nicht | Quarantäne / weiterer Test, **keine Freigabe** |

Mögliches maschinenlesbares Decision-Record:

```yaml
# Fiktives Beispiel, KEIN behaupteter 1.23-Diff
change_id: VS-DIFF-0001
source_version: "1.22.7"
target_version: "1.23.0"
classification: API_BREAK
affected_bismut: [BISMUT-REA-G2V-001, antidotum.terrain]
current_contract_valid: false
action: SHIM_REQUIRED
alternative: KEEP_EXISTING_RUST_AND_DX12
release_scope: COMPATIBILITY_PATCH
tests: [hook_contract, payload_parity, removal_lifecycle, world_reset, live_regression]
confidence: PENDING_VALIDATION
release_gate: TESTS_REQUIRED
```

**Der Analyzer darf ausdrücklich „KEIN UPDATE NÖTIG“ melden.** Der bloße Fund neuer APIs ist kein Implementierungsauftrag. Ist Bismuts bisherige Lösung besser oder gleichwertig, dürfen wir sie behalten. Wenn eine Vanilla-Änderung unseren Workaround überflüssig macht, kann **Code entfernen** die beste Lösung sein.

## 7. Update-Größen / Release-Policy

| Bismut-Release | Wann | Umfang |
|---|---|---|
| Compatibility Patch | neue Spielversion mit unverändertem ABI oder kleinem Adapter-Fix | Manifest/Adapter + Contract- und Live-Tests |
| Minor Update | absichtlich übernommene neue Funktion, kompatibel zu bestehenden Bismut-Verträgen | Feature + Performance-/Regressionstests |
| Major Update | inkompatible eigene Bridge-/Native-ABI- oder Produktverträge | koordinierte Migration und Versionierung |

Die Spielversionsnummer bestimmt **nicht automatisch** die Bismut-Semver. Funktionierende Altversionen bleiben separat freigegeben, solange gewartet.

**Vor der Übernahme:** Unbekannte DLL/Signatur/Fingerprint- oder Contract-Verletzung ⇒ fail-closed, kein riskanter Hook; soweit technisch möglich Vanilla unverändert starten.  
**Nach DX12-Übernahme:** nicht pauschal „hot fallback“ versprechen; ggf. kontrollierter Neustart nötig.  
**PC2:** weiterhin nicht als Test- oder Update-Target verwenden, bis PC1 Stable freigegeben ist.

## 8. Beispielreport 1.22.7 → 1.23.0 (rein illustrativ)

Die nachfolgenden Zahlen und Änderungen sind **erfunden**, nicht aus echten VS-1.23-Binaries gewonnen.

| Änderung | Bismut betroffen? | Entscheidung | Update |
|---|---|---|---|
| neue Worldgen-Hilfs-API | nein | NO_IMPACT | nichts |
| neue effiziente Chunk-Upload-API | optional | OPTIONAL_FEATURE | Future Request |
| veränderte ChunkRenderer-Signatur | ja, Terrain-Hook | SHIM_REQUIRED | Antidotum-Patch + Regression |
| Tessellation-/Dispose-Reihenfolge verändert | möglicherweise | UNKNOWN, bis Trace | isolierter Lifecycle-Test |

**Gewünschter Top-Line-Report:**
```text
BISMUT REA · COMPATIBILITY REPORT (BEISPIEL)
Vintage Story 1.22.7 -> 1.23.0
Detected: 47 changes (fictitious)
Impacting Bismut: 3 (fictitious)
Required fixes: 1 (fictitious)
DX12 Core: unchanged (only if verified)
Rust Core: unchanged (only if verified)
Recommendation: Antidotum Compatibility Patch
Release: BLOCKED until adapter + runtime regression PASS
```

## 9. Artefakte / Schnittstellen

Geplante Artefakte, **noch nicht angelegt / freigegeben**:

- `BISMUT_REA_TEMPLATE_SCHEMA_V1.yaml` — gemeinsame Syntax für Hook-Semantik.
- `BISMUT_REA_GAME_TO_VIDEO_V1.yaml` — G2V-001 als konkreter Referenz-Hook.
- `BISMUT_REA_VERSION_DIFF.json` — maschinenlesbare Signatur-/IL-/Verhaltensänderungen.
- `BISMUT_COMPATIBILITY_IMPACT_REPORT.md` — Bismut-bezogene Auswirkung, Evidenz und Risiko.
- `BISMUT_ANTIDOTUM_ADAPTER_PLAN.md` — minimaler Migrationsplan, Tests, Rollback.
- `BISMUT_ANTIDOTUM_HOOK_MANIFEST_V1.json` — versionierte, geprüfte Release-Hook-Auswahl.

**Verknüpfung zum bereits vorgesehenen Vintage Story API Radar:** Externe Pre-/RC-/Stable-Meldung ist nur ein **Trigger zur Prüfung**. REA vergleicht neue lokale Assemblies; kein autonomer Umbau einer produktiven Installation. Nicht behaupten, der Radar führe automatisch IL-Diffs aus oder könne ungeprüfte Methoden semantisch verifizieren.

## 10. Nächster Implementierungsschritt (noch nicht beauftragt)

1. **Ist-Aufnahme:** vorhandene Bismut Hooks, Bridge-Verträge, Adapter, Version-Probes und Cartography-Daten erfassen; Doppelarbeit vermeiden.
2. **Template-V1:** minimale Schemafelder und referenzierte Contract-Evidenz mit einem realen 1.22.7-Terrain-Hook reproduzierbar ausfüllen.
3. **Offline A/B Harness:** selbst bei identischer Assembly `NO_IMPACT` und bei kontrolliert geänderten Fixtures `BREAKING / SHIM_REQUIRED / UNKNOWN` korrekt klassifizieren.
4. **Release-Gate:** geprüfte Hook-Manifeste laden, auf unbekannten Builds fail-closed, keine Runtime-Autodiscovery.
5. **Echte neue Version:** isolierte Assembly-Diffs, priorisierte Impact-Liste, minimaler Adapter-Patch, Tests; **kein** PC2-Einsatz.

## Gültigkeit dieser Notiz

Dies ist ein **materialisierter Gesprächs- und Architekturvorschlag**, kein Beleg, dass Antidotum, REA Semantic Discovery oder Compatibility Decision Engine bereits implementiert sind. Verbindliche Spezifikationen, Arbeitsaufträge, Quellcode, Tests und Freigaben bleiben in den jeweiligen kanonischen Bismut-Repositories. Der zuvor abgelegte Performance Talk ist hier: [Bismut Performance Talk & Engine Cartography](2026-11-07-bismut-performance-talk-engine-cartography.md).
