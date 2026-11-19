# ADR BISMUT Q22 | Signierte Komponenten, Delta-Updates, Portable/Offline, einfache UX

- Datum: 2026-11-19
- Owner-Entscheidung: **Q22 Option E = B + C + D als gestufter Hybrid**
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Produkt: **Bismut Client** (Standard), optional **Bismut Client Extended**, Titelleiste in beiden „Bismut Client“
- Interner BismutX-Name bleibt technische Edition-/Paketkennung, kein öffentliches Produktbranding.
- Launcher-Dev-Gate **LAUNCHER_READY_TO_CONNECT** unverändert; Q21 Vercel-Diagnose-Gate darf nicht mit Update-/ModDB-Transport vermischt werden.

## Beschluss

Es gibt **einen Installer, einen gemeinsamen Updater und ein signiertes, versioniertes Komponentenmodell**. Standard ist die Default-Installation und bleibt ohne Bismut-Engine, Native-DX12-Binaries oder Cloud-Anmeldung vollständig als Vanilla-/Modded-Launcher funktionsfähig. Extended wird als optionales, kompatibilitätsgeprüftes Paketset zum selben Kern hinzugefügt und kann entfernt werden, ohne Standard-/World-/Mod-/Auth-Daten zu zerstören.

**UX-Invariante:** Benutzer sehen „Update verfügbar“, „Aktualisieren“, „Später“ und bei Bedarf „Problem beheben“ bzw. „Vorherige Version wiederherstellen“. Paketgraph, Delta-Metadaten, Signaturen, Hashes, Dependency Resolver, ABI-IDs, Update-Kanäle und Downloadendpunkte bleiben aus der normalen Oberfläche verborgen. Erweiterte Details sind nur über „Details“/lokalen Supportbericht zugänglich. Kein gesondertes BismutX-Fenster oder zweiter Installationsassistent.

## Artefakte und Vertrauensgrenze

| Artefakt | Inhalt | Owner |
|---|---|---|
| **Core** | Launcher-/Client-Shell, Standard-Mod Hub, Profile/Environment/WorldBinding, Vanilla-Launch, Settings, Recovery | Launcher/Distribution Core |
| **Extended Native Runtime** | Rust Native DX12 + passender VS-.NET-Adapter/Bootstrap, versionierte ABI/Device/Present-Contracts | optionaler qualifizierter Runtime-Provider |
| **Extended Mod-/Graphics-Packs** | Q15 Bismut Loader/SDK, Q17 Benchmark-World-Fixtures, Q19 Advanced Graphics/NVIDIA Assist | optionale Feature-Packs, an Runtime-/VS-Version gebunden |
| **Release-Manifest** | Channel, Release/Build-ID, Komponenten-/Dependencies-/Compatibility-Ranges, Größen, SHA-256, Provenienz, verifizierbare Publisher-Signatur und Signatur-/Key-Rotation-Metadaten | Release-/Trust-Service |
| **Install-Receipt** | installierter Satz, exakte Versions-/Hash-Pins, Channel, vorherige Last-Known-Good-Version, Pending-/Applied-/RolledBack-Status | ein kanonischer Package-/Update-Owner |

**Sicherheit:** Manifest **und** Komponenten gegen einen vorab vertrauenswürdig gebundenen Signierschlüssel prüfen. Hash allein beweist Integrität gegen Übertragungsfehler, nicht den Herausgeber. Vertrauenskette/Key-Rotation/Revoke/Expiry/Anti-Rollback-Metadaten als eigener Vertrag; unbekannte/abgelaufene Signaturen **fail closed**. HTTPS allein genügt nicht als Release-Verifikation. Keine signierbaren privaten Schlüssel, Provider-Tokens oder Benutzer-Credentials in Client-Binaries oder Git.

## Update-Algorithmus

```text
Read-only discover (UpdateCoordinator)
    → reconcile installed inventory + target channel
    → resolve Core/Extended dependency graph
    → show simple update offer + consent
    → verify signed manifest + compatibility/space
    → choose valid delta OR full component
    → download into content-addressed cache
    → verify every payload hash/signature
    → stage in side-by-side versioned directory
    → create Last Known Good / install snapshot
    → quiesce active game/client/update locks
    → atomically switch via external updater helper
    → smoke/health/LaunchPlan verification
    → record success OR restore previous version
```

**Kein In-place-Overwrite laufender EXE/DLLs.** Für Windows wird eine minimale externe Updater-/Bootstrap-Komponente vorgesehen; Installer/Updater braucht kontrollierte Rechte und darf im Normalbetrieb keine Admin-Anforderungen erfinden. Kein Start einer ungeprüften Runtime/Native-DLL. Fail/Crash/Power-Loss muss vom aufzeichnbaren Update-Journal aus reparierbar sein.

## Delta-Updates und Cache (Q22-C)

- Eine Delta-Kette ist nur zulässig, wenn Ausgangsversion/-Hash und Zielversion/-Hash **exakt verifiziert** sind und genügend freier Speicher zur sicheren Rekonstruktion verfügbar ist. Hash-/Signaturprüfung gilt für das rekonstruierte **vollständige** Zielartefakt.
- Fehlende Basis, Cache-Korruption, Patch-Fehler oder schlechter Delta-Nutzen: **automatisch auf signiertes Vollpaket zurückfallen**, ohne dem Nutzer Interna zuzumuten.
- Content-addressed Cache nach vertrauenswürdigen Artefakt-Hashes, bounded Cache/GC, keine Cross-Edition-Asset-Verwechslungen und keine unsignierten Abkürzungen.
- Delta-Kompression/Algorithmus, CDN und Cache-Quota werden erst nach Distribution-Benchmarks festgelegt. Kein vermeintlich bereits implementierter Differential-Updater.

## Offline / Portable (Q22-D)

- **Offline-Installer:** vollständig gebündelte und signierte Core-Pakete; Extended-Offline-Bundle nur bei passenden, ebenfalls signierten Extra-Paketen. Kein Cloud-/Vercel-LLM-Zwang, keine Stillschweigend-Downloads. Offline-Paket kann installierte Vintage-Story-Version importieren; die Spiel-Lizenz-/Auth-Regeln bleiben unverändert.
- **Portable:** unabhängiger Paket-/Datenpfad mit eindeutigem DataRoot; keine stillen Registry-/Treiber-/System-Änderungen; expliziter Hinweis bei Windows-/Game-API-Beschränkungen, damit „Portable“ nicht fälschlich als vollständige OS-Unabhängigkeit verkauft wird.
- **Offline Updates:** importiertes signiertes Bundle und gleiches Staging/Verify/Apply/Rollback-Protokoll; späterer Online-Check optional. Portable, Standard und Extended teilen Signatur-, Manifest-, Receipt- und Safety-Verträge.

## Extended / Versionsicherheit

- Die Edition **ist kein getrenntes Produkt und kein zweiter Updater**. Runtime-Packs können unabhängig vom Core aktualisiert oder entfernt werden **nur**, wenn Versions-/VS-/ABI-/API-/Game-/Mod-Constraints vorher qualifiziert sind.
- **Core/Extended-Versionsmatrix**: Bismut Launcher Host, VS-Version, native Runtime/Loader/SDK ABI, Windows-/GPU-/Treiber-/Shader-Support, Benchmark-Packs und ggf. Modpack-/WorldBinding. Unbekannte Kombination = `UNSUPPORTED/UNKNOWN`, nicht stilles Freischalten.
- **Downgrade Extended → Standard:** Native-Packs entfernen oder deaktivieren; Bismut Client, Profile, Mods, Modpacks, Welten, Save-/WorldBindings und Vanilla-Fallback funktionsfähig erhalten. Nutzer-Rollback des Programms darf keine irreversiblen Save-/Schema-Downgrades erzwingen.
- Eine native DX12-Runtime darf erst als `SUPPORTED` angeboten werden, wenn die echten DX12/GL-Zero-/Mod-Kompatibilitäts-/Recovery-Gates bestanden sind; `EXPERIMENTAL` oder `BLOCKED` entsprechend ausweisen.
- Default bleibt **Bismut Client (Standard)**. Kein Engine-Download oder Bismut-Provider-Requirement ohne Extended-Opt-in. `LAUNCHER_READY_TO_CONNECT` darf durch den Installer nicht umgangen werden.

## Bestehende Launcher-Grenzen (geprüfter Dev-Stand)

- `docs/architecture/ADR-001_LAUNCHER_READY_TO_CONNECT.md` definiert Standalone-Vanilla als Pflicht und `BLOCKED` vor Provider-Integration.
- `docs/architecture/BISMUT_LAUNCHER_ROADMAP.md` sieht signierte Distribution, Portable, Self-Update, Cache und Rollback als **Ziel**, nicht als fertig bewiesenes Produkt.
- `docs/architecture/update-and-publishing.md` und `electron/update/updateCoordinator.cjs` betreffen einen lesenden Update-/Provider-/Cache-Pfad, getrennt von nutzerbestätigter mutierender Installation. Q22 belegt **keinen bereits vorhandenen finalen Updater**.
- Install/Update-Änderungen verwenden den vorhandenen kanonischen `MutationTransaction`-/Snapshot-/Last-Known-Good-Vertrag, statt eine zweite Mutation Authority neben Mod-/World-Operations aufzubauen. Ein App-Replacement verlangt ggf. separaten Bootstrap-Helfer, aber keine zweite fachliche Quelle.

## Slices und Tests

| PT | Umsetzung | Exit-/Negativtest |
|---|---|---|
| **BC-DIST-01** | Inventur Release-/Update-/Installer-/UpdateCoordinator-Contracts, Package Manifest V1 und Signatur-/Key-Rotation-Design | Manifest-/ABI-/VS-/Dependency-/Signature-Fixture Tests, Unsigned/Revoked/Expired Fail-Close |
| **BC-DIST-02** | Side-by-side Installation, external Updater Helper, journaled Apply/Health/Rollback, Core Standard allein | Unterbrochener/abgestürzter Update-/Power-Loss-Run; vorherige Version bleibt verwendbar |
| **BC-DIST-03** | Signed Extended Component Packs, Upgrade/Remove Standard↔Extended und Version Resolver | Fehlende/inkompatible Engine/Loader/VS/GPU blockiert nur Extended, Standard läuft |
| **BC-DIST-04** | Delta-Kette + hash-/signaturverifiziertes Full-Package-Fallback und bounded Cache | Base-Hash-Mismatch, Cache Damage, truncated/expired downloads, out of space |
| **BC-DIST-05** | Portable/Offline Bundles + einheitliches UI + Release-/Upgrade-/Downgrade-E2E | Offline ohne Engine/Vercel, Original-Welten/Profile/Auth unberührt, Recovery PASS nur bei Beleg |

**PASS wird erst vergeben**, wenn reale Installer-Builds, Install-/Update-/Downgrade-/Offline-/Abbruch-/Rollback-Tests auf sauberem Windows und mit existierendem Nutzerprofil nachweisbar sind. Signatur-/Keys-/Provider-/API-Stand wird vor Umsetzung technisch konkretisiert. Dokumentation alleine ist kein Test.

## Schnittstellen und Nicht-Ziele

- Q21 **Vercel-Gate** gilt exklusiv für alle online Diagnose-/KI-Hilfepfade. Q22 Update-/Release-/ModDB-Transporte bleiben separat gemäß UpdateCoordinator/Provider-Verträgen; keine unbeschlossene Umleitung durch Vercel.
- Q23 entscheidet Release-Channels, automatische vs nutzerbestätigte Updates, Beta-/Nightly-Opt-in, gestaffelte Rollouts und Update-Policy. **Q22 legt nur die zugrunde liegende sichere Distributionsarchitektur fest.**
- Kein ungefragter Cloud-Upload, kein Hersteller-Driver-Tuning, kein Game-Save-Downgrade, kein unkontrolliertes Auto-Update, kein zweiter Launcher und keine automatische Q8-OpenGL-Zero-Promotion.

## Referenzen

- [Roadmap V2.5](BISMUT_VS_CLIENT_ROADMAP_V2_5.md)
- [Q18A Editionen](ADR_BISMUT_Q18A_PRODUCT_EDITIONS_INSTALLER_2026-11-19.md)
- [Q21 Support-Gate](ADR_BISMUT_Q21_SIMPLE_SUPPORT_VERCEL_GATE_2026-11-19.md)
- Launcher Canonical: https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/BISMUT_LAUNCHER_ROADMAP.md
- Launcher Integration: https://github.com/Manus-Vindicte/Bismut-Launcher-Dev/blob/feat/publish-suite/docs/architecture/ADR-001_LAUNCHER_READY_TO_CONNECT.md
