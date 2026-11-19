# Bismut VS Client | Roadmap V2.0 (Git-Entscheidungsfassung)

- Datum: 2026-11-19
- Status: **Q1–Q8, Q12, Q15–Q18 ARCHITECTURE_DECIDED / Q9–Q11 + Q13–Q14 EVALUATION_APPROVED, ARCHITECTURE_OPEN / Q19 OPEN**
- Diese kompakte Git-Fassung enthält die ratifizierten Architekturentscheidungen, Evaluierungs-Gates und Umsetzungsschritte. Die ausführliche Chat-Roadmap V2.0 (einschließlich historischer Details und Quellverweise) bleibt ein separater Artefakt-Export und muss im lokalen Engine-Projekt mit dem tatsächlichen Stand reconciliert werden. **Kein Implementation-PASS** durch dieses Dokument.
- Produkt: **Bismut VS Client**, Rust-EXE mit eigenem Top-Level-HWND, Vintage Story .NET Game Runtime in-process, langfristig Rust-native Direct3D 12 (nicht wgpu). Vanilla OpenGL als separater Recovery-/Vergleichspfad.

## A. Architekturentscheidungen

| Q | Owner-Beschluss | Verbindliche Bedeutung |
|---|---|---|
| 1 | D: Rust-first Hybrid | Rust RenderWorld/Client, .NET-Spielmechanik erhalten; C++-DX12 nur bis zu qualifiziertem Rust-Cutover; Go nicht im Render-Hotpath |
| 2 | B: Frame-Snapshot + Resource Queue | Atomarer, unveränderlicher Frame-Snapshot mit versionierter C-ABI; getrennte Ressourcen-Commands, Generation-Handles und GPU-Fence Retirement |
| 3 | A + C: Dual Animation | VS Pose-Owner für Vanilla; DX12-GPU-Skinning; optionaler Rust Native Animator als Bismut-Mod-SDK, nicht als impliziter VS-Ersatz |
| 4 | D: Hybrid GUI | VS-/Mod-GUI-Logik + Rust Client UI, ein DX12-UI-Compositor |
| 5 | B: Direkter Rust Client | Rust EXE/Window-Lifecycle, VS .NET in-process; keine getrennte Game-Prozess-Grenze |
| 6 | C: Rust Native DX12 | Vollständiger nativer D3D12-GPU-Core in Rust als Endzustand, C++-Core weiterhin qualifizierte Referenz |
| 7 | D: Volle VS-Mod-Kompatibilität als V1-Ziel | Bismut Launcher/Installer mit versioniertem VS-System, BCIL/REA-Mod-Analyse und Atlas headless Mod-Tests; Custom-GL-Mods benötigen eigene DX12-Adapter und Live-Evidence |
| 8 | C: OpenGL-Zero-Evidence | Statische Inspektion + prozessweite Runtime-Traces + echte DX12-Frames + Mod-/Negativtests; ein finaler DX12-Present-Owner |
| 9 | A/B/C/D testen | Original-Entry/Harmony, Managed hostfxr Bootstrap, isolierte IL-Patches, begrenzter eigener Runtime-Integration-Slice; Architektur OPEN |
| 10 | E: A/B/C/D Threading vergleichen | Window/Game/Render/Task-Scheduler-Varianten; kein endgültiger Thread-Owner beschlossen |
| 11 | E: A/B/C/D Speicher vergleichen | Rust Native Arenas D ist Hypothesenfavorit, B-Ownership kann dazu komplementär sein; kein Lifetime-Entscheid |
| 12 | B: Rust Input Router | Rust Win32/Input-/Focus-Owner, versionierter .NET-Adapter erhält Gameplay-/Mod-Hotkeys; optionales Replay nicht automatisch Teil von Q12-B |
| 13 | E: A/B/C/D Shader-Pipelines vergleichen | Kein produktiver Shader-Manager beschlossen; [ADR Q13](ADR_BISMUT_Q13_SHADER_PIPELINE_EVALUATION_2026-11-18.md) |
| 14 | E: A/B/C/D GPU Resource Management vergleichen | Allokations- und Streaming-Lanes mit gemeinsamen Lifetime-/Fence-Gates; [ADR Q14](ADR_BISMUT_Q14_GPU_RESOURCE_MANAGEMENT_EVALUATION_2026-11-18.md) |
| 15 | B + Bismut Loader: Dual .NET-/Rust-Mod-SDK, Forge-inspirierte Loader-Lifecycle-/Registry-Schicht | VS-Mod-Loader bleibt Owner der bestehenden VS-Mods; Rust Loader orchestriert Bismut-native Mods über versionierte C-ABI; [ADR Q15](ADR_BISMUT_Q15_DUAL_SDK_BISMUT_LOADER_2026-11-18.md) |
| 16 | B + D: Rust Recovery Supervisor + externer Watchdog | In-process Recovery State Machine für kontrollierbare Fehler; externer Helper für Crash-/Hang-Evidence, Launcher-Handoff und sichere Folgestarts; [ADR Q16](ADR_BISMUT_Q16_RUST_RECOVERY_SUPERVISOR_EXTERNAL_WATCHDOG_2026-11-18.md) |
| 17 | B + Benchmark Worlds: Stage-Frame-Time-Budgets, p50/p95/p99 und reproduzierbare geskriptete Testwelten | Feste Route/Fixture/Mod-/Grafikprofile, Warm-up, Capture, compare vs Vanilla nur bei Parität; [ADR Q17](ADR_BISMUT_Q17_FRAME_BUDGETS_BENCHMARK_WORLDS_2026-11-19.md) |
| 18 | D+ Unified Launcher/Client Shell mit Backend-Picker, Q17 Benchmark Worlds und optionalem NVIDIA Assist | Eine Rust-native Shell mit Home/Game/Control-Center; Installation als Launcher-Service, VS-.NET in-process nur auf Bismut-DX12-Pfad; [ADR Q18](ADR_BISMUT_Q18_UNIFIED_LAUNCHER_CLIENT_BACKEND_BENCH_NVIDIA_2026-11-19.md) |
| 19 | OPEN: Grafiksettings und Presets | Profile, Quality/Performance, Benchmark-Autotune, NVIDIA/Device-Capability-Gates präzisieren |

[Q12 ADR](ADR_BISMUT_Q12_RUST_INPUT_ROUTER_2026-11-18.md) ist dokumentiert; Q9–Q11 sowie Q13–Q14 sind **Evaluationen, nicht qualifizierte Implementierungen**.

## B. Q13 Shader-Pipeline-Testvertrag

| Lane | Arbeitsinhalt | Qualifikation |
|---|---|---|
| Q13-A | Native HLSL/DXC, DXIL, Root Signatures, DX12 PSO-Cache | Cold/Warm-Compile, PSO-Build, Failure-Diagnostics |
| Q13-B | Rust Shader Manager, Reflection, Content-/Include-Hashes, Shader Variants, PSO-Cache, transaktionaler Hot Reload | deterministischer Cache, korrekter Rollback, p50/p95/p99 |
| Q13-C | Begrenzte Shader-IR/-Graph-Ausgabe auf HLSL | Binding-/Pixel-Parität und Komplexitätsgrenzen |
| Q13-D | VS-/Mod-Kompatibilitätsadapter auf denselben DX12-Pfad | World/Avatar/Entity/UI/Shadow/Custom-Shader-Semantik, UNKNOWN wo ungetestet |

**Testbedingungen:** gepinnte Versionen von DXC/SDK/Rust/Shader Model/GPU/Treiber, identische Szenen, Shader-Hashes, Source/Include/PSO-Manifest, Cold/Warm-Baselines, drei kontrollierte Läufe, Negative-Tests für Shader-Fehler, Cache-Korruption, Root-Signature-Drift, Concurrent Reload, Device Lost und Shutdown. Atlas-Server-PASS ersetzt keine Render-Evidence. Alle A–D-Lanes bis zur wirklichen Ausführung **NOT_ATTEMPTED**. Ein automatischer Benchmark-Sieger ist kein ratifizierter Architekturentscheid.

## C. Routen und priorisierte Umsetzung

| Phase | Arbeit | Exit Gate |
|---|---|---|
| P0 / R0–R2 | Lokalen Engine-/Launcher-/Mod-Checkout, Handoff, Dirty State, VS-/GL-Call-Chain inventarisieren; Q9/Q10/Q11-Lab-Vergleich; Frame-/Resource-ABI; echter DX12 Avatar Vertical Slice | Qualifizierte Managed Pose-/Mesh-Ingestion mit tatsächlichem Avatar, nicht nur native Unit-Tests |
| P1 / R3 | Rust Client HWND, .NET in-process, Rust-DX12 Device/Queues/PSO/Present passweise migrieren; C++-Fallback erhalten | Live VS Game Loop + ein valider DX12 Present Owner, Recovery-/Watchdog-Verträge und qualifizierte Exit-Pfade |
| P2 / R4–R6 | Entity/GUI/Input/Loading, native Mod SDK/Animation, Q16-Recovery-Slices, Shader-Q13- und Resource-Q14-Evaluierungen sowie Q15-Dual-SDK-/Loader-Vertrag und Umsetzung nach Gate | Game-/Mod-Szenarien mit klarer Statusmatrix, keine ungenehmigte GL-Rückkehr |
| P3 / R7 | Launcher-Bismut-VS-System-Installer, BCIL-Version-/Capability-Matrix, Atlas-Headless-Lane, Bismut-Loader-Mod-Matrix, Unified Shell, NVIDIA Assist und echter Client-E2E-Harness | Reproduzierbare OpenGL-Zero-Evidence, Mod- und Q17-Benchmark-Welten und Frame-Budgets, Crash-/Hang-Recovery und Rollback/Vanilla |

**Keine Gate-Promotion:** DX12-03/05/06, aktuelle Live-Welt, Custom-GL-Mods und OpenGL Zero sind durch die Planung nicht PASS. Originale Saves/Profile/Auth, Dirty-Worktree und ungeklärte Lizenz-/Shader-Provenienz unverändert lassen.

## D. Q14 EVALUATION APPROVED | GPU Resource Management

**Owner-Entscheidung: Q14 Option E.** Wir vergleichen A–D experimentell, ohne voreilig eine GPU-Allocator-/Residency-Architektur festzulegen.

| Lane | Strategie | Mess- und Qualifikationsziel |
|---|---|---|
| Q14-A | D3D12 Committed Resources pro Asset | Reproduzierbare Baseline, Alloc/Free-Kosten und Fence-Safety |
| Q14-B | Rust GPU Resource Manager (Heap-Pools, Suballocation, Upload-Ring, Handles) | Fragmentierung, Ressourcenauslastung, Staging-/Fence-Lifetime |
| Q14-C | D3D12 Memory Allocator / vergleichbarer Allokator über Rust-FFI | FFI-Lifecycle und Allocator-Wirkung, Lizenz-/Versionsprüfung |
| Q14-D | VRAM-Budget-/Residency-first Streaming, Eviction, gegebenenfalls Defragmentierung | Budget-Stalls, Residency, Thrash, korrekte Recovery unter Speicherdruck |

**Methodik:** A/B/C sind primär Allocation-/Lifetime-Alternativen; D ist eine dazu kombinierbare Streaming-/Residency-Policy und darf nicht isoliert als FPS-Ranking gegen A betrachtet werden. Gleiche Mesh-/Textur-/Chunk-/Avatar-/GUI-Last, gepinnte SDK-/Compiler-/Treiberstände, mindestens drei kontrollierte Läufe; messen p50/p95/p99 von Alloc/Upload/Frame-Time, VRAM committed/resident/budget, Fragmentierung, Upload-Bandbreite, Fence-Waits, Device-Lost und Recovery. Negative Tests für Stale Handles, doppelte Releases, Device Removal, OOM/Descriptor-Erschöpfung und ausstehende GPU-Fences.

**Status aller Lanes: NOT_ATTEMPTED.** Q14-D VRAM-Policies sind nicht dasselbe wie Q11-D CPU-Memory-Arenas. Q2 Resource Queue und GPU-Fence-Retirement gelten in jeder Lane. Das Ziel Q8 OpenGL Zero und Atlas-Headless-Tests bleiben separate Evidenz-Gates. Der C++-DX12-Referenzpfad wird nicht vorzeitig entfernt.

[ADR Q14 mit detailliertem Testvertrag](ADR_BISMUT_Q14_GPU_RESOURCE_MANAGEMENT_EVALUATION_2026-11-18.md)

## E. Q15 DECIDED | Dual SDK + Forge-inspirierter Bismut Loader

**Owner-Entscheidung: Option B, erweitert um Bismut Loader.** VS-.NET-Mods nutzen weiterhin den originalen Vintage-Story-Mod-Loader mit modinfo.json und ModSystem. Die Bismut-.NET-Fassade erlaubt optionalen Zugriff auf native Bismut-APIs. Für neue native Bismut-Mods gibt es Rust SDK Crates und einen **Bismut Loader** mit Manifest-Discovery, Dependency Resolver, Capability Negotiation, Event-/Lifecycle-Scheduling, Registry-Verträgen, Diagnostics und Shutdown. Keine zweite, konkurrierende VS-Mod-Ladeinstanz.

| Schicht | Owner | Grenze |
|---|---|---|
| Bismut Launcher / Bismut VS System Installer | Deployment, Profiles, Modpacks, Version-Resolver, BCIL, Rollback | Bereitstellung und Vertrauensprüfung vor Client-Start |
| Vintage Story Mod Loader | VS .NET Runtime | Bereits vorhandene VS-Mods bleiben in ihrem Loader |
| Bismut Loader (Rust) | Bismut VS Client | Registrieren und Aktivieren Bismut-nativer Mods; Forge-artige Lifecycle-/Dependency-Ideen, kein Forge-Fork |
| Bismut .NET SDK + Rust SDK | Public API | Versionierte C-ABI/API-Tabelle mit ABI Major/Minor, struct_size, Capabilities, Ownership/Handle, Errors, Thread-Reentrancy |
| Rust RenderWorld + Native DX12 | Ein GPU-/Present-Owner | Mods registrieren provider-/eventbasierte Schnittstellen, keinen eigenen ungeregelten Device-/Swapchain-Owner |

**V1-Start-Slices:** duale ABI-Smokes, deklarative Loader-Metadaten, dependency order/conflict checks, Legacy-VS-Mod + .NET-Bismut-Fassade, native Rust-Demo mit Asset/UI/Animator-Registry, Negativtests. **Keine Code-Hot-Reload-/Unloading-Garantie für native DLLs** und keine Sandbox-Behauptung für in-process nativen Code. Volle Q7-D Mod-Kompatibilität und Q8-C OpenGL Zero bleiben gesonderte Game- und DX12-Evidence-Gates.

[ADR Q15 · Dual SDK und Bismut Loader](ADR_BISMUT_Q15_DUAL_SDK_BISMUT_LOADER_2026-11-18.md)

## F. Q16 DECIDED | Rust Recovery Supervisor + externer Watchdog

**Owner-Entscheidung: B + ergänzendes D.** Eine Rust Recovery State Machine klassifiziert im Bismut VS Client Mod-/Adapter-/Shader-/GPU-Fehler, führt kontrolliertes Quiesce, explizit freigegebene Neuinitialisierung, Failure-Receipts und sicheren Exit aus. Ein minimaler **externer Watchdog** erkennt Client-Prozessabsturz bzw. qualifizierten Heartbeat-/Hang-Zustand und liefert Crash-/Hang-Evidence an den **Bismut Launcher**; der Launcher besitzt Rollback-/Vanilla-/Neustart-Entscheidungen.

| Ebene | Scope | Grenze |
|---|---|---|
| Rust Supervisor (in-process) | Preflight/Running/Degraded/Quiesce/Recovery/RestartRequired/Stopped; begrenzte Retries | Nur für als sicher beweisbare Fehlerklassen; keine Fortsetzung nach möglicher Speicherkorruption |
| Watchdog (separater Helper) | Prozess-/Heartbeat-Überwachung, Exit-/Crash-/Hang-Evidence, sichere Launcher-Übergabe | Kein zweiter .NET-Game-/DX12-/Present-Owner, kein blindes Beenden fremder Prozesse |
| Launcher | Quarantäne für Folge-Session, Paket-/Profil-Rollback, kontrollierter Neustart, Vanilla-Modus | Keine automatischen Änderungen produktiver Saves/Auth oder stiller Wechsel mitten im DX12-Frame |

**Qualifikationsgates:** Mod-/ABI-/Managed-Fehler, Device Removed/Reset/Fence, Prozesscrash und Stale Heartbeat per isolierter Fault Injection prüfen. Originale Saves/Profile schützen; Recovery-GPU-Lifetime und Q2 Resource-Generations respektieren. Native In-process Plugins sind nicht sandboxed. Keine automatisierten Neustartschleifen oder unbelegten Live-PASS-Claims.

[ADR Q16 mit Failure Classes, State Machine und PT-Slices](ADR_BISMUT_Q16_RUST_RECOVERY_SUPERVISOR_EXTERNAL_WATCHDOG_2026-11-18.md)

## G. Q17 DECIDED | Stage Frame Budgets + Benchmark Worlds

**Owner-Entscheidung: Option B und zusätzliche Bismut Benchmark Worlds.** Ein CS2-Benchmark-Map-artiger Testmodus mit festen Vintage-Story-World-Fixtures, Scripted Camera Routes, Warm-up und reproduzierbaren Messfenstern wird in den Bismut VS Client und den Launcher eingebunden. Keine Übernahme fremder Maps oder Assets.

| Benchmark | Last | Mess-Gate |
|---|---|---|
| BC-BENCH-01 Baseline | Konstante Standardregion + fest definierte Kameraroute | Game-/Bridge-/Render-/GPU-/Present-Zeiten |
| BC-BENCH-02 Chunk Stress | Schnell bewegte Kamera, Chunk-Aufbau und Streaming | Stalls, Uploads, Chunk Ready vs Warm-/Cold-Capture |
| BC-BENCH-03 Entity Stress | Realer Avatar, Animation, GPU-Skinning, Instanzen | VS Managed Ingestion und native GPU-Render-Parität |
| BC-BENCH-04 Graphics Stress | Licht, Schatten, Transparenz, Effekte und GUI | tatsächliche Renderpass-Aktivierung und Messintegrität |
| BC-BENCH-05 Mod Stress | Gepinnte Vintage-Story- und Bismut-Mods, UI/Asset/Renderer | Loader-/SDK-Kompatibilität und echter Client-Present |

**Messverfahren:** Auswertung von Present-Framedauer und Pipeline-Stages über p50/p95/p99, FPS, Frame-/Tick-/Bridge-IDs, CPU-/GPU-Zeiten (nur mit verifizierter Quelle), RAM/VRAM/Resident-/Budget, GPU-/Chunk-/Shader-Stalls, Shader-/Asset-Caches und erfasster Instrumentierungs-Overhead. Mindestens drei gültige Vergleichsruns je Fixture. CPU-/GPU-Stages können überlappen und dürfen nicht unkritisch summiert werden.

**Reproduzierbarkeit:** gepinnte Version/Hashes von World-Fixture, Saves in isolierten Kopien, Modset, Assets, Kameraroute, Einstellungen, Auflösung/Render-Scale/FOV, Spielzeit/Wetter, Hardware-/GPU-/Treiber-/SDK-/Build-Profil. Erst WORLD_READY + CHUNKS_READY + RENDER_READY, danach Warm-up, dann Capture. Der bekannte historische Spawn-Chunks-0%-Hänger ist ein BOOT_BLOCKED und kein FPS-Result.

**Vergleich:** Vanilla OpenGL, C++-DX12-Referenz und Rust DX12 erhalten getrennte, ausdrücklich benannte Backend-Profile. Kein Vergleichs-PASS bei fehlenden Avatar-/GUI-/Mod-/Depth-/Shader-Pässen. Das Performance-Budget-Design ist **DECIDED**, konkrete numeric PASS-Schwellen je Szene/Hardware noch **NEEDS_BASELINE**. Das frühere 70-FPS-Ziel ist nicht als PASS nachgewiesen.

[ADR Q17 · Frame Time + Benchmark Worlds](ADR_BISMUT_Q17_FRAME_BUDGETS_BENCHMARK_WORLDS_2026-11-19.md) · [Benchmark Suite V1 Spec](BISMUT_VS_CLIENT_BENCHMARK_SUITE_V1_SPEC_2026-11-19.md)

## H. Q18 DECIDED | Ein Launcher ist zugleich der Client (D+)

**Owner-Entscheidung: D mit erweiterter Unified Shell.** Eine zusammenhängende Oberfläche umfasst **Pre-Game-Launcher und nativen Game Client**, nicht zwei konkurrierende Frontends. Im Bismut-Modus bleibt Q5 bindend: Rust-EXE mit eigenem Top-Level-HWND und in-process gehosteter VS-.NET-Game-Runtime. Die existierende Launcher-Installation/Modpacks/Publishing-Funktionalität bleibt als fachlich eigener Dienst erhalten und wird über Adapter in die gemeinsame Shell überführt. Kein unqualifizierter Full-Rewrite des existierenden Launcher-Checkouts.

| UI-Bereich | Kernfunktion | Grenze |
|---|---|---|
| Home / Spielen | Version, Profile, Spielwelten, Modpacks, Start/Resume | Installer-/Profilservice als Single Source of Truth |
| Backend & Graphics | **Bismut Native DX12 / Vanilla OpenGL** explizit auswählen, Eignung und effective Backend anzeigen | Wechsel nur vor Session oder nach kontrolliertem Restart |
| Benchmarks | **Q17 Benchmark Worlds** auswählen, starten, mit gleichen Settings vergleichen | World-/Chunk-/Render-Ready, pro Backend separate Run-IDs |
| NVIDIA Assist | GPU-/Driver Detection, opt-in Optimierungsvorschläge für Vintage Story, Render-/Frame-Pacing-/Latency-Profile | Feature-Gates, explizites Consent; kein globaler ungeprüfter Driver-Tweak |
| Mod-Management | VS-Legacy-Mods, Bismut Loader, Modpacks, Version-/Capability-Matrix | Launcher Packaging vs Loader Runtime klar getrennt |
| In-Game Control Center | Settings, Performance, Benchmark, Debug, Loader-/Recovery-Status, VS GUI über Q4 | Q12 Rust Input Router und Q4 DX12 GUI-Compositor |

**NVIDIA-Plan:** zunächst optionale Empfehlungen und überprüfbare per-Backend-Presets; NVIDIA-App-spezifische Programmprofile nur über unterstützte Wege mit Nutzerfreigabe, kein unerlaubtes Registry-/Treiber-Tuning. Reflex, NIS, DLSS/Streamline erst als spätere SDK-/GPU-/Shader-/Depth-/Motion-Vector-/Lizenz-gated Integrationskandidaten. Eine GeForce-Karte allein beweist keine Verfügbarkeit und keine FPS-Steigerung; andere GPU-Marken müssen neutral funktionieren.

**Stufengates:** F0 minimaler Unified Start/Backend-Status/Vanilla-Fallback; F1 In-Game Control Center und Benchmark-Auswahl; F2 NVIDIA Assist mit Capability- und Benchmark-A/B-Evidence; F3 Konsolidierung von Launcher-Services und UI bei Parität der vorhandenen Launcher-Funktionen. Für den Vanilla-Fallback kann der normale VS-Spielprozess separat starten; **das ist kein zweiter DX12-Owner und kein heimlicher GL-Fallback in der Bismut-Runtime**. Bei fehlendem DX12-Live-Gate zeigt die UI EXPERIMENTAL/BLOCKED statt Produkt-PASS.

[ADR Q18 · Unified Launcher/Client mit Backend, Benchmark und NVIDIA Assist](ADR_BISMUT_Q18_UNIFIED_LAUNCHER_CLIENT_BACKEND_BENCH_NVIDIA_2026-11-19.md)

## I. Q19 OPEN | Grafiksettings und Presets

**Frage:** Wie organisieren wir allgemeine Grafiksettings, Backend-spezifische DX12-/GL-Optionen und den NVIDIA Assist, ohne unsupported Features zu aktivieren oder Bismut Benchmark-Welten zu verfälschen?

| Option | Modell | Trade-off |
|---|---|---|
| A | Vanilla-nahe manuelle Settings mit gesonderten DX12-Optionen | Transparent, wenig Automatik |
| **B** | Capability-aware Presets (Quality/Balanced/Performance) plus Advanced-Konfiguration pro Backend | Konsistent und kontrollierbar, braucht Support-/Feature-Matrix |
| C | Benchmark-basierter Auto-Tuner mit reproduzierbarer Q17-World-Baseline und Nutzerziel | Hardware-nahe Optimierung, nur mit qualifizierten Baselines |
| D | NVIDIA Assist als eigener opt-in Optimierungsmodus mit belegten GPU-/Treiber-/Render-Funktionen | NVIDIA-spezifisch, muss neutralen Fallback behalten |
| E | Stufen-Hybrid B + C + D, von Basis-Presets zu benchmarkbasiertem Tuning und Vendor Assist | Umfangreicher, dafür langfristig konsistent |

**Q19 bleibt OPEN.** Festgelegt durch Q18 ist die **Existenz** von Backend-Auswahl, Benchmark und NVIDIA Assist, nicht deren abschließende Preset-/Auto-Tuning-Policy. Performance-Grenzwerte benötigen Q17-Live-Baselines, Treiber-/SDK-Features sind keine impliziten Produktmerkmale.

## J. Reconciliation in kanonischen lokalen Engine-Docs

Vor Implementierung per lokalem Agenten die aktuelle Codebasis und alle maßgeblichen Handoff-/Roadmap-/ADR-Indizes lesen, Versionen/Pfade gegen HEAD/Dirty State und Evidenz abgleichen, dieses Git-Register und die vollständige ausführliche Roadmap integrieren, Duplikate vermeiden und keine historischen Gates ohne erneute Tests promoten. Dieses Git-Dokument ändert weder den lokalen Checkout noch Game- oder Renderer-Binaries.
