# ADR BISMUT Q14 | GPU Resource Management A/B/C/D Evaluation

- Datum: 2026-11-18
- Owner-Beschluss: **Q14 Option E**
- Status: **EVALUATION_APPROVED / ARCHITECTURE_OPEN / ALL_LANES_NOT_ATTEMPTED**
- Produkt: Bismut VS Client (Rust-EXE, VS-.NET-Game-Runtime in-process, langfristig Rust-natives D3D12)

## Entscheidung

Q14-A bis Q14-D werden unter reproduzierbaren Bedingungen als GPU-Ressourcenstrategien evaluiert. **Keine dieser Strategien ist als Produktimplementierung ratifiziert.** Eine belegte Kombination ist zulässig, braucht aber einen separaten Owner-Beschluss. Q11 behandelt CPU-/Frame-Speicher und GC-/FFI-Lifetime, Q14 GPU-Heaps, VRAM/Residency, Deskriptoren, Transfers und GPU-Fence-Retirement.

Q6-C bleibt die Zielarchitektur: ein Rust-nativer DX12-Owner für Device, Command Queues, GPU-Ressourcen und Present. Der bestehende C++-DX12-Core bleibt Referenz und Recovery während der Migration. Q2-Resource-Command-Queue und Generation-Handles sind bindende Schnittstellengrenzen.

## Vergleichslanes

| Lane | Umfang | Qualifikationspunkt |
|---|---|---|
| **A** | D3D12 Committed Resources pro Mesh/Texture/Buffer; minimaler direkter Pfad | Korrekte Lebensdauer, Kosten von Allokation, Upload und Freigabe als Referenz |
| **B** | Eigener Rust GPU Resource Manager: Heap-Pools, Suballocation, Upload-/Readback-Ring, Descriptor-Generations, Fence-Retirement | Allokator-Korrektheit, Wiederverwendung, Fragmentierung, FFI-freier GPU-Hotpath |
| **C** | D3D12 Memory Allocator oder vergleichbare nach Lizenz/Version geprüfte native Allocation-Komponente über explizite Rust-FFI; Ressourcen-Lifetime bleibt Rust-owned | Suballocation und Stabilität unter FFI-/Allocator-Versionen, Error- und Release-Verhalten |
| **D** | VRAM-Budget-/Residency-first Streaming, kontrollierte Eviction und gegebenenfalls Defragmentierung für Chunks/Textures | Sichtbarkeits-/Streaming-Korrektheit bei Last, Stalls, Rückgewinnung und Wiederaufnahme |

**Testachsen trennen:** A/B/C primär als Allokations-/Lifetime-Varianten vergleichen; D ist wesentlich eine dazu kombinierbare Streaming-/Residency-Policy. D mit einer definierten Allocator-Baseline testen. Ein einfaches A-vs-D-FPS-Ranking wäre methodisch irreführend. Keine doppelte D3D12-Device-/Present-Ownership.

## Gleiche Testbedingungen

1. **Gepinnte Inputs:** identische Mesh-/Texturgrößen, Chunk-Muster, Alpha/Shadow/UI/Avatar-Last, Resource-Command-/Frame-Snapshot-Corpus; Hardware-/Treiber-/SDK-/Rust-/Windows-/Commit-Hashes pro Run und Assets-Hashes protokollieren.
2. **Messwerte:** alloc/free p50/p95/p99, Upload-/Readback-Durchsatz und p95/p99-Latenz, CPU-/GPU-Frame-Time p50/p95/p99, VRAM allocated/committed/resident/budget, Peak Usage, Fragmentierung, Anzahl/Größe von Heaps, Descriptor-Usage, Fence-Waits, Evictions und Stalls. Mindestens drei reproduzierbare Durchläufe pro Lastprofil.
3. **Lastprofile:** Cold Start, ruhige Szene, dichtes Chunk-Streaming, Teleport/Chunk-Thrash, Texturwechsel, Avatar/Entities, GUI, lange Session, GPU-Speicherdruck. Budget-/OOM-Behandlung auch auf kleinerer VRAM-Klasse prüfen; keine pauschalen FPS-Gewinne behaupten.
4. **Lifetimes:** Create/Update/Release mit Handle-Generation, keine Freigabe vor GPU-Fence-Abschluss, keine stale Descriptor- oder Resource-Referenzen, begrenzte Ringbuffer-Backpressure, deterministischer Shutdown.
5. **Negativtests:** künstlich kleines Budget, Heap-/Descriptor-Exhaustion, doppelte Release, Stale Handle, Out-of-order Resource-Commands, Resize, Minimize, Device Removed/Lost, Failure bei Upload/Fence, unterbrochenes Streaming, korrupte Cache-/Resource-Metadaten.

## Evidenz und Gate

- Pro Lane: Testplan, Build-/Binary-/Dependency-Manifest, metrische JSON-/CSV-Receipts, Logs, GPU-Captures wo verfügbar, Failure-/Recovery-Klassifikation, Kosten-/Komplexitätsvergleich, Lizenz-/Provenienz-Audit für externe Allocators.
- Statusstufen: NOT_ATTEMPTED, STATIC_REVIEWED, SYNTHETIC_LAB_QUALIFIED, NATIVE_DX12_QUALIFIED, VS_LIVE_QUALIFIED, MOD_COMPAT_QUALIFIED oder BLOCKED/FAIL/PARTIAL. Nur tatsächlich gemessene Stufen behaupten.
- **Auswahl-Gate:** Safety und deterministisches Fence-/Lifetime-Verhalten zuerst; danach VRAM-/Frame-Time-/Streaming-Metriken, Implementierungs- und Wartungsaufwand. Entscheidung durch Owner nach vergleichender Evidence-Matrix.
- **Kein Gate-Ersatz:** Headless Atlas validiert VS-/Server-Mod-Funktionen, nicht GPU-Lifetime; Q8-C OpenGL Zero wird separat mit realem Client, processweiten Traces und Negativtests geprüft.
- **Umfang:** Experimente nicht zum Blocker des bestehenden DX12-Avatar-Vertical-Slice machen; A als instrumentierte Baseline beginnen, B/C isoliert vergleichen, D mit definiertem Allokator für Streaming belasten.

## Nicht tun

Keine voreilige Wahl B/C/D, keine produktiven Spielstände oder Profile ändern, keine fremden Prozesse anhalten, keine Auth-/Cache-Secrets committen, keine ungeprüften Allokator-Binärteile übernehmen. Bestehenden C++-DX12-Recovery-Pfad nicht entfernen. Q9, Q10, Q11, Q13 bleiben Evaluierungen und sind nicht durch diesen ADR qualifiziert.

## Referenzen

- [Bismut VS Client Roadmap V1.6](BISMUT_VS_CLIENT_ROADMAP_V1_6.md)
- [Q13 Shader Pipeline Evaluation](ADR_BISMUT_Q13_SHADER_PIPELINE_EVALUATION_2026-11-18.md)
- [Q12 Rust Input Router](ADR_BISMUT_Q12_RUST_INPUT_ROUTER_2026-11-18.md)
