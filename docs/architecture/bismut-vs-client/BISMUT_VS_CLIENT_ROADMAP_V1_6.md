# Bismut VS Client | Roadmap V1.6 (Git-Entscheidungsfassung)

- Datum: 2026-11-18
- Status: **Q1–Q8 und Q12 ARCHITECTURE_DECIDED / Q9–Q11 + Q13–Q14 EVALUATION_APPROVED, ARCHITECTURE_OPEN / Q15 OPEN**
- Diese kompakte Git-Fassung enthält die ratifizierten Architekturentscheidungen, Evaluierungs-Gates und Umsetzungsschritte. Die ausführliche Chat-Roadmap V1.6 (einschließlich historischer Details und Quellverweise) bleibt ein separater Artefakt-Export und muss im lokalen Engine-Projekt mit dem tatsächlichen Stand reconciliert werden. **Kein Implementation-PASS** durch dieses Dokument.
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
| 15 | OPEN: Mod-SDK und ABI-Versionierung | Öffentliche Mod-/Plugin-Schnittstellen auf Basis des bereits beschlossenen versionierten Q2 C-ABI-Vertrags festlegen |

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
| P2 / R4–R6 | Entity/GUI/Input/Loading, native Mod SDK/Animation, Shader-Q13- und Resource-Q14-Evaluierungen sowie Q15-Mod-SDK-/ABI-Entscheid und Umsetzung nach Gate | Game-/Mod-Szenarien mit klarer Statusmatrix, keine ungenehmigte GL-Rückkehr |
| P3 / R7 | Launcher-Bismut-VS-System-Installer, BCIL-Version-/Capability-Matrix, Atlas-Headless-Lane und echter Client-E2E-Harness | Reproduzierbare OpenGL-Zero-Evidence, Mod- und Performance-Akzeptanz, Rollback/Vanilla |

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

## E. Q15 OPEN | Mod-SDK und öffentliche ABI-Versionierung

Q2 hat den internen versionierten C-ABI-/Frame-/Resource-Vertrag bereits beschlossen. **Q15 entscheidet die öffentliche Mod-/Plugin-SDK-Oberfläche und Versionierungsstrategie**, nicht erneut die Notwendigkeit des internen C-ABI-Vertrags.

| Option | Öffentliche SDK-Strategie | Trade-off |
|---|---|---|
| A | .NET-first SDK mit bestehender VS-Mod-API und internem Bismut-Adapter | Höchste VS-Mod-Nähe, eingeschränkte native Erweiterbarkeit |
| **B** | Dual SDK: .NET-Fassade + Rust-Crate über stabile versionierte C-ABI, Capability-/Version-Negotiation | VS- und native Bismut-Mods, klarer ABI-Vertrag (Empfehlung) |
| C | Native Rust Plugins (cdylib), nur mit expliziter stabiler C-ABI an Plugin-Grenzen | Maximale native Erweiterbarkeit, hohe Unsafe-/Update-Verantwortung |
| D | Manifest-/Capability-basierte isolierte WASM-/IPC-Erweiterungen neben VS-Adapter | Gute Isolation, aber deutlich höherer Funktions- und Perf-Aufwand |
| E | A–D experimentell vergleichen | Keine Vorfestlegung, höherer Evaluierungsaufwand |

Unabhängig von der Option benötigen öffentliche ABI-Verträge klare Major-/Minor-Versionen, Feature Bits, struct-size/version checks, Fehlercodes, Handle-Generations, Lifetime-/Thread-/Reentrancy-Regeln und abgesicherte Mod-Provenienz. **Kein stabiles Rust-native ABI voraussetzen.** Q7-D volle Kompatibilität bleibt V1-Ziel, kein ungetesteter PASS.

## F. Reconciliation in kanonischen lokalen Engine-Docs

Vor Implementierung per lokalem Agenten die aktuelle Codebasis und alle maßgeblichen Handoff-/Roadmap-/ADR-Indizes lesen, Versionen/Pfade gegen HEAD/Dirty State und Evidenz abgleichen, dieses Git-Register und die vollständige ausführliche Roadmap integrieren, Duplikate vermeiden und keine historischen Gates ohne erneute Tests promoten. Dieses Git-Dokument ändert weder den lokalen Checkout noch Game- oder Renderer-Binaries.
