# ADR BISMUT Q23 | Nur Stable-Releases, manuell bestätigte Updates

- Datum: 2026-11-19
- Ownerentscheidung: **Q23 Option A**
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Öffentliche Editionen: **Bismut Client** (Standard), **Bismut Client Extended**, Titelleiste jeweils **Bismut Client**
- Q22 Distributionsvertrag bleibt bindend: signierte Komponenten, optional Delta-/Offline-/Portable-Bundles, transaktionaler Updater/Rollback
- Q21 bleibt: lokales Crash-Panel mit Loganalyse und sicherem Lösungsvorschlag vor optionalem Vercel-Gate-Fehlerbericht

## Entscheidung

**Es gibt genau einen öffentlich auswählbaren Release-Channel: STABLE.**

- Keine öffentlichen Beta-, Nightly-, Alpha-/Experimental-Update-Channels, keine kanalwechselnde Benutzeroberfläche. Auch Extended hat **keinen eigenen Beta-/Nightly-Channel**.
- Neue Versionen werden als **signierte Stable-Releases nach manuellem Release-Gate** veröffentlicht. Release-/Update-Discovery ist **nur lesend**; Update-Download/-Installation erfordern Nutzerbestätigung. Kein ungefragter Hintergrund-Download, Auto-Apply oder Neustart.
- Benutzer sehen lediglich **„Update verfügbar“ → „Aktualisieren“ / „Später“** sowie bei Bedarf einfache Fehler- und Wiederherstellungsoptionen. Die Namen von Build-Pipelines, Channels, CI, SDK, ABI, Update-Signaturen und Dev-Lanes erscheinen nicht in der normalen Oberfläche.
- Ein Extended-Paket darf nur in einen öffentlichen Stable-Release aufgenommen werden, wenn sein tatsächlicher Game-/DX12-/ABI-/Mod-Support und die Komponentenkompatibilität die jeweilige Release-Matrix erfüllt. Fehlt ein Extended-Gate, **Standard Stable trotzdem unabhängig releasen**; Extended bleibt auf dem letzten verifizierten kompatiblen Stand oder wird für diese Kombination sichtbar als nicht verfügbar markiert, niemals im produktiven Launcher auf Nightly gewechselt.
- Interne Entwicklungs-/Test-Builds sind **erlaubt**, aber ausschließlich in getrennten CI-/Lab-/Staging-Artefakten und isolierten Testumgebungen. Sie werden **nicht** über öffentliche Update-Manifeste ausgeliefert und sind keine Benutzerkanäle. Entwickler können diese gesondert manuell in eigenen Testprofilen installieren. Ein interner Test-Build ist kein Stable-PASS.

## Einfache UI

| Situation | Sichtbar für Benutzer | Verbotene implizite Aktion |
|---|---|---|
| Alles aktuell | „Bismut Client ist aktuell.“ | Kein unnötiger Kanal-/Pipeline-Dialog |
| Update angeboten | „Update verfügbar“ + Version/kurzer Änderungsüberblick + „Aktualisieren“ / „Später“ | Keine automatische Installation oder Game-Unterbrechung |
| Spiel läuft | „Update wird nach dem Beenden möglich.“ | Keine laufenden EXE-/DLL-/Mod-/Save-Dateien überschreiben |
| Update blockiert | „Update kann derzeit nicht installiert werden“ + sichere Wiederholung/Details | Kein ungeprüfter Patch oder erzwungene Engine-Version |
| Update fehlgeschlagen | „Die vorherige Version wurde wiederhergestellt“ nur bei verifiziertem Rollback; sonst „Wiederherstellung erforderlich“ | Keinen unbewiesenen Rollback-PASS behaupten |
| Extended nicht qualifiziert | „Erweiterte Funktionen für diese Version noch nicht verfügbar“ | Kein stiller Beta-/DX12-Fallback |

Eine **manuelle „Nach Updates suchen“**-Aktion steht in Einstellungen/Über bereit. Ein rein informativer Updatehinweis aus dem vorhandenen UpdateCoordinator ist zulässig, soweit Datenschutz-/Nutzerpräferenzen gelten; er führt niemals automatisch Download/Apply aus.

## Interne Qualität vor Stable-Promotion

Das Release-Team kann beliebig viele interne Build-/CI-/Lab-Kandidaten testen, aber nur **explizit freigegebene, validierte Artefakte** werden öffentlich als Stable signiert/veröffentlicht.

| Gate | Mindestnachweis |
|---|---|
| **REL-01 Core** | Installation/Import/Vanilla-Start ohne Bismut Engine; canonical Environment/WorldBinding/LaunchPlan/RunRecord und Mod Hub/Profiles unabhängig benutzbar |
| **REL-02 Integrity** | Signatur/Hash/Manifest/Version-/Abi-Abgleich, kein Revoked/Unsigned-Release, reproducible Release-Bundle und Trust-Key-Handling |
| **REL-03 Update Safety** | User opt-in, Backup/Stage/Verify/External Updater Helper, Crash/Locked File/Interrupted Download/Power Loss und Last Known Good/Rollback qualifizieren |
| **REL-04 Editions** | Standard↔Extended Install/Remove/Upgrade/Downgrade und alte Profil-/Mod-/World-Bindings ohne Save-/Auth-Korruption; keine Engine-Pflicht im Standard |
| **REL-05 Compatibility** | Game-Version, ModGraph, WorldBinding, Vanilla-/Extended-Adapter- und Runtime-Qualifikationsmatrix; Q7 Full Mod Compatibility / Q8 OpenGL Zero nur bei echter separater Evidence |
| **REL-06 UX/Support** | Einfache Settings/Update-Ansicht, lokale Q21 Crash-Panel-/Log-Analyse vor optionalem Fehlerbericht, Q16 Recovery/Watchdog und Offline-Fallback |
| **REL-07 Distribution** | Signed Full + Delta-Fallback, Portable-/Offline-Bundles, fresh/existing-user Windows-Installation, Repair und Restore |

**No metric invented:** Keine festen Crash-/Performance-/Mod-Compatibility-Schwellen oder Release-SLAs ohne reale Messungen und Owner-Freigabe. Ein UI- oder Synthetic-Test allein qualifiziert kein VS-Live-/DX12-PASS. Release-Promotion ist ein dokumentierter, verantworteter Entscheid mit signiertem Release-/QA-Manifest, nicht nur ein umbenanntes Branch-Tag.

## Update-Policy und Recovery

- Existing Launcher `UpdateCoordinator` bleibt der read-only discovery-/cache-/provider-Owner. Q22 `MutationTransaction` plus externer Update Helper übernehmen nach Zustimmung die Installation und atomaren Version-Wechsel. Nicht parallel mehrere Updater/Release Truth Stores aufbauen.
- Versionen/Channels sowohl von Standard als auch von Extended müssen im öffentlichen Release-Manifest zusammen **kompatibel gepinnt** sein; einzelne Extended-Komponenten dürfen erst nach Dependency-/ABI-/VS-/Game-/Driver-Gates ausgetauscht werden.
- Release-Abbruch bei unbekannter Publisher-Signatur, nicht erreichbarer/inkompatibler Vanilla-/Runtime-Version, modifiziertem Base-Hash, aktiver World-Session oder unqualifiziertem Crash-/Recovery-Zustand. Bei Delta-Fehler signiertes Vollpaket anbieten.
- Letzte verifizierte Version und lokale Recovery-Optionen bleiben verfügbar, solange technisch möglich. Keine unautorisierte Save-Migration oder heimliche Mod-/Profil-Mutation.
- Reaktivierende/entziehende Revocation-Metadaten können signierte Releases als gefährlich blockieren, aber die Richtlinie für präventives Blocking bereits installierter Versionen erfordert eine eigene Safety-/Update-Policy, nicht eine versteckte Auto-Deinstallation.
- Reports/Crash-Facts werden standardmäßig **nicht automatisch hochgeladen**; Q21 Cloud-Support bleibt nach Datenvorschau und ausdrücklicher Zustimmung über **Vercel-Gate**, lokal verständliche Crash-Hilfe funktioniert offline.

## Umsetzungs-/Evidence-Slices

| PT | Arbeit | Exit |
|---|---|---|
| **BC-REL-01** | Release manifest/channel contract: genau 1 öffentlicher Stable-Channel, Manual Apply/Consent/UI copy | Installable/Allowed-Build-Checks, Beta-/Nightly-Metadaten im Public Index geblockt |
| **BC-REL-02** | interne CI/Lab/Release-Candidate-Pipeline strikt vom öffentlichen Stable-Manifest trennen | Keine versehentliche Veröffentlichung von Lab-Artefakten, signierter Release-Approval-Prozess |
| **BC-REL-03** | Q22 Safe Updater mit Release-/Component-Compatibility-Matrix und Rollback-Gates verbinden | Fresh/existing-user Standard/Extended, Offline-/Portable-/Rollback-E2E |
| **BC-REL-04** | Q21 Crash Panel + Update Recovery + Support-Consent unter Stable-Releases testen | Offline/Cloud-Outage, Standard-/Extended-Fehler, klare User-Statusmeldungen |
| **BC-REL-05** | Stable Release Acceptance Review, Last Known Good und signierte QA-Receipts | manuelle Freigabe nur nach reproduzierbaren Tests; sonst BLOCKED |

## Grenzen / Folgeentscheidung

- **Q23-A entscheidet die öffentliche Release-Politik**. Automatische gestaffelte Rollouts, öffentliche Beta-/Nightly-Kanäle und erzwungene automatische Updates sind **nicht** Bestandteil.
- **Q24 bleibt OPEN:** First-Run, Import-/Migrationsassistent und Berechtigungs-/Account-UX für bestehenden Vintage-Story-Installationen/Instanzen/Welten. Q24 kann die Onboarding-/Discovery-Strategie definieren, ohne die Stabilitäts-/Mod-/Installer-Verträge zu ändern.
- Kein Zugriff auf lokale Windows-Checkouts oder Ausführung von Installer-/Release-/Game-Tests durch das Git-Dokument.

## Referenzen

- [Roadmap V2.6](BISMUT_VS_CLIENT_ROADMAP_V2_6.md)
- [Q22 Distribution](ADR_BISMUT_Q22_DISTRIBUTION_SIGNED_COMPONENTS_DELTA_OFFLINE_2026-11-19.md)
- [Q21 Local Crash Panel + Vercel Support](ADR_BISMUT_Q21_SIMPLE_SUPPORT_VERCEL_GATE_2026-11-19.md)
- [Q18A Editions](ADR_BISMUT_Q18A_PRODUCT_EDITIONS_INSTALLER_2026-11-19.md)
- Launcher update architecture: https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/update-and-publishing.md
