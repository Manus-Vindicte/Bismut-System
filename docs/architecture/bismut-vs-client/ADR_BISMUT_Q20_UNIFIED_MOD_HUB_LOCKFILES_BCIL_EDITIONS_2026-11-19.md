# ADR BISMUT Q20 | Unified Mod Hub + reproduzierbare Modpacks + BCIL (Editionsgrenzen)

- Datum: 2026-11-19
- Ownerentscheidung: **Q20 Option E = B + C + D, gestuft**
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Öffentliche Editionen: **Bismut Client** (Standard) und **Bismut Client Extended** (Extended). Gemeinsame Titelleiste: **Bismut Client**.
- Interne Namen „Bismut VS Client - Standard“ / „BismutX VS Client - Extended“ sind technische Kennungen, kein öffentliches Produktbranding.
- Q20 präzisiert Q7/Q15/Q18A; kein Ersatz bestehender Launcher-Contracts oder deren Gate-Status.

## Beschluss

Der **vollständige normale Mod Hub** gehört verbindlich zur Standardedition **Bismut Client**. Die Extended-Edition enthält **alle** Standardfunktionen und ergänzt nur Bismut-spezifische native Runtime-/DX12-/BCIL-Capabilities. Ein Installer, eine Launcher-/Client-Shell, ein kanonisches Environment-/ModGraph-/ModLock-/MutationTransaction-Modell. **Kein zweiter Modmanager oder Dependency Resolver.**

### Editionsmatrix

| Bereich | Bismut Client (Standard) | Bismut Client Extended |
|---|---|---|
| ModDB-Modbrowser, Suche, Kategorien, Details, Update-Hinweise | Enthalten | Enthalten |
| Installieren, Entfernen, Aktivieren, Deaktivieren, Version-Pinning | Enthalten | Enthalten |
| Profile, Welten-/Instanz-Bindung, Modpacks und lokale Verwaltung | Enthalten | Enthalten |
| Abhängigkeits-/Konfliktauflösung, Version-/Game-Kompatibilität, ModGraph | Enthalten | Enthalten |
| **Modpack-Lockfiles, exakte Release-/Asset-Hashes, Provenienz, Import/Export** | Enthalten | Enthalten |
| Transaktionale Installation, Preview, Snapshot, Verify, Last Known Good, Rollback | Enthalten | Enthalten |
| ModDB Publishing (wenn Provider/Auth verfügbar) | Enthalten, optional und ausdrücklich qualifiziert | Enthalten |
| VS-eigener Mod-Loader für vorhandene VS-Mods | Bestehender VS-Ladepfad | Derselbe VS-Mod-Loader |
| **Bismut Loader** und native Rust-/C-ABI-Mod-SDK-Capabilities | Nicht erforderlich, kein Engine-Zwang | Optionaler Extended-Provider nach Q15-Gates |
| **DX12-Render-/Custom-GL-Mod-Adapter** | Nicht benötigt für Vanilla OpenGL | Extended-Provider, Status nur mit Live-Evidence |
| **Erweiterte BCIL-/REA-Kompatibilitätsanalyse** | Normale deklarative Game-/Version-/Dependency-Checks bleiben Standard | Zusätzlich API-/Shader-/GL-/DX12-Risikoanalyse, Adapter-Vorschläge und Evidence-Matrix, kein automatisches PASS |

**No artificial feature gating:** Eine ursprünglich normale Launcher-/Modpack-Funktion darf nicht zu einer kosten-, online- oder Engine-gebundenen Extended-Funktion gemacht werden. Extended ist ein additiver Capability-Satz.

## Fachliche Ownership und Paketgrenzen

- **Launcher Core:** Installation, Environment, Instance, WorldBinding, ModArtifact/Release, ModGraph, ModLock, Quellprovider (ModDB/Import), Cache, Modpack/Profilverwaltung, MutationTransaction, LaunchPlan, RunRecord, Update/Publishing-Pipeline und Recovery. Lokales Vanilla-Handling **ohne** Bismut-DX12/Engine-Binaries.
- **VS-Mod-Loader:** lädt bereits vorhandene Vintage-Story-Mods weiter. Kein zweites VS-Mod-Assembly-Loading durch Bismut.
- **Bismut Loader (Q15, Extended):** manifestbasierte Registrierung/Capability-/Lifecycle-Verträge für Bismut-native Mods, ohne eigene Downloads/Installs/Resolver als konkurrierende Source of Truth. Plugin-ABI nur über versionierte C-ABI/API-Tabelle und Handle-/Thread-/Ownership-Gates.
- **BCIL (Extended):** **liest** canonical ModGraph/ModLock, statische Metadaten und zulässige Analyseevidence; liefert advisory Compatibility Reports. BCIL ist **kein** modifizierender Package-Owner und promoted keinen reinen Static-Report zu DX12-LIVE/PASS.
- **UI:** eine Mod-Hub-Oberfläche in beiden Editionen. Extended erweitert Details/Diagnose-Tabs und zeigt Capability-/Kompatibilitätsinformationen, wenn der Provider tatsächlich installiert/qualifiziert ist.

**Bestehender konkreter Anknüpfungspunkt im Launcher-Dev-Repo (Stand der Einsicht, kein lokaler Test):**
- `electron/moddb-client.cjs`: existierender ModDB-Transport.
- `electron/modpack/bismutPckContracts.cjs`: derzeitiges Schema V1 mit Mod-ID, Version, Game-Constraint, Source, Release-ID, SHA256, Dependencies sowie Preflight für Duplikate/fehlende Hashes/Version-/Dependency-Probleme.
- `electron/modpack/bismutPckService.cjs`: bestehende Lockfile-/Previous-Lock-/Staging-/Rollback-Grundlage. Diese Implementierung ist **PARTIAL** relativ zum kanonischen Environment-/MutationTransaction-Ziel; Dateiexistenz ist kein Nachweis für vollständige atomare Install-/Rollback-Sicherheit.
- `docs/architecture/BISMUT_LAUNCHER_ROADMAP.md` und `ADR-001_LAUNCHER_READY_TO_CONNECT.md`: Launcher ist eigenständiger Vanilla-fähiger Produktkern. Das **LAUNCHER_READY_TO_CONNECT-Gate** bleibt separat bis zur Qualifikation BLOCKED; Extended darf es nicht umgehen.

## Standard-Mod-Hub-Workflow

`Discover → Resolve → Explain Conflicts → Lock → Preview Environment Diff → Snapshot → Verify Artifacts → Stage → Apply → Validate → Record → Rollback`

1. **Explore:** ModDB/Cache/Import, Tags/Filters/Sortierung, deklarierte Game-/Side-/Dependency-Eignung. Update-Readmodels sind side-effect-free; Anbieter-Daten sind niemals der kanonische Mod-Graph.
2. **Resolve:** deterministischer ModGraph mit Versionsintervallen, transitive Dependencies, Konflikten/Duplikaten, Client/Server/Universal-Seite. Kein stilles Dependency-Downgrade, Mod-Konflikt oder Mod-Version-Upgrade.
3. **Lock:** exaktes `ModLock` mit Schema-Version, modid, Version, Source, ReleaseID, Artifact-Hash, Game/Environment-Binding und Resolver-Provenienz; keine Tokens, Login-/Passwortdaten oder private Pfade in exportierte Lockfiles.
4. **Stage / Atomic Mutation:** Downloads/Importe in isolierten Staging-Bereich, Hash-/Package-/Provenienzprüfung, explizite Nutzerfreigabe, verifizierter Snapshot/Last-Known-Good, gemeinsame MutationTransaction-Commit-/Rollback-Ownership. Bei aktiver World-/Run-Bindung nicht in-place modifizieren.
5. **Import/Export & Repro:** Modpacks reproducible exportieren (Lock + optional erlaubte Dateien) und offline importieren; fehlende/vergriffene/retracted Releases deutlich melden; kein illegaler Redistribution-Pfad. Provider-Auth/Publishing ist optional.
6. **Runtime:** Settings/Mod-Lock beim Launch als unveränderliche Environment-ID/LaunchPlan übergeben, nach Run als RunRecord erfassen. Binary-Modwechsel erst nach geordnetem Neustart; keine stillen Live-Reload-Zusagen.

## Extended: Bismut Loader und BCIL Compatibility Intelligence

- **Q15 Native Loader:** Capability-/ABI-Versionen, Manifest-/Dependency-Lifecycle, side classification und native Engine-Registries, getrennt vom standardmäßigen Launcher Resolver.
- **BCIL-Lanes:** VS-Game-/API-Version; Managed/C#- und native Bismut-Capabilities; historische Renderer-/OpenGL-/Shader-Abhängigkeiten; potenzielle DX12-Konvertierungs-/Adapter-Anforderungen, Shader-/RootSignature-/Resource-/Input-/GUI-Interop; optionale REA-Hinweise nur als technische Klassifikation.
- **Statusmodell:** `SUPPORTED_EVIDENCED`, `INCOMPATIBLE_EVIDENCED`, `POTENTIAL_ADAPTER`, `STATIC_ONLY`, `UNKNOWN`, `BLOCKED` jeweils mit Quelle, Analyzer-/Runtime-Version, Mod-/Artifact-Hash, Ziel-Backend/VS-Version und Datum. **UNKNOWN ist kein PASS**; heuristische Risiken dürfen als mögliche Risiken und nicht als bewiesene Crash-Ursachen dargestellt werden.
- **Evidenztrennung:** Static/Headless-Atlas-Mod-Tests, native DX12 Lab und echte VS Client-Mod-/GL-Zero-Traces sind verschiedene Gates. Atlas PASS garantiert keine Client-Render-/Custom-GL-Kompatibilität. Q7 „volle Kompatibilität“ bleibt V1-Ziel, nicht nachgewiesene Tatsache.
- **Keine ungefragten Patches:** Kein automatisches Mod-Rewriting/IL-Injection, Shader-Transformation oder Veröffentlichung dritter Mod-/Game-Binärteile ohne separate Freigabe, Lizenz- und Provenienzprüfung.
- **Degradation:** Fehlt Extended oder ist BCIL deaktiviert, funktioniert der gesamte Standard Mod Hub weiter. Ein Extended-Crash wird über Q16 klassifiziert und darf keine Standard-Profile/Mod-Archives/Saves kompromittieren.

## Umsetzungs- und Evidence-Gates

| PT | Inhalt | Exit Gate |
|---|---|---|
| **BC-MOD-01** | Launcher-Dev aktuelle Environment-/Mod-/Modpack-/Publishing-Owner inventarisieren, bestehendes BismutPck an canonical ModLock binden | Kein zweites Graph-/Lock-/Mutation-Modell, bestehender Vanilla-Pfad stabil |
| **BC-MOD-02** | Standard UI für Browse/Profiles/Update/Install/Enable/Disable + Resolver-Diff + explicit confirm | Readmodels/Provider sauber getrennt, deterministische Conflict-/Dependency-Tests |
| **BC-MOD-03** | Modpack Import/Export + Hash-/Provenienzprüfung + Environment-/WorldBinding-/Snapshot-/Rollback-Vertrag | Offline-/Corrupt-/Missing-/Retracted-/Active-Run-Negativtests und Last Known Good |
| **BC-MOD-04** | Extended Bismut Loader Adapter und BCIL Readmodel (nur advisory zuerst) | Capability-Version-Mismatch, UNKNOWN-Cases und keine Standard-Regressions |
| **BC-MOD-05** | Integration Atlas-/VS-Live-/DX12-Mod-Matrix, Install/Remove Extended, Crash-/Recovery-Checks | Echte Client-Evidence nur dann qualifizieren, keine automatische OpenGL-Zero-Promotion |

**Nicht tun:** kein neues separates Produkt/zweites ModDB-System, keine Engine-Abhängigkeit im Standard, keine irreversible Mod-/Save-Mutation, keine unqualifizierten Custom-GL-PASS, keine Änderung an aktiven Binary-Mod-Sets. Der Installer zeigt **Bismut Client** / **Bismut Client Extended** und die Titelleiste bleibt in beiden **Bismut Client**.

## Folgefrage

**Q21 OPEN:** Developer Tools und Diagnosefunktionen. Standard braucht verständliche Logs, Mod-Konflikt- und Restore-Hilfe; Extended kann Live Engine/Renderer/BCIL/Shader-/GPU-Inspektoren und Evidence-/SDK-Tools ergänzen. Keine finale Q21-Wahl durch Q20.

## Referenzen

- [Roadmap V2.3](BISMUT_VS_CLIENT_ROADMAP_V2_3.md)
- [Q18A Editions-ADR](ADR_BISMUT_Q18A_PRODUCT_EDITIONS_INSTALLER_2026-11-19.md)
- [Q15 Dual SDK/Loader ADR](ADR_BISMUT_Q15_DUAL_SDK_BISMUT_LOADER_2026-11-18.md)
- [Q16 Recovery ADR](ADR_BISMUT_Q16_RUST_RECOVERY_SUPERVISOR_EXTERNAL_WATCHDOG_2026-11-18.md)
- Launcher Source: https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/BISMUT_LAUNCHER_ROADMAP.md
- Launcher readiness: https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/ADR-001_LAUNCHER_READY_TO_CONNECT.md
