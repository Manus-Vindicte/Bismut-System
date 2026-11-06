# BISMUT ENGINE · Performance Talk / Engine Cartography

**Stand:** 2026-11-07  
**Charakter:** Gesprächs- und Research-Snapshot; keine neue Freigabe, kein kanonischer Performance-Nachweis.  
**Scope:** Vintage Story 1.22.7, Bismut Mod und native DX12-Video-Engine, Rust-Mesher/Scheduler, Referenz-Mods, DirectX- und GPU-Optimierung.  
**Testsystem:** PC1 Ryzen 5 5600X, RTX 3070 8 GB, Windows 11, 1080p. PC2 RTX 3050 bleibt PARKED.

## 1 · Kerngedanke / Entscheidung

Wir haben Vintage Story weitgehend auf einen eigenen DX12-Ausgabepfad übertragen, aber noch **keinen vollständigen DX12-nativen Optimierungsdurchlauf über alle Funktionen** abgeschlossen. Der erste Fokus war Funktionsfähigkeit, dann direkte Game→Video-Übernahme und GPU-Driven-Prototypen. Als Nächstes wird **die gesamte Engine kartografiert**, nicht von einem lokalen FPS-Fix zum nächsten gesprungen.

**Arbeitsregel:** DISCOVER → TRACE → MEASURE → MAP → PRIORITIZE → isoliert OPTIMIZE → A/B VERIFY. Zunächst Cartography/Diagnostik, keine neuen Performance-Features auf Verdacht.

Ein vermeintliches Vintage-Story-Limit ist nicht automatisch ein Game-Engine-Bottleneck: Erst gleiche Szenen für Vanilla OpenGL, bisherigen DX12-Hook-Pfad und neue entkoppelte DX12-Bridge messen. Game-/Simulationstakt, Render-Takt, Chunk-Streaming und GPU-Laufzeit separat betrachten.

## 2 · Aktuelles Architekturmodell (berichtet, noch live vollständig zu qualifizieren)

1. **Vintage Story Game/World:** Simulation, Chunk-I/O, Worldgen, Tessellation-Worker, Completion Queues.
2. **Boundary:** `ChunkRenderer.AddTesselatedChunk(TesselatedChunk, ClientChunk)` liefert fertige MeshData vor OpenGL-Pooling/Dispose. Read-only-Hook und C#-Datenvertrag vorhanden.
3. **Native Delta-Übergabe:** unmanaged Vertex-/Index-Payloads (laut Bericht 28 B/Vertex), Chunk-Versionen, Remove/ClearAll; Ownership und Freigaben absichern.
4. **Bismut Video State:** 64-bit Kamera/Scene Snapshots, persistente Chunk-Residency; DX12 DEFAULT-Heap via UPLOAD-Staging optional.
5. **GPU-Ausführung:** Compute-Frustum-Culling, GPU-generierte Argument-/Count-Buffer, `ExecuteIndirect`, D3D12/DXGI Present; Deferred Releases über Fences.
6. **Noch nicht vollständig entkoppelt:** GUI, Entities, Animationen, Partikel und Sonderpasses. Das GUI-PBO ist asynchroner GL→CPU-Readback, keine GL-freie native Lösung.

**Kritische Frage:** Wie viele reale Millisekunden kosten die Übergänge zwischen diesen Stationen? Der Renderloop darf unabhängig präsentieren, doch frische Weltzustände kommen weiterhin aus der Simulation. Ein erneut präsentierter Frame ist kein neuer Game-Tick.

## 3 · Gemeldete Meilensteine und Messwerte

| Ereignis | Berichteter Befund | Einordnung / Vorbehalt |
|---|---|---|
| Ursprünglicher Direct-DX12-Prototyp | Modus C ~3,26 FPS; Terrain-Rasterzeit ~1,72 ms | Rasterzeit ist **nicht** die Gesamtframetime; stark instrumentierter Altstand |
| Readback/Telemetry-Fixes | synchrone I/O und `GL.ReadPixels` weitgehend aus kritischem Pfad entfernt | Später tauchte GUI-Composite-Readback erneut als separate Last auf |
| Metrics Grabber | BismutMetrics; Evidence-ZIP; Crash-Recovery nach hartem Prozessabbruch; A/B-Analyzer | 60-s-Lauf gemeldet; unabhängige Verifikation der erfassten Presents weiterhin sinnvoll |
| Rust-Mesher synthetisch | 4 Threads: ~2.988 Chunks/s Rust vs. ~1.362 C++; 64-Chunks-Parität berichtet | Compiler-/Algorithmus-/Worker-Fairness prüfen, kein direkter Game-FPS-Speedup |
| Rust Shadow im Spiel | 300 echte Chunks; 60.445 Quads; ~818,61 µs/Chunk | Shadow-Ausführung berichtet; aktive Producer-Übernahme und echte Mesh-Gleichheit für breiten Weltmix separat prüfen |
| PBO + Coalescing | Overall 43,09 → 49,14 FPS; Steady-State 63,80 → 68,12 FPS | MINIMUM-POTATO-/Testbedingungen; Cold/Steady-Partition und Presenter-Zähler prüfen |
| Game→Video Boundary | REA für VS 1.22.7, 36/36 Boundary-Tests; Delta-Tap | Unit- und Implementierungsnachweis, keine vollständige Live-End-to-End-Parität |
| Native Bridge + GPU-Driven | 28/28 native Gates; DEFAULT-Heap; Compute-Culling; `ExecuteIndirect`; Deferred Releases | Testumfang vs. reale Renderfamilien und Stresstests unterscheiden |
| Native Video Engine Live | 32/64/128/256 **Chunks**: 121,9/87,7/67,5/61,7 FPS laut 300-s-Qualifikationsbericht | Chunkzahl **nicht** Sichtweite in Metern; exakte Profil-, Zeitfenster-, Present-, P95/P99- und Bildparitäts-Evidence erforderlich |
| DEFAULT vs. UPLOAD Mikrobm. | DEFAULT 2,502 ms, UPLOAD 1,994 ms | Der konkrete Mikrobm. zeigt **keinen** DEFAULT-Gewinn; 448 GB/s ist GPU-Spezifikation, keine eigene gemessene Bandbreite |

Die Zahlen sind **vom Engineering-Agenten gemeldete Ergebnisse** aus der bisherigen Unterhaltung und keine vom Autor dieses Snapshots erneut ausgeführten Tests. PASS in einem Agentenbericht ≠ vollständige unabhängige Produktqualifikation. Aussagen wie „0 % CPU“, „100 % VRAM-leakfrei“ oder „60 FPS Full Stable“ gelten nur innerhalb der konkret beobachteten Messbedingungen.

**Genannte Projekt-Commits (aus Berichten):** Metrics `251117654bab17210893d7ba4089c5753a16b537`; Graphics Sweep / Rust Shadow `077d896`; High-Distance `01c3197` / `1ae7292`; Boundary `068e639`; Bridge `2f756bc` / `4938d6e`; Live Qualification `0f87d44` / `9805dc2`. Ursprüngliche Entwicklung auf `feat/dx12-live-pass-pt02c` und `feat/native-mod-manager-takeover`; Push-Status pro Bericht verschieden, letzter Live-Qualifikationsbericht meldet PUSH YES.

## 4 · Was Vintage Story selbst bereits mitbringt

- Asynchrones Chunk-Laden / Worldgen und Terrain-Tessellation mit Worker-/Completion-Queues.
- Der Render-/Hauptthread führt jedoch bestimmte Completion-/Upload-Aufgaben und Spiel-/Netzwerkaufgaben aus; ein Burst kann Frames verlängern.
- OpenGL verfügt bereits über Mesh-Wiederverwendung, Pooling, Culling und bewährte Shader-/Resource-Lifecycles.
- Bismut muss unterscheiden zwischen **VS-internem Queue-/Tick-Stall**, **Bridge-Copy-/Ownership-Kosten**, **DX12-Upload-/Fence-Stall** und **GPU-Raster-/Compute-Zeit**.

## 5 · Externe Referenzimplementierungen (Forschung, nicht Codeübernahme)

| Referenz | Konkretes Muster | Bismut-Frage |
|---|---|---|
| Vintage Story **Komet** | `ChunkLoadBudget` begrenzt Main-Thread-Chunk-Jobs (Default ~3 ms, Backlog-Boost ~8 ms im untersuchten nightly-Code) | Verursachen Ankunftsbursts lange Game-/Renderframes? |
| Komet | `ChunkBudget` begrenzt Mesh-Uploads je Frame; `TessWorkers` eigene ChunkTesselator-Instanzen; `TessGovernor` reagiert auf Worker-/GC-Druck | Brauchen wir Frame-Budgets und eine adaptive Worker-Obergrenze an mehreren Stellen? |
| Komet | `IndirectDraw`, Frustum-/Occlusion-Culling, Pacer, Vulkan `GlTap`, FrameBridge mit vorherigem Frame | Welche Techniken bringen objektiv Gewinn und welche erhalten bloß optische Flüssigkeit? |
| Vintage Story **OptiTime** | Upload-, Entity-/GUI-/Shader-nahe Einzeloptimierungen | Isolierbare Vergleichsprofile statt Totalumbau |
| **Sodium** | Region-Buffer, Mesh-Builder, Section-Manager, Culling und gebündelte Draws | Ist unser ResidentChunk-Store effizient genug? |
| **C2ME** | Paralleles Chunk-Loading/Generieren/I/O und Scheduling | Welche Queues benötigen Backpressure / weniger Context-Switches? |
| **ImmediatelyFast** | GUI-, Text-, Entity- und Particle-Batching | Nach Terrain der nächste getrennte Renderfamilien-Test |
| **Lithium / FerriteCore** | Game-Logic-Optimierung bzw. Speicherdatenstrukturen | CPU-/GC-Bottlenecks, die DX12 nicht löst |
| **Distant Horizons / Voxy** | Persistente Fernsicht-LOD-Architektur | Separate Quality-/Fernsichtoption, **keine** versteckte Qualitätsminderung im Ultra-Benchmark |
| **Nvidium** | GPU-/Mesh-Shader-lastige Terrainverarbeitung | Forschungsbenchmark gegen bestehendes `ExecuteIndirect`, nicht automatisch schneller |

**Konkrete Komet-Quellen (untersuchter `nightly`-Stand, nicht automatisch stabile Release-Funktionen):**
- https://github.com/xToast-dev/Komet/blob/nightly/Komet.Chunks/ChunkLoadBudget.cs
- https://github.com/xToast-dev/Komet/blob/nightly/Komet.Chunks/ChunkBudget.cs
- https://github.com/xToast-dev/Komet/blob/nightly/Komet.Tessellation/Scheduling/TessWorkers.cs
- https://github.com/xToast-dev/Komet/blob/nightly/Komet.Tessellation/Scheduling/TessGovernor.cs
- https://github.com/xToast-dev/Komet/blob/nightly/Komet.Rendering/Pools/IndirectDraw.cs
- https://github.com/xToast-dev/Komet/blob/nightly/Komet.Vulkan/Engine/VulkanRenderer.Only.cs
- https://github.com/xToast-dev/Komet/blob/nightly/Komet.Vulkan/Frame/FrameBridge.cs

**Bisherige Schlussfolgerung:** Nicht „alle Worker maximal auslasten“, sondern CPU-/GC-/I/O-/Upload-Arbeit so budgetieren, dass der kritische Frame-Pfad nicht verdrängt wird. Vor Umsetzung anhand eigener Traces bestätigen. Quellen/Lizenzen und VS-Versionskompatibilität beachten.

## 6 · DX12-, NVIDIA- und AMD-Optimization-Atlas (Kandidaten, keine beschlossenen Änderungen)

| Thema | Hypothese / Prüfung | Priorität |
|---|---|---|
| D3D12 Memory Allocator, VRAM-Suballocation | Weniger große Ressourcen, besserer Residency-/Fragmentierungsstatus | P0 |
| Copy Queue, Asynchronous Uploads, Enhanced Barriers / Fences | Transfer und GPU-Rendering überlappen, unnötige Stalls vermeiden | P0 |
| PIX, Nsight Graphics, ETW/WPA, dotnet-trace, BismutMetrics | Vollständige CPU/GPU/Queue-Timeline und reproduzierbare Ursache | P0 |
| Nsight Aftermath / DRED | GPU-Crash-/Device-Hang-Evidence in Metrics-Graber | P0 |
| Mesh Shaders / Meshlets | Terrain-Submesh-Ausführung gegen bisherigen Indexed/Indirect-Pfad testen | P1 |
| Work Graphs | GPU-internes Scheduling isoliert gegen Compute + ExecuteIndirect benchmarken | P1 |
| GPU Upload Heaps, Sampler Feedback, Tiled Resources | Geräteabhängige Upload-/Textur-Optimierungen | P1–P2 |
| NVAPI / DLSS / Reflex / DXR | Nur optionale NVIDIA-spezifische Erweiterungen, portable Baseline behalten | später |

**DX12-Feature-Gates:** Agility SDK / Runtime, Shader Model, GPU-Tiers, Treiber und `CheckFeatureSupport` am echten Gerät erheben. Als besprochene SDK-Standpunkte: 1.620 Retail und 1.722 Preview (keine ungeprüfte Preview-Produktionsmigration). Feature-Verfügbarkeit oder FPS-Gewinn niemals allein aus GPU-Markennamen ableiten.

**AMD-Teststrategie:** Ein `AMD_COMPAT`-Profil kann NVIDIA-only Engine-Pfade abschalten und Fallbacks testen; das **emuliert keinen AMD-Treiber**. Microsoft WARP testet D3D12-Software-Korrektheit, nicht Radeon-Leistung. AMD Radeon GPU Analyzer bietet eingeschränkte Offline-Shader-Analyse; echte Treiber-/RDNA-Performance benötigt später physische AMD-Hardware bzw. geeigneten Windows-Testdienst. GPU-/Vendor-IDs nicht fälschen.

## 7 · Cartography-Programm vor den nächsten Performance-Fixes

| Lieferobjekt | Inhalt / Abnahmekriterium |
|---|---|
| ENGINE_MAP | End-to-End Game→Video→GPU-Datenfluss inkl. Thread-IDs, Queue-Kanten, Locks, Synchronisation |
| FUNCTION_ATLAS | Tatsächliche VS-Assembly-/Methoden-/Hook-Pfade, native ABI, Häufigkeit, Ownership, Aufrufkontext |
| BOTTLENECK_MATRIX | Funktion, Auslöser, CPU-/GPU-ms, Queue-/I/O-/GC-Effekt, Test-Run-ID, Evidenzstufe |
| REFERENCE_GAP_MATRIX | Vanilla vs. Komet/OptiTime/Sodium/C2ME/etc. vs. Bismut: vorhanden / anders / fehlt |
| DX12_CAPABILITY_ATLAS | Per Device/Driver unterstützte APIs, Queue- und Heap-Fähigkeiten; Testharness statt Annahmen |
| OPTIMIZATION_PLAN | Abhängigkeits- und Wirkungssortierung mit einzeln messbaren Experimenten, Rollback und Gate |

**Messmatrix:** Auf PC1 Vanilla GL, bisheriger DX12-Hook-Pfad und neue Bismut-Video-Engine; identische Welt, Kameraposition/Fahrt, Auflösung, Mods, ViewDistance in **Metern** und tatsächliche sichtbare Chunk-Zahlen getrennt ausweisen. 32/64/256 m mit Cold (Worldgen/Streaming), Warm (stationär), Bewegung, Block-Edit, Chunk-Unload/Worldwechsel; jeweils Minimum und festes Maximum-Qualitätsprofil. GPU Presents gegen Metrics Frames abgleichen; P50/P95/P99, 1%-Low, Tickzeit, Worker-/Queue-Druck, Upload-/VRAM-, Copy-, Compute-, Draw-, Fence- und Present-Zeiten dokumentieren.

**Zu klärende Inkonsistenzen:** Cold- und Steady-FPS teils beide über Overall; 300-s-Gesamtlauf vs. genaue Zeit je Konfiguration; `chunks` vs. Meter; Snapshot-Parität vs. vollständige GUI/Entity/Particle-Parität; ZERO-COPY-Anspruch vs. belegte Kopier- und Ownership-Kosten; „0 % CPU Culling/Submission“ präzisieren; heap-Mikrobm. richtig interpretieren.

## 8 · Kurze Roadmap nach der Karte

| Phase | Nächster Schritt | Gate |
|---|---|---|
| 01 · Kartografieren | REA, Profiler, Referenzen, komplette Bottleneck- und DX12-Capability-Matrix | Ursachen mit echten Traces belegbar |
| 02 · Isolieren | Größten nachgewiesenen Engpass einzeln testen, keine parallelen Architekturänderungen | Reproduzierbarer A/B-Unterschied |
| 03 · DX12 nativ optimieren | Speicher, Queues, Culling, PSOs, Shader, Meshlets **nur nach Atlas-Priorität** | Bildparität und bessere P95/P99/CPU-/GPU-Zeit |
| 04 · Game/Video erweitern | GUI, Entity, Animation und Partikel getrennt entkoppeln | Renderfamilien korrekt, Vanilla-Fallback möglich |
| 05 · PC2 Real-World | Nach PC1 Stable Bismut + Metrics installieren, spielen, Evidence-ZIP übertragen | RTX 3050 stabil, Vergleich gegen Vanilla derselben Hardware |

## 9 · Maßgebliche vorhandene Projektartefakte

- `docs/architecture/INVESTIGATION_GAME_TO_VIDEO_BOUNDARY_RE.md`
- `docs/architecture/INVESTIGATION_GAME_TO_VIDEO_RUNTIME_TRACING.md`
- `docs/architecture/SPEC_BISMUT_GAME_TO_VIDEO_INTERFACE_V1.md`
- `docs/reports/BISMUT_NATIVE_VIDEO_ENGINE_LIVE_QUALIFICATION_300S_REPORT.md`
- `BISMUT_NATIVE_VIDEO_ENGINE_LIVE_QUALIFICATION_TRACKER.md`
- `BISMUT_DX12_FULL_STABLE_PERFORMANCE_TRACKER.md`
- `HIGH_DISTANCE_PERFORMANCE_AND_ADAPTIVE_CHUNK_TRACKER.md`
- `AUTOMATED_GRAPHICS_SETTINGS_SWEEP_TRACKER.md`
- `BISMUT_METRICS_GRABBER_TWO_PC_EVIDENCE_TRACKER.md`

**Status:** Ideen- und Performance-Gespräch dokumentiert. Dieser Snapshot ersetzt keine Original-Repos, Logs, Benchmarks, Git-Diffs oder kanonischen Tracker. Nächster fachlicher Schritt ist die vollständige Engine Cartography und erst danach evidenzbasierte Optimierungsarbeit.