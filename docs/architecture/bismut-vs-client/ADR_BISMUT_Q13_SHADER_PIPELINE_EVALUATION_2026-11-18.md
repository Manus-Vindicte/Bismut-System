# ADR BISMUT Q13 | Shader Pipeline A/B/C/D Evaluation

- Date: 2026-11-18
- Owner: **Q13 Option E**
- Status: **EVALUATION_APPROVED / ARCHITECTURE_OPEN / ALL_LANES_NOT_ATTEMPTED**
- Product: Bismut VS Client (Rust EXE, VS .NET in-process, long-term Rust native D3D12)

## Entscheidung

Die vier Shader-Pipeline-Modelle Q13-A/B/C/D werden mit kontrolliertem, einheitlichem Testkorpus evaluiert. **Q13-E ratifiziert keinen technischen Renderer- oder Shader-Manager als Produktarchitektur.** Die finale Auswahl, auch eine belegte Kombination mehrerer Varianten, bedarf eines neuen Owner-Beschlusses. A/B/D können als Stufen einer gemeinsamen Pipeline komplementär sein.

Q6-C bleibt bindend: Ziel ist **Rust-native D3D12**, nicht wgpu. Ein Device-/PSO-/Swapchain-/Present-Owner. Vorhandener C++-DX12-Core einschließlich Shader-Hot-Reload bleibt Referenz und Recovery, nicht automatisch aktueller Live-PASS.

## Experiment-Matrix

| Lane | Technischer Umfang | Hauptnachweise |
|---|---|---|
| **A** | Direkte HLSL/DXC-Kompilierung zu DXIL, Root Signature, native DX12 PSO-/Pipeline-Library | Cold-/Warm-Compile, PSO-Lifecycle, Debugdiagnose, reproduzierbare Builds |
| **B** | Rust Shader Pipeline Manager über HLSL/DXC mit Reflection, Content-/Include-Hash, PSO-Cache, atomarem Hot Reload | Cache-Korrektheit, Varianten, Dependency-Invalidierung, Rollback |
| **C** | Begrenzter Shader-IR/-Graph-Spike mit HLSL-Output; kein alternativer GPU-Device-Owner | Reproduzierbarkeit, Binding-Layout und Rendering-Parität, Komplexität |
| **D** | VS-/Mod-Shader-Compatibility-Adapter, Ausgabe auf denselben Rust-DX12-PSO-/Resource-Pfad | Shader-/Render-State-/Depth-/Blend-/Shadow-Semantik; unbekannte Features UNKNOWN |

**D ist keine pauschale GLSL→DXIL-Kompatibilitätszusage.** Custom-GL-Mods mit Treiberzustand, Hooks oder GL-Ressourcen brauchen gezielte Adapter und eigene Live-Evidence.

## Vergleichsbedingungen

1. **Identische Eingaben:** kontrollierte Testshader für Triangle, World/Chunk, Avatar GPU-Skinning, Entity/Instancing, GUI/Scissor/Fonts, Shadow/Alpha, positive und absichtlich fehlerhafte Varianten. Mod-Beispiele nur mit gültiger Provenienz und Lizenzen.
2. **Gepinnte Umgebung:** Git-Hashes, Rust/C++ Compiler, DXC, Windows SDK, Shader Model, DXIL, GPU, Treiber, OS, Compile-Flags, Root Signature, PSO-Descriptor, Shader-/Include-Hashes, Cold-/Warm-Cachezustand.
3. **Metriken:** Compile p50/p95/p99, PSO-Creation, Cache Hits/Misses, CPU-/GPU-Speicher und Residency, GPU-/CPU-Frame-Time, Hot-Reload-to-Present, Failure-/Rollback-Kosten, mindestens drei reproduzierbare Läufe.
4. **Negative-Tests:** Syntaxfehler, fehlende Includes, Binding-/Root-Signature-Drift, korrupte Cache-Einträge, Shader-Model-/Treiberwechsel, parallele Compiles, Reload bei GPU-Nutzung, Device Lost, Resize, Shutdown. Keine stillen Stale-PSOs, Speicher-/Lifetime-Fehler oder doppelte Present-Owner.
5. **Evidenzstufen:** STATIC_REVIEW → SYNTHETIC_LAB → NATIVE_DX12 → VS_LIVE → MOD_COMPAT → Q8_GL_ZERO. Zwischenstufen nicht automatisch promoten. Headless Atlas belegt Game-/Server-Mod-Verhalten, **nicht** DX12-Shader-/Client-Renderparität.

## Akzeptanz- und Safety-Gates

Pro Lane: Evidence-Manifest, Builds/Logs, DXIL-/PSO-/Source-Hashes, Fehlerklassifikation, Performancevergleich, Pixel-/Depth-Captures wo reale Frames laufen, Regressionen, Lizenz-/Provenienz-Vermerk. Status nur QUALIFIED, PARTIAL, BLOCKED, FAIL, NOT_ATTEMPTED entsprechend der tatsächlich ausgeführten Tests. Entscheidungen erst nach schriftlicher Vergleichsmatrix und Owner-Ratifikation.

Q7-D volle Mod-Kompatibilität bleibt Produktziel, keine Garantie. Q8-C OpenGL-Zero-Gate bleibt separat. Q9/10/11 sind Evaluationen. Keine modifizierten Original-Saves, Credentials oder fremde Prozesse. Keine Veröffentlichung dekompilierter VS-Shader oder fremder Mod-Assets ohne Rechteprüfung.

**Empfohlene Experimentreihenfolge (keine Produktwahl):** A als native Referenz, B für Lifecycle/Cache/Reload, C als begrenzter IR-Spike, D mit VS-/Mod-Corpus. Avatar-Slice und Q9-Bootstrap dürfen nicht blockiert werden.

## Verweise

- [Bismut VS Client Roadmap V1.5 (Git-Fassung)](BISMUT_VS_CLIENT_ROADMAP_V1_5.md)
- [Q12 Rust Input Router](ADR_BISMUT_Q12_RUST_INPUT_ROUTER_2026-11-18.md)
