# ADR BISMUT Q17 | Frame-Time-Budgets + Benchmark Worlds

- Datum: 2026-11-19
- Owner-Entscheidung: **Q17 Option B + eigene Bismut Benchmark Worlds**
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Produkt: **Bismut VS Client** (Rust-EXE, Vintage Story .NET Game Runtime in-process, langfristig Rust-native D3D12)
- Inspiration: CS2-artige Benchmark-Maps als **Konzept** einer reproduzierbaren geskripteten Testsequenz, keine Übernahme fremder Maps, Assets oder Benchmark-Ergebnisse

## Entscheidung

Bismut erhält eine **instrumentierte, reproduzierbare Benchmark-Suite mit dedizierten Vintage-Story-Testwelten und automatisierten Kamerarouten**. Sie misst Frame-Time-Budgets nach Pipeline-Stufe statt nur Durchschnitts-FPS.

Der Bismut Launcher stellt qualifizierte Benchmark-Profile und Testwelten bereit; der Bismut VS Client startet die definierten Szenarien mit der Rust-Input-/Camera-/Frame-Snapshot-Brücke (Q2, Q12), sammelt Live-Metriken und speichert Evidence. **Die Game- und VS-World-Runtime bleibt .NET-owned**. Ein separater Vanilla-OpenGL-Lauf ist nur Vergleichsbaseline, nicht Teil der Bismut-OpenGL-Zero-Assertion.

Mit Q17-B werden **keine universellen FPS-Ziele, Hardware-Tiers (Q17-C) oder umfangreiche Always-on-Telemetry-Produktpolitik (Q17-D)** ratifiziert. Minimale Diagnosedaten für Frame-Budget-Evidence sind Bestandteil von B; konkrete numerische Pass/Fail-Schwellen werden erst nach qualifizierten reproduzierbaren Baselines beschlossen. Das historische 70-FPS-Ziel bleibt ein Ziel, **kein gemessener PASS**.

## Benchmark-Suite v1

| Scenario ID | Arbeitsname | Belastung | Mindest-Gate |
|---|---|---|---|
| BC-BENCH-01 | Bismut Baseline | Gleiche statische Szene, definierter Blickwinkel, Game/Bridge/Render/Present | Stabile Frame-ID-Korrelation und sauberes Warm-up |
| BC-BENCH-02 | Bismut Chunk Stress | Geskriptete Kamerafahrt/Bewegung mit Chunks, Sichtweite, kontrolliertem Streaming | World-/Chunk-Readiness, Stream-/Frame-Stalls getrennt klassifiziert |
| BC-BENCH-03 | Bismut Entity Stress | Definierte Anzahl Assets/Entities, Pose-/GPU-Skinning, instanced Entities | Tatsächliche Managed Avatar-/Entity-Ingestion, keine Dummy-only-Promotion |
| BC-BENCH-04 | Bismut Graphics Stress | Beleuchtung, Schatten, Transparenz, Partikel/Atmosphäre mit festem Tageslicht | Gleichwertige Szene/Pass-Aktivierung, CPU-/GPU-Zeiten |
| BC-BENCH-05 | Bismut Mod Stress | Gepinnte repräsentative VS-/Bismut-Mod-Sets, GUI-/Material-/Animation-Registries | Loader-/Mod-/DX12-Client-Evidence, UNKNOWN bei unqualifizierten Custom-GL-Pfaden |

Die Namen sind Design-Labels; Testwelten, Szenen, Assets und Routen sind noch nicht implementiert. Tests dürfen schrittweise freigegeben werden. **Keine Mod-Stress- oder Avatar-PASS-Claims ohne echten Client-/Renderpfad.**

## Reproduzierbarkeitsvertrag

1. **Fixture:** pro Welt eigener schreibgeschützter Source-Fixture/Schematic, gepinnte Vintage-Story-Version, World-ID/Seed, World-/Asset-Hashes, Chunk-Koordinaten, Zeit/Wetter, Mods/Versionen/Load-Order, Game-/Render-Einstellungen, Auflösung, Render Scale, FOV, Preset, Shader-/Pipeline-Version.
2. **Ablauf:** Preflight → Kopie in isoliertes Testprofil → Boot/World Ready → Chunk-/Asset-/GPU-Ready → Warm-up → deterministische Kamera-Route → Capture → Validierung → Ergebnisexport → geregeltes Exit/Cleanup. Keine bestehenden produktiven Saves verändern.
3. **Route:** geskriptete Waypoints mit Zeit/Tick/Position/Quaternion/FOV und definiertem Interpolationsmodus; Kamera- und Update-/Render-Takt getrennt führen. Eingaben werden über Q12 Router oder qualifizierte Kamera-Test-API bereitgestellt, nicht durch störende unkontrollierte OS-Inputs.
4. **Messungen:** echte Frame-/Present-Timestamps, p50/p95/p99 und Min/Max, optional 1%-Low/0.1%-Low mit dokumentierter Berechnung; getrennte Stufen für VS Game Tick, Managed-to-Rust Bridge, RenderWorld/Submit, GPU und Present; FPS/Frames/Frame-ID, CPU-/RAM-/VRAM-/Committed-/Resident-Budget, Upload-, Chunk-/Mesh-/Shader-Cache-Stalls, Warm-up/Readiness-Zeiten separat.
5. **Reproduzierbarkeit:** mindestens drei Capture-Runs pro Profil auf identischer Hardware und Build; Hardware/OS/CPU/GPU/Treiber/VRAM/Clock/Power-/Thermal-Mode, Rust/.NET/DXC/SDK-/Mod-/Runtime-Hashes, start/end timestamps und Capture-Tool-Version protokollieren.
6. **Messintegrität:** monotone Uhr und einheitlicher Frame-ID-Join; CPU/GPU-Zeit nicht naiv aufsummieren, da Stufen überlappen. VSync, FPS-Limit, Upscaling/Render Scale, API-Backend und Fensterauflösung explizit fixieren. Telemetrie-Overhead in A/B-Läufen messen; bei fehlender GPU-Timestamp-Unterstützung als UNKNOWN markieren.
7. **Vergleich:** Vanilla-OpenGL und Bismut-DX12 nur bei gleichen World-/Game-/Mod-/Grafik- und Hardwarebedingungen vergleichen; divergierende Semantik, Mod-Unsupported oder fehlende Rendering-Pässe => NICHT VERGLEICHBAR, kein künstlicher Performance-Sieger.

## Readiness- und Fehlerregeln

- **GATE_WORLD_READY:** echte Welt lädt, Spawn und benötigte Testchunks sind vorhanden; gegenwärtiger historischer 'spawn chunks 0%' Hänger ist ein *Boot-/World-Readiness-BLOCKER*, kein FPS-Ergebnis.
- **GATE_RENDER_READY:** DX12-Device, Shader-/Resource-Queues, benötigte Pässe und ein finaler Present-Owner tatsächlich aktiv. Child-Window-/synthetische Demo ersetzt keine vollständige Bismut VS Client-Qualifikation.
- **GATE_BENCH_VALID:** Route vollständig, Frame-IDs ohne nicht klassifizierte Lücken, Messfenster vollständig, keine unzulässige Nutzerinteraktion, definierte Instrumentierungs-Overhead-Spanne.
- **GATE_NEGATIVE:** fehlender Chunk, abweichender Save-/Asset-Hash, Mod-ABI-Mismatch, Shader-Compile-Failure, GPU Device Lost, Watchdog-Reset, Thermal-/Power-Throttling, OOM/VRAM-Eviction oder nicht vergleichbares Backend führen zu BLOCKED/PARTIAL/INVALID, nicht zu gemittelten FPS ohne Warnung.
- **GATE_RELEASE:** belastbare Schwellen erst anhand von Messläufen und Owner-Ratifizierung festlegen; getrennte PASS-Gates für Performance, Q7-Mod-Kompatibilität und Q8 OpenGL Zero.

## Artefakte und Reports

Ein Run erzeugt maschinenlesbares JSON/CSV mit run_id, scenario_id, fixture_hash, route_hash, version-/hardware-/settings-profile, commit_ids, Backend, Messfenster, Framedauer-Perzentile pro Stage, Ressourcenmetriken, Stalls, Fehlerklassifikation, Readiness-Gates und Evidence-Pfaden. Vergleichsreport zeigt Bismut DX12 vs Vanilla OpenGL (falls beide qualifiziert) plus Regression gegenüber dem letzten **gleichwertigen** Referenzrun. Keine Cloud-Telemetrie ohne expliziten separaten Beschluss; Reports lokal und ohne Auth-/Save-/User-Secrets.

## Umsetzungsschritte

| PT | Was | Exit |
|---|---|---|
| BC-BENCH-01 | Szenario-/Fixture-/Route-Schema + Benchmark-Welt-Generator/Isolationsregeln | Hash- und Fixture-Replay-Smoke |
| BC-BENCH-02 | Frame-ID korrelierte CPU/Bridge/Render/Present-Timer, GPU-Timestamp falls verfügbar | Synthetic Frametiming + Overhead-Test |
| BC-BENCH-03 | Erste Baseline-Welt mit fixer Kamerafahrt und World-Ready-Preflight | Reproduzierbare echte VS-Session mit vollständiger Capture |
| BC-BENCH-04 | Chunk-/Entity-/Graphics-Stress-Profile mit eigenen Readiness- und Mod-Checks | Vergleichbare Ergebnisse, Failure-Szenarien |
| BC-BENCH-05 | Launcher-Integration, Benchmark-UI und lokale Vergleichs-/Regression-Reports | Verifizierter Report mit Baseline-/Regression-/Invalid-Klassifikation |

Das Benchmark-Framework darf vor dem fertigen Rust Client als Test-Harness auf dem bestehenden DX12-Recovery-Pfad prototypisiert werden. Erzielte Ergebnisse gelten nur für den tatsächlich getesteten Host-/Renderer-Pfad und werden nicht automatisch zum zukünftigen Rust-Client-PASS.

## Abhängigkeiten

Q1–Q8, Q12, Q15, Q16 sind Architekturentscheidungen. Q9 Bootstrap, Q10 Threading, Q11 CPU-Memory, Q13 Shader und Q14 GPU-Resource bleiben Evaluationen. Atlas ist ausschließlich für headless VS-/Mod-Game-Szenarien relevant und kein GPU-Benchmark-Renderer. GPU-Tools wie PIX/PresentMon/ETW nur verwenden, wenn tatsächlich installiert und verfügbar. Q8 OpenGL-Zero-Messung separat mit Prozess-Tracing von Start bis Exit.

## Status

**DECIDED** betrifft die Architektur und die Benchmark-Suite-Anforderung. Keine Benchmark-Welten, Kamerarouten oder Tools sind durch diesen ADR installiert, ausgeführt oder als PASS nachgewiesen.

Referenzen: [Roadmap V1.9](BISMUT_VS_CLIENT_ROADMAP_V1_9.md), [Benchmark Suite Spec](BISMUT_VS_CLIENT_BENCHMARK_SUITE_V1_SPEC_2026-11-19.md), [Q16 Recovery](ADR_BISMUT_Q16_RUST_RECOVERY_SUPERVISOR_EXTERNAL_WATCHDOG_2026-11-18.md).
