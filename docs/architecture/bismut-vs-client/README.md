# Bismut VS Client · Architektur- und Q&A-Dokumente

**Aktueller Git-Stand (2026-11-19):** [Roadmap V2.8](BISMUT_VS_CLIENT_ROADMAP_V2_8.md)

## Entscheidungen und laufende Evaluierungen

- **Q1–Q8:** Architekturentscheidungen RATIFIED (zusammengefasst in Roadmap V2.8).
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
- **Q21:** [Option A: lokales Crash-Panel / Loganalyse / Lösung zuerst, optional „Fehlerbericht senden“ über Vercel, DECIDED](ADR_BISMUT_Q21_SIMPLE_SUPPORT_VERCEL_GATE_2026-11-19.md). Freiwillige Cloud-Diagnose ausschließlich über Vercel. Intern bevorzugt Gemini API Free Tier mit geprüfter Free-Alternative und regelbasiertem Fallback; keine ungeprüften Kosten oder ungeklärten Free-Tier-Datenschutzrisiken.
- **Q22:** [E: signierte Komponenten, Delta-Updates, Portable-/Offline-Pakete, DECIDED](ADR_BISMUT_Q22_DISTRIBUTION_SIGNED_COMPONENTS_DELTA_OFFLINE_2026-11-19.md). Ein Installer/Updater und Recovery-Gate für Standard/Extended, keine Engine-Pflicht im Standard.
- **Q23:** [Option A: genau ein öffentlicher Stable-Kanal, manuell bestätigte Updates, DECIDED](ADR_BISMUT_Q23_STABLE_ONLY_MANUAL_UPDATES_2026-11-19.md). Keine öffentliche Beta-/Nightly-Auswahl oder automatische Installationen; interne isolierte Lab-Builds bleiben zulässig.
- **Q24:** [Option E: minimaler First-Run, Read-only Discovery, freiwilliger Import und sichere isolierte Migration, DECIDED](ADR_BISMUT_Q24_FIRST_RUN_DISCOVERY_SAFE_IMPORT_2026-11-19.md). Originalsaves, Mods, Instanzen und Auth-Daten nicht ungefragt verändern; Standard ohne Engine oder Cloud.
- **Q25:** [Option E: bestehender Launcher bleibt Start-/Server-/World-UI, DECIDED](ADR_BISMUT_Q25_EXISTING_LAUNCHER_CLIENT_SESSION_2026-11-19.md). Preflight/LaunchPlan/RunRecord/Session-Lifecycle und Return-to-Launcher ergänzen, kein zweiter World-/Server-Browser.
- **MASTER GOAL:** [Team-Building HARD GATE, genau zwei Slices Engine + Client](MASTER_GOAL_BISMUT_ENGINE_CLIENT_TWO_SLICES_2026-11-19.md). Owner-reported DX12 Avatar Runs 16/17/19 sind lokale Ausgangsbasis, nicht neu verifiziert. Bestehende Dynamic-Mesh-Bridge verwenden, passMask/visibility + FrameId erst nach eigenem ADR und Owner Approval; Defekt E/AV-06/07 bleiben manuell OPEN. Status READY_FOR_LOCAL_AGENT_HANDOFF, keine lokale Code-Umsetzung.

## Laufende Impact-Studien

- **VS 1.23.0-pre.1:** [PT-VS-1.23-PRE1-ENGINE-CLIENT-IMPACT-STUDY, PRELIMINARY / SOURCE_PENDING](PT_VS_1_23_0_PRE1_ENGINE_CLIENT_IMPACT_STUDY_2026-11-19.md). Zwei getrennte Engine-/Client-Study-Threads, Pre-Release nur isoliert im Lab, keine Stable-/World-/Mod-Migration. Die exakte offizielle Pre.1-Release-Seite war bei Erstellung nicht abrufbar; sie ist vor abschließender Quellenauswertung zwingend zu lesen.

## Reconciliation-Grenze

Dieser Git-Pfad enthält versionierte **Entscheidungen und einen kompakten Umsetzungsstand**. Er ist nicht automatisch ein Ersatz für bestehende lokale Engine-/Launcher-/Mod-Handoff-Docs und Runtime-Evidence. Vor Übernahme in den Windows-Checkout die tatsächlichen HEADs, Dirty States, Kanon-Indizes und qualifizierten Gates prüfen. Keine Tests werden durch das Anlegen dieser Dokumente zu PASS.
