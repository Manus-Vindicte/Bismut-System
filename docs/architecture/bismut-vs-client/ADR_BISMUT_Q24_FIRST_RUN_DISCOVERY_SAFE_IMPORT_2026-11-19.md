# ADR BISMUT Q24 | Schneller Erststart, optionaler Import und sichere Migration

- Datum: 2026-11-19
- Owner-Entscheidung: **Q24 Option E = D + B + C als gestufter Hybrid**
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Öffentliche Produktnamen: **Bismut Client** (Standard) / **Bismut Client Extended**; Titelleiste immer **Bismut Client**
- Leitplanken: **minimalistische Benutzeroberfläche, existierende Vintage-Story-Installationen und Welten schützen, öffentlich nur Stable (Q23)**

## Entscheidung

Beim ersten Start bietet Bismut einen **sofort nutzbaren, nicht invasiven Einstieg**. Es sucht in zulässigen lokalen Installations- und Datenpfaden nach vorhandenen Vintage-Story-Versionen, Instanzen, Mods und Welten, **ausschließlich lesend** und mit beobachtbarem Herkunfts-/Integritätsstatus. Die Erkennung verändert weder die Originalsaves noch Auth-Dateien, Spielkonfiguration oder Mod-Ordner.

- **D: Schnellstart zuerst.** Bei eindeutiger, lauffähiger Installation ohne zwingende Migration: Start-/Weltauswahl zeigen. Bei keinem Treffer: „Vintage Story auswählen“ bzw. über qualifizierten offiziellen Installationspfad installieren. **Kein erzwungener Account, keine Cloud-Verbindung, keine Engine-Pflicht**.
- **B: Import auf Wunsch.** Ein kompakter optionaler Assistent zeigt eine Vorschau vorhandener Installationen, Welten, Instanzen, Modpacks, Mods und Profilkonfigurationen. Nutzer wählt gezielt die zu übernehmenden Einträge; es gibt **Überspringen/Später importieren**, nie einen stillen Massenimport.
- **C: Sichere Migration nur transaktional.** Bei tatsächlichem Transfer/Änderung: isolierte Zielkopie, verifizierter Snapshot, Environment-/WorldBinding-/ModLock-Abgleich, Hash-/Provenienz-/Schemaprüfung, Commit/Verify und Rollback. Quelle unverändert, solange Benutzer keine ausdrücklich gesonderte, qualifizierte Originaloperation freigibt (Q24 umfasst **keine** destruktive Source-Migration).

Die Erststart-UI bildet lediglich den Zustand des kanonischen Launcher-Modells ab. Keine konkurrierende Rust-/Electron-Datenbank oder zweite „Import-Engine“.

## UX-Kontrakt: Wenige Aktionen, Details nur bei Bedarf

| Zustand | Benutzeroberfläche | Automatische Nebenwirkung |
|---|---|---|
| Eine eindeutige VS-Installation erkannt | **„Vintage Story gefunden“**, **„Spielen“**, optional **„Welten und Mods übernehmen“** | Nur read-only Discovery + lokale, nicht mutierende Anzeige |
| Mehrere Installationen erkannt | **„Welche Version möchtest du verwenden?“** mit verständlicher Auswahl | Keine unsichtbare Versions-/Profilmigration |
| Keine Installation erkannt | **„Vintage Story auswählen“**, optional **„Spiel installieren“** (nur qualifizierter offizieller Provider) | Keine ungefragte Download-/Account-Aktion |
| Bestehende Welten/Mods erkannt | **„Vorhandene Daten gefunden“**, **„Übernehmen“** / **„Später“** | Bis zur Bestätigung nur Vorschau, keine Kopie/Schreibzugriffe |
| Migration benötigt oder Konflikt erkannt | **„Übernahme prüfen“** mit einfacher Zusammenfassung der betroffenen Daten; bei Bedarf „Details“ | Änderungen ausschließlich nach plan/snapshot/verify/rollback-Vertrag |
| Import fehlgeschlagen | **„Deine Originaldaten sind unverändert“** nur bei tatsächlich verifiziertem Quellschutz; **„Erneut versuchen“** / **„Überspringen“** | isoliertes Staging aufräumen, unklare Zustände BLOCKED statt scheinbarer Erfolg |

**Ersten Spielstart nicht von einem großen Assistenten abhängig machen**, sofern vorhandene Installation und LaunchPlan ohne Mutation tatsächlich verwendbar sind. „Spielen“ bedeutet nur bei validiertem Vanilla-LaunchPlan/RunRecord-Vertrag eine freigegebene Startoption, keine aus reinen Dateien abgeleitete PASS-Garantie.

## Canonical Discovery & Import Contract

**Discovery V1 (Read-only):**
- Erfasst gefundene Installation/Version/Executable/InstallDir, prüfbare Ownership-/Datei-Hashes, DataPath/Instanzkandidaten, World-Metadaten, aktive Mod-/Modpack-Kandidaten und relevante Config-/Game-Version, **ohne kompletten Save-Inhalt, Zugangsdaten oder unnötige personenbezogene Pfade in Logs zu exportieren**.
- Behandelt Windows-/VS-Standardpfade, vom Benutzer gewählte Pfade und portable/side-by-side Installationen als Kandidaten, nicht als unfehlbare Detection. Netzlaufwerke/symbolische Links/Junctions/ACLs/unerreichbare Quellen müssen klassifiziert werden; keine Pfad- oder Symlink-Traversal in unkontrollierte Schreibziele.
- Provenienzfelder: `source_kind`, `source_id`, `source_version`, `read_only`, `supported`, `confidence`, `discovered_at`, `source_fingerprint`, `blocking_conflicts`. Keine Speicherung von Geheimnissen oder vollständigen privaten Pfaden in einem exportierbaren Report.
- Eindeutige Zuordnung vorhandener Daten erfolgt nicht anhand von Namen allein; gleichnamige Welten, widersprüchliche Save-Metadaten, Dubletten und fremde Instanz-IDs bleiben **NEEDS_REVIEW**, kein automatischer Merge.

**Import Plan V1:** `source_id`, `target_environment_id`, `selected_worlds`, `selected_mods`, `selected_instances`, `game_version`, `source_hashes`, `world_binding_policy`, `copy_policy`, `expected_size`, `conflicts`, `snapshot_plan`, `user_approval`, `dry_run_result`. Der vollständige Plan ist vor der Mutation festgelegt und erhält eine eindeutige idempotente Transaktions-ID. Der Benutzer sieht nur die kurze Liste übernommener Inhalte und ggf. Konflikte.

**Transaktion:** `Discover → Plan → Validate → Snapshot → Copy to staging → Verify hashes/schema → Create canonical Environment/WorldBinding/ModLock → Commit → Smoke → Record`, bei einem Fehler `Abort/Rollback`. Reale Spielstände nicht öffnen/mutieren, während Vanilla/Bismut/Game/Server eine aktive Instanz oder WorldBinding hält; welt- und instanzweite exklusive Locks. Quell-Saves bleiben read-only, die Zielkopie gehört einer getrennten DataRoot/Instance-Identität. Ein eventuelles direktes „Use existing in place“ darf nur ein qualifiziertes explizites Nutzermodell sein, niemals stillschweigendes Default.

**Credentials/Account:** Vintage-Story-Auth nicht duplizieren, bestehende Credentials/Tokens/Cookies nicht in exportierbare Bundles/Docs kopieren. OS Credential Store / sichere AccountRef-Verträge aus dem kanonischen Launcher verwenden, wenn nötig. Publisher-/ModDB-Auth ist kein First-Run-Gate. Der Offline-Standard-Client bleibt benutzbar, soweit VS-Authentifizierungs- und Lizenzbedingungen den gewünschten Start zulassen.

## Editionen und Architekturverträglichkeit

| Geltung | Standard | Extended |
|---|---|---|
| Read-only Vintage Story Discovery, Welcome, optionaler Import | Ja | Ja |
| Isolierte sichere World-/Mod-/Profile-Migration mit Snapshot/Rollback | Ja | Ja |
| Vanilla-Launch und bestehende VS-Mod-Umgebung | Ja, keine Bismut Engine benötigt | Ja |
| Backend-Eignung native DX12 / Q15 Loader / BCIL | Keine zusätzliche Engine-Abhängigkeit | Optionale, separat qualifizierte Compatibility-Prüfung **nach** `LAUNCHER_READY_TO_CONNECT` |
| Grafik- und Modpack-Schutz | Q19 und Q20 nutzen denselben Environment-/Settings-/ModLock-Owner | Gleiche kanonische Daten, keine zweite Import-Implementation |

Das bisherige Launcher-Dev-Repository hat **partielle** Instanz-/Config-/World-Adapter (z. B. `electron/instance-manager.cjs`, `electron/game-content-manager.cjs`). Das sind wiederzuverwendende Grundlagen, **kein Nachweis** einer vollständigen sicheren Import-/WorldBinding-/Mutation-Pipeline. Die kanonischen Launcher-Waves PT-BL-001 bis PT-BL-005 und das weiterhin als BLOCKED dokumentierte Gate `LAUNCHER_READY_TO_CONNECT` dürfen nicht übersprungen werden.

## Implementierung und Tests

| Slice | Was | Gate |
|---|---|---|
| **BC-ONBOARD-01** | Bestehende Installer-, Instance-/World-/Mod-/Auth-Discovery-Owner inventarisieren; read-only Scan mit provenance | Keine Mutation/Secret-Leaks beim Scannen, Negative Path-/Symlink-/ACL-Fixtures |
| **BC-ONBOARD-02** | Minimaler First-Run, eindeutiger Installations-Fund, „Spielen“ und optionaler Import | 0/1/mehrere Installationen, portable/offline, Skip/Resume, Accessibility |
| **BC-ONBOARD-03** | Import-Preview, selektive Wahl und versionierter Dry-Run/Conflict/Space-Plan | Doppel-IDs, defekte Quellen, Incompatible Game/Mod-Sets, Source Read-Only |
| **BC-ONBOARD-04** | Isolierte Copy/Snapshot/Hash/WorldBinding-/ModLock-/Mutation-Transaktion mit Rollback | Crash/Power Loss, Disk Full, Locked/Active World, Original-Hashes vor/nachher identisch |
| **BC-ONBOARD-05** | E2E mit bestehenden echten Vintage-Story-Profilen/Save-Kopien auf Windows, standard und extended; Launcher-/RunRecord-Smoke | Vanilla ohne Engine; Import/Downgrade/Rollback ohne Originalsaves zu verändern; Gated Evidence |

**Nicht tun:** erzwungene Migration, Account-Duplikation, unbemerktes Verschieben oder Löschen von Save-Dateien, Umbenennen der Original-DataRoots, Upload von User-Dateien, automatisches „Reparieren“ von Benutzerwelten oder Auto-Switch zu nicht Stable Engines.

## Anschlussfrage

**Q25 OPEN:** Spielstart-, Welt-/Server-Schnellzugriff und Session-Lifecycle der Unified Shell. Welche Kombination aus „Spielen“/„Weiter“, Backend-Auswahl, sicherem World-/Mod-Preflight, Return to Launcher und Recovery-Fallback soll die sichtbare Startoberfläche bieten? Q25 ist nicht durch Q24 vorentschieden.

## Referenzen

- [Roadmap V2.7](BISMUT_VS_CLIENT_ROADMAP_V2_7.md)
- [Q18A Editionen](ADR_BISMUT_Q18A_PRODUCT_EDITIONS_INSTALLER_2026-11-19.md)
- [Q20 Mod Hub](ADR_BISMUT_Q20_UNIFIED_MOD_HUB_LOCKFILES_BCIL_EDITIONS_2026-11-19.md)
- [Q22 Distribution](ADR_BISMUT_Q22_DISTRIBUTION_SIGNED_COMPONENTS_DELTA_OFFLINE_2026-11-19.md)
- [Q23 Stable only](ADR_BISMUT_Q23_STABLE_ONLY_MANUAL_UPDATES_2026-11-19.md)
- [Launcher canonical roadmap](https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/BISMUT_LAUNCHER_ROADMAP.md)
- [Launcher ownership/WorldBinding gap map](https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/BISMUT_CURRENT_TO_TARGET_MATRIX.md)
