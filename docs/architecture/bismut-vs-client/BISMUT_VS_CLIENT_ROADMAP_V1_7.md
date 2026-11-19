# Bismut VS Client | Roadmap V1.7 (Git-Entscheidungsfassung)

- Datum: 2026-11-18
- Status: **Q1–Q8, Q12 und Q15 ARCHITECTURE_DECIDED / Q9–Q11 + Q13–Q14 EVALUATION_APPROVED, ARCHITECTURE_OPEN / Q16 OPEN**
- Diese kompakte Git-Fassung enthält die ratifizierten Architekturentscheidungen, Evaluierungs-Gates und Umsetzungsschritte. Die ausführliche Chat-Roadmap V1.7 (einschließlich historischer Details und Quellverweise) bleibt ein separater Artefakt-Export und muss im lokalen Engine-Projekt mit dem tatsächlichen Stand reconciliert werden. **Kein Implementation-PASS** durch dieses Dokument.
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
| 16 | OPEN: Runtime Recovery und Fehlerisolation | Crash-/Device-Lost-/Mod-Fehlerpfade, Restart/Failover-/Rollback-Modell festlegen |

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
| P1 / R3 | Rust Client HWND, .NET in-process, Rust-DX12 Device/Queues/PSO/Present passweise migrieren; C++-Fallback erhalten | Live VS Game Loop + ein valider DX12 Present Owner, funktionierendes Recovery |
| P2 / R4–R6 | Entity/GUI/Input/Loading, native Mod SDK/Animation, Shader-Q13- und Resource-Q14-Evaluierungen sowie Q15-Dual-SDK-/Loader-Vertrag und Umsetzung nach Gate | Game-/Mod-Szenarien mit klarer Statusmatrix, keine ungenehmigte GL-Rückkehr |
| P3 / R7 | Launcher-Bismut-VS-System-Installer, BCIL-Version-/Capability-Matrix, Atlas-Headless-Lane, Bismut-Loader-Mod-Matrix und echter Client-E2E-Harness | Reproduzierbare OpenGL-Zero-Evidence, Mod- und Performance-Akzeptanz, Rollback/Vanilla |

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

## F. Q16 OPEN | Runtime Recovery und Fehlerisolation

**Frage:** Wie erholt sich der Bismut VS Client von defekten Mods, nicht verfügbaren Game-Runtime-Adaptern, DX12 Device Removed und Fehlern während Start/Shutdown, ohne VS-Saves zu beschädigen?

| Option | Recovery-Modell | Trade-off |
|---|---|---|
| A | Fail-fast + sicherer Rücksprung zu Vanilla | Einfach, harte Unterbrechung |
| B | Rust Supervisor mit Recovery State Machine, GPU-/Mod-/Game-Fehlerklassen, Quarantäne und Launcher-Rollback | Kontrolliert, aber in-process native Speicherfehler nicht sicher isolierbar |
| C | In-process modulare Restart-/Reload-Strukturen für DX12-/Managed-Komponenten | Schnellere Teil-Recovery, anspruchsvolle ABI-/Thread-/Lifetime-Gates |
| D | Externe Watchdog-/Helper-Prozesse für crash-sichere Diagnose und isolierten Neustart | Robuster bei Prozesscrash, mehr Prozess-/IPC-Komplexität; Gameplay bleibt in-process |
| E | A–D im Lab mit reproduzierbaren Fault-Injections vergleichen | Entscheidung erst nach Evidence |

**Q16 noch OPEN.** Ein Loader, Capability-Checks oder Panic-Catch machen native in-process Plugins nicht sandboxed. Vor Restart/Shutdown müssen VS-World-/Save-Lifetime, GPU-Fences, Resource-Generations und Adapter-Ownership berücksichtigt werden.

## G. Reconciliation in kanonischen lokalen Engine-Docs

Vor Implementierung per lokalem Agenten die aktuelle Codebasis und alle maßgeblichen Handoff-/Roadmap-/ADR-Indizes lesen, Versionen/Pfade gegen HEAD/Dirty State und Evidenz abgleichen, dieses Git-Register und die vollständige ausführliche Roadmap integrieren, Duplikate vermeiden und keine historischen Gates ohne erneute Tests promoten. Dieses Git-Dokument ändert weder den lokalen Checkout noch Game- oder Renderer-Binaries.
