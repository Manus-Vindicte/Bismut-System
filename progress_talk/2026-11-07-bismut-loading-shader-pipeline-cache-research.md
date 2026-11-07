# Bismut · Mod Loading, Shader Compilation & Caching Research

**Stand:** 2026-11-07  
**Status:** RESEARCH ONLY; keine Implementierung und keine Performance-Freigabe  
**Anschluss:** Voxel Mod Ecosystem Inspiration Radar (gleicher `progress_talk/`-Bereich)
**Scope:** Vintage Story, Minecraft, Veloren, Bismut Graphics / native DirectX 12, Launcher und Forge.

## Ausgangsfrage

Welche Ansätze aus der Mod-Landschaft verkürzen Mod-Startup, World Loading, Shader Compilation und GPU-Pipeline-Erstellung? Welche Cache-Identität und Invalidierung benötigt Bismut, ohne Mod- und Hardwarekompatibilität zu verletzen?

## Recherche-Funde

| Projekt/Technik | Prinzip | Bismut-Potenzial | Quelle |
|---|---|---|---|
| Vintage Story **Shader Cache** | Kompilierte Shader wiederverwenden; experimentell, Versionsunterstützung beachten | Referenz für Shader-Caching, aber OpenGL-Artefakte nicht mit DX12-PSO gleichsetzen | https://mods.vintagestory.at/show/mod/8546 |
| Vintage Story **Optimum** | Shape-/Animator-Cache, Texture Prewarming, bounded retessellation | First-encounter stalls und wiederholte aufwändige Shape-Arbeit reduzieren | https://mods.vintagestory.at/show/mod/57052 |
| Vintage Story **Fast Map** | Persistente komprimierte Map-Seiten und GPU-Atlas | Cacheformat, Reload-Budgets und Atlas-Residency | https://mods.vintagestory.at/fastmap |
| Minecraft **DashLoader** | Prepared asset cache für Folgestarts | Warmstart, Modprofil-Invaliderung | https://modrinth.com/mod/dashloader |
| Minecraft **ModernFix** | Lazy Model Loading, veränderte Asset-Ladepfade | Kosten durch späteren Bedarf vermeiden statt pauschal vorladen | https://github.com/embeddedt/ModernFix |
| Minecraft **LazyDFU** | Teure DataFixerUpper-Initialisierung verzögern | On-demand statt unconditional startup | https://modrinth.com/mod/lazydfu |
| Veloren **AssetCache** | AssetHandles, Wiederverwendung, Hot-Reload-Abhängigkeiten | Selektive Invalidation für Shader, Modelle, Texturen | https://gitlab.com/veloren/veloren |
| Microsoft **D3D12 Cached PSO / PipelineLibrary** | GPU/Treiber-abhängiger Pipeline Cache | PSO Warmstart bei gleicher Hardware-/Treiberkonfiguration | https://learn.microsoft.com/en-us/windows/win32/direct3d12/pipeline-state-cache |

Projektspezifische Messungen sind **keine** übertragbaren Bismut-Benchmarks. VS Shader Cache hatte dokumentiert experimentellen Charakter; auch Mod-Kompatibilität und genaue VS-Versionen sind vor Installation neu zu prüfen.

## Im Bismut-Engine-Quellcode gefundene Ansatzpunkte

Repo: `Manus-Vindicte/Bismut-Engine-Dev`, Branch `feat/native-mod-manager-takeover` (geprüfter Stand Nov 2026).
Dateien:
- `native/bismut-d3d12/src/game_swapchain_v1.cpp`: `D3DCompile` für VS/PS/Compute und mehrere `CreateGraphicsPipelineState` / `CreateComputePipelineState`-Aufrufe (z. B. um Zeilen 478–481, 562, 804–807, 856, 1432, 1464, 1584, 1592).
- `native/bismut-d3d12/src/device_draw_v1.cpp`: weitere Shader-Kompilation/PSO-Erstellung (z. B. Zeilen 24–27 und 51).
- In diesen untersuchten Dateien keine persistente `GetCachedBlob`, `CachedPSO` oder `ID3D12PipelineLibrary`-Nutzung gefunden. Das ist kein umfassender Negativbeweis für das gesamte Repo.

Quelle: https://github.com/Manus-Vindicte/Bismut-Engine-Dev/blob/feat/native-mod-manager-takeover/native/bismut-d3d12/src/game_swapchain_v1.cpp

## Drei getrennte Cacheklassen

1. **Effective shader source / dependency cache:** effektiver HLSL-/GLSL-Quelltext nach Patches/Includes plus Hash.
2. **Compiled shader bytecode cache:** z. B. DXBC/DXIL; Compiler-Version und Flags Teil der Identität.
3. **Native D3D12 PSO cache:** fertige Pipeline oder CachedPSO/PipelineLibrary. Adapter-/Treiberwechsel darf zu Invalidierung führen.

Mod-Shaderpatches können den effektiven Shader ohne Dateinamenwechsel ändern. Deshalb reicht ein filename-only CacheKey nicht.

### Cache Identity V1 (Forschungsentwurf)

```text
SHA256(ArtifactType + EffectiveSourceHash + DependencyHashes
     + CompilerVersion + CompilerFlags + PipelineConfiguration
     + EngineABIVersion + RelevantModProfileHash
     + GPUAndDriverIdentity)
```

Cacheklasse bestimmt, welche Felder relevant sind: GPU+Driver für PSO, nicht zwingend für reine Mod-Metadaten. Atomisches Schreiben/Ersetzen, Bounds/Size Budget, Versionierung und sichere Rebuild-Fallbacks vorsehen. Bei Hot-Reload alte gültige Ressource halten bis neuer Build und GPU-Synchronisierung sicher abgeschlossen sind.

## Phasen (Forschung, kein Auftrag)

| Phase | Thema | Erwarteter Gate-Beleg |
|---|---|---|
| L0 | Read-only Startup-/World-Load-Profiler | Baseline, Mod discovery, dependency resolution, assembly init, atlas, shader compile, PSO creation, first-playable-frame |
| L1 | DX12 CachedPSO/PipelineLibrary Prototyp | Kalter/Warmstart, korrekte Invalidation, gleiche Bildausgabe, Zeitgewinn oder kein Gewinn dokumentiert |
| L1 | Shader/Asset Dependency Graph | Rebuild nur betroffener Pfade, Hot-Reload korrekt |
| L2 | Asset-/Shape-Prewarming | weniger First-Encounter-Spikes, kein Memory-Leak |
| L2 | Modprofile/Launcher-Cache | geänderte Versionen/Mods gezielt invalidieren |
| L3 | Persistente GPU-fertige Ressourcendaten | belegter Nutzen gegen Disk-/VRAM-Kosten |

## Instrumentierungsentwurf

- Stopwatch/Trace: mod discovery, dependency resolution, assembly load/init, asset parsing, atlas preparation, shader compile, PSO create, initial scene ready.
- Counter: Cache hit/miss/invalidation/corruption/rebuild; I/O bytes; shader count; PSO count.
- Testfälle: cold, warm, changed mod, modified shader, corrupt cache, GPU-driver upgrade, parallel startup.

## Governance

**RESEARCH ONLY.** Vor Umsetzung BN-Recovery und aktuelle Governance beachten. Contract: GOAL → ACCEPTANCE CRITERIA → IMPLEMENTATION → NEGATIVE TEST → SOURCE-GROUNDED EVIDENCE → INDEPENDENT REVIEW → STATUS. GPU-Ready ≠ In-Game-Render-PASS. Kein frei erfundener FPS-/Startup-Gewinn.

**Nächste Recherche:** Chunk Loader: dauerhaft aktive Chunks versus Chunk Streaming / Loading / Residency / Budgets.
