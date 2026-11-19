# MASTER GOAL | Bismut Engine + Client · genau zwei Slices · Team-Building Pflicht

- Datum: 2026-11-19
- Status: **READY_FOR_LOCAL_AGENT_HANDOFF / IMPLEMENTATION_NOT_STARTED_BY_THIS_GOAL**
- Auftraggeber-Entscheid: **Q25 E**, aber vorhandene Launcher-Startseite beibehalten. Kein zweiter World-/Server-Browser.
- Umsetzung: **SLICE 1 ENGINE** und **SLICE 2 CLIENT**. Genau zwei **Implementierungs**-Slices; Audit/Team-Building sind gemeinsame Vorbedingungen und kein drittes Implementierungsslice.
- Öffentlicher Produktname/Titelleiste: **Bismut Client**, optional **Bismut Client Extended**. Nur Stable-Releases öffentlich; Standard braucht keine Bismut-Engine.
- Zentrale Grenze: derzeitige **lokale Engine-Commits sind NICHT gepusht**, also in diesem Git-Repo **nicht unabhängig verifiziert**. Der folgende Engine-Stand ist ein **Owner-/Agenten-Report**, keine neue Test-Ausführung.

## MASTER-GOAL-OUTCOME

Nutze den tatsächlich vorhandenen Bismut Launcher und die **bestehende DX12-Bridge**. Qualifiziere die nächste minimal erforderliche **Render-Dispatch-/Command-List-Erweiterung** im existierenden Engine-Pfad, dann verbinde den bereits vorhandenen Launcher-World-/Server-Start mit dem passenden Runtime-/Session-Vertrag. Zeige verständliche UI, keine neue Launcher-UI. Erhalte Vanilla-Start, WorldBinding, Originalsaves, Modpacks und Auth.

**Nicht tun:** kein „DX12 von Null“, keine zweite Mesh-/Animation-/Avatar-Bridge, keine Rust-Neuimplementierung des bereits nachgewiesenen Avatar-Pfads, kein neuer Home-/World-/Server-Screen, kein ungeprüfter DX12-Produkt-PASS und keine unbeschlossene Komplett-Portierung. Rust Native DX12 bleibt **Q5/Q6-Zielarchitektur**; der aktuelle live belegte Pfad ist eine vorhandene C++-DX12-Referenz/Bridge. Kein automatisches Gleichsetzen.

## 0. HARD GATE: TEAM BUILDING vor jeglichem Code

Der Lead muss zunächst über das **tatsächlich verfügbare** Agenten-/Orchestrator-System echte, getrennte Agenten/Worker instanziieren und Aufgaben übergeben. **Team-Building ist Pflicht und darf nicht durch textuell simulierte Rollen erfüllt werden.**

| Rolle | Owner | Verantwortlich für |
|---|---|---|
| Lead / Integrator | 1 reale koordinierende Session | Repos/Authority, Branch-/Lease-Zuteilung, Abhängigkeiten, Owner-Entscheidungen und Integration |
| Engine Worker | eigene reale Session/Worker | bestehende Dynamic-Mesh-/Avatar-/DX12-Bridge und Engine-Slice |
| Client Worker | eigene reale Session/Worker | vorhandener Launcher RuntimeSupervisor, World-/Server-/LaunchPlan-/RunRecord-Integration |
| Independent Validator | unabhängig von Engine-/Client-Implementation | reproduzierbare Tests, negative Fälle, Evidence und Claim-Review |

**Team Evidence:** echte Agent-ID/Session-ID, Arbeitsordner, Branch/HEAD, erlaubte Schreibbereiche, Rolle, zugewiesener Task, tatsächliche Ack/Handoff-Receipts. Validator darf nicht aus dem Implementierer „umbenannt“ werden.

**Bei fehlender Teamfähigkeit:** `BLOCKED_TEAM_NOT_AVAILABLE`, reporten welche konkrete Capability fehlt, **keine** Codeimplementation und kein behauptetes Team. Bei fehlenden Repos, Owner-Leases oder schreibgeschützten Bereichen: `BLOCKED_REPO_OR_AUTHORITY`.

## 1. Gemeinsame Bestandsaufnahme / Reconciliation (vor Slice 1 und Slice 2)

1. Verifiziere **lokale** Engine-/Launcher-/Bridge-Repos, Worktrees, HEADs, Branches, Dirty State, AGENTS.md/ADRs, Update-/Launch-/Mod-/WorldBinding-Verträge und Testkommandos. Öffentliche `Bismut-System`-Git-Dokumente sind kein lokaler Engine-Codecheckout.
2. **Engine-User-Report verifizieren, nicht überschreiben:** letzter Worktree `pt-bl-native-window-mod-20261117` **clean**, Commits lokal **`17e127f`**, **`aa4efa3`**, **`6ccbf05`** und **kein Push**. Referenzen: `docs/PT-BISMUT-RENDER-DISPATCH-01_GAP_ANALYSIS.md` (**RESEARCH_DONE / PENDING_DECISION**), `docs/PT-BISMUT-NATIVE-WINDOW-01_W3_RESULT_PARTIAL.md`, `BISMUT_AVATAR_TEXTURE_MANAGER_LIB_V1_ACCEPTANCE.md`. Dateien und Hashes erst **lokal** öffnen, anschließend ihre Aussagen bestätigen.
3. **Avatar-Live-Stand aus Owner-Report (noch nicht durch diesen Auftrag neu geprüft):**
   - **Run 16:** texturierter Third-Person-Avatar auf DX12-Readback, `entityAtlasUploaded=true`, `entitySkinnedDraws=12408`, BMP-SHA `93C1239C…` passend zum Receipt, `PROBE_TP_FRAME.png`.
   - **Run 17:** `BISMUT_ENTITY_ATLAS_SOLID=1` setzt gesamten Avatar flach magenta, `PROBE_SOLID_FRAME.png`, SHA `66796A68…`; SRV Slot 1 gebunden und gesampelt.
   - **Run 19:** AV-05 Pose-Palette **1×/Frame** belegt: Frame 1980→2100, `poseChanges` 2199→2319 (**Δ120/120 Frames**), `entityDrawsTotal` 11816→12536 (**6 skinned draws/frame**), Guard-Build `1EDB170B`.
   - **Deploy-Falle:** `dotnet build` aktualisiert **nicht** automatisch `bin\\Release\\net10.0\\BismutGraphics\\`; Runs 16–18 nutzten **stale DLL `CF4DD9C` ohne E-Guard**, Run 19 war erster Live-Build mit Guard `1EDB170B`. **Vor jedem Run die tatsächlich geladene DLL und Hash/Deploy-Quelle protokollieren**, sonst alle E-Guard-Claims BLOCKED.
   - AV-01..AV-05 sind laut Owner-Report implementiert und live teilweise/einzeln belegt. **Gesamter Avatar-Pass und W3 bleiben PARTIAL**, nicht PASS.
4. **Eingrenzung:** Defekt E ist noch nicht requalifiziert: benötigt tatsächliche **manuelle** Sequenz **Escape → Pausenmenü offen lassen → Fenster schließen**. Keine injizierten Input-Events, kein automatisch nachgestellter Happy-Path als Ersatz. AV-06/07: sichtbare W-Bewegung/Equipment/transparente Teile mit echten Third-Person-Frames prüfen; ohne Nutzermitwirkung ggf. `BLOCKED_MANUAL_INPUT` bzw. `NOT_ATTEMPTED`.
5. **Technische echte Lücken (Owner-Report):** `passMask/visibility`-Deskriptor, frame-gebundene Command-List/Frame-ID, transparente Entity-Teile, Entity-Schatten, GUI/Effects-Produzenten, Resource-Handles, Instancing. **G statisch:** Opaque PSO/Blend FALSE ist belegt, aber Discard bei `base.a < 0.005` nicht gemessen. **Keine neue Pipeline** als Abkürzung.

## SLICE 1 / ENGINE | Bestehende Dynamic-Mesh-Bridge erweitern

**Was:** **Bestehenden** Render Dispatch/Dynamic-Mesh-ABI nutzen und den kleinsten verantwortbaren **Frame-/Visibility-/Pass**-Vertikalschnitt integrieren, statt Avatar-/Bridge-/DX12-Rendering neu aufzubauen.

**Wie:**
1. Baseline verifizieren: lokale Commits, Readbacks/Receipts, native DLL Hash, GUARD, Source-Lines/PSO, VS-Prozessende, aktive Lease. Kein Arbeiten auf einem stale Paket oder anderem Worktree ohne Abgleich.
2. **ADR + explizite Owner-Freigabe als HARD GATE vor ABI-Änderung:** vorhandenes `PT-BISMUT-RENDER-DISPATCH-01_GAP_ANALYSIS.md` lesen; einen minimalen **additiven, versionierten** `passMask/visibility + FrameId`-Contract mit Command-List-Framebindung, Ownership, Lifetime, Backward Compatibility und Negativtests zur Entscheidung vorlegen. Bis zur Freigabe **nur Analyse/Tests/Entwurf**, keine ABI-Änderung.
3. Nach Freigabe exakt **eine** bereits existierende Dynamic-Mesh-Producer-/Consumer-Grenze erweitern. Vorhandene Entity Atlas/21 Joints/Model Matrix/GPU Skinning/Pose-Update bewahren. Keine zweite Bridge. Transparente Entities/Schatten/GUI/Effects/Instancing bleiben explizit separat gegated, sofern nicht klein und tatsächlich nachweisbar innerhalb des genehmigten Contracts.
4. Tests: native Build + richtige Deploy-DLL/Hash, Manager-Unit baseline **5/5 laut Report** neu ausführen, deterministische `FrameId`-/PassMask-/Visibility-Tests, reale Third-Person-Frame-Readbacks, Stale-DLL- und Device-/Shutdown-Negativtests; Avatar-Pose-/Textur-Regressionen müssen weiterhin Evidence liefern.
5. Defekt E und AV-06/07 **nur im erlaubten manuellen Prüfpfad** testen; ohne legitime Handinteraktion korrekt `BLOCKED_MANUAL_INPUT`, keine Test-Fälschung. Jeder behauptete PASS braucht genaue Run-ID, geladenen DLL-Hash, Frame-/Receipt-/Bild-Hänge, Repo-Commit und Command.

**Exit Gate:** `ENG_RENDER_DISPATCH_V1` nur bei genehmigter ADR, funktionierender bestehender Bridge, live `passMask/visibility/FrameId`-Evidence und unverändertem Avatar-Readback als `PASS`. Andernfalls `PARTIAL`/`BLOCKED`; keine automatische Qualifikation von Defekt E, AV-06/07, transparenter Entity-/Shadow-/GUI-/Effect-Parität. Rust-Native-DX12-Migration als **separater, nach Q9/Q10/Q13/Q14 gereifter Zielpfad**, nicht implizites Ergebnis des aktuellen Bridge-Slices.

## SLICE 2 / CLIENT | Vorhandenen Launcher an bestehenden Client-Start binden

**Was:** die existierenden Singleplayer-/Multiplayer-Karten und UI-Aktionen an einen **kanonischen LaunchPlan/RunRecord-Preflight und Session-Lifecycle** binden. Keine neue Launcher-/Welt-/Serveroberfläche.

**Wie:**
1. Bestand `Bismut-Launcher-Dev` (zuletzt geprüft `feat/publish-suite`) gegen tatsächlichen lokalen HEAD vergleichen: `electron/runtime-supervisor.cjs`, Home-Readmodels, `direct-world`/`join-server`, `instance-manager.cjs`, `game-content-manager.cjs`, Auth/Mods-/Profile-/Environment-/WorldBinding-Contract. Bestehende GUI beibehalten.
2. Ein Start-Intent → **ein** immutable `LaunchPlan` mit `requestId/runId`, World-/Server-/Environment-/ModLock-/Version-/Backend-/Lease-/Readiness-Preflight. Kein zweiter RuntimeSupervisor, kein stilles Umschalten DX12→OpenGL, keine World-/Save-Source-Mutation.
3. Session-States `preflight/starting/loading/ready/stopping/exited/crashed/recovery` und `RunRecord` nach tatsächlichem Lifecycle; kontrolliertes Save/Shutdown und Rückkehr in die **vorhandene** Launcher-Startansicht. Falls Host nach Crash weg ist, externer Q16 Watchdog/Launcher-Handoff statt vorgetäuschtem In-Process-Fenster.
4. **Nur qualifizierte Engine-/Runtime-Adapter anbinden**: `LAUNCHER_READY_TO_CONNECT` aus Launcher ADR-001 bleibt **BLOCKED**, bis canonical `Installation/Instance/Environment/WorldBinding/LaunchPlan/RunRecord`, Vanilla unabhängig und `MutationTransaction` nachgewiesen sind. Bei fehlendem Gate isolierter Lab-Adapter und Status `PARTIAL`, nicht produktiver Provider. Q5 Rust in-process .NET-Ziel ist nicht mit dem aktuellen C++ Bridge-Live-Pfad gleichzusetzen.
5. E2E mit isolierten Testwelten und bekannten Server-Fixtures: Singleplayer, Server Join, VS/Mod/Backend-Mismatch, Duplicate Launch, World Lock, Exit/Stop/Crash/Recovery, Vanilla ohne Engine, Avatar-/DX12-Lane nur nach tatsächlichen Capability-Gates. Nach Crash **zuerst lokale Q21 Log-/Crash-Panel-Lösung**, **danach optional** Vercel-gated Bericht; keine Cloud-Pflicht.

**Exit Gate:** Launcher UI und RuntimeSupervisor bleiben Eigentümer der sichtbaren Start-/Session-UX, LaunchPlan/RunRecord und sichere Session-Gates sind durch reale E2E-Receipts bestätigt; ansonsten `PARTIAL/BLOCKED`. Keine Umbenennung von „Launcher gestartet“ zu „vollwertiger nativer VS-Client“.

## Gemeinsame Abnahme und Reporting

- **Lead** sammelt ein gemeinsames Evidence-Register pro Run, korrekt geladenen DLL-Hash, Commit, Branch, Lease, Ausgangs- und Endstatus, echte Testbefehle sowie unabhängige Validator-Checks. Slices können bei konfliktfreien Dateien parallel vorbereitet werden; Client darf keinen unqualifizierten Engine-Provider produktiv promoten.
- **Manager-Unit 5/5 PASS, Build 0 Errors** sind laut altem Report Baseline, **nicht** hier neu ausgeführt. CS0162 ist berichtete bestehende Warnung. Jede Re-Testabweichung dokumentieren.
- Trennung der Status: `TEAM_BUILD`, `ENGINE`, `CLIENT`, `INTEGRATION`, `DEFECT_E`, `AV06_AV07` mit `PASS/PARTIAL/BLOCKED/NOT_ATTEMPTED` je Gate.
- **One-stop Master Goal Report**: reale Team-Matrix, Repo-/HEAD-/Dirty-Inventar, ADR-Freigaben, genau zwei Slice-Summaries, vollständige Evidence-/Hash-/Test-Liste, Known Issues, manuelle Blocker und next gate. Keine fake GUI-E2E- oder FPS-Zahlen.
- Verboten: unautorisierter Push/Merge, Owner-Approval umgehen, manuell geschützte Escape-/Equipment-Events automatisieren, produktive Welten modifizieren, zweite Bridge, Rust-Wegwerf-Rewrite oder Experimente mit ungeprüften Treibertweaks.

## Hand-off-Text für den lokalen Agenten

**„MASTER GOAL STARTEN. Erst echtes TEAM BUILDING (Lead, Engine, Client, Independent QA) mit Belegen. Danach genau zwei Implementierungsslices: S1 vorhandene DX12 Dynamic-Mesh-/Avatar-Bridge um genehmigte passMask/visibility/FrameId-Grenze erweitern, S2 bestehenden Launcher mit LaunchPlan-/RunRecord-/Client-Session-Lifecycle verbinden. Die zuletzt gemeldeten Runs 16/17/19 und lokalen Commits 17e127f/aa4efa3/6ccbf05 auf dem tatsächlichen Worktree verifizieren. Kein neuer Avatar-Renderer/keine zweite Bridge. Defekt E/AV-06/07 bleiben manuell gegated. Existing Launcher UI erhalten; Vanilla ohne Engine; Stable-only; Q21 Crash-Panel lokal vor optionalem Vercel-Bericht. Zuerst Owner-Approval für neue ABI-ADR, sonst PARTIAL/BLOCKED. Liefere ein einziges belegtes Team-/Slice-/E2E-Reportpaket und keine unbelegten PASS.“**

## Referenzen

- [Q25 Existing Launcher / Client ADR](ADR_BISMUT_Q25_EXISTING_LAUNCHER_CLIENT_SESSION_2026-11-19.md)
- [Roadmap V2.8](BISMUT_VS_CLIENT_ROADMAP_V2_8.md)
- [Q16 Recovery/Watchdog](ADR_BISMUT_Q16_RUST_RECOVERY_SUPERVISOR_EXTERNAL_WATCHDOG_2026-11-18.md)
- [Q20 Mod Hub](ADR_BISMUT_Q20_UNIFIED_MOD_HUB_LOCKFILES_BCIL_EDITIONS_2026-11-19.md)
- Launcher ADR-001: https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/ADR-001_LAUNCHER_READY_TO_CONNECT.md
