# Bismut VS Client | Benchmark Suite V1 Specification

- Datum: 2026-11-19
- Status: **SPEC_APPROVED / NOT_IMPLEMENTED**
- Owner: Q17 **B + Benchmark Worlds**
- Zweck: eine wiederholbare, in-game Benchmark-Route über fixierte Testwelten, vergleichbar im Prinzip mit einer CS2-Benchmark-Map, ohne fremde Karten-/Asset-Inhalte zu kopieren

## V1 Scope

Die Suite ist eine **Spielwelt-basierte, clientseitige Performance-Messung** für Bismut VS Client. Sie ist nicht identisch mit den Q9/10/11/13/14-Mikrobenchmarks und ersetzt keine serverseitigen Atlas-Mod-Tests. Ziel: reproduzierbare Ausführung und belastbare Zuordnung von Stalls zu Game Runtime, Bridge, CPU RenderWorld, GPU oder Present.

### Fünf Szenenprofile

| Szenario | Fixture-Inhalt | Kameraroute | Gemessenes Risiko |
|---|---|---|---|
| baseline | Kompakte Standardregion, feste Lichter, konstante Entity-Zahl | ruhige Strecke mit definierter Pan-/Rotation | Basis-Frame-Time, Render-/Bridge-Overhead |
| chunk-stress | Testregion aus gepinnten World-/Chunk-/Schematic-Dateien und festem Sichtprofil | vorgegebener schneller Traversal mit Warm-/Cold-Streaming-Phasen | Chunk-Builds, Upload-Queues, Frame-Spikes |
| entity-stress | definierte Instanzen, Skelette, Pose-Quellen und Animationssynchronisierung | Orbit und Streckenflug um Entitäten | CPU Pose, Managed Ingestion, GPU-Skinning, Draw Calls |
| graphics-stress | Beleuchtung, Schatten, transparente Materialien, Partikel, GUI-Overlay | definierter Blickwinkelwechsel mit hoher Overdraw-Last | PSO, Pixel/Depth-Parität, GPU-Pass-Frame-Time |
| mod-stress | zwei oder mehr gepinnte zulässige Mod-Familien, C# Legacy und Bismut Native | kurze Mod-/GUI-/Asset-Sequenz mit deterministischen Aktionen | Mod-Loader-ABI, Compositor, Asset-/Shader-Streaming |

Alle konkreten Counts, Weltgrößen und Schwellen folgen erst aus technisch geprüfter Test-Hardware und der Verfügbarkeit realer Renderfeatures. Ein nicht implementierter Avatar-, GUI- oder Mod-Pass bleibt **NOT_READY**, nicht der vermeintlich schnelle Benchmark.

## Manifest V1 (Normativer Schema-Entwurf)

Ein versioniertes JSON-Szenario enthält mindestens folgende Felder:

- schema_version: 1
- scenario_id: stabile Identität wie bc-bench-baseline-v1
- vintage_story_version, game_assembly_hashes, bismut_client_build, engine_commit
- world_fixture_id, world_fixture_hash, world_seed, required_chunks, required_assets
- mods: Liste von Mod-ID, Version, Hash, Side, erwarteter Capability
- graphics: Backend, resolution, render_scale, FOV, preset, VSync, frame_cap, shader_hashes
- run: warmup_seconds, capture_seconds, repeats, camera_route_id, route_hash, readiness_timeout
- environment: Hardware-/Treiber-/Windows-Version, GPU-VRAM, CPU, RAM, Power-/Thermal-Policy
- evidence: local_output_dir, desired_trace_channels, privacy_policy

Eine Route enthält monotone Zeit-/Tickmarker, Kamera-Position, Orientierung (Quaternion), FOV und ggf. scripted action IDs. **Kamera-Replay und Game-Ticks sind separat aufzunehmen**, damit Viewpath-Konsistenz ohne stillschweigende Gameplay-Fälschung testbar bleibt.

## Ausführungsvertrag

1. **PREFLIGHT:** Manifest-/Asset-/Game-/Mod-/Version- und Hash-Prüfung, Benchmark-Hardware-Tier dokumentieren. VS-Save-Fixture kopieren; Quelle read-only lassen.
2. **BOOT:** gewählten Vanilla- oder Bismut-Modus starten; tatsächliche Window-/Device-/Runtime-Ownership protokollieren. Nicht gleichzeitig GL und DX12 als einen Run zählen.
3. **WORLD_READY:** Welt, Spawn, benötigte Chunks, Shader-/Mesh-/Textur-Assets und Render-Pässe melden überprüfbare Bereitschaft. Timeout/Chunk 0% => **BLOCKED_BOOT**.
4. **WARMUP:** Render-/Shader-/Asset-Pfade stabilisieren; Warm-up getrennt protokollieren und nicht in Frame-Time-Perzentile mischen. Cold-Start nur in separaten Profilen als solcher messen.
5. **CAPTURE:** gleiche Kamera-Waypoints, definierte Zeitfenster und Instrumentierungs-Konfiguration. Interaktive Fremdeingaben => invalid oder klassifizierte Unterbrechung.
6. **VALIDATE:** Route vollständig? Count/Hash/Frame-Sequence/Present/Readiness? GPU-Timestamp verfügbar? Instrumentierungs-Overhead innerhalb gemessener Grenzen? Erst dann Aggregate bilden.
7. **EXPORT:** lokale maschinenlesbare Frame-/Session-Receipts, Vergleichs- und Regression-Reports. Ein fehlgeschlagener Run wird nicht mit einem gültigen gemittelt.

## Metric Dictionary

| Metrik | Einheit | Quelle / Caveat |
|---|---|---|
| displayed fps / avg frame time | frames/s bzw. ms | Present-basierte Timestamps; nur vollständig gezeigte oder konsistent klassifizierte Presents |
| frame_time_ms p50/p95/p99 | ms | Per-frame Present-Abstände, dropped/late Frames getrennt markieren |
| stage game_tick / bridge / render_cpu / submit | ms | Instrumentierte CPU-Timestamps; überlappende Stufen nicht aufsummieren |
| gpu_time | ms | DX12 GPU Timestamp Queries (falls verfügbar, sonst UNKNOWN); Queue-/Pass-Abgrenzung dokumentieren |
| present / latency | ms | Verifizierbare Present-/Queue-/Display-Messmethodik; kein ungeprüftes Input-to-Photon-Versprechen |
| cpu_ram / gpu_vram / residency | MiB | committed/resident/budget und peak getrennt; Treiber-/Tool-Abhängigkeiten dokumentieren |
| chunk_load / mesh_upload / shader_compile | counts, ms | Runtime- und Loader-Telemetrie, echte Readiness-/Backpressure-Signale |
| invalid / stutter / cpu_limited / gpu_limited | Klassifikation | dokumentiertes Heuristik-/Threshold-Schema; ohne Beleg UNKNOWN |

1%-Low/0.1%-Low können ergänzt werden, aber ihre mathematische Ableitung und Fenstergröße müssen offengelegt werden. p50/p95/p99 sind zwingend. Aus FPS allein darf keine Pipeline-Stage-Performance abgeleitet werden.

## Compare Contract

- Teste pro Szene mindestens drei gültige Wiederholungen nach Aufwärmen; dokumentiere Run-to-Run-Varianz und die Baseline-Messunsicherheit.
- Gleiche PC-/Treibersettings, Auflösung, Render-Scale, View Distance, Shader/Content, Time/Weather, Mods, Spieler- und Entity-Zustände, Route und Versionen für Vanilla-GL vs Bismut-DX12. Unterschiede und fehlende Feature-Parität explizit als NOT_COMPARABLE behandeln.
- Für die Render-Cutover-Phase kann der aktuelle C++-DX12-Host gegen Rust-DX12 auf derselben Fixture getestet werden. Diese Results bleiben backend-/build-spezifisch.
- Keine falschen numerischen Budgets: 70 FPS ist nur ein nicht qualifiziertes historisches Ziel. Ein Q17-Threshold-ADR folgt später, sobald baselined Hardware-/Scene-Profile existieren.
- Ausgabe im Launcher/Client zunächst als lokaler Report. Cloud-Upload, öffentliche Leaderboards und nutzerübergreifende Datensammlung sind **nicht beschlossen**.

## Gates und Nicht-Ziele

- Testwelt-Kopie und negative Asset-/Save-Hash-Tests, kein produktives Save-Mutieren.
- Frame-ID, World Ready, Live Present, Render-Pass-Parität und instrumentierte Messfenster müssen nachgewiesen sein.
- OpenGL-Zero-Test aus Q8-C ist ein separater Prozess-Trace-Gate; GL-Vanilla-Baselines sind ausdrücklich erlaubt, aber als anderes Backend gekennzeichnet.
- Q7-D Mod-Kompatibilität ist Ziel, nicht Benchmark-PASS ohne Client-/Mod-Evidence.
- Kein unbewiesener 'auto replay works' oder 'benchmark world exists' Claim.

## Dependency / PT

Start: **BC-BENCH-01**, vollständiges Manifest, reproduzierbare Welt-Fixture und eine statische Baseline-Kameraroute. Nächstes: BC-BENCH-02 Frame-Timer und Report-Schema, dann BC-BENCH-03 echte World-Ready+Capture-Session. Erst bei qualifizierter Avatar-/Chunk-/GUI-Bridge weitere Szenen hinzufügen.

Referenzen: [Q17 ADR](ADR_BISMUT_Q17_FRAME_BUDGETS_BENCHMARK_WORLDS_2026-11-19.md), [Roadmap V1.9](BISMUT_VS_CLIENT_ROADMAP_V1_9.md).
