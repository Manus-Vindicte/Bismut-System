# Bismut · Notebook-Essenz: Voxel-Rendering-Architektur & Optimierung

**Datum:** 2026-11-07  
**Dokumenttyp:** Verdichtete Forschungsnotiz / Decision Support  
**Status:** `RESEARCH_ONLY` · keine Implementierungsfreigabe · kein Benchmark-Nachweis  
**Quelle:** Vom Nutzer bereitgestellte Notebook-Ausarbeitung *„Architekturevaluation und Optimierungsblaupause für modernste Voxel-Rendering-Engines“* (Abschnitte 1–4, 34 Referenzeinträge).  
**Projektkontext:** Bismut DX12-Renderer, Rust-/C#-Bridge, Workpack V1, laufender PT-26 und ausstehende unabhängige Prüfung.

## Executive Essence

Der Bericht schlägt fünf Entwicklungsrichtungen vor: **Work Graphs + Mesh Nodes**, **zweistufiges HZB-Occlusion-Culling**, **64-Bit-Vertex-Packing**, **GPU_UPLOAD-Heaps mit Resizable BAR** sowie **eine zweistufige Nah-/Fernfeld-Pipeline mit SVDAG/LOD**. Kernziel ist die Skalierung auf größere Sichtweiten bei weniger Draw-Overhead, geringerer VRAM-/Upload-Last und stabileren Frametimes.

**Unsere Bewertung:** Als *Forschungslandkarte* brauchbar. Nicht als neue verbindliche Engine-Roadmap übernehmen: Zwei vorgeschlagene Sofortmaßnahmen widersprechen ausdrücklich unseren bisherigen V1-NO-GOs; einige Technologie- und Performanceaussagen sind unzureichend belegt oder zu absolut formuliert.

## Entscheidungsübersicht

| Vorschlag aus dem Notebook | Notebook-Hypothese | Bismut-Bewertung | Nächstes zulässiges Gate |
|---|---|---|---|
| **Work Graphs + Mesh Nodes** | GPU-gesteuerte Arbeit ersetzt Compute-Culling + `ExecuteIndirect`; weniger Steuerpuffer und Synchronisierung | **FUTURE / RESEARCH**: Work Graphs und Graphics/Mesh Nodes technologisch trennen; Hardware-/SDK-/Treiber-Support zuerst abfragen | DX12 Capability Probe, PIX-Traces, A/B gegen bestehenden `ExecuteIndirect`-Pfad |
| **Two-Pass HZB** | Verdeckte Chunks vor Rasterisierung unterdrücken | **V1 NO-GO BEIBEHALTEN**: Bismut-PT-12 dokumentiert fehlende Depth-SRV/2-Pass-Integration und Disocclusion-Popping | Erst vollständiger konservativer Depth-Ansatz mit Bildparität und Kamera-/Teleport-Negativtests |
| **64-Bit-`uint2`-Vertices** | Weniger Vertex-Bytes und Upload-Bandbreite | **V1 NO-GO BEIBEHALTEN**: Notebook-Schema mit 5 Bit pro Achse kann die reale VS-Geometrie (Sub-Block-Details, Überhänge, vollständige Flags) nicht verlustfrei abbilden; BN-05/PT-10 hat schon 16-Byte-Alternative verworfen | Feldgenauer Verlustfreiheitsnachweis und Shader-/Materialparität vor erneutem Formatvorschlag |
| **`D3D12_HEAP_TYPE_GPU_UPLOAD` + ReBAR** | CPU schreibt direkt in GPU-sichtbaren Upload-Heap und spart Copy-Stufen | **GEEIGNET FÜR ISOLIERTEN A/B-VERSUCH**: Vorteil ist hardware-/Zugriffsmuster-abhängig, keine garantierte Stutter-Eliminierung | `D3D12_OPTIONS16.GPUUploadHeapSupported`, ReBAR-Probe, Upload-/PCIe-/p99-Messung, Fallback |
| **Near/Far Hybrid: Mesh-Shader + SVDAG/LOD** | Nahbereich vollständig, Fernbereich komprimiert/hierarchisch rendern | **V2-KANDIDAT**: passt zum vorhandenen Ghost→High-Detail-Handover-Prototyp; sehr hohe Integrations- und Update-Komplexität | Resident-Ready-Handover, keine Löcher/Z-Fighting, VRAM-/Sichtweiten-A/B |

## Welche Einsichten wir bewahren

1. **Messen vor Migrating:** Indirect-Argument-Puffergröße, GPU-Queue-Stalls, Overdraw, VRAM-Residency und Chunk-Upload-Latenz müssen als getrennte Kostenstellen betrachtet werden.
2. **Occlusion ist ein Qualitätsproblem:** Frustum-Culling entfernt Offscreen-Chunks; konservative Verdeckung benötigt gültige Tiefe und Disocclusion-Schutz. „Weniger Draws“ ist kein PASS, wenn Gelände sichtbar aufpoppt.
3. **Vertex-Layout folgt Vintage Story, nicht Minecraft-Demos:** Exakte Position, Formen, Chisel-/Overhang-Daten, UVs, Licht-/Materialinformationen und Flags sind vertragliche Daten; keine pauschale 32³-Rasterannahme.
4. **Upload-Latenz ist ein End-to-End-Pfad:** CPU-Meshing → Staging/Host-Visible Memory → DX12 Submission → Fence → Draw-Ready messen. „Upload submitted“ ist nicht `GPU_READY`.
5. **Extreme Sichtweite verlangt Datenstufen:** Authoritative Full Chunk, persistenter Cache und visuelles Far-LOD trennen; fehlende Daten nicht durch unmarkierte, unzuverlässige Ersatzgeometrie verstecken.

## Explizite Korrekturen / Quellenrisiken

- **Komet-Namenskollision:** Die Ausarbeitung verwechselt den für uns relevanten Vintage-Story-Performance-Mod `xToast-dev/komet` mit einem Rust/GPUI-Agentenprojekt gleichen Namens. Keine Architekturfolgerungen aus diesem Quellenmix übernehmen. Relevantes VS-Repo: https://github.com/xToast-dev/komet
- **9.400 MB `ExecuteIndirect`-Steuerpuffer:** Im Notebook zitierte Fremdzahlen nicht auf Bismut übertragen. Ein nackter `D3D12_DRAW_INDEXED_ARGUMENTS`-Eintrag ist 20 Byte; tatsächliche Descriptor-/Root-Argumente und Allocator-Kosten separat erfassen.
- **Mesh Shader und Work Graphs sind nicht dieselbe Fähigkeit:** `CheckFeatureSupport`, Agility-SDK-Version, Treiber und Architektur unterscheiden. Speziell *Graphics/Mesh Nodes* nicht als bereits allgemein verfügbaren Ersatz voraussetzen.
- **„64-Bit = verlustfrei“ und „77 % weniger“:** Aus dem vorgeschlagenen Notebook-Bitlayout folgt keine tatsächliche VS-Parität. Die Prozentangabe setzt eine bestimmte Vergleichsbasis voraus und ist kein Bismut-Messergebnis.
- **ReBAR / `GPU_UPLOAD` ist nicht kostenlos:** Nicht jedes System unterstützt den Heap; direkte CPU-Schreibzugriffe können wegen PCIe- und Write-Combining-Eigenschaften schlechter sein. Kein obligatorischer Standardpfad ohne A/B.
- **„Rust eliminiert GC/Locks vollständig“:** Falsch als Systemaussage: Die Vintage-Story-/C#-Seite bleibt managed; Rust-Worker beseitigen nicht automatisch sämtliche Locks, GC-Pausen oder Queue-Stalls.
- **„Work Graphs beseitigen alle Barrieren / FPS unabhängig von Sichtweite“:** Spekulative Architekturbehauptungen statt verifizierter Bismut-Ergebnisse.
- **Referenzen der Notebook-Ausarbeitung:** Mehrere Sekundär-/Community-/Aggregatorquellen; Leistungsversprechen und technische Details vor Umsetzung gegen Primärquellen und unseren Quellcode verifizieren.

## Priorisierung ohne Scope-Drift

| Zeitpunkt | Arbeit | Status |
|---|---|---|
| **Jetzt** | PT-26: Workpack V1 Integration-/Evidence-Abweichungen beheben und Tracker bereinigen | **LAUFEND** |
| **Danach** | Unabhängige Hoplite-Gegenprüfung mit realem Produktpfad und raw logs | **REVIEW_PENDING** |
| **Später R&D** | Hardware-Capability-Probes für GPU_UPLOAD / Work Graphs; Mikrobenchmarks nur als getrennte Probe | **RESEARCH** |
| **Bismut V2** | LOD-Residency/Handover, GPU-Arena, konservatives Occlusion-Konzept, mögliche alternative Geometrie-Pipeline | **FUTURE** |

**Invariant:** Der produktive V1-Pfad bleibt vorerst beim **verlustfreien 28-Byte-Vertexformat**, beim **GPU-Compute-Frustum-Culling** und dem bestehenden **`ExecuteIndirect`-Renderer**, bis belastbare Gegenbelege und explizite Freigaben vorliegen.

## Evidence-Auftrag für spätere Forschung (kein aktueller PT)

- **Vorab:** GOAL / Acceptance Criteria / Architektur- und Feature-Capability-Check.
- **A/B:** deterministische Szene, identische Sichtweite, VRAM-Budget, Warmup, Qualitätsvergleich.
- **Messwerte:** GPU-Zeit p50/p95/p99, Upload-Bytes/-Zeit, Queue-Waits, Live-Resident-Chunks, Sichtbarkeits- und Geometrie-Parität.
- **Negativtests:** GPU ohne Feature-Support; Kameraschwenk/Teleport; Device-Loss; verspäteter Upload; Geometrie mit Überhängen/Chisel/Flags; Cache-/Epoch-Mismatch.
- **Status:** `PASS` erst bei produktiver Erreichbarkeit, source-grounded Evidenz und allen erforderlichen Reviews. Isolierte Harnesses = Probe, nicht Produkt-/Gameplay-PASS.

---

**Materialisierungszweck:** Dies ist eine knappe, belastbar eingeordnete Essenz der Notebook-Ausarbeitung. **Kein neuer Implementierungsauftrag, keine Freigabe zum Überschreiben der NO-GOs und keine Änderung an PT-26 oder PT-25.**
