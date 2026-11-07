# Bismut · Chunk Loaders, Pregeneration, Streaming & Residency Radar

**Datum:** 2026-11-07  
**Status:** RESEARCH ONLY / NO IMPLEMENTATION AUTHORITY  
**Untersuchung:** Vintage Story, Minecraft, Veloren; Server-Chunk-Loading vs. Pre-Generation vs. Client Streaming vs. DX12 GPU Residency.
**Hinweis:** Hersteller/Mod-Performancewerte sind Claims, keine unabhängigen PC1-A/B-Tests. Versionskompatibilität bei Installationen neu prüfen.

## Wichtigste Trennung

- **Forced / pinned world chunks**: Server hält Chunk-Spieldaten auch ohne nahe Spieler resident, evtl. für Maschinen/Ticks. Nicht automatisch identisch mit Spieler-Simulation, Client-Geometrie oder GPU-Residency.
- **Pregeneration**: Chunks vor Erkundung generieren, um spätere Worldgen-Spitzen zu mindern; spart nicht automatisch Meshing/Upload/Renderkosten.
- **Chunk Streaming**: Anfragen, Priorisieren, I/O/Generation, Transport zum Client, Tessellation, Native-Upload, Ready/Unload.
- **Visual LOD**: Ferne, vereinfachte Geometrie, ohne vollständige Simulationschunks.
- **GPU residency**: Bismut-DX12-Buffer, Upload/retire per Fence und Sichtbarkeitsentscheidung. Nicht mit serverseitigem KeepLoaded verwechseln.

## Primärquellen Vintage Story API

- `IWorldManagerAPI.LoadChunkColumn(x,z,keepLoaded)`: async normal priority; `keepLoaded` hält Spalte geladen.
- `IWorldManagerAPI.LoadChunkColumnPriority(..., ChunkLoadOptions)`: async priority path für Spalte oder rechteckigen Bereich.
- `ChunkLoadOptions.KeepLoaded`, `OnLoaded`, `ChunkGenParams` dokumentiert.
- `ChunkColumnLoaded` und `ChunkColumnUnloaded` Events, beim Shutdown kann Unloaded zehntausendfach feuern.
- `/chunk forceload x1 z1 x2 z2` als Vanilla Adminfunktion.
- **Performancewarnung:** `IWorldManagerAPI.AllLoadedChunks` erstellt einen Clone unter Chunk-Dictionary-Lock. Nicht pro Frame pollen. `IWorldAccessor.LoadedChunkIndices` ebenfalls laut API nicht pro Frame abrufen.

Quellen:
- https://apidocs.vintagestory.at/api/Vintagestory.API.Server.IWorldManagerAPI.html
- https://apidocs.vintagestory.at/api/Vintagestory.API.Server.ChunkLoadOptions.html
- https://apidocs.vintagestory.at/api/Vintagestory.API.Server.IServerEventAPI.html
- https://apidocs.vintagestory.at/api/Vintagestory.API.Common.IWorldAccessor.html
- https://wiki.vintagestory.at/List_of_server_commands

## Mod-/Runtime-Landschaft Vintage Story

| Projekt | Prinzip | Erkenntnis | Quelle / Einschränkung |
|---|---|---|---|
| Chunk Loader (dejavu4you) | player-/rollen-begrenzte Force-Loading Claims, Anzeige/Commands | Quotas und klare Ownership; vergleiche mit Vanilla-Kept-Chunks | https://mods.vintagestory.at/show/mod/58202 ; ausdrücklich 1.22.0–1.22.3 |
| ChunkQueueTweaks | Queue-Overflows durch adaptive Priorisierung/Throttling abfangen | Bounded Admission und Stale-Request-Cleanup | https://mods.vintagestory.at/chunkqueuetweaks ; ausgewiesen 1.22.0 |
| VS Chunk Manager | selektives Vorladen/Neugenerieren/Löschen auf der Karte | Vorberechnungs-/Wartungswerkzeug, kein Echtzeit-GPU-Streamer | https://mods.vintagestory.at/show/mod/41441 |
| PreGen Chunk | Spiral PreGen, Resume nach Neustart, Existing-Skip und TPS-basiertes Backoff | durable cursor, pause/resume und adaptive Lastkontrolle | https://mods.vintagestory.at/pregenchunk ; 1.22.0-pre.1–1.22.7; KI-Unterstützung laut Autor |
| Stratum | Version-gepatchte Server-Runtime mit fair chunk send, generation budget, movement-aware priority, stale cleanup | konkrete Serverarchitektur-Referenz; hoher Patch-/Kompatibilitätsumfang | https://stratumvs.dev/ ; 1.22.7-stratum.3-indev.1 vom 04.11.2026; nicht kompatibel mit Synergy/Tungsten |

Weitere Einordnung: Interview-Nachbetrachtung mit VS-Entwickler, ausdrücklich kein aktueller Engine-Quellcode: https://ogyrecdev.github.io/posts/vintage-story-engine-notes/ . Modell: server-driven visibility, separate request admission, lifecycle-retention, CPU/GPU mesh lifetimes.

## Minecraft

| Projekt | Problem | Architektur-Idee | Quelle |
|---|---|---|---|
| FTB Chunks | forced chunks + ownership/Teams/offline policy | Quotas pro Spieler/Team, offline-policy, Administratorinspektor | https://docs.feed-the-beast.com/mod-docs/mods/suite/Chunks/ |
| Chunky | große Weltregionen vorab generieren, pausierbar und resumierbar | permanente Job-Cursor, Fortschritt, Backpressure | https://modrinth.com/mod/Chunky |
| C2ME | Chunk Generierung/I/O/Loading parallelisieren | Work Queue/Fairness/Schutz übriger CPU-Tasks | https://modrinth.com/mod/c2me-fabric |
| T2ME | bounded FULL chunk futures, adaptive request admission nach tick-time EWMA, heap, CPU, players | Ticket-/Lease-Cleanup auf success/failure/cancel/shutdown, Retry/Timeout, Resume | https://github.com/MimoAlexer/T2ME |
| DynamicDistance | pro Spieler begrenzte View- und Sim-Distanz ohne separate Send-Pipeline | server-authoritative demand budget; kein pauschales Chunk-Pinning | https://github.com/Codetaker56/dynamicdistance |
| Chunky Pregenerator | Bewegungstrigger, angepasste Spiralen, TPS Pause, Voxy Hooks | Prediction und LoD-Schnittstellen nur unter Budget | https://github.com/uhneer/chunkypregen |

## Veloren

- `terrain_sync.rs`: räumlich begrenzte Empfängerwahl, `rayon`-parallele Modifikationsweitergabe, EventBus für ChunkSendEntry; https://docs.veloren.net/src/veloren_server/sys/terrain_sync.rs.html
- `chunk_send.rs`: getrenntes serialisiertes Chunk-Message-Handling über crossbeam channel, per-client send und Metrik; https://docs.veloren.net/src/veloren_server/sys/chunk_send.rs.html
- `terrain_persistence.rs`: **Cache für persistierte Terrain-Modifikationen**, nicht pauschal volle Chunk-Meshes; LRU nach Anzahl Blocks, atomische FileWrites und Dirty-only Save. Achtung: `maintain()` TODO merkt an, dass reines Writeback beim Unload nicht alle Crash-Szenarien schützt. https://github.com/veloren/veloren/blob/master/server/src/terrain_persistence.rs
- Historische Engineering-Erklärung zum Chunkformat und Transfer/Kompression: https://veloren.net/blog/devblog-117/

## Bismut Architektur-Hypothese: Event-driven Chunk Lifecycle, getrennte Owner

```text
Vintage Story Server (authoritative):
   visibility/keepLoaded/generation/network ownership
      -> chunk-ready / unload / changed events
         -> Game-to-Video Tap (contract + epoch/generation)
            -> bounded request admission / dedup / cancellation
               -> Rust scheduler / tessellation work where allowed
                  -> DX12 upload / residency / draw-ready
                     -> fence-safe deferred retirement
```

**Keine ungeprüfte Server-Worldgen-Übernahme durch Bismut.** Rendering und serverseitige Simulations-Chunks getrennt halten. Native Renderersatz erst sichtbar, wenn Backend und relevante Atlas/Materialdaten `READY` sind; LOD erst entfernen, wenn High-Detail-Chunk wirklich renderbar.

### Hypothetische Request-States

`REQUESTED -> ACCEPTED -> LOADING -> MEShing -> UPLOADING -> GPU_READY -> ACTIVE -> RETIRING -> RELEASED` plus `CANCELLED` / `FAILED` / `SUPERSEDED`. Dies ist ein Designvorschlag, **keine Behauptung über existierenden Code**.

### Prioritäts- und Budgetmodell

- **Priority 0:** Spielerumgebung und aktuell sichtbare Chunks; nie ohne Budgetschutz.
- **Priority 1:** voraussichtliche Bewegungsrichtung, neue durch Gameplay angeforderte Regionen.
- **Priority 2:** LOD / Cache-Warmup / hintergründige Vorbereitung.
- **Priority 3:** vom Benutzer bestellte offline PreGen, pausierbar unter Last.
- Bounded in-flight per I/O, CPU meshing, upload bytes/frame, VRAM resident, renderer swap.
- Job keys: WorldId, Dimension, ChunkCoord, ChunkGeneration/Epoch, RequestType. Überholte Jobs nicht in aktive Residency schreiben lassen.
- Explicit lease/refcount ownership: client-visible interest, force-load interest, LOD interest, GPU fence interest sind **verschiedene Domänen**, nicht ein gemeinsames Boolean `loaded`.

## Risiken und Gates

- **Zero Quality Cuts:** Kein Fake-Chunk, keine verlorenen Chisel-/Block-Updates, keine abgeschnittenen Chunk-Ränder.
- **Memory:** unbounded keepLoaded, PreGen oder Cache führt zu RAM/VRAM-Druck; Quotas und Backpressure unverzichtbar.
- **Race:** teleport, rapid travel, unload/reload, world change, server disconnect, device lost, dimension swap.
- **Mod-Compatibility:** Andere Mods können laden/entladen, Serverruntime tief patchen; Hooks via Antidotum versioniert/fail-closed.
- **Deadlock/Lock pressure:** `AllLoadedChunks` und `LoadedChunkIndices` nicht pro Renderframe pollen.
- **Tests:** negative stale-job-apply, duplicate request, late completion after unload, cancellation, no GPU resource released before fence, queue overflow, forced-chunk quota; PC1 replay/frame/VRAM/network measurements.
- **Evidence:** Research ≠ integration. Abschlussstatus nur G1 produktive Erreichbarkeit, G2 echte negative Regression, G3 geeignete PC1-Live-Messung, unabhängiger Review.

## Empfehlenswerte Reihenfolge (nur R&D)

| Reihenfolge | Thema | Entscheidungsgate |
|---|---|---|
| 0 | BN-01..08 Recovery | vorhandene Gates |
| 1 | Read-only Chunk Queue/Residency Atlas | warten, queued, stale, cancellation, upload bytes, VRAM |
| 2 | Bounded Admission / stale dedup | gleiche Qualität, kürzere p95/p99 |
| 3 | Prediction / priority policy | schnelleres sichtbar-READY ohne mehr CPU/GPU-Stalls |
| 4 | Separates optionales PreGen-Tool | Server-Autorität, kein Gameplay-Abfall, Persistenz |
| Future | LOD handover / GPU arena | echtes Ready-Gate und Hardware-Parität |

**Status:** Forschung materialisiert. Kein Auftrag, kein Code geschrieben, keine Benchmarks ausgeführt.
