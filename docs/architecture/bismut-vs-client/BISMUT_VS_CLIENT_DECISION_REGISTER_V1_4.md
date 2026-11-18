# Bismut VS Client | Architektur-Q&A-Entscheidungsregister V1.4

- Stand: **2026-11-18**
- Quelle: Owner-Entscheidungen aus der Bismut-VS-Client-Q&A, zusammengefasst für **Git-Versionierung**
- Status: **Q1–Q8 RATIFIED; Q9–Q11 EVALUATION APPROVED / ARCHITECTURE OPEN; Q12 DECIDED; Q13 OPEN**
- Zielprodukt: **Bismut VS Client**, separate native Rust-EXE mit Vintage-Story-.NET-Game-Runtime in-process und langfristig Rust-nativem D3D12-Renderer
- WICHTIG: **Architekturplanung ist keine Implementierungs-Evidence.** Hier wird keine Game-/GL-Zero-/Mod-Kompatibilität als PASS behauptet.

## Owner-Entscheidungen Q1–Q12

| ID | Beschluss | Verbindliche Grenze |
|---|---|---|
| Q1 | **D Rust-first Hybrid** | VS-.NET-Gameplay behalten, Rust RenderWorld/Client; vorhandenen C++-DX12-Core vorerst als Referenz erhalten; Go nicht im Frame-Hotpath |
| Q2 | **B Immutable Frame-Snapshot + Resource-Command-Queue** | Ein versionierter C-ABI, Frames atomar, Resource Handles/Generations, geordnete Create/Update/Release, GPU-Fence Retirement |
| Q3 | **A + C Dual Animation** | VS bleibt Pose-Owner für Vanilla-Avatar/Entities, DX12 GPU-Skinning; optionaler Rust Native Animator als Library für Bismut-Mods |
| Q4 | **D Hybrid GUI** | VS-Game-/Mod-GUI-Semantik + native Rust Client UI, gemeinsamer DX12-UI-Compositor, ein Present-Owner |
| Q5 | **B Direkter Rust Client** | Rust-EXE besitzt Top-Level-HWND, .NET via In-process-Hosting; kein separater Gameplay-Prozess |
| Q6 | **C Rust Native DX12** | Langfristig vollständige D3D12-Device/PSO/Resource/Swapchain/Present-Ownership in Rust, **nicht wgpu**; inkrementell portieren |
| Q7 | **D Vollständige VS-Mod-Kompatibilität als V1-Ziel** | Launcher/Installer verwaltet Versionen und Bismut-VS-System-Pakete; BCIL mit REA-Analyse; Atlas prüft serverseitige Mods; Custom-GL-Adapter benötigen echte Evidence |
| Q8 | **C OpenGL-Zero-Evidence-Gate** | Statische Analyse + prozessweite GL-Traces Start-to-Exit + echte DX12-World-/Avatar-/UI-/Mod-Frames + Negativtests |
| Q9 | **Alle A/B/C/D Bootstrap-Wege testen** | Keine Variante ratifiziert: Original-Entry/Harmony, eigener Managed Adapter/hostfxr, isolierte IL-Transformation, begrenzter Runtime-Fork-Spike |
| Q10 | **E Threading-Benchmark A/B/C/D** | Keine einzelne Thread-/Scheduler-Architektur ratifiziert |
| Q11 | **E Memory-Benchmark A/B/C/D; D im Fokus** | Rust Memory Arenas als Hypothese; Safety und Ownership vor Speed; keine Variante ratifiziert |
| Q12 | **B Rust Input Router** | Rust physischer Input-/Focus-/Capture-Owner; VS-Adapter erhält Game-/Mod-Hotkey-Semantik; [ADR Q12](ADR_BISMUT_Q12_RUST_INPUT_ROUTER_2026-11-18.md) |

## Q13 OPEN | Shader Pipeline

Wie sollen wir Source-HLSL, DXC, Reflection, DXIL, PSO, Content-Hash-Cache, Shader-Varianten, Hot Reload und VS-/Mod-Shader-Compatibility organisieren?

| Option | Modell |
|---|---|
| A | HLSL + DXC + native D3D12 PSO-Cache |
| B | **Rust Shader Pipeline Manager** auf HLSL/DXC, mit Reflection, Content-Hash-/PSO-Cache und transaktionalem Hot Reload (Vorschlag) |
| C | Shader-IR/-Graph und austauschbare Compiler |
| D | Zwei Quellen: VS-kompatible Übersetzung und separate Bismut Shader Pipeline |
| E | A–D zunächst evidenzbasiert vergleichen |

**Q13 ist nicht entschieden.** Kein Tooling-Benchmark oder Live-Hot-Reload-PASS impliziert.

## Priorisierte technische Phasen

1. **P0**: Repo-/Evidence-Audit, Q9/10/11-Lanes im isolierten Lab, einheitliche C-ABI, echter DX12-Avatar-Vertical-Slice; vorhandenen Child-Window-Fallback nutzen.
2. **P1**: Native Rust-EXE/Top-Level-HWND, In-process-.NET-Host, Rust-DX12-Cutover passweise gegen C++-Core qualifizieren.
3. **P2**: Native GUI-/Input-/Animation-/Shader-/Asset-Pipelines und VS-/Mod-Kompatibilitätsadapter; Loading ohne GL.
4. **P3**: Bismut Launcher/BCIL/Installer, Atlas headless Mod-Lane, echte DX12 Client-E2E-Suite, Q8 OpenGL Zero, Performance und Recovery.

Die Q7-D-Mod- und Q8-C-OpenGL-Zero-Qualifikation dürfen nicht aus Atlas-Server-PASS oder aus vorhandenem C++-Child-Window-/Offline-Lab-PASS abgeleitet werden. Altbinaries, Shader/Assets und VS-Interna dürfen nicht ungeprüft redistribuiert werden.

## Git-Integration und Follow-up

Dieses Register und der Q12-ADR bilden die **Git-Materialisierung des Entscheidungsstands**, nicht die vollständigen historischen Q1–Q11-Volltextdokumente. Die umfassende Chat-Roadmap V1.4 und historische ADRs müssen beim anschließenden lokalen Projekt-Audit in die kanonischen Engine-Docs übernommen und auf Pfade, aktuelle Heads, Dirty State und Herkunft abgeglichen werden. **Keine automatische Promotion in die Engine-Foundation.**
