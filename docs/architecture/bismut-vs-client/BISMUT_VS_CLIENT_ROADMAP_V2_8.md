# Bismut VS Client | Roadmap V2.8 (Git-Entscheidungsfassung)

- Datum: 2026-11-19
- Status: **Q1–Q8, Q12, Q15–Q25 + Q18A EDITIONS DECIDED / Q9–Q11 + Q13–Q14 EVALUATION APPROVED, ARCHITECTURE OPEN / MASTER GOAL TWO SLICES READY_FOR_LOCAL_AGENT_HANDOFF**
- Diese kompakte Git-Fassung enthält die ratifizierten Architekturentscheidungen, Evaluierungs-Gates und Umsetzungsschritte. Die ausführliche Chat-Roadmap V2.8 (einschließlich historischer Details und Quellverweise) bleibt ein separater Artefakt-Export und muss im lokalen Engine-Projekt mit dem tatsächlichen Stand reconciliert werden. **Kein Implementation-PASS** durch dieses Dokument.
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
| 18A | **Öffentlich: Bismut Client / Bismut Client Extended**; intern: Bismut VS Client - Standard / BismutX VS Client - Extended | Ein Installer, Standard ohne Engine als Default, Extended optional; **Titelleiste immer Bismut Client**. [ADR Q18A](ADR_BISMUT_Q18A_PRODUCT_EDITIONS_INSTALLER_2026-11-19.md) |
| 19 | E: Manual/Custom + capability-aware Presets + Q17 Benchmark Auto-Tuner + opt-in NVIDIA Assist | Quality/Balanced/Performance/Custom mit desired/effective/supported, Nutzervorschau, A/B-Messung, Rollback und Backend-Gates; [ADR Q19](ADR_BISMUT_Q19_GRAPHICS_PRESETS_AUTOTUNER_NVIDIA_ASSIST_2026-11-19.md) |
| 20 | **E = B+C+D** Unified Mod Hub + reproduzierbare Lockfiles + BCIL Intelligence | **Standard** erhält ModDB, Modpacks, Profile, Resolver, Lockfiles, Snapshots/Rollback; **Extended** ergänzt Q15 Native Loader, DX12-Mod-Adapter, BCIL-/REA-Analyse. [ADR Q20](ADR_BISMUT_Q20_UNIFIED_MOD_HUB_LOCKFILES_BCIL_EDITIONS_2026-11-19.md) |
| 21 | **A + „Fehlerbericht senden“**: Minimal-Support-UI, Cloud-Diagnose ausschließlich über Vercel Gate, parallele Dev-Client-/kostenbewusste LLM-Auswertung und verständliche Hilfeergebnisse | Technik/LLM/Provider/Receipts bleiben in Standard-UI verborgen; lokaler Offline-Support, explizites Upload-Opt-in, Redaction und Fail-Closed; [ADR Q21](ADR_BISMUT_Q21_SIMPLE_SUPPORT_VERCEL_GATE_2026-11-19.md) |
| 22 | **E = B+C+D:** signierte Core/Extended-Komponenten, Delta-/Full-Fallback, Portable/Offline, transaktionales Apply/Rollback | Ein gemeinsamer Installer/Updater, Standard ohne Engine; [ADR Q22](ADR_BISMUT_Q22_DISTRIBUTION_SIGNED_COMPONENTS_DELTA_OFFLINE_2026-11-19.md) |
| 23 | **A: nur Stable**, manuell bestätigte Updates, keine öffentlichen Beta-/Nightly-Release-Kanäle | Interne CI/Lab-Tests bleiben isoliert; signierter QA-/Release-Gate, kein Auto-Download/Apply; [ADR Q23](ADR_BISMUT_Q23_STABLE_ONLY_MANUAL_UPDATES_2026-11-19.md) |
| 24 | **E = D+B+C:** minimaler First-Run, optionale Discovery-/Importvorschau und isolierte, transaktional abgesicherte Migration | Original-Installationen/Welten/Auth erhalten, read-only Discovery, WorldBinding-/ModLock-/Snapshot-/Rollback-Gates; [ADR Q24](ADR_BISMUT_Q24_FIRST_RUN_DISCOVERY_SAFE_IMPORT_2026-11-19.md) |
| 25 | **E (A+B+C+D) mit Scope-Korrektur:** vorhandene Launcher-Singleplayer-/Multiplayer-Startseite behalten; Client-Preflight, Session-/RunRecord-Bridge, Save/Exit/Recovery ergänzen | Kein zweiter Launcher, World-/Server-Browser oder Avatar-/DX12-Bridge-Rewrite; [ADR Q25](ADR_BISMUT_Q25_EXISTING_LAUNCHER_CLIENT_SESSION_2026-11-19.md) |
| MG-01 | **Team-Building Pflicht; genau zwei Implementierungsslices Engine + Client** | lokale Engine-Runs 16/17/19 und stale DLL vor Code verifizieren; S1 vorhandene Render Dispatch ABI nur mit ADR/Owner Approval, S2 Launcher Client-Bridge; [Master Goal](MASTER_GOAL_BISMUT_ENGINE_CLIENT_TWO_SLICES_2026-11-19.md) |

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
| P3 / R7 | Launcher-Bismut-VS-System-Installer, BCIL-Version-/Capability-Matrix, Atlas-Headless-Lane, Bismut-Loader-Mod-Matrix, Unified Shell, Q19 Capability-Presets/Auto-Tuner, NVIDIA Assist und echter Client-E2E-Harness | Reproduzierbare OpenGL-Zero-Evidence, Mod- und Q17-Benchmark-Welten und Frame-Budgets, Crash-/Hang-Recovery und Rollback/Vanilla |

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

## I. Q19 DECIDED | Manual Settings + Capability Presets + Q17 Auto-Tuner + NVIDIA Assist

**Owner-Beschluss: Option E**, umgesetzt als explizit stufenweiser Hybrid aus **A** (Manual/Custom), **B** (capability-aware Quality/Balanced/Performance-Presets), **C** (benchmark-basierter Auto-Tuner) und **D** (optionaler NVIDIA Assist).

| Capability | Funktionsziel | Guardrail |
|---|---|---|
| Manual / Advanced | Editierbare VS-kompatible Settings und Bismut-DX12-Optionen, pro Backend/pro Profil | Q18 Single-Source-of-Truth, desired/effective/supported getrennt |
| Quality / Balanced / Performance / Custom | Versionierte und capability-geprüfte Presets für VS-Vanilla GL und Bismut Rust-DX12 | nicht unterstützte Werte nicht aktivieren; Custom Overrides erhalten |
| Q17 Benchmark Auto-Tuner | Geskriptete Benchmark Worlds, p50/p95/p99, CPU/GPU-/VRAM-/Stall-Metriken und Änderungsvergleich | World/Chunks/Render Ready, gleiche Fixture, mehrere valide Runs, Nutzervorschau und Opt-in vor Apply |
| NVIDIA Assist | NVIDIA GPU/Treiber-/Feature-Erkennung, Programm-/Backend-spezifische Vorschläge für Vintage Story | optional, keine stillen globalen NVIDIA-/Windows-/Registry-Tweaks; neutrale AMD-/Intel-Profile |
| Advanced NVIDIA SDK | NVAPI, Reflex, NIS, DLSS/Streamline als spätere Integrationskandidaten | echte Backend-/GPU-/Shader-/Depth-/Motion-Vector-/SDK-/Lizenzqualification erforderlich, nicht als derzeit verfügbar behaupten |

**Gates:** Q17 liefert noch keine numerisch ratifizierten Performance-Ziele und keine vorhandenen vollständigen Benchmark-Welten. Der historische 70-FPS-Wert ist kein PASS. Auto-Tuning bleibt vor gültigen Q17-VS-Live-Baselines **BLOCKED/READ_ONLY**. Saved Settings mit Profil-/Game-Version/Backend/Modset/Capabilities/Driver/Route-/Build-Hashes, Change Preview, Rollback-Snapshot und Restart Required für Backend-Wechsel. Kein Auto-Apply/Cloud-Upload ohne Freigabe. Features müssen AVAILABLE, UNAVAILABLE, NEEDS_INTEGRATION oder UNVERIFIED ausweisen.

**Reihenfolge:** Existing-Settings-/Launcher-Audit → Schema/Capability Registry/Presets → read-only Auto-Tuning-Vorschläge → qualifizierter Apply/Rollback → NVIDIA Assist. Testmatrix enthält Non-NVIDIA GPU, Unsupported Feature, World/Render Not Ready, Device Lost, Mod-Mismatch und Profil-Rollback.

[ADR Q19 · Presets, Benchmark-Auto-Tuner und NVIDIA Assist](ADR_BISMUT_Q19_GRAPHICS_PRESETS_AUTOTUNER_NVIDIA_ASSIST_2026-11-19.md)

## J. Q18A DECIDED | Standard und Extended Produkteditionen

**Öffentliche Anzeigenamen und interne Produktkennungen sind getrennt:**

- **Bismut Client** (*interne Kennung: Bismut VS Client - Standard*): Standardinstallation und normaler Vanilla-/Modded-Launcher für Vintage Story, **keine Bismut Engine als zwingende Abhängigkeit**. Bestehende Launcher-Funktionen zu ModDB, Modpacks, Instanzen, Welten, Updates, Recovery bleiben in Standard und dürfen nicht hinter BismutX verschwinden.
- **Bismut Client Extended** (*interne Kennung: BismutX VS Client - Extended*): Option beim selben Installer mit Standard **plus** versionierten BismutX-Komponenten für Rust Native DX12, Q15 Bismut Loader/SDK, Q17 Benchmark Worlds und Q19 Advanced Graphics/NVIDIA Assist, jeweils erst nach tatsächlicher Capability-/Version-/Live-Qualifikation.

**UI-Branding:** In beiden Editionen lautet die Titelleiste immer **Bismut Client**. Öffentliche Menü-/Taskbar-/Shortcut-Branding-Texte verwenden **Bismut Client**, der Installer zeigt als Wahl **Bismut Client** und **Bismut Client Extended**. In der Oberfläche kann die konkrete Edition als Status/Label unter Über/Komponenten, Installation und Extended-Funktionen erscheinen. **BismutX** bleibt auf technische Paket-/Repository-/Capability-Namen beschränkt; es ist kein öffentlicher Produktname. Der Benutzer erkennt den funktionalen Unterschied anhand der bereitgestellten Module und Optionen, nicht durch eine zweite Marke.

**Installer:** Default Standard, Extended als bewusst auszuwählendes Add-on; lokal/offline entpackbare oder verifizierte herunterladbare Komponenten, transparente Version-/Disk-Space-/Dependency-/Fallback-Anzeige. Standard → Extended und Extended → Standard sind transaktionale Upgrade-/Downgrade-Wege mit Snapshots und Rollback. Beim Entfernen von Extended keine Spielstände, Mods oder Standard-Profile löschen. Game-/DX12-Feature-Gates bleiben unverändert; ein installiertes Paket ist kein Rendering-PASS.

**Abgleich zu Launcher-Dev:** die unabhängige Vanilla-/Environment-/WorldBinding-/MutationTransaction-Architektur gilt; LAUNCHER_READY_TO_CONNECT bleibt vor Engine-Provider-Anbindung ein eigenes Gate und wird nicht durch diese Editionierung übersprungen. Der gegenwärtige Electron-/React-Launcher ist ein vorhandenes Substrat, keine bereits bewiesene reine Rust-Binary. Kein paralleler zweiter Launcher, Installer oder Mod-Resolver.

[ADR Q18A · Produkteditionen und Installer](ADR_BISMUT_Q18A_PRODUCT_EDITIONS_INSTALLER_2026-11-19.md)

## K. Q20 DECIDED | Standard Mod Hub + Extended Loader, DX12 und BCIL

**Owner-Beschluss Q20-E: Hybrid B+C+D, stufenweise.** Bismut behält **einen einzigen Mod Hub**, eine kanonische ModGraph-/ModLock-/Environment-/MutationTransaction-Ownership und einen gemeinsamen Launcher. Alle normalen Modverwaltungs-/Modpack-Funktionen gehören zur **Standardedition Bismut Client**, nicht exklusiv zu Extended.

| Mod-Funktion | Bismut Client (Standard) | Bismut Client Extended |
|---|---|---|
| ModDB Browser/Cache, Mods suchen/filtern, Install/Update/Remove/Enable/Disable | Enthalten | Enthalten |
| Profile, Modpacks, Mods pro Welt/Instanz, Versionierung, Konflikt-/Dependency Resolver | Enthalten | Enthalten |
| Reproduzierbare ModPack-Lockfiles mit exakten Versionen/Hashes/Provenienz und Import/Export | Enthalten | Enthalten |
| Snapshot/MutationTransaction/Preflight/Verify/Last Known Good/Rollback | Enthalten | Enthalten |
| Basic VS-/Game-Version-/Side-/Dependency-Kompatibilität | Enthalten | Enthalten |
| Optionales ModDB Publishing (Provider-/Auth-gated) | Enthalten | Enthalten |
| Bismut Native Loader und public Rust-Mod-SDK | Nicht als Engine-Abhängigkeit erforderlich | Extended-only nach Q15-/ABI-/Runtime-Qualifikation |
| DX12/Custom-GL Mod Bridge und native Renderer-/Shader-Adapters | Nicht erforderlich für Vanilla-Betrieb | Extended-only, keine automatische Mod-Parität |
| BCIL/REA tiefe Mod-/API-/GL-/DX12-Kompatibilitätsanalyse | Kein Engine-Zwang, normale deklarative Checks bleiben | Extended-only, advisory Evidence mit UNKNOWN/STATIC_ONLY/Live-Qualifikationsstatus |

**Owner-Grenzen:** Launcher besitzt Download/Package/Resolver/Environment/WorldBinding/Lockfile/ModDB/Publishing/Mutation/Rollback; der vorhandene VS-Mod-Loader bleibt Owner existierender VS-Mods; Q15 Bismut Loader aktiviert nur Bismut-native Plugins. **BCIL liest und klassifiziert, modifiziert nicht**. Profil-/Binary-Modset-Wechsel nur über qualifizierte Shutdown-/Neustart-Transaktionen, nicht im laufenden Game.

**Vorhandene Launcher-Dev-Grundlage:** ModDB-Transport existiert; `electron/modpack/bismutPckContracts.cjs` hat Schema-V1-Mod-Lockfile mit Hash-/Version-/Dependency-Preflight, `bismutPckService.cjs` stellt eine Partial Lock/PreviousLock/Staging/Rollback-Grundlage bereit. Diese Grundlage muss in kanonischen `Environment`/`ModGraph`/`ModLock`/`MutationTransaction`-Owner konsolidiert werden, nicht neu als Schattenmodell angelegt werden. `LAUNCHER_READY_TO_CONNECT` bleibt laut Launcher-ADR-001 separat zu qualifizieren, nicht durch Extended-Installation zu umgehen.

**Gates:** deterministische Resolver-/Lockfile-Tests, hash-/source-verifizierte Offline-Importe, Provider-/Mod-Version-/Game-/Side-Konflikte, aktive World-/Save-Mutation BLOCKED, Snapshot/Verify/Rollback, Last Known Good, Custom-GL-/BCIL-UNKNOWN-Status, echter VS Client-/DX12-Mod-Frame-PASS nur bei Live-Evidence. Q7 volle VS-Mod-Kompatibilität bleibt V1-Ziel, Q8 OpenGL Zero ein separater Prozess-Gate. Keine angebliche vollständige Implementierung durch diesen ADR.

**Branding unverändert:** öffentlich Bismut Client / Bismut Client Extended; Titelleiste immer **Bismut Client**; `BismutX` nur interne technische Kennung.

[ADR Q20 · Mod Hub, Lockfiles und BCIL mit Editionstrennung](ADR_BISMUT_Q20_UNIFIED_MOD_HUB_LOCKFILES_BCIL_EDITIONS_2026-11-19.md)

## L. Q21 DECIDED | Benutzerfreundlicher Support mit einem Send-Button

**Ownerentscheidung: Option A, lokale Crash-Analyse zuerst, vertiefter Cloud-Support freiwillig.** Keine Developer-Cockpit-/GPU-/Shader-/CLI-Flut in der normalen Oberfläche. Bismut Client und Bismut Client Extended zeigen dieselbe schlichte Hilfe:

1. **Nach Spiel-/Client-Absturz lokales Crash-Panel anzeigen**: Q16 Launcher/Watchdog stellt letzten RunRecord/Logs/Mod-/Version-/Environment-Befund bereit; lokale Regeln klassifizieren **erkannt / vermutet / unbekannt** ohne Netz oder LLM. Eine mögliche sichere Lösung steht an erster Stelle.
2. **Erst als zweite Möglichkeit „Fehlerbericht senden“** anbieten: „Wenn du genauere Hilfe möchtest ...“. Vor dem freiwilligen Upload Datenvorschau und ausdrückliches Senden/Abbrechen, lokale und Gate-Redaction.
3. **Vertieftes Hilfeergebnis** fasst evidenzgeprüfte Dev-/LLM-Rückmeldung als verständliche Kurzursache und bis zu drei Schritte zusammen. Keine ungeprüften Auto-Fixes; keine Vercel-/LLM-/Provider-/Receipt-Details in normaler UI.

**Hinter der Oberfläche:** Der Client sendet **sämtliche Cloud-Diagnoseanfragen nur an das autorisierte Bismut Vercel Gate**. Dieses kontrolliert Auth/Rate Limit/Redaction/Datenaufbewahrung/Kosten/Budgets und startet serverseitig **parallel** einen Dev-Client-Analysepfad und einen bevorzugt kostenlosen oder günstig begrenzten LLM-Hilfepfad. Die beiden Ergebnisse werden nach Evidence-/Policy-Validierung zu einem internen `ResolutionReceipt` zusammengeführt; dem Benutzer erscheint ausschließlich der **Lösungsvorschlag**. Kein Client-Direktzugriff auf Dev-Client-/LLM-Provider; kein freies LLM-Autofix; ein kostenloser LLM-SLA ist **nicht garantiert**. Bei Gate-/LLM-Ausfall bleibt lokale Hilfe erreichbar.

**Wichtige Scope-Grenze:** Der Vercel-Egress-Gate gilt für **alle Cloud-Transportpfade der Q21 Diagnose-/KI-Hilfe**. Es ist **keine** stillschweigende Umlenkung vorhandener ModDB-/Update-/Publishing-Transporte. Benutzer müssen **nicht** Vercel, LLM, Dev Client, Rust, DX12, technische Receipts oder Provider kennenlernen. In erweiterten exportierbaren Supportdetails dürfen IDs und Traces bleiben; normale UI bleibt frei von Architektur.

**Datenschutz/Gates:** Kein Versand ohne Einwilligung, keine Saves/Tokens/private Pfade/minidumps by default; strukturierte Allowlist/Secret-Scanner lokal und am Gate; keine Cloud-Pflicht oder unautorisierte Systemänderungen. Vercel-/Provider-Verfügbarkeit, Quoten, Rate/Budget, DPA/Retention, LLM-Auswertung, Prompt-Injection und Fallback sind technische Implementierungsgates, noch kein PASS. Q16 RunRecord/Watchdog und Q20 ModGraph werden wiederverwendet, keine zweite Datenwahrheit.

[ADR Q21 · Einfacher Support über ein Vercel-Gate](ADR_BISMUT_Q21_SIMPLE_SUPPORT_VERCEL_GATE_2026-11-19.md)

## M. Q22 DECIDED | Ein Installer, signierte Komponenten + Delta + Portable/Offline

**Owner-Entscheidung: Option E = B+C+D, gestufte Umsetzung.** Eine gemeinsame Distribution für **Bismut Client (Standard)** und **Bismut Client Extended**; der Installer bietet **Standard als Default** und optionale Extended-Komponenten mit geprüfter Versions-/ABI-/VS-/Hardware-Kompatibilität. Sichtbare Titelleiste in beiden Editionen: **Bismut Client**.

| Baustein | Vertrag und Zuständigkeit | Qualifikationsgate |
|---|---|---|
| **Signierte Komponenten (B)** | versioniertes Release-Manifest, Publisher-Signatur/Trust-Root/Rotation, Core/Extended-Packages, SHA-256, Compatibility-/Dependency-Graph | unknown/invalid/expired/revoked Signatur oder fehlender Runtime-Support => **BLOCKED**, ohne Beschädigung des Standards |
| **Transaktionaler Updater (B)** | Provider/UpdateCoordinator Discover (read-only) → Nutzer-Preview/Consent → Resolve → Verify → Stage → Quiesce → externer Updater Helper → Health/Commit oder Rollback | kein EXE-/DLL-In-place-Overwrite während Game-Run; Crash/Power Loss/insufficient space / Locked File negativ testen |
| **Delta & Cache (C)** | Delta nur bei exaktem Base- und Target-Hash, rekonstruiertes Full-Payload verifizieren, content-addressed Cache | Cache-/Patch-Korruption => signiertes Vollpaket-Fallback, kein ungeprüfter DLL-Load |
| **Portable / Offline (D)** | signierte vollständige Offline-Core-/Extended-Archive, portable DataRoot, gleicher Update-/Rollback-Vertrag | kein Internet-/Vercel-/Engine-Zwang, keine versteckten globalen OS-/Registry-Änderungen |
| **Standard ⇄ Extended** | Komponenten per Capability-/Version-Graph installieren/entfernen, Standard-Profile/Welten/Modpacks beibehalten | Install/Update/Downgrade/Remove/Repair/Offline/Existing-User-E2E, Last Known Good |

**Simple UX:** „Update verfügbar“ → „Aktualisieren“ bzw. „Später“. Nur bei Fehlern „Problem beheben“ / „Vorherige Version wiederherstellen“; technische Paketgraphen, Kanäle und Signaturen sind unter optionalen Details verborgen. Standard-Funktionalität bleibt ohne Bismut Native DX12 und ohne Engine-Provider verfügbar. Extended-installiert allein ≠ DX12-LIVE-PASS oder Q8 OpenGL-Zero-PASS.

**Single Source of Truth:** Launcher besitzt Installation, Environment/WorldBinding, mod/config/world MutationTransaction, Profile, UpdateCoordinator/Provider und Release-Receipts. Ein externer Bootstrap-Update-Helper darf laufende Binaries ersetzen, aber keine zweite fachliche Datenwahrheit schaffen. `LAUNCHER_READY_TO_CONNECT` bleibt eigenständiges BLOCKED-Gate vor optionaler Engine-Provider-Integration. Keine Veränderung existierender Saves/Auth.

**Q21 UX-Reconciliation:** Ein Spielcrash führt **zuerst** zu einem lokalen Crash-Panel mit Log-/RunRecord-/Mod-/Environment-Analyse, klassifizierter Ursache (identified/suspected/unknown) und einer möglichen sicheren Lösung. **Nur sekundär**: „Wenn du genauere Hilfe möchtest, sende den Fehlerbericht“. Cloud-Diagnose läuft mit Einwilligung ausschließlich durch das Vercel-Gate; der Installer-/Update-Providertransport wird dadurch **nicht** umgeleitet.

[ADR Q22 · Signierte Komponenten, Delta und Offline](ADR_BISMUT_Q22_DISTRIBUTION_SIGNED_COMPONENTS_DELTA_OFFLINE_2026-11-19.md)

## N. Q23 DECIDED | Nur Stable, manuell bestätigte Updates

**Owner-Entscheidung Q23-A:** Genau **ein öffentlicher Release-Kanal: Stable**. Weder Standard noch Extended zeigen Beta-/Nightly-/Lab-Kanalwahl. Es gibt **keine** stillen Update-Downloads, Auto-Installationen, erzwungenen Restarts oder verschleierte Upgrade-Migrationen. Die normale UI zeigt „Update verfügbar“ → „Aktualisieren“ / „Später“ und bei Fehlern „Vorherige Version wiederherstellen“ nur mit belegtem Status.

| Regel | Umsetzung | Gate |
|---|---|---|
| Ein Public Channel | `stable` im öffentlichen signierten Release-Manifest | andere Kanäle dürfen nicht per Standard-/Extended-Updater ausgeliefert werden |
| Manuelle Updates | read-only UpdateCoordinator discovery; vor Download/Apply explizite Nutzerbestätigung | laufendes Spiel, Snapshot/WorldBinding und Q22 Transaction/Updater korrekt |
| Interne CI/Lab-Tests erlaubt | isolierte Test- und Release-Candidate-Artefakte ohne öffentliche Updater-/UI-Exposition | keine unbeabsichtigte Beta-/Nightly-Promotion |
| Stable-Promotion | manuelles signiertes QA-/Release-Approval mit Q22 Trust/Integrity, Engine-/Mod-/World-/Recovery-Gates | nicht gemessene Client-/GL-Zero-/DX12-Gates bleiben BLOCKED |
| Extended-Sicherheit | nur zur Stable-Core-/VS-/Runtime-/ABI-Matrix kompatible Extended Packs | bei fehlender Extended-Evidence Standard Stable separat freigeben |
| Rollback | signierte vorherige qualifizierte Version + lokaler Last Known Good | kein unbewiesener Rollback-PASS, kein Save-/Auth-Downgrade |

**Q21 Ergänzung, Owner:** Die lokale Crash-/Log-/RunRecord-Analyse zeigt **zuerst** ein verständliches Crash-Panel mit möglicher Lösung. **Erst danach** kann der Benutzer „Fehlerbericht senden“ für genauere Hilfe wählen. Serverseitig hinter dem **einzigen Vercel-Gate für Q21-Cloud-Diagnose** läuft ein free-first **Gemini API Free Tier → ein weiterer nach Privacy-/Free-Tier-Audit zugelassener Anbieter → regelbasierte Dev-Analyse ohne LLM** Fallback. Keine automatische kostenpflichtige Abrechnung oder Provider-/Modellwahl für Benutzer. Da Gemini-Free-Tier-Daten zur Produktverbesserung verwendet werden können, darf der Free-Tier nur strikt minimierte, anonymisierte, allowlist-basierte Diagnosedaten erhalten und auch nur nach dokumentierter Einwilligung/Datenverarbeitungsprüfung; sonst **LLM BLOCKED**, lokale Hilfe bleibt aktiv. Vercel AI Gateway-Billing nicht mit Gemini-API-Free-Tier gleichsetzen. Kein automatischer Support-Upload.

[ADR Q23 · Stable-only und manuelle Update-Policy](ADR_BISMUT_Q23_STABLE_ONLY_MANUAL_UPDATES_2026-11-19.md) · [Q21 ADR aktualisiert um Free-Tier-Fallback](ADR_BISMUT_Q21_SIMPLE_SUPPORT_VERCEL_GATE_2026-11-19.md)

## O. Q24 DECIDED | Schnellstart, freiwilliger Import und sichere Migration

**Ownerbeschluss: Option E = D + B + C.** Der erste Start soll einfach sein. Bismut erkennt eine vorhandene Vintage-Story-Installation und deren Instanzen/Welten/Mods **lesend** und zeigt bei eindeutigem, qualifiziertem Ergebnis direkt „Spielen“. Gibt es mehrere Installationen oder fehlt VS, führt nur ein kurzer verständlicher Dialog zur Auswahl. Der Importassistent ist **optional**, keine erzwungene Migration oder doppelte Anmeldung.

| Stufe | Sichtbarer Ablauf | Technischer Schutz |
|---|---|---|
| First Run / Fast Path | „Vintage Story gefunden“ → „Spielen“ oder „Vorhandene Daten übernehmen“ | keine Quellmutation, ohne Bismut Engine/Cloud im Standard |
| Discovery / Preview | Found installations, worlds, mods, profiles; Auswahl und „Später“ | nur read-only Scanner; Herkunft, Version, Pfadkonflikte, IDs und Hashes validieren |
| Plan / Dry Run | kompakte Übernahme-Vorschau, Konflikte und Speicherbedarf | kanonische `Environment`/`WorldBinding`/`ModLock`-Ownership, keine Schatten-Datenbank |
| Copy / Verify / Commit | Zustimmung → isolierte Kopie in separate Instance/DataRoot → Verify | Snapshot, Laufzeit-/World-Lock, Hash-/Schema-/Provenienz-Check, MutationTransaction, Rollback |
| Fehler / Resume | „Übernahme nicht abgeschlossen“, Retry/Skip/Details | Originalsaves unverändert nachgewiesen; kein Fake-PASS; keine Auth-/Save-/Mod-Mutation ohne Freigabe |

**Existing-state-gates:** `electron/instance-manager.cjs` (partielle Create/Clone/Import/Archive-Funktionen) und `electron/game-content-manager.cjs` (partielle Settings-/World-Adapter) sind wiederzuverwendende Launcher-Grundlagen. Die kanonischen Launcher-Docs klassifizieren WorldBinding und gemeinsame MutationTransaction bislang als fehlend/partial. Erst die PT-BL-001..005-Verträge und reale Windows-E2E-Tests qualifizieren einen sicheren importierten `Environment`. `LAUNCHER_READY_TO_CONNECT` bleibt für Extended-Provider unabhängig davon BLOCKED, bis der separaten Launcher-ADR-001-Gate erfüllt ist.

**Privacy und Releases:** kein automatischer Upload von Welt-/Accountdaten, keine Token-/Auth-Duplikation, portable/offline Pfade und VS-Lizenz-/Auth-Regeln respektieren; öffentliche Downloads ausschließlich Stable (Q23). Standard ohne Bismut Engine; Extended darf für Import keine zusätzliche Runtime erzwingen. Die Erststart-UI zeigt keine internen technischen Architekturbegriffe.

[ADR Q24 · Read-only Discovery und sichere Migration](ADR_BISMUT_Q24_FIRST_RUN_DISCOVERY_SAFE_IMPORT_2026-11-19.md)

## P. Q25 DECIDED | Bestehender Launcher als Client-Session-UI

**Ownerentscheidung Option E mit expliziter Korrektur:** Die bestehende Bismut-Launcher-Startseite besitzt bereits Singleplayer-/Multiplayer-World-/Server-Karten und die Aktionen **„Spielen“ / „Beitreten“**. Keine neuen Seiten und keine Duplikate. Q25 E fügt an genau diese Aktionen einen kontrollierten **LaunchPlan-Preflight**, den tatsächlichen Runtime-/Backend-Start und den **Session-Lifecycle (RunRecord, Exit, Crash, Return to Launcher)** an.

| Schritt | Owner | Gate |
|---|---|---|
| World / Server auswählen | vorhandene Launcher UI und Launcher RuntimeSupervisor | ein Intent/Run, eindeutige WorldBinding-/Server-Identität |
| Silent Preflight | canonical Launcher Environment/WorldBinding/ModLock/LaunchPlan | Game-/Mod-/Version-/Lease-/Backend-Konflikte melden, keine stille Save-/GL-Mutation |
| Vanilla / Extended Launch | Standard Vanilla ohne Engine; Extended nur mit qualifiziertem Runtime-Provider | `LAUNCHER_READY_TO_CONNECT` bleibt bis echter Qualifikation BLOCKED; kein DX12-/OpenGL-Zero-PASS durch einfachen Start |
| Save / Exit / Return | Launcher/Client/RunRecord/Watchdog nach echten Ownership-Verträgen | kein verlorener Save, kein doppelter Runtime Owner; Crash zeigt Q21 lokales Panel vor optionalem Vercel-Bericht |
| Reuse | `electron/runtime-supervisor.cjs`, Launcher Home, `direct-world` / `join-server` | kein zweiter World Picker, Launcher Rewrite oder System-Truth |

### Zwingend: zuletzt berichtete Engine-Realität berücksichtigen

Der Owner meldete **lokale, noch ungepushte** Commits `17e127f`, `aa4efa3`, `6ccbf05` auf `pt-bl-native-window-mod-20261117` (clean laut Bericht). **Run 16** texturierter Third-Person-Avatar auf DX12; **Run 17** SOLID-Magenta-Oracle und SRV 1; **Run 19** AV-05 Pose-Änderung **120/120 Frames** (Guard-Build `1EDB170B`, 6 Draws/Frame). Runs 16–18 liefen laut Bericht mit **stale DLL `CF4DD9C` ohne Guard**, weil `dotnet build` das `bin\\Release\\net10.0\\BismutGraphics\\`-Paket nicht selbst aktualisiert. Diese Fakten sind **Owner-reported / LOCAL_VERIFICATION_REQUIRED** und kein neues Git-E2E-PASS. Avatar/W3 bleiben **PARTIAL**; Defekt E (Escape Pause → Fenster schließen) ist manuell nicht requalifiziert. AV-06/07 sichtbare Bewegung/Equipment/Transparenz bleiben zu prüfen.

**Engineering-Konsequenz:** kein neuer Avatar Renderer und **keine zweite Dynamic-Mesh-/DX12-Bridge**. Reale nächste Lücken sind `passMask/visibility + FrameId`-Command-List-Bindung sowie transparente Entity-Passes, Shadows, GUI/Effects, Resource-Handles, Instancing. `docs/PT-BISMUT-RENDER-DISPATCH-01_GAP_ANALYSIS.md` ist nach Report **RESEARCH_DONE / PENDING_DECISION**. **Vor jeder ABI-Mutation erst additive Contract-ADR und explizite Owner-Freigabe**. Der C++-DX12-Live-Pfad ist nicht identisch mit der Q5/Q6 Rust-Native-Zielarchitektur.

### Master Goal: Team-Building Pflicht, zwei Umsetzungsslices

- **Team-Gate vor Code:** echte Lead-/Engine-/Client-/Validator-Agenten/Delegationen mit Role-/Task-/Branch-/Lease-Evidence; wenn nicht verfügbar **BLOCKED_TEAM_NOT_AVAILABLE**.
- **Slice 1 ENGINE:** verifizierte aktuelle C++-/DX12 Dynamic-Mesh-Bridge **minimal und versioniert** weiterführen, nach ADR/Owner-Freigabe `passMask/visibility + FrameId`; Avatar-/Pose-/Atlas-Gates erhalten. Defekt E/AV-06/07 sind manuell gegated, kein erfundener PASS.
- **Slice 2 CLIENT:** bestehende Launcher UI und RuntimeSupervisor mit `LaunchPlan/RunRecord/Preflight/Exit/Watchdog` koppeln; Vanilla unabhängig, Extended nur nach echten Provider-/Game-/DX12-Live-Gates.
- **Ein gemeinsamer Abschlussbericht**: realer Teamnachweis, Repo-/Commit-/DLL-Hashes, genau zwei Slice-Status, Testkommandos/Receipts, blockierte manuelle Prüfungen, Integrations-/Safety-Gates. Keine Codeumsetzung durch das Anlegen dieser Docs.

[ADR Q25 · Existing Launcher/Client Session](ADR_BISMUT_Q25_EXISTING_LAUNCHER_CLIENT_SESSION_2026-11-19.md)

[MASTER GOAL · Engine + Client, Team-Building, genau zwei Slices](MASTER_GOAL_BISMUT_ENGINE_CLIENT_TWO_SLICES_2026-11-19.md)

## Q. Next Gate | Local Implementation and Owner Review

Nicht weitere Launcher-Oberflächen entwerfen: **zuerst** Master Goal an das lokale Agenten-Orchestrator-Team übergeben, Team-Building und tatsächliche Git-/Engine-/Launcher-HEADs verifizieren. **Nächste Owner-Entscheidung: additive Dynamic-Mesh-/Render-Dispatch-ABI (`passMask/visibility + FrameId`) auf Basis der lokalen Gap-Analyse.** Ohne diese explizite Freigabe bleibt Slice-1-Code-Gate gesperrt. Q9/Q10/Q11/Q13/Q14-Lanes bleiben offen, nicht durch Q25 entschieden.

## R. Reconciliation in kanonischen lokalen Engine-Docs

Vor Implementierung per lokalem Agenten die aktuelle Codebasis und alle maßgeblichen Handoff-/Roadmap-/ADR-Indizes lesen, Versionen/Pfade gegen HEAD/Dirty State und Evidenz abgleichen, dieses Git-Register und die vollständige ausführliche Roadmap integrieren, Duplikate vermeiden und keine historischen Gates ohne erneute Tests promoten. Dieses Git-Dokument ändert weder den lokalen Checkout noch Game- oder Renderer-Binaries.
