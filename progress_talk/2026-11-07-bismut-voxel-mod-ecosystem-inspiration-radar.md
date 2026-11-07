# Bismut · Voxel Mod Ecosystem Inspiration Radar (Research Snapshot)

**Stand:** 2026-11-07  
**Status:** RESEARCH / IDEAS ONLY, keine Implementierungsfreigabe  
**Scope:** Vintage Story, Minecraft, Veloren mit Relevanz für Bismut Engine (Rust/C# / DirectX 12), Antidotum/REA, Launcher, Forge und Regisseur.
**Ausgangspunkt:** Gesprächsrecherche zu Rendering, GPU-Residency, Chunk-Streaming, Fernterrain/LOD, Mod-Konflikten, Telemetrie und wiederholbaren Benchmarks.
**Hinweis:** Projektbeschreibungen aus öffentlichen Repos/Modseiten sind keine PC1-Benchmarks oder Bismut-Implementierungsnachweise. Lizenz, Aktualität und tatsächlicher Codepfad vor Adaption erneut prüfen.

## Executive Findings

- **Sodium:** GPU-Buffer-Arenas, inkrementelle Defragmentierung, Batching und Chunk-Render-Management; als Architektur-Referenz für GPU-Residency, **nicht** als direkter Patch.
- **C2ME:** paralleles Chunk-I/O und Generierung, Scheduling und Backpressure als Referenz für Rust Worker.
- **ImmediatelyFast:** UI/Text/Immediate-Mode Draw-Batching **am realen Aufrufer**; für BN-07 als Beleg, dass ein reiner Batch-Algorithmus ohne Eingriff in den Render-Hook nicht genügt.
- **Komet, OptiTime, Tungsten:** Hook-Konflikt-Handling, einzeln abschaltbare Optimierungen, Diagnosezähler, begrenzte Workloads und sichere Fallbacks.
- **GhostChunks / Distant Vistas / Voxy:** dedizierte Fernterrain-Daten- und Rendering-Ebene, persistenter World-/LOD-Cache, abgestufte Genauigkeit und zuverlässiges Übergangsprotokoll zu residenten High-Detail-Chunks.
- **Nvidium / Meshelium:** Hersteller-spezifische bzw. portablere GPU-getriebene Mesh-Shader-Ansätze als langfristige Vergleichsbasis zu Bismuts Compute + ExecuteIndirect.
- **Veloren:** Rust-Crate-Architektur, getrennte Spiel-/Rendering-Layer, WASM-Pluginkonzepte, Profiling und instrumentierte Workloads.
- **ReplayMod:** deterministische Kamera-/Replay-Szenarien als mögliche Basis reproduzierbarer Render-A/B-Tests; **Axiom** als UI-/World-Editing-Referenz für Forge.

## Referenzlandkarte

| Ökosystem | Projekt | Inspirationsfeld | Quelle |
|---|---|---|---|
| Minecraft | Sodium | GPU Buffer Arenas / Defragmentierung / Chunk Renderer | https://github.com/CaffeineMC/sodium |
| Minecraft | C2ME | paralleles Chunk Loading / Scheduling | https://github.com/RelativityMC/C2ME-fabric |
| Minecraft | ImmediatelyFast | GUI/Text-/Immediate-Mode Batching | https://github.com/RaphiMC/ImmediatelyFast |
| Minecraft | EntityCulling | asynchrone Visibility Checks | https://github.com/tr7zw/entityculling |
| Minecraft | Nvidium | NVIDIA Mesh Shaders | https://modrinth.com/mod/nvidium |
| Minecraft | Meshelium | Vulkan Mesh Shader / Cross-Vendor | https://modrinth.com/mod/meshelium |
| Minecraft | Voxy | Very-long-range voxel LOD | https://modrinth.com/mod/voxy |
| Minecraft | ReplayMod | Kamera-Replays / deterministische Szenarien | https://github.com/replaymod/replaymod |
| Minecraft | Axiom | World Editing / Brush Preview / UX | https://github.com/Moulberry/AxiomDocumentation |
| Vintage Story | Komet | tiefe Engine Hooks / GPU / Compatibility / Fallback | https://github.com/xToast-dev/komet |
| Vintage Story | OptiTime | modulare Optimierungen / Diagnose | https://mods.vintagestory.at/optitime |
| Vintage Story | Tungsten | Speicher / Server / Patch-Konflikte | https://mods.vintagestory.at/tungsten |
| Vintage Story | Fast Map | persistente GPU-/Karten-Seiten | https://mods.vintagestory.at/fastmap |
| Vintage Story | Terrain Sampler Lib | Terrainhöhen ohne volle Chunkgenerierung | https://mods.vintagestory.at/algernonsterrainsampler |
| Vintage Story | GhostChunks | LOD Fernterrain / World Cache | https://mods.vintagestory.at/ghostchunks |
| Vintage Story | Distant Vistas | persistente 3D-LODs | https://github.com/leroysquad/DistantVistas |
| Vintage Story | VSPT | alternative Renderer / RTX / GUI Komposition | https://mods.vintagestory.at/vspt |
| Veloren | Veloren | Rust-Architektur / Renderer / WASM / Telemetrie | https://gitlab.com/veloren/veloren |

## Bismut-Ideen, nach Priorität (keine Umsetzung autorisiert)

| Phase | Forschung | Vorbilder | Mess-/Entscheidungs-Gate |
|---|---|---|---|
| R0 | BN-01–08 Recovery abschließen | Eigene Engine | Produktpfad + Negativtest + PC1-Hardware-Evidenz |
| R1 | GPU Chunk Residency / Fragmentierung | Sodium | GPU allocations, VRAM, frame pacing, upload stalls |
| R1 | Hook-Kompatibilität / Ownership | Komet, OptiTime, Tungsten | Konflikterkennung und fail-closed ohne Gameplay-Schäden |
| R1 | Echtes GUI-Batching | ImmediatelyFast | Live-Aufrufer, batch draw counts, Bildparität |
| R2 | Priorisierte Chunk-Worker | C2ME, Veloren | queue p95/p99, load/mesh throughput bei realem Gameplay |
| R2 | Persistenter LOD-World-Cache | GhostChunks, Distant Vistas, Voxy | kein Loch beim High-/Low-LOD Handover |
| R3 | Optionaler Mesh-Shader-Pfad | Nvidium, Meshelium | A/B zur DX12 ExecuteIndirect Baseline; Vendor-Portabilität |
| R3 | Reproduzierbare Kamera-Replays | ReplayMod | gleicher Spielzustand, Welt, Kamera, Residency, Preset |
| Future | Sandbox Plugins | Veloren WASM | API/Permission-Isolation und Governance |

## Technische Kernideen

1. **GPU-Arena-/Residency-Allocator:** GPU Buffer nach Region/Größe zusammenfassen; Defragmentierung nur inkrementell unter Frame-Budget. Erst bestehende Residency messen.
2. **Compatibility-Aware Hook Ownership:** Antidotum prüft Hook- und Mod-Konflikte; einzelne Hook-Gruppen separat abschaltbar, Fail-Closed.
3. **Mehrstufige World Data Residency:** aktive echte Chunkmeshes, persistente 3D-LODs, nur-oberflächliche Terrain-Prognosen. LOD darf erst verschwinden, wenn hochauflösender Renderersatz **tatsächlich ready** ist.
4. **Feature-Evidenz statt PASS-Text:** Aktivierungsflag, realer Produkt-Aufruf, vermiedene Arbeit, Kosten, negative tests und Live-Gates werden getrennt erfasst.
5. **Replayable Performance Evidence:** reproduzierbare Kamerafahrt als Vergleich über gleiche Welt, Einstellungen, Warmup und Metrikfenster.

## Nicht übernehmen / Risiken

- Keine automatischen Qualitätskürzungen im Standardprofil; Zero-Quality-Cuts gilt.
- Kein unbesehenes Kopieren fremder Implementierungen; MIT/LGPL/GPL und andere Lizenzen projektbezogen prüfen.
- Kein ungeprüftes Wechseln auf Vulkan oder Mesh Shader; bestehender DX12-Pfad zuerst qualifizieren.
- Keine neuen Implementierungsaufträge bis BN-Recovery/Governance die jeweils vereinbarten Gates bestehen.
- Aussagen wie „mehr FPS“ sind erst mit belastbarer PC1-A/B-Evidenz gültig.

## Governance der Recherche

**STATUS: RESEARCH-ONLY.**  
Jede Umsetzung folgt dem Engineering Contract: GOAL → ACCEPTANCE CRITERIA → IMPLEMENTATION → NEGATIVE TEST → SOURCE-GROUNDED EVIDENCE → INDEPENDENT REVIEW → STATUS. Kein PASS von Mock, synthetischer Szene oder Dokumentationsbehauptung allein.

**Nächster Forschungsstrang:** Mod-Ladezeiten, Shader-Compilation, Pipeline State Object Caching, Asset-Cache und Startup-/World-Load-Optimierung.
