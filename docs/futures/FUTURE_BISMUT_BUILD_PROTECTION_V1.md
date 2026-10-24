# FUTURE-BISMUT-BUILD-PROTECTION-V1

Status: FUTURE
Decision: DEFER
Priority: P3 — after native DX12 renderer stabilization
Scope: Bismut-owned release binaries and build pipeline
Owner: Bismut System / release engineering
Created: 2026-10-24
TASK_CLASS: future_request
DELEGATION: ERWÜNSCHT (only when explicitly activated)
TOOLS: EXPLICIT_USE (only when explicitly activated)
EXECUTION: NOT_AUTHORIZED

## ABSICHT — WAS

Preserve a future initiative to make casual decompilation and unauthorized source reconstruction of Bismut-owned release binaries harder, **without destabilizing** the Bismut Engine, Launcher, Graphics Mod, or their public integration contracts.

This is a deferred architectural candidate, **not an implementation ticket**. Do not install, change release pipelines, ship an obfuscator, or add runtime countermeasures as a result of this document.

## WARUM

Bismut combines .NET/C# components, Harmony patch points, native DirectX 12 interop, mod-facing APIs and performance-critical render loops. Once the native DX12 pipeline is mature, we may want to raise the effort required to trivially reconstruct proprietary internals while retaining reliable crash diagnosis, compatibility, user trust and performance. Executed local software cannot be made impossible to reverse engineer; protection is a cost/effort measure, not a security boundary.

## KANDIDATEN — WIE (to evaluate later)

1. **Selective managed-code obfuscation:** Rename private types/methods and apply bounded control-flow transformations only to code that does not depend on reflection, serialization names, external ABI or deterministic performance behavior.
2. **Build-specific diversification:** Investigate reproducibly tracked release variations from pinned seeds. Require a private build manifest and retained symbol/mapping files for support and incident analysis.
3. **Native AOT feasibility:** Explore only for isolated compatible libraries. Do not assume Harmony, reflection, dynamic loading, mod hooks or current P/Invoke entry points will survive unchanged.
4. **Release provenance:** Keep signing, artifact hashing, SBOM/dependency provenance and private crash-symbol mapping separate from the obfuscation stage.
5. **Threat-model review:** Distinguish IP friction from actual tamper resistance and application security; assess user/debugger compatibility, antivirus false positives, accessibility, observability and maintenance costs.

**Explicit exclusions:** No randomized breakpoints, anti-debug crashes, deceptive runtime behavior, secret extraction, malware-like packing, kernel drivers, forced online activation, obfuscation of public contracts or random control-flow tricks inside the DX12 hot path. No claim of preventing decompilation.

## TECHNICAL BOUNDARIES

- Preserve public modding APIs, serialization compatibility, Harmony target signatures, reflection-discovered members and managed/native export names.
- Exclude real-time GPU command submission, synchronization, frame identity, native ABI layouts and shader/resource contract surfaces unless independent evidence establishes safety.
- Keep developer/diagnostic builds unobfuscated and retain private release symbol maps for reproducible debugging.
- Treat signing and access control as distinct concerns: obfuscation is not authentication, authorization or a security fix.
- Verify that license terms and legitimate user/modder interoperability needs are respected.

## REQUIRED PRE-WORK (after future activation)

1. Inventory release assembly boundaries, exports, reflection/Harmony surfaces, crash-reporting paths and available build tools.
2. Establish an explicit threat model and choose a narrowly scoped proof of concept on a **non-critical** library.
3. Run compatibility, reproducibility, cold-start, CPU/GPU frame-time and memory regressions; compare unobfuscated and protected builds.
4. Preserve artifact hashes, pinned tool versions, transformation configuration, deterministic seed provenance and private recovery mapping.
5. Have an independent reviewer inspect release safety and the ability to debug a protected crash.
6. Record tool usage and justified non-usage; delegate independent discovery/review if available.

## FUTURE ACCEPTANCE (not satisfied today)

- Protected release behaves identically on supported mod/API and serialization compatibility tests.
- Render-path performance and stability stay within predeclared budgets; no new frame pacing or GPU-lifecycle regressions.
- Debug symbols/mappings enable diagnosis of crashes in shipped artifacts.
- Supply-chain/signing checks pass, obfuscation tools are legally usable and do not trigger unacceptable AV/EDR false positives.
- Rollback to an unprotected signed release is possible.
- An explicit owner approval promotes this FUTURE to PLANNED; only then create an executable PT.

## DEPENDENCIES / STOP CONDITIONS

**Do not activate before:** native DirectX 12 vertical slice and renderer ownership are stable; release packaging, CI gates and versioned API contracts are qualified.

**Stop:** evidence of game/mod interoperability breakage, debugger crashes, nondeterministic rendering, unbounded build diversification, performance regressions or loss of crash diagnosability.

## LINKS / TRACEABILITY

- System future lifecycle: [docs/process/RESEARCH_LIFECYCLE.md](../process/RESEARCH_LIFECYCLE.md)
- Future implementation should reference the owning Engine/Launcher/Graphics release pipeline and its then-current SDK/toolchain manifest.
- This request is strategic and **does not alter** the current DX12 G2.3, N3 or G3 gates.
