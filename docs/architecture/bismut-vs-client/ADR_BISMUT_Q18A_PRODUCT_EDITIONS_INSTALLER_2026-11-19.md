# ADR BISMUT Q18A | Produkteditionen und Installer

- Datum: 2026-11-19
- Owner-Beschluss: **Einheitlicher öffentlicher Markenname mit interner Editionierung**
  - **Öffentlich:** Bismut Client (Standardinstallation)
  - **Öffentlich:** Bismut Client Extended (optionale erweiterte Installation)
  - **Interne technische Bezeichnungen:** Bismut VS Client - Standard / BismutX VS Client - Extended
  - **Titelleiste in beiden Editionen:** Bismut Client
- Status: **PRODUCT_DECIDED / INSTALLER_NOT_IMPLEMENTED**
- Bezug: Q18 (Unified Launcher-/Client-Shell); Q20 Mod-Hub-/Modpack-Details bleiben OPEN
- Grundlage: Launcher-Dev als unabhängiges Vanilla-fähiges Produkt und Q18 Unified Shell. Das Dokument ist eine **additive Editionierungsentscheidung**, kein Ersatz für die Launcher-ADR-001-Integration-Gates.

## Produktvertrag

**Ein Installer, ein gemeinsamer Launcher-/Client-Kern, zwei Ausbaustufen**. Editionen sind Installations-/Capability-Profile, **keine zwei konkurrierenden Programme oder Datenbanken**. Nach außen werden ausschließlich folgende Namen angezeigt, mit der gemeinsamen Titelleiste **Bismut Client**:

| Produktname | Installationswahl | Enthalten |
|---|---|---|
| **Bismut Client** | **Default / Standard** | Eigenständig nutzbarer Vintage-Story-Launcher mit Vanilla-Launch, Versionen/Instanzen, Welten, Mods/Modpacks, Profilen, Updates, Backups/Recovery und den jeweils tatsächlich qualifizierten Standard-Settings |
| **Bismut Client Extended** | **Optional erweiterte Installation** | Alles aus Standard **plus** optionale BismutX-Runtime-Pakete: Bismut Native DX12 (nach qualifiziertem Live-Gate), Q15 Bismut Loader/Native SDK, Q17 Benchmark Worlds, Q19 Advanced Graphics/Auto-Tuner/NVIDIA Assist und später qualifizierte Extended-Features |

**UI-Regel:** Titelleiste, Taskbar-/Shortcut-Name und Haupt-Branding bleiben unabhängig von der Edition **Bismut Client**. Die Edition wird nur dort angezeigt, wo sie inhaltlich nützt: Installer-Auswahl, Funktionsübersicht, Einstellungen → Über/Komponenten sowie bei Extended-only-Funktionen und deren Verfügbarkeit. Kein öffentliches „BismutX“-Branding. Interne Paket-/Provider-IDs dürfen weiterhin das Präfix bismutx tragen, sind aber keine Produktnamen.

**Keine künstlichen Standard-Limits:** Features aus dem bisherigen unabhängigen Bismut Launcher werden nicht rückwirkend exklusiv für X gemacht. Extended bedeutet zusätzliche native Runtime-/Graphics-/Developer-Capabilities, nicht ein kostenpflichtiges Abo oder separate Login-/Profildatenbank. **Keine Lizenz-/Preis-/Monetarisierungsentscheidung** getroffen.

## Installer-Führung

Der Installer erklärt die Wahl in Alltagssprache:

**„Wie möchtest du Bismut installieren?“**

1. **Bismut Client (empfohlen)**: „Verwaltet und startet Vintage Story. Keine zusätzliche Bismut-Engine erforderlich.“
2. **Bismut Client Extended (optional)**: „Enthält alle Standardfunktionen und ergänzt bei unterstützter Version den erweiterten Client mit DX12-Backend, Benchmark-Welten und erweiterten Grafikeinstellungen.“

Der Installer kann aus demselben signierten Paket-/Channel-System **lokale/offline Pakete entpacken** oder **versioniert herunterladen, verifizieren, atomar installieren**. Eine Onlineverbindung ist nicht zwingend für vorhandene Offline-Pakete und den bereits installierten Standard-Launcher. Größe, Runtime-/Game-Version, Installationsort, fehlende Komponenten und Neustart-Hinweise vor Bestätigung sichtbar machen. Kein automatischer Engine-Download im Standard-Modus.

### Technische Paket- und Capability-Trennung (Schema zu qualifizieren)

- `bismut-client-core`: Shell/Launcher, canonical Environment/WorldBinding/LaunchPlan/RunRecord, Install/Update/Modpack/Recovery-Prozesse.
- `bismutx-runtime-dx12`: Rust-DX12-Client und versionierter VS-.NET-Bootstrap/Adapter, nur bei unterstützten Game-/Build-Kombinationen.
- `bismutx-loader-sdk`: nativer Loader-/SDK-Support, nur falls kompatibel mit Core/VS-/Backend-Capabilities.
- `bismutx-benchmarks`: versionierte Q17-World-Fixtures, Benchmark Runner/Reports; Reporting aus standardmäßigem Launcher-Core bleibt sinnvoll nutzbar.
- `bismutx-graphics-assist`: Q19 Advanced Tuning, Backend-Capability-Provider und optional NVIDIA Assist.

**Diese Paket-IDs sind Vorschläge im Architekturvertrag, keine Behauptung vorhandener Builds.** Feature-Grants hängen an Installation + kompatiblem Contract + echten Capability-/Readiness-Gates, **nicht** an einem einzigen Boolean „X“.

## Integration mit bestehenden Entscheidungen

- **Launcher-Unabhängigkeit (Launcher ADR-001):** Standard-Installation, Vanilla-Launch und die Launcher-Kernfunktion müssen ohne Bismut-Engine funktionieren. Der Gate `LAUNCHER_READY_TO_CONNECT` ist laut aktuell geprüften Launcher-Docs BLOCKED, bis die dort definierten Kriterien bestanden sind. X darf diese Voraussetzung nicht per UI-Flag umgehen.
- **Q18:** eine Shell; bei aktiviertem Bismut DX12 folgt sie der Q5 Rust-EXE mit in-process Vintage-Story-.NET-Game-Runtime. Beim Vanilla-Modus bleibt ein separater Vanilla-VS-Spielprozess zulässig. Kein zweiter Settings-/Mod-Resolver-/Auth-Owner.
- **Backend-Wahl:** Standard bietet Vanilla OpenGL; mit qualifiziertem Extended-Paket erscheint Bismut Native DX12 abhängig von Supported/Experimental/Blocked/Unknown. „Extended installiert“ ist **kein** Beweis für OpenGL Zero oder DX12-Live-Readiness.
- **Q15:** VS-Mod-Loader bleibt für vorhandene VS-Mods verantwortlich; BismutX Loader als optionale Capability-Schicht, keine zweite Modinstanz.
- **Q16:** Upgrade/Removal ist kontrollierte Transaktion mit Snapshot/Verify/Rollback. Entfernen von X löscht **keine** Vanilla-Installationen, Welten, Mods, Profile, Auth-Informationen oder Launcher-Kerndaten. X-spezifische Settings werden versioniert deaktiviert/erhalten oder explizit zur Löschung angeboten.
- **Q17/Q19:** Benchmark-Auswahl und NVIDIA Assist werden nur als wirklich unterstützte Features aktiv; Benchmark darf bei WORLD/RENDER_NOT_READY nicht als erfolgreich gelten. Keine automatischen globalen Treiberänderungen.

## Installer-Akzeptanztests

1. **Fresh Standard:** installiert und startet normal Vanilla, selbst ohne Engine-/GPU-/BismutX-Binaries; kein Engine-Download oder stiller Zusatzservice.
2. **Fresh Extended:** Standard zuerst, danach ausschließlich passende Extended-Pakete; prüft Version/Hash/Provenienz/Signer, Host- und VS-Version, Disk Space und Restart-Semantik.
3. **Upgrade Standard → Extended:** gleiche Environment-/World-/Mod-/Settings-Ownership; neu aktivierte Capabilities erst nach Gate; kein verlustbehafteter Import.
4. **Downgrade Extended → Standard:** Rust DX12/Loader/Bench-Extras werden deaktiviert oder deinstalliert, Standard und Vanilla-Fallback bleiben funktionsfähig; keine Save-Mutationen.
5. **Failure/Offline:** unterbrochener Download, korrupter Hash, unsupported GPU/VS-Version oder unvollständiger Runtime-Provider führen zu erkennbarem PARTIAL/BLOCKED + Rollback; Standard bleibt nutzbar.

## Nicht beschlossen / Status

Nicht entschieden: Preis/Lizenz, Markenrechtslage, eigentliche technische Installer-Implementierung, finale UI-Gestaltung, Release-Zeitpunkt, DX12-Mod-Kompatibilitäts-PASS. **Die sichtbaren Editionsnamen und die einheitliche Titelleiste sind dagegen beschlossen.** **Q20 bleibt explizit offen**, auch wenn Standard bereits vorhandene Modpack-/ModDB-Funktionen behält. Kein automatisches Umetikettieren alter Launcher-Versionen zu „Rust Native“ und keine ungeprüften Package-/Renderer-Rewrites.

## Referenzen

- [Roadmap V2.2](BISMUT_VS_CLIENT_ROADMAP_V2_2.md)
- [Q18 Unified Launcher/Client Shell](ADR_BISMUT_Q18_UNIFIED_LAUNCHER_CLIENT_BACKEND_BENCH_NVIDIA_2026-11-19.md)
- Launcher canonical plan: https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/BISMUT_LAUNCHER_ROADMAP.md
- Launcher integration gate: https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/ADR-001_LAUNCHER_READY_TO_CONNECT.md
