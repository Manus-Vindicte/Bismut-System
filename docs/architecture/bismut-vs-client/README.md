# Bismut VS Client · Architektur- und Q&A-Dokumente

**Aktueller Git-Stand (2026-11-19):** [Roadmap V2.2](BISMUT_VS_CLIENT_ROADMAP_V2_2.md)

## Entscheidungen und laufende Evaluierungen

- **Q1–Q8:** Architekturentscheidungen RATIFIED (zusammengefasst in Roadmap V2.2).
- **Q9:** vier Bootstrap-Lanes zur Evaluation freigegeben, Architektur OPEN.
- **Q10:** vier Threading-Lanes zur Evaluation freigegeben, Architektur OPEN.
- **Q11:** vier Memory-Ownership-Lanes zur Evaluation freigegeben (Rust Arenas D als Hypothese), Architektur OPEN.
- **Q12:** [Rust Input Router, DECIDED](ADR_BISMUT_Q12_RUST_INPUT_ROUTER_2026-11-18.md).
- **Q13:** [Shader-Pipeline A–D Evaluation, APPROVED / OPEN](ADR_BISMUT_Q13_SHADER_PIPELINE_EVALUATION_2026-11-18.md).
- **Q14:** [GPU Resource Management A–D Evaluation, APPROVED / OPEN](ADR_BISMUT_Q14_GPU_RESOURCE_MANAGEMENT_EVALUATION_2026-11-18.md).
- **Q15:** [Dual .NET-/Rust-SDK + Bismut Loader, DECIDED](ADR_BISMUT_Q15_DUAL_SDK_BISMUT_LOADER_2026-11-18.md). Der VS-Mod-Loader bleibt für bestehende VS-Mods verantwortlich.

- **Q16:** [Rust Recovery Supervisor + externer Watchdog, B+D DECIDED](ADR_BISMUT_Q16_RUST_RECOVERY_SUPERVISOR_EXTERNAL_WATCHDOG_2026-11-18.md).
- **Q17:** [Stage Frame-Time Budgets + Benchmark Worlds, B+Suite DECIDED](ADR_BISMUT_Q17_FRAME_BUDGETS_BENCHMARK_WORLDS_2026-11-19.md). [Benchmark Suite V1 Spec](BISMUT_VS_CLIENT_BENCHMARK_SUITE_V1_SPEC_2026-11-19.md). Konkrete Performance-PASS-Schwellen bleiben NEEDS_BASELINE.
- **Q18:** [D+ Unified Launcher/Client Shell, Backend-Auswahl, Benchmark und NVIDIA Assist, DECIDED](ADR_BISMUT_Q18_UNIFIED_LAUNCHER_CLIENT_BACKEND_BENCH_NVIDIA_2026-11-19.md). Vorhandene Launcher-Dienste integriert statt doppelt implementiert.
- **Q18A:** [Editionen: Bismut VS Client - Standard / BismutX VS Client - Extended, DECIDED](ADR_BISMUT_Q18A_PRODUCT_EDITIONS_INSTALLER_2026-11-19.md). Standard ohne Engine als Default, Extended optionale versionierte Komponenten; gleicher Launcher-/Client-Kern.
- **Q19:** [Graphics Presets, Benchmark Auto-Tuner und NVIDIA Assist, E DECIDED](ADR_BISMUT_Q19_GRAPHICS_PRESETS_AUTOTUNER_NVIDIA_ASSIST_2026-11-19.md); numeric Performance-Budgets weiterhin NEEDS_BASELINE.
- **Q20:** Mod Hub, Modpacks, Lockfiles und BCIL-Kompatibilitätsprüfung, OPEN.

## Reconciliation-Grenze

Dieser Git-Pfad enthält versionierte **Entscheidungen und einen kompakten Umsetzungsstand**. Er ist nicht automatisch ein Ersatz für bestehende lokale Engine-/Launcher-/Mod-Handoff-Docs und Runtime-Evidence. Vor Übernahme in den Windows-Checkout die tatsächlichen HEADs, Dirty States, Kanon-Indizes und qualifizierten Gates prüfen. Keine Tests werden durch das Anlegen dieser Dokumente zu PASS.
