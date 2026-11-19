# Bismut VS Client · Architektur- und Q&A-Dokumente

**Aktueller Git-Stand (2026-11-19):** [Roadmap V2.5](BISMUT_VS_CLIENT_ROADMAP_V2_5.md)

## Entscheidungen und laufende Evaluierungen

- **Q1–Q8:** Architekturentscheidungen RATIFIED (zusammengefasst in Roadmap V2.5).
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
- **Q18A:** [Öffentliche Editionen: Bismut Client / Bismut Client Extended, DECIDED](ADR_BISMUT_Q18A_PRODUCT_EDITIONS_INSTALLER_2026-11-19.md). **Titelleiste immer Bismut Client**; intern bleiben Bismut VS Client - Standard / BismutX VS Client - Extended die technischen Kennungen. Standard ohne Engine als Default, Extended optionale versionierte Komponenten, gemeinsamer Launcher-/Client-Kern.
- **Q19:** [Graphics Presets, Benchmark Auto-Tuner und NVIDIA Assist, E DECIDED](ADR_BISMUT_Q19_GRAPHICS_PRESETS_AUTOTUNER_NVIDIA_ASSIST_2026-11-19.md); numeric Performance-Budgets weiterhin NEEDS_BASELINE.
- **Q20:** [E: Unified Mod Hub, reproduzierbare Modpack-Lockfiles und BCIL, DECIDED](ADR_BISMUT_Q20_UNIFIED_MOD_HUB_LOCKFILES_BCIL_EDITIONS_2026-11-19.md). **Bismut Client Standard:** ModDB/Modpacks/Profiles/Resolver/Lockfiles/Rollback; **Extended:** Bismut Loader, DX12-Mod-Adapter und BCIL-/REA-Kompatibilitätsanalyse.
- **Q21:** [Option A: lokales Crash-Panel / Loganalyse / Lösung zuerst, optional „Fehlerbericht senden“ über Vercel, DECIDED](ADR_BISMUT_Q21_SIMPLE_SUPPORT_VERCEL_GATE_2026-11-19.md). Cloud-Diagnose nur freiwillig und ausschließlich über Vercel; interne Dev-/LLM-Auswertung bleibt hinter der einfachen Nutzeroberfläche.
- **Q22:** [E: signierte Komponenten, Delta-Updates, Portable-/Offline-Pakete, DECIDED](ADR_BISMUT_Q22_DISTRIBUTION_SIGNED_COMPONENTS_DELTA_OFFLINE_2026-11-19.md). Ein Installer/Updater und Recovery-Gate für Standard/Extended, keine Engine-Pflicht im Standard.
- **Q23:** Release-Channels, Beta-Tests und Update-Policy, OPEN.

## Reconciliation-Grenze

Dieser Git-Pfad enthält versionierte **Entscheidungen und einen kompakten Umsetzungsstand**. Er ist nicht automatisch ein Ersatz für bestehende lokale Engine-/Launcher-/Mod-Handoff-Docs und Runtime-Evidence. Vor Übernahme in den Windows-Checkout die tatsächlichen HEADs, Dirty States, Kanon-Indizes und qualifizierten Gates prüfen. Keine Tests werden durch das Anlegen dieser Dokumente zu PASS.
