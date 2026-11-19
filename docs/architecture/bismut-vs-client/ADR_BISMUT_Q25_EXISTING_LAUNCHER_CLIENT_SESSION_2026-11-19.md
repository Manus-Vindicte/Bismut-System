# ADR BISMUT Q25 | Existing Launcher → Client Start/Resume and Session Lifecycle

- Datum: 2026-11-19
- Ownerentscheidung: **Q25 Option E (A+B+C+D), präzisiert: bestehende Launcher-Startseite erhalten; E gilt der Client-Anbindung, nicht dem Neubau des Launchers**
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Produkt: öffentlich **Bismut Client** / **Bismut Client Extended**, gemeinsame Titelleiste **Bismut Client**
- Architekturabhängigkeiten: Q5 Rust-native DX12 Client mit in-process VS-.NET-Runtime **als Ziel**, Q12 Input, Q16 Crash/Recovery, Q18 gemeinsame Shell, Q20 ModLock, Q24 WorldBinding/Import. Q9/Q10/Q11/Q13/Q14 bleiben offene Evaluierungen.

## Entscheidung

Der bestehende **Bismut Launcher** besitzt bereits eine sichtbare **Startseite mit Singleplayer-Welten, Multiplayer-Serverliste, Auswahl und Beitreten**. **Diese UI ist der fachliche Start-/Session-Einstieg**. Q25 beschreibt keinen zweiten World Browser, keinen zweiten Server Picker, keine zweite Start- oder Launcher-App und keine blinde Reimplementation in Rust.

**Q25 E bedeutet:** Die vorhandenen Launcher-Aktionen starten einen **einzigen qualifizierten LaunchPlan**, dessen preflight die korrekten Game-/World-/Mod-/Backend-Verträge erfüllt. Im Bismut-Extended-DX12-Modus übernimmt der native Client die Laufzeit. Beim Beenden meldet der Client den finalen RunRecord zurück und Bismut zeigt wieder den bekannten Launcher. Bei Fehlern zeigt der Launcher nach Q16/Q21 **zuerst lokale Diagnose und Lösung**, **erst dann optional Fehlerbericht senden**.

## Ownership und Startfluss

```text
Bestehende Bismut Launcher UI (Home / Singleplayer / Multiplayer)
    |
    | Start World / Join Server (vorhandene UI-Aktion)
    v
Launcher LaunchPlan Owner + canonical Environment/WorldBinding/ModLock
    |
    | Silent Preflight: VS build, world/server target, mods,
    | login/connection prerequisites, requested/effective backend,
    | runtime capability, lease/active session, save safety
    +--> BLOCKED: einfache, konkrete Fehlermeldung, keine stille Mutation
    |
    +--> Vanilla OpenGL: normaler qualifizierter VS-Start (eigener Prozess zulässig)
    |
    +--> Bismut Client Extended: Rust-native DX12 Client
                                + VS .NET runtime in-process (Q5 Ziel)
                                + Q2 Frame snapshots / Q4 GUI (nach Gates)
                                + Single HWND / Device / Present Owner
    |
    v
Save/Shutdown or Crash → Session Receipt / RunRecord → Back to Launcher
    |
    +--> Crash: lokales Crash-Panel, Lösungsvorschlag;
         freiwilliger Bericht nur via Vercel nach Zustimmung
```

- Launcher Core bleibt Owner von Environment, Installation, WorldBinding, RunRecord, Mod-/Profilpaket-Verwaltung und Update/Publishing. **Kein zweiter Statusspeicher** durch Rust Client/Renderer.
- Ein Start-/Server-Intent hat eine eindeutige `requestId` und ein `runId`; `LaunchPlan` wird vor Prozess-/Runtime-Start validiert, ist während Run immutable und dokumentiert **requested vs effective Backend**. Silent Fallback von DX12 auf GL ist ausgeschlossen; Vanilla separat.
- Beim Weltstart darf der Client nie ein produktives Save blind kopieren, patchen, neu binden oder mergen. Bei aktiven oder unklaren Welt-/Instance-Leases **BLOCKED**, außer nach separate qualifizierter Benutzeraktion.
- Bei Multiplayer ist Join ein Nutzerintent mit korrekter Host-/Port-/Game-/Mod-Version und erforderlicher Auth/Connectivity. Keine automatische Server-Join-Aktion durch einen Wetter-/News-/Kachel-Refresh.
- Bei Game-/Client-Crash endet der ursprüngliche In-process-Host möglicherweise vollständig. Q16 **externer minimaler Watchdog/Launcher-Handoff** liefert dann die Crash-Hilfe; die Client-UI darf nicht als weiterlaufend behauptet werden.

## Bereits im Launcher vorhanden, weiterverwenden

Das Launcher-Dev-Repo `Manus-Vindicte/Bismut-Launcher-Dev` (geprüfter Ref `feat/publish-suite`) besitzt eine Home-Singleplayer-/Multiplayer-UI und eine `electron/runtime-supervisor.cjs`-Grundlage mit `direct-world` und `join-server` LaunchTarget-Klassifikation, `preflight/starting/loading/ready/stopping/recovery`-Status, World-/Server-/Instance-IDs, `rendererRequested/Resolved/ProofState`-ähnlichen Readmodel-Feldern und Exit-/Crash-Klassen. Sie ist **nicht automatisch ein qualifiziertes Rust-DX12-Backend**.

**Kein Rewrite-by-Assumption:** RuntimeSupervisor, Home Readmodels, LaunchTarget, `instance-manager.cjs`, `game-content-manager.cjs` und kanonische Launcher-Docs müssen gegen die **tatsächlichen lokalen HEADs/Dirty States/Tests** geprüft werden. Die Rust-/C++-DX12-Repos sind über das aktuelle Bismut-System-Git-Dokumenten-Repo nicht eindeutig bestimmt. Der implementierende Agent muss sie als erstes finden; keine erfundenen Pfade, Branches oder PASS-Receipts.

## Minimaler sichtbarer UX-Vertrag

| Situation | UI |
|---|---|
| Launcher offen | bestehende Home-Seite: Singleplayer und Multiplayer wie bisher |
| Welt auswählen | vorhandene World Card / Start-Aktion; ggf. qualifizierter „Weiter“-Shortcut für letzten gültigen Run |
| Server auswählen | vorhandene Server Card / „Beitreten“ |
| Preflight ok | Spielstart; keine technische Start-Checkliste für Benutzer |
| Preflight blockiert | verständlicher Grund und sichere Option, keine automatischen Mod-/Save-/Backend-Mutationen |
| Sitzung beendet | Rückkehr zur bestehenden Launcher-Oberfläche, letzte Welt/Server als validierter Readmodel-Hinweis |
| Crash | Q21: lokales Crash-Panel, logbasierte mögliche Ursache, sichere Lösung, optionaler Support-Upload nach Consent |

### Gates und Negativtests

1. **Launcher→Plan:** Ein Klick führt genau zu einem Request/Run; keine Doppelevents; Version/WorldBinding/ModLock/Backend-Preflight wahrheitsgemäß. Keine neue Launcher-UI.
2. **Vanilla independence:** Standard startet Vanilla ohne Engine. Extended darf bei fehlendem DX12-Provider nicht fälschlich „supported“ oder GL-Zero-PASS zeigen.
3. **Extended runtime:** Native DX12-Engine und Client werden erst nach echtem HWND/Device/Swapchain/Present/Managed-Bootstrap-/Game-Readiness-Test qualifiziert; Q9 Bootstrap und Q10 Threading sind nicht durch Q25 beantwortet.
4. **Session lifecycle:** Start/Ready/Stop/Exit/Crash/Hang/Duplicate Launch, geordneter Save/Shutdown, kein Datenverlust, Return-to-Launcher/Watchdog beim Host-Absturz.
5. **UX and privacy:** Crash-Panel offline mit identifiziert/vermutet/unbekannt; Fehlerbericht ausschließlich freiwillig hinter Vercel Gate, keine erzwungene KI-/Cloud-Anbindung. Öffentliche Releases ausschließlich Stable (Q23).

## Verbindlicher Ausführungsplan

Umsetzung **nicht als weitere Q-Runde zerlegen**, sondern als **[Master Goal mit zwei Umsetzungsslices](MASTER_GOAL_BISMUT_ENGINE_CLIENT_TWO_SLICES_2026-11-19.md)**:

- **Team-Building-Pflicht vor Code:** unabhängige Engine-/Launcher-/Validation-Owner und Lead mit dokumentiertem Ownership-Handoff.
- **Slice 1 Engine:** aktuelle DX12-Codebasis und Evidence inventarisieren, nur ein enges native Rust-DX12-Frame-/Present-/Lifecycle-Inkrement gegen die bestehenden Q-Gates bauen.
- **Slice 2 Client:** bestehenden Launcher-Start-/Join-Flow an die qualifizierten Runtime-Capabilities/LaunchPlan-/RunRecord-Verträge anbinden; keine UI-Duplikate, kein ungeprüfter Engine-Provider.

Ergebnisstatus muss pro Slice und Gate als `PASS/PARTIAL/BLOCKED/NOT_ATTEMPTED` mit echten Test-Receipts berichtet werden; Dokumentation allein ist **kein** PASS.

## Referenzen

- [Roadmap V2.8](BISMUT_VS_CLIENT_ROADMAP_V2_8.md)
- [Master Goal: Engine + Client, zwei Slices](MASTER_GOAL_BISMUT_ENGINE_CLIENT_TWO_SLICES_2026-11-19.md)
- [Q18 Unified Launcher / Client](ADR_BISMUT_Q18_UNIFIED_LAUNCHER_CLIENT_BACKEND_BENCH_NVIDIA_2026-11-19.md)
- [Q24 Safe First Run](ADR_BISMUT_Q24_FIRST_RUN_DISCOVERY_SAFE_IMPORT_2026-11-19.md)
- [Q16 Recovery + Watchdog](ADR_BISMUT_Q16_RUST_RECOVERY_SUPERVISOR_EXTERNAL_WATCHDOG_2026-11-18.md)
- [Launcher ADR-001](https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/ADR-001_LAUNCHER_READY_TO_CONNECT.md)
