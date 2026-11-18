# Bismut VS Client · Architektur- und Q&A-Dokumente

**Aktueller Git-Stand (2026-11-18):** [Roadmap V1.8](BISMUT_VS_CLIENT_ROADMAP_V1_8.md)

## Entscheidungen und laufende Evaluierungen

- **Q1–Q8:** Architekturentscheidungen RATIFIED (zusammengefasst in Roadmap V1.8).
- **Q9:** vier Bootstrap-Lanes zur Evaluation freigegeben, Architektur OPEN.
- **Q10:** vier Threading-Lanes zur Evaluation freigegeben, Architektur OPEN.
- **Q11:** vier Memory-Ownership-Lanes zur Evaluation freigegeben (Rust Arenas D als Hypothese), Architektur OPEN.
- **Q12:** [Rust Input Router, DECIDED](ADR_BISMUT_Q12_RUST_INPUT_ROUTER_2026-11-18.md).
- **Q13:** [Shader-Pipeline A–D Evaluation, APPROVED / OPEN](ADR_BISMUT_Q13_SHADER_PIPELINE_EVALUATION_2026-11-18.md).
- **Q14:** [GPU Resource Management A–D Evaluation, APPROVED / OPEN](ADR_BISMUT_Q14_GPU_RESOURCE_MANAGEMENT_EVALUATION_2026-11-18.md).
- **Q15:** [Dual .NET-/Rust-SDK + Bismut Loader, DECIDED](ADR_BISMUT_Q15_DUAL_SDK_BISMUT_LOADER_2026-11-18.md). Der VS-Mod-Loader bleibt für bestehende VS-Mods verantwortlich.

- **Q16:** [Rust Recovery Supervisor + externer Watchdog, B+D DECIDED](ADR_BISMUT_Q16_RUST_RECOVERY_SUPERVISOR_EXTERNAL_WATCHDOG_2026-11-18.md).
- **Q17:** Performance-Budgets und Telemetrie, OPEN.

## Reconciliation-Grenze

Dieser Git-Pfad enthält versionierte **Entscheidungen und einen kompakten Umsetzungsstand**. Er ist nicht automatisch ein Ersatz für bestehende lokale Engine-/Launcher-/Mod-Handoff-Docs und Runtime-Evidence. Vor Übernahme in den Windows-Checkout die tatsächlichen HEADs, Dirty States, Kanon-Indizes und qualifizierten Gates prüfen. Keine Tests werden durch das Anlegen dieser Dokumente zu PASS.
