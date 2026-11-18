# ADR BISMUT Q18 | Unified Launcher + Client, Backend Picker, Benchmark, NVIDIA Assist

- Datum: 2026-11-19
- Ownerentscheidung: **Q18 D, erweitert: „Launcher ist auch der Client“**
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Produkt: Bismut VS Client / Bismut Launcher als gemeinsame Benutzeroberfläche
- Keine Runtime-, GPU- oder UI-Testausführung mit diesem Dokument

## Entscheidung

Bismut bietet **eine zusammenhängende Launcher-/Client-Shell statt zwei eigenständiger Frontends mit gespiegelten Zuständigkeiten**. Die Benutzeroberfläche umfasst vor dem Spielstart Bibliothek, Worlds, Modpacks/Profiles, Backend-Auswahl, Grafik-/NVIDIA-Settings, **Q17 Benchmark Worlds**, Installation/Update und Diagnose. Beim Start des Bismut-DX12-Modus geht **dieselbe native Shell** in den Game-Modus über und stellt Game View, die über Q4 geroutete Vintage-Story-.NET-GUI sowie das kontextuelle In-Game-Control-Center bereit.

**Keine Doppel-Ownership:** Es gibt einen nutzerseitigen Launcher/Client; Download-/Publishing-/Installations-Dienste bleiben getrennte fachliche Module. Der vorhandene Bismut Launcher (separates existierendes Produkt-/UI-Repository) wird funktional über Adapter und schrittweise Migration integriert, nicht blind gelöscht oder mit einem zweiten Paketmanager gespiegelt. Q5 (native Rust-EXE + Top-Level-HWND + in-process Vintage-Story-.NET-Runtime) gilt **für den Bismut-Backend-Pfad**. Ob und wie vorhandene Launcher-Electron-/Web-UI-Fragmente migriert werden, muss am echten Repo und UI-Vertrag geprüft werden.

## Gemeinsame Shell, Navigation und Ownership

| UI-Bereich | Funktion | Owner / technische Grenze |
|---|---|---|
| **Home / Spielen** | Versions-/Profilwahl, Weltliste, Last Played, Play/Resume, Bismut/Vanilla Modus | Unified Shell + vorhandene Launcher-Profil-/Publishing-Dienste |
| **Grafik & Backend** | explizite Renderer-Wahl, Kompatibilitäts-/Readiness-Status, Presets, Auflösung, FPS/VSync/Frame Pacing | Shell schreibt gewünschte Konfiguration; tatsächliche Feature-Unterstützung vom Backend belegen |
| **Benchmark** | Q17 festgelegte Baseline/Chunk/Entity/Graphics/Mod-World-Profile, Lauf und lokaler Reportvergleich | Q17 Benchmark Harness; nur World-/Render-Ready-gültige Ergebnisse |
| **NVIDIA Assist** | GPU-/Treibererkennung, Hardware-/Feature-Matrix, optionale VS-bezogene Performance-/Qualitätsprofile | NVIDIA-Funktionen opt-in, nie als zwingende Treiberänderung |
| **Mods/Modpacks & Loader** | Installieren, Auflösen, Aktualisieren, Quarantäne, kompatibles Startprofil | Launcher/BCIL für Pakete; Q15 Rust Loader nur für Runtime-Aktivierung |
| **In-Game Control Center** | Gameplay-unabhängige Grafik-, Performance-, Benchmark-, Diagnose- und Mod-Status-Oberfläche | Q4 DX12 GUI-Compositor + Q12 Rust Input Router; Vanilla-VS-GUIs bleiben semantisch im VS-Adapter |
| **Recovery/Logs** | Crash-/Hang-Reports, Quarantäne für Folgestart, Safe Launch/Vanilla, Exit/Restart | Q16 Rust Supervisor + externer Watchdog + Launcher-Fallback |

**Ein primärer UI-Owner je Funktion.** Kein paralleler Launcher- und Game-Settings-Speicher mit auseinanderlaufenden Werten. Eine versionierte Settings-Quelle mit Profil-Overrides und Provenienz „desired vs effective vs supported“. Keine zwei konkurrierenden Install-/Auth-/Publishing-Owner.

## Backend-Auswahl: Strikter Vertrag

- **Bismut Native DX12**: Rust-EXE besitzt HWND, Device, RenderWorld, Swapchain und einen finalen Present-Owner; VS-.NET-Gameplay-Runtime in-process. Sichtbarer Status **SUPPORTED / EXPERIMENTAL / BLOCKED / UNKNOWN** abhängig von tatsächlichen Tests; aktuelle Architekturentscheidung ist keine Freigabe des Live-Backends.
- **Vanilla Vintage Story OpenGL**: eigener qualifizierter Vanilla-Launch-Pfad als Kompatibilitäts-/Recovery-Option, bei Bedarf eigener Spielprozess. **Nicht** als versteckter Renderer im Bismut-DX12-Prozess; Q8 OpenGL-Zero-Traces dürfen beide Modi nicht vermischen.
- **Legacy C++-DX12-Lab**: sichtbar nur als ausdrücklich benannter Engineering-/Recovery-Modus, nicht als Rust-Native-DX12-Produkt-PASS.
- **Switching**: nur vor Sessionstart bzw. über kontrolliertes Save-/Shutdown-/Relaunch-Gate. Nach Fehlstart wird der gewünschte Backend-Wert nicht stillschweigend auf „DX12 funktioniert“ gestellt; effective backend und Fallback-Reason im Audit.
- **Profiles**: Backend, VS-Adapter-Version, Spielversion, Mod-/Loader-Capabilities, Grafikpreset, Benchmark-Settings und NVIDIA Assist sind pro Profil versioniert, export-/importierbar und rollbackfähig, ohne produktive Saves/Auth umzuschreiben.
- **Performance-Vergleich**: gleiche Welt-/Mod-/Einstellungen und Render-Parität nötig; andere Feature-Unterstützung => NOT_COMPARABLE. Ein DX12-Benchmark auf fehlender GUI-/Entity-Pipeline ist kein valider Vergleich.

## NVIDIA-Optimierung für Vintage Story

Der Beschluss umfasst einen **optional aktivierbaren NVIDIA Assist-Bereich**. Er ist herstellerübergreifend sauber: nicht-NVIDIA-GPUs bekommen neutrale Settings und keine NVIDIA-spezifischen Fehlermeldungen als Featureverlust.

1. **Discovery & Presets V1:** GPU Vendor/Device, VRAM, Treiber-/SDK-Version und verfügbare Features ermitteln; Vorschläge zu VSync/Frame Cap, Presentation/Low-Latency-Strategie, Skalierung, Power-/Quality-Presets und spiel-/backendspezifischer Konfiguration. Nur tatsächlich unterstützte Parameter anwenden und den effektiven Zustand verifizieren.
2. **Vanilla-/Treiberprofil**: bei Vanilla-OpenGL differenzieren zwischen VS-eigenen Optionen und NVIDIA-App-/Treiberprofil-Optionen. Standardmäßig **Anleitung/Deep-Link** zu NVIDIA App mit per-Programm-Einstellungen statt undokumentierten globalen Treiber-Manipulationen. NVIDIA-Profile oder NVAPI Driver-Settings **nur mit explizitem Nutzer-Opt-in, Rückgängig-Möglichkeit und unterstützter API** ändern, falls überhaupt qualifiziert.
3. **Bismut DX12 Native**: DX12-spezifische Adapter/Timestamps, Shader-/PSO-Caches, GPU-/VRAM-Budgets und Present-/Frame-Pacing messen. NVIDIA Reflex/NIS/Streamline/DLSS als **optionale spätere Integrations-Spikes mit Device-/Treiber-/SDK-/Lizenz-/Render-Pass-/Motion-Vector-/Depth-Verträgen**, keine implizite Verfügbarkeit.
4. **Feature-Provenienz**: Jede Option erhält einen Status **AVAILABLE / UNAVAILABLE / NEEDS_INTEGRATION / UNVERIFIED**, GPU-/Treiber-Eignung, Modus sowie Rücksetzung. Keine pauschale „NVIDIA Optimierung = automatisch mehr FPS“-Behauptung; Benchmark muss Benefit gegen Stabilität/Qualität/Latency messen.
5. **Safety**: Kein Driver-Tweak auf globaler Ebene ohne explizite Freigabe, keine versteckten Registry-Edits, kein automatisches Overclock/Undervolt, keine ungeprüften NVIDIA-DLLs im Bundle. Diagnose lokal, ohne Auth-Daten im Report.

Relevante offizielle Quellen (nur als technische Grundlage für einen späteren Audit, nicht als Nachweis der Integration):
- https://www.nvidia.com/en-us/geforce/news/gfecnt/202411/nvidia-app-download-and-features/
- https://developer.nvidia.com/nvapi/get-started
- https://developer.nvidia.com/rtx/streamline/get-started
- https://developer.nvidia.com/performance-rendering-tools/reflex

## Stufenmodell D+

| Stufe | Umsetzung | Exit Gate |
|---|---|---|
| **F0 / Minimum** | Unified Window/Shell mit Home, Game Launch, eindeutiger Backend-Picker und Vanilla-Fallback; Status/Version/Profil eindeutig | Ein Startpfad pro Backend und korrektes desired/effective Backend-Receipt; DX12 kann BLOCKED bleiben |
| **F1 / Native Control Center** | Gemeinsame Settings, In-Game-Overlay, Mods/Loader-Status, Q17 Benchmark-Launcher/-Reports und Diagnose | UI-Lifecycle/Focus, Save-/Restart-Sicherheit, Benchmark World-Ready/Frame-ID-Evidence |
| **F2 / NVIDIA Assist** | NVIDIA-Detection, Empfehlungen und per-Backend-Presets; optional qualifizierte NVAPI-/Reflex-/DLSS-Spikes | Feature-Capability-Matrix, Opt-in, Rollback, Benchmark-A/B, keine falschen GPU-/Feature-Claims |
| **F3 / Konsolidierung** | Launcher-Install/Publishing/Modpacks vollständig als Module derselben Shell; UI-/Settings-/Auth-Ownership vereinheitlichen | Parität zur bestehenden Launcher-Funktionalität, Update-/Recovery-Tests, kein Doppeldienst |

**Wichtiger Übergang:** Das bestehende Launcher-UI/-Repository wird erst nach funktionaler Inventur und Gate übernommen. „Unified“ beschreibt das Zielbild, nicht eine schon durchgeführte Electron→Rust-Portierung.

## Negativtests und Akzeptanz

- Kein DX12-fähiges Profil, fehlender VS-Adapter, Shader/Engine blockiert: Fehlerstatus statt funktionslosem DX12-Start; Vanilla separat erreichbar.
- Nicht-NVIDIA-GPU, nicht unterstützte NVIDIA-SDK-Funktion oder fehlerhaftes Treiber-Query: Assist sauber degraded; keine Zwangsinstallation.
- Backend-Wechsel während Game/Save/Benchmark: explizites **Restart required** mit geordnetem Shutdown, kein Live-Swap.
- Q17 World-/Chunk-Readiness ausstehend, ungleiche Grafikpässe oder fehlendes Present: Benchmark **BLOCKED/NOT_COMPARABLE**.
- NVIDIA-Tweak verworfen, Profil-Rollback oder Renderer-Absturz: Ausgangseinstellungen und FallBack-Pfad nachweisbar.
- Mod-Overlay/Input-Fokus/GUI: Q4/Q12-Gates, keine Doppel-Dispatches.
- Invasive Driver-/Firmware-/Power-Änderungen sind **out of scope**.

## Abhängigkeiten

Bindend Q1–Q8, Q12, Q15, Q16, Q17. **Q9, Q10, Q11, Q13, Q14: experimentelle Architekturpfade weiterhin OPEN**. **Q19 wird Grafik-Settings und Preset-Policy konkretisieren**, insbesondere NVIDIA Assist, Hardware-Tiers, Quality/Performance und per-Backend-Feature-Gating. Kein Game-/GPU-/Benchmark-PASS durch diesen ADR.

Referenzen:
- [Roadmap V2.0](BISMUT_VS_CLIENT_ROADMAP_V2_0.md)
- [Q17 Benchmark Worlds](ADR_BISMUT_Q17_FRAME_BUDGETS_BENCHMARK_WORLDS_2026-11-19.md)
- [Q15 Dual SDK + Loader](ADR_BISMUT_Q15_DUAL_SDK_BISMUT_LOADER_2026-11-18.md)
