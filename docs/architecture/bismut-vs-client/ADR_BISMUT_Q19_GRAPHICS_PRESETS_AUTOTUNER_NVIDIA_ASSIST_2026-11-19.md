# ADR BISMUT Q19 | Unified Graphics Presets, Benchmark Auto-Tuner und NVIDIA Assist

- Datum: 2026-11-19
- Ownerentscheidung: **Q19 Option E (A + B + C + D als gestufter Hybrid)**.
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED / PERFORMANCE_THRESHOLDS_NEED_BASELINE**
- Produkt: **Bismut VS Client / Unified Launcher-Client Shell** (Q18)
- Scope: manual settings, capability-aware presets, Q17 Benchmark Worlds Auto-Tuner, optional NVIDIA Assist und herstellerneutraler Fallback

## Entscheidung

Die gemeinsame Bismut Launcher-/Client-Shell erhält **eine Grafiksettings-Plattform mit vier zusammenhängenden Modi**:

1. **Manual / Advanced (A):** Original-Vintage-Story-Optionen sowie qualifizierte native Bismut-DX12-Optionen in einer nachvollziehbaren, aber backend-spezifischen Oberfläche. Keine ungültigen Werte oder Hidden GL-Paths im Rust-DX12-Modus. Per-Option Provenienz.
2. **Capability-aware Presets (B):** Quality, Balanced, Performance und Custom, basierend auf Hardware/Backend-/VS-Adapter-/Mod-/Shader-Capabilities. Preset-Mapping **nur für tatsächlich unterstützte Features**, nicht aufgrund eines GPU-Namens.
3. **Benchmark Auto-Tuner (C):** nutzt die durch Q17 definierten reproduzierbaren Benchmark Worlds als **Vorschlagsgenerator** für konfigurierbare Ziele (Frametime, p95/p99, Qualität, begrenzter VRAM-/Streaming-Overhead), nicht als magischen GPU-Optimierer. Ohne gültige Benchmarks: Empfehlungen UNKNOWN, keine Änderung.
4. **NVIDIA Assist (D):** optionale lokale NVIDIA-Vendor-/Treiber-/Feature-Erkennung, Performance-/Latency-/Qualitäts-Empfehlungen pro VS-OpenGL- und Bismut-DX12-Profil, mit Nutzer-Opt-in und reversiblem Settings-Change. AMD/Intel und andere GPUs bleiben mit generischen Presets voll bedienbar.

Das ist eine **Architektur- und Funktionsentscheidung**, keine Bestätigung, dass NVIDIA Reflex, NIS, DLSS, Streamline, DX12-Feature-Passes, Benchmark Worlds oder Auto-Tuner bereits in der Engine existieren. Der Benutzer behält stets manuelle Override-Möglichkeit und eine klar erkennbare Custom-Konfiguration.

## Settings- und Capability-Ownership

| Ebene | Responsibility | Korrektheitsgrenze |
|---|---|---|
| Unified Shell | eine gemeinsame, versionierte Oberfläche, Profile, Export/Import, Änderungs-Vorschau | Single Source of Truth für **gewünschte** Settings, kein paralleler Launcher-/Client-Settings-Owner |
| Backend Adapter | gewünschte Optionen zu Vintage-Story-/OpenGL- bzw. Bismut-DX12-Semantik validieren | **supported / effective / desired** getrennt; nicht passende Werte werden nicht still übernommen |
| Capability Registry | echte Renderer-Passes, Shader-/GPU-/Treiber-/Mod- und Monitor-Features melden | AVAILABLE, UNAVAILABLE, NEEDS_INTEGRATION, UNVERIFIED; nicht aufgrund bloßer DLL-/GPU-Präsenz freischalten |
| Benchmark Harness | World-/Chunk-/Render-Ready, Fixture-/Route-/Build-/Driver-Hashes, Frame-/Stage-Metriken | Q17 p50/p95/p99 und mehrere gültige Runs; Vergleich nur bei Szene-/Renderer-Parität |
| Auto-Tuner | Vorschläge aus qualifizierten Benchmark-Runs und Nutzerziel bilden | keine eigenmächtige dauerhafte Settings-Mutation; Ergebnis/Unsicherheit/Tradeoff anzeigen |
| NVIDIA Assist | nur qualifizierte NVIDIA-Funktionen und reversible per-App-/backend-spezifische Empfehlungen | keine heimlichen globalen Treibertweaks, Registry-Manipulation, Overclock/Undervolt |
| Q16 Recovery | Save-/Settings-Rollback, Crash-/Device-Lost-Safe-Start | Fehler vor allem bei experimentellem DX12 nicht als Optimierungsgewinn fehlinterpretieren |

**Profilschema V1 (normative Felder):** schema_version, profile_id, vs_version, mod_set_hash, game_backend, backend_adapter_version, gpu_vendor/device/driver, monitor/display, resolution, render_scale, vsync, frame_limit, view_distance, graphics_preset, manual_overrides, desired_settings, effective_settings, supported_capabilities, pending_restart, last_qualified_benchmark_run_id, last_applied_change_id, rollback_snapshot_hash. Secrets/Authentifizierung gehören **nicht** in Grafikprofile.

**Backends strikt trennen:** Vanilla OpenGL kann ein eigener VS-Prozess sein; Bismut Rust-DX12 mit Q5 in-process .NET. Einstellungen können semantisch gleich benannt sein, aber intern abweichend implementiert und unterstützt werden. Ein Backendwechsel ist ein **Restart-Gate**, kein Live-API-Tausch.

## Preset-Semantik

| Profil | Ziel | Prinzip |
|---|---|---|
| Quality | höchste durch Capability verifizierte Qualität | Fidelity erhalten; VRAM-/GPU-Budget offenlegen |
| Balanced | ausgewogene Darstellung bei stabilen Frametimes | Default-Kandidat, konkrete Schwellen erst nach Baselines |
| Performance | niedrigere Frametimes und stabile p95/p99 | bewusst qualitätsreduzierende Eingriffe dokumentieren |
| Custom | vollständig kontrollierbare manuelle Einstellungen | Manual Overrides erhalten, Preset nie unsichtbar erzwingen |

Bei nicht unterstützten Funktionen wird der spezifische Toggle als **UNAVAILABLE/UNVERIFIED** dargestellt, nicht pseudoaktiviert. Feature-Parität zwischen Vanilla-GL und Rust-DX12 ist **nicht angenommen**. VS-Mods können Grafikoptionen hinzufügen; deren Registrierung erfolgt ausschließlich über validierte Q15-Mod-/Q4-GUI-Adapter.

## Auto-Tuner: kontrollierter Ablauf

1. **Preflight:** user-selected objective (beispielsweise p95 Frame-Time-Ziel, Qualität/VRAM-Präferenz), VS-Client-/Backend-/Build-/Mod-/Driver-/Fixture-Hashes, Hardware- und Power-/Thermalzustand; nicht erreichbare Ziele als Schätzung kennzeichnen.
2. **Baseline:** Benchmark-World in isolierter Kopie, WORLD_READY + CHUNKS_READY + RENDER_READY, definierter Warm-up, kontrollierte Kamera-Route und mindestens drei gültige Repeats; Q17 überlappende CPU/GPU-Stufen getrennt behandeln.
3. **Candidate Search:** begrenzte, transparente Preset-/Option-Varianten testen; Änderung jeweils einzeln oder mit dokumentiertem Design; Time/quality budgets und maximalen Benchmark-Aufwand definieren. Kein blindes Durchtesten aller Kombinationen.
4. **Validate:** gleiche Fixture und Szene mit Candidate erneut messen; p50/p95/p99, Frame-Stalls, VRAM/Residency, Rendering- und Feature-Parität, Mod-Support und Überhitzungs-/Power-Einflüsse klassifizieren. **NOT_COMPARABLE**, wenn Effekte/Passes/Mods verschwinden.
5. **Propose:** Report zeigt vorher/nachher, Unsicherheit, Tradeoffs, Änderungsliste, supported/effective, mögliche Restart-Pflicht. **Erst nach Nutzerfreigabe anwenden**; Custom Override respektieren; Versioniertes Snapshot für Rollback anlegen.

**Hard Blockers:** World-Spawn bei 'spawn chunks 0%', fehlender DX12-Game-/Avatar-/GUI-Pass, Device Lost, Mod-ABI-Mismatch, unvollständige Kameraroute, inkonsistente GPU-Timestamps, stark abweichende Thermal-/Treiber-Settings => keine gültige Optimierung. **70 FPS** bleiben ein historisches Ziel, keine pauschale Mindestanforderung oder PASS.

## NVIDIA Assist: Scope und Sicherheitsvertrag

- **V1:** Vendor/Device/Driver identifizieren, bei NVIDIA GPU per-Backend unterstützte NVIDIA-App-/Game-Settings als Empfehlung darstellen; bei unbekannter NVIDIA-Funktion status **UNVERIFIED** und keinen Treiberwert schreiben.
- **V2 Spikes:** NVAPI- oder andere unterstützte Schnittstellen für programmbezogene Einstellungen nur nach Lizenz-/SDK-/API-Prüfung und **Opt-in + Snapshot + Rollback**. Falls nicht validierbar: manuelle Anleitung statt automatischer Änderung. Keine globalen Treiberänderungen als Standard.
- **V3 Render-Integration:** Reflex, NIS, DLSS/Streamline oder andere NVIDIA-Funktionen sind **keine Preset-Toggles ohne echte Renderer-/GPU-/SDK-/Motion-Vector-/Depth-/Frame-Pacing- und Lizenzqualifikation**. Nicht jede Funktion ist im Vanilla-OpenGL-Backend oder auf jeder GeForce verfügbar.
- **Nicht-NVIDIA-Fallback:** generische Presets und Benchmark-Tuner funktionieren ohne NVIDIA SDK; Vendor Assist ist nur ein optionaler Provider.
- **Nutzerdaten:** lokale Profiles/Reports als Default, keine unbeschlossene Cloud-Telemetrie, keine Auth-/Save-/Driver-Secrets im Git-/Evidence-Report.

## Test- und Akzeptanzmatrix

| Gate | Positivtest | Negativtest / Block |
|---|---|---|
| CFG-01 schema | Profil laden, migrieren und exportieren; desired/effective/supported stimmen | Unknown-Key/ABI-/Schema-Mismatch, Rollback nach Fehler |
| CFG-02 manual | Custom Overrides über Relaunch erhalten, UI/Backend-Wert identisch | nicht unterstützte Optionen werden nicht als aktiv behauptet |
| CFG-03 preset | gleiche Hardware/Mods deterministische Quality/Balanced/Performance-Werte | Feature weg, Mod-Konflikt, Shader-/Resource-Lane noch UNKNOWN |
| CFG-04 benchmark | Q17-baseline-/candidate Runs mit p50/p95/p99 + Fixture-/Route-Hash | Boot-Blocker, World not ready, inkomparables Backend => kein Auto-Tune |
| CFG-05 approval | Änderungsvorschau, Opt-in, Snapshot, Rückgängigmachen, Restart required | kein stiller apply oder globaler NVIDIA-Driver-Tweak |
| CFG-06 safety | Q16 Recovery stellt gültiges Profil nach Crash/Device-Lost wieder her | Watchdog/Absturz manipuliert keinen Save-/Auth-Kontext |
| CFG-07 capability | NVIDIA und nicht-NVIDIA Geräte mit Feature-Status passend geroutet | unsupported Vendor-/SDK-Funktion wird nicht aktiv angeboten |

**Resultstatus:** IMPLEMENTATION_NOT_ATTEMPTED, LAB_QUALIFIED, VS_LIVE_QUALIFIED, MOD_COMPAT_QUALIFIED, PERF_BASELINED und Q8_OPENGL_ZERO (separat). Ein erfolgreicher UI-Prototyp ist kein Live-Renderer-PASS.

## Umsetzungsreihenfolge

| Slice | Was | Exit Gate |
|---|---|---|
| BC-GFX-01 | Bestehende Vanilla-/Launcher-/Bismut-Settings und Versions-/Ownership-Verträge inventarisieren; versioniertes Schema | Kompatibilitäts-/Roundtrip-Test, keine unnötige Doppelkonfiguration |
| BC-GFX-02 | Capability Registry + Basic Manual/Quality/Balanced/Performance/Custom Oberfläche | Supported/Effective getrennt, Vanilla- und Bismut-Pfade verwechslungssicher |
| BC-GFX-03 | Q17 Benchmark Reports als lokale Eingabe, Auto-Tuner zunächst read-only Vorschlagsmodus | drei validierte Repeats, transparente Empfehlungen oder BLOCKED |
| BC-GFX-04 | Nutzerfreigabe/Apply/Rollback/Restart, Settings Change Receipts | keine stillen Mutationen, Crash-Recovery-Regressionen |
| BC-GFX-05 | Opt-in NVIDIA Assist + Hardware-/Feature-Gating und neutrale anderen-Vendor-Ansicht | Vendor Tests, Rollback, dokumentierte nicht unterstützte Funktionen |

## Abhängigkeiten / Nicht-Ziele

Q4 Hybrid GUI, Q5 Rust EXE, Q7 Launcher/BCIL, Q8 GL Zero, Q12 Input Router, Q15 Loader, Q16 Recovery, Q17 Benchmark Worlds, Q18 Unified Shell gelten. Q9/Q10/Q11/Q13/Q14 bleiben Evaluierungen. Q19 beschließt **die Produktfunktion und Guardrails, nicht Performance-Schwellen oder Treiberfeature-Erfolgswerte**. Keine automatischen globalen Windows-/Treiber-/Registry-Änderungen. Kein vorschnelles Redesign des bestehenden Launchers.

Referenzen: [Roadmap V2.1](BISMUT_VS_CLIENT_ROADMAP_V2_1.md), [Q18 Unified Shell](ADR_BISMUT_Q18_UNIFIED_LAUNCHER_CLIENT_BACKEND_BENCH_NVIDIA_2026-11-19.md), [Q17 Benchmark Worlds](ADR_BISMUT_Q17_FRAME_BUDGETS_BENCHMARK_WORLDS_2026-11-19.md).
