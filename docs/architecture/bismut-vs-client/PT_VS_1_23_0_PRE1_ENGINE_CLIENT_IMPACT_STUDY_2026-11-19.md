# PT · Vintage Story 1.23.0-pre.1 | Engine + Client Impact Study

- Studie gestartet: 2026-11-19
- Status: **PRELIMINARY / RELEASE_ARTICLE_NOT_RETRIEVED / NO_CODE_CHANGE**
- Owner: Bismut Engine Thread + Bismut Client Thread; getrennte Findings, eine gemeinsame QA-/Lead-Abnahme
- Originalquelle: https://www.vintagestory.at/blog.html/news/v1230-pre1-the-combat-nature-and-fancy-physics-update-r450/
- **Hard source gate:** Die obige Pre.1-Release-Seite ließ sich am 2026-11-19 über verfügbare Retrieval-Zugriffe nicht direkt öffnen, war in der aktuellen Suchindizierung noch nicht abrufbar. **Keine spezifische Pre.1-Änderung darf als gelesen oder bestätigt behauptet werden, bevor der Artikel tatsächlich gesichert und vollständig gelesen ist.**
- Produktpolicy Q23: **einzig öffentlich Stable**, Preview nur als **isolierte interne Labor-Fassung**, nie stilles Upgrade von Nutzerwelten.
- Keine vorhandenen lokalen Repos/Tests wurden durch das Erstellen dieser Studie verändert.

## Ziel

Die neue Vintage-Story-Version anhand **der unveränderten Original-Release-Notes** gegen die vorhandenen Bismut-Engine-/Launcher-/Client-Verträge prüfen. **Nicht sofort migrieren.** Ermitteln: API-/ABI-/Runtime-Breakages, neue Render-/Animation-/Physics-Produzenten und Release-/Mod-/World-Kompatibilitätsrisiken. Team liefert einen **Evidence-gestützten Kompatibilitätsbericht**, nicht einen unqualifizierten PASS.

## Quelllage, strikt trennen

### Exakte Pre.1-Quelle (noch nicht inspiziert)
- Titel gemäß Nutzerlink: „v1.23.0-pre.1 - The Combat, Nature and Fancy Physics Update“
- Vollständiger Changelog, genaue Änderungen, mögliche API-Breaking-Changes, Downloads/Hashes und Kompatibilitätslisten: **UNKNOWN / SOURCE_PENDING**, nicht aus Überschrift ableiten.

### Bereits bekannte, frühere 1.23-Entwicklungsaussagen (Hintergrund, KEINE Pre.1-Bestätigung)
- Offizielles Anego-Studios-Update vom 2026-06-16: Kampftreffer, Animationsvarianten, Waffen-/Rüstungsarbeit; dreiteiliges Rüstungsslotsystem mit mehrlagigen Bestandteilen; Verbesserungen an Mod-Support und weitere geplante Features. Der Artikel sagt ausdrücklich, dass nicht alles zwingend in 1.23 gelangt: https://www.vintagestory.at/blog.html/news/anego-studios-development-update-june-2026-r446/
- Entwickler-Fortschrittsbericht 2026-10-12 (reproduziert auf der unabhängigen `vintagestory.info`-Seite): physikalische Melee-Trefferprüfung, animation-authoritative Hit-Detection, Waffen-Swing-Animation, unterschiedliche Schadenstypen/Knockback, Rüstung. Optionaler alter Combat-Modus als Entwicklungsplan, **kein bestätigter Pre.1-Inhalt**: https://vintagestory.info/en/news/combat-update-1-23-development-progress/
- VS Wiki zum Mod-Update-Zyklus: Previews können weiterhin API-Änderungen enthalten; Release Candidates sollen API stabilisieren; Stable nur in Ausnahmen API ändern. Wiki-Seite war zuletzt für 1.19.8 verifiziert, also als Prozesshinweis, nicht aktuelle API-Truth nutzen: https://wiki.vintagestory.at/Modding:Mod-Engine_Compatibility
- API-Update-Index enthält im abgefragten Stand 1.22-Inhalte; 1.23-API-Diff ist **nicht** daraus ableitbar: https://wiki.vintagestory.at/Modding:Modding_API_Updates

## Start-Gates vor dem Vergleich

1. **SOURCE GATE:** Release-Artikel vollständig sichern (Originaltext, Veröffentlichungsdatum/Version, URL, ggf. verlinkte offizielle Patch-/API-Notes). Jede Änderung als Originalzitat-/Abschnittsreferenz klassifizieren, einschließlich `API`, `Render`, `Animation`, `Physics`, `World`, `Networking`, `Modding`, `UI`, `Install`.
2. **BASELINE GATE:** Echte lokale und gepushte Engine-/Launcher-Repo-Branches/Commits **nach dem neuen Push** reconciliieren. Das im Bericht genannte Engine `Run 16/17/19` ist nicht automatisch dieselbe Testserie wie der spätere Audit-Report mit gleich nummerierten Runs. Pro Evidence **Run-ID, Datum, Repo, Commit, Backend, DLL-SHA** vermerken. `BismutGraphics` stale-DLL-Falle vermeiden.
3. **ENVIRONMENT GATE:** Vintage Story 1.22.x als qualifizierte Referenz **unverändert**. 1.23.0-pre.1 nur in **separater, explizit freigegebener Testinstallation mit kopierten Testwelten**, neutralen und modded Baselines; keine produktiven Saves, Accounts, Profile oder Modpacks migrieren. Kein Preview im Q23-Stable-Updater.
4. **TEST GATE:** Keine neuen API-/Physik-/Combat-/Render-Annahmen aus Titeln oder älteren Roadmaps in Code übernehmen. Erst offizielle Release-Notizen und echte Assembly-/Asset-/Mod-Live-Inspektion, dann Verträge/ADRs/Owner Approval.

## ENGINE THREAD · Study Scope

| Prio | Hypothese / Prüffrage, KEIN Pre.1-Fakt | Evidence / Regression |
|---|---|---|
| P0 | Entity-/Player-/Equipment-/Armor-Modelle: ändern sich Shape-/Atlas-/Joint-/Pose-/Attachment-Verträge? | Asset-/Assembly-Diff 1.22.x→1.23.0-pre.1; Avatar Run 16/17/19 als korrekt versionierte Baseline, AV-06/07 realer Third-Person-Readback |
| P0 | Animation-authoritative Combat: welche per-frame Pose-/Weapon-/Hit-/Model-Matrix-Daten sind im Renderpfad sichtbar? | Source/API-Diff; Render-Gesamtpass ohne neues Hit-Detection-Authority-Modell |
| P0 | Neue/andere Physics-/Collider- und bewegte Objekte? | Nur tatsächlich in Pre.1-Diff bestätigte Featurefälle in Frame/Visibility/Pass-/Command-List-/Transform-Matrix aufnehmen |
| P1 | OpenGL-/Shader-/Render-API oder GPU-Resources geändert? | Q8 OpenGL-Zero-Evidence getrennt; existing Dynamic-Mesh/PassMask/FrameId-Bridge, Frame-/SRV-/DLL-Hash/PSO-Negativtests |
| P1 | Shutdown/GUI/Animation/Effects/Particles regressieren? | Defekt E weiterhin MANUAL_ESCAPE_REQUIRED; Runtime/Watchdog/Save-/Host-Lifecycle; Effects-Produzenten, Alpha, Equipment und Entity Shadows als separate Gates |

**Keine neue Avatar-/Skinning-Bridge und kein Rust-DX12-Rewrite.** Bestehende C++-/DX12-Referenz mit belegten Avatar-Features bleibt Baseline. Q5/Q6 Rust-Zielarchitektur separat; keine ABI-Änderung ohne neue ADR + Owner-Approval.

## CLIENT THREAD · Study Scope

| Prio | Frage | Evidence / Regression |
|---|---|---|
| P0 | 1.23.0-pre.1 als eigene, **nicht öffentliche** Version erkennen und **vom Stable-Hauptpfad isolieren** | Read-only Installer/Game-Version-Discovery, separate DataRoot/Environment; Q23 Stable-only Updater unverändert |
| P0 | VS Game-Version → LaunchPlan/WorldBinding/RunRecord/ModLock und Server Join Constraints | Neue Pre.1 als `UNVERIFIED / LAB_ONLY`; kein automatisches World-/Mod-/Save-Upgrade |
| P0 | Welche offiziellen API-/Mod-Schema-/Dependency-Diffs brechen bestehende VS-Mods? | Release-/Assembly-API-Diff, BCIL UNKNOWN bis Mod-Klassen live getestet |
| P1 | Q21 Crash-Panel / Diagnosedaten / Versioned Error-Codes | Lokales Hilfepanel zuerst; optionaler Bericht ausschließlich nach Einwilligung über Vercel |
| P1 | Update-/ModDB-/Publishing-/Server-Feed-Version-Handling | Kein neuer Version-Parser ohne Compat-Tests; keine 1.23-Pre-Promotion zum Stable-Channel |

**Existierende Singleplayer/Multiplayer Launcher-UI bleibt Owner**. Keine neue Welt-/Serveransicht und kein zweiter RuntimeSupervisor.

## Gemeinsames Ergebnisformat

- `VS_1_23_PRE1_CHANGE_REGISTER`: jede tatsächlich bestätigte Release-Änderung mit Quellabschnitt, betroffene Datei/API/Vertrag, Prio und `CONFIRMED / SUSPECTED / UNKNOWN`.
- `ENGINE_DIFF_REPORT`: relevant vs no impact, Reproduktion, Test-Baselines, Build-/Runtime-/AV-/Renderer-Evidence.
- `CLIENT_DIFF_REPORT`: Game-/Launcher-/Mod-/World-/Server-Kompatibilität, Stable-only Gate, Isolation-/Rollback-Evidence.
- `OWNER_DECISION_QUEUE`: höchstens fünf nötige ADR-/Upgrade-Entscheidungen, keine Vorab-Freigabe.
- `FINAL_STUDY_STATUS`: `RESEARCH_DONE` erst nach verifizierten Original-Release-Notes; `TEST_PASS` nur nach echten qualifizierten Tests. **Kein Push/Merge auf Produktivbranches und kein automatisch ausgeführtes Update.**

## Nicht tun

- Pre-Release nicht in den öffentlichen Stable-Kanal schieben.
- Keine Originalwelten modifizieren, keine Auth-/Credential-Dateien kopieren.
- Keine Architektur-Duplikate (Launcher-/ModHub-/DX12-/Avatar-Bridge).
- Keine Änderungen an Produktivcode ohne separate Änderungsfreigabe.
- Kein behauptetes „1.23 Pre1 API/Render kompatibel“ ohne offizielle Notes plus Build/Runtime/Mod-Evidence.
