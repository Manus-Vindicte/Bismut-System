# PT-BISMUT-VS123-NEXT-01 | Post-Study Owner Gates + zwei Agent-Threads

- Datum: 2026-11-19
- Status: **OWNER_DECISIONS_PROPOSED / NOT_RATIFIED / CODE_NOT_AUTHORIZED**
- Eingabe: **vom Owner im Chat übermittelter Statusbericht** zur lokal materialisierten VS-1.23.0-pre.1-Studie, nicht die fünf vollständigen Berichtsdateien. Der genaue lokale Commit `c12c0df3` ist nach Owner-Bericht noch nicht gepusht; vollständige Report-Artefakte konnten hier **nicht geöffnet** werden.
- Offizielle Quelle: https://www.vintagestory.at/blog.html/news/v1230-pre1-the-combat-nature-and-fancy-physics-update-r450/
- **Wahrheitsgrenze:** Im Chat gemeldete 20+9 Engine-Einträge, 24 Client-Einträge, QA `ACCEPT_WITH_CORRECTIONS`, 5 Korrekturen sind **OWNER_REPORTED**, nicht durch dieses Git-Dokument unabhängig nachgeprüft.
- Bestehende Entscheidungen: Q23 **Stable-only**, Q25 Launcher-Startseite behalten. Engine-ABI gemäß jüngstem ENGINE-Report **native PASS / Managed + Live BLOCKED**; Client ADR-006 **PROPOSED / Astra REJECT**. Kein automatisches Freigeben offener Owner-Gates.

## 0. Einstieg: erst die richtige lokale Evidenz lesen

Jeder Thread öffnet und zitiert die fünf **vollständigen lokalen Studienartefakte** statt nur diese Chat-Zusammenfassung:

```text
docs/development/reports/PT-BISMUT-VS123-STUDY-01/
  VS123_PRE1_IMPACT_STUDY_RESULT.md
  ENGINE_VS123_IMPACT_REPORT.md
  CLIENT_VS123_IMPACT_REPORT.md
  QA_VS123_REVIEW.md
  DELEGATION_EVIDENCE.md
```

Dazu die lokalen Engine-Reports zu `PT-BISMUT-RENDER-DISPATCH-01` (Native ABI/ENGINE_HANDOFF_V1), Client-Reports zu `PT-BL-CLIENT-02` (Astra F1–F8, ADR-006, D6) und alle zugehörigen HEAD-/Branch-/Remote-/Dirty-State-Receipts. Reports ohne belegte Pfade/Commits dürfen nicht als Canon/Runtime-PASS akzeptiert werden. Keine Aktualisierung historischer Reports durch Übernahme bloßer Summary-Aussagen.

## 1. Fünf Owner-Entscheidungen, PROPOSAL ONLY

| ID | Freigabevorschlag | Ohne Owner-Freigabe |
|---|---|---|
| **OD-01 PRE_RELEASE** | `1.23.0-pre.1` und alle VS-Prereleases als `LAB_ONLY/UNVERIFIED`, **nicht** wegen semver `[1,23,0] >= [1,22,0]` als Stable-kompatibel zulassen. `preReleaseOf()`-Semantik anhand tatsächlichen Parsers und Tests konsolidieren. | Kein produktiver Versionsparser-/Kompatibilitäts-Contract-Change |
| **OD-02 LAB_DIFF** | Read-only 1.22↔1.23-pre1 Assembly-/Asset-/API-Diff freigeben; mögliche Live-Tests nur **separate Pre1-Installation + unabhängiger DataRoot + kopierte Testwelt** und konkrete Test-Authorization/Lease. | Keine Downloads, Updates, produktive Welten, Auth-Kopien oder Live-Spielstarts |
| **OD-03 RENDER_ABI** | **Vorhandene** owner-freigegebene native `set_dynamic_mesh_pass_v1` / `get_dynamic_mesh_pass_caps_v1` / `get_frame_id_observation_v1` als Baseline prüfen und nur fehlenden **Managed Consumer** planen; keine neue `passMask/FrameId`-Bridge. Weitergehende TRANSPARENT/SHADOW-/Frame-Consumer-Vertragsänderungen benötigen eigenen ADR/GO. | Kein ABI-Rewrite, keine TRANSPARENT/SHADOW-Aktivierung |
| **OD-04 ADR006** | Client-ADR-006 D-A bis D-D **zur finalen Owner-Abnahme vorbereiten**, aber `PROPOSED` lassen, bis F1/F2 (Bridge-Replay/Pending) unabhängig dynamisch geprüft sind; Vorschlag: nach Reconnect **erneute Startbestätigung**, Pending `worldId`-/Intent-ID-gebunden + gültiger TTL. | Keine neue produktive Start-Authority oder Lifecycle-Promotion |
| **OD-05 STABLE** | Q23 unverändert bindend: öffentliche Release-/Update-Manifeste bleiben **stable-only**; Pre1 ausschließlich Lab und Opt-in, keine Auto-Migration. | Keine Pre1-Promotion und kein Update aktiver Nutzerprofile |

**Wichtig:** OD-03 ersetzt die ältere Studienformulierung „kein passMask/FrameId-Mechanismus“ nicht stillschweigend. Reconcile *welcher* Mechanismus existiert: (a) native Export/Capability/Frame observation bereits getestet vs. (b) Managed-/Game-Consumer/Live-Frame-Identity noch fehlend. Beide Aussagen können für unterschiedliche Schichten gleichzeitig zutreffen.

## 2. PT-ENGINE-VS123-02 · Engine Thread

**Was:** Pre1-Animation/Physics/Rendering-Impact gegen die **heutige** DX12-Bridge und native PASS-ABI verifizieren; managed und live weiter **BLOCKED** bis autorisiert.

**Wie:**
1. Release-/API-Abschnitte einschließlich `API Changes` mit lokalen Engine- und Mod-Reports abgleichen; Keyframe-Größen **248→88 bzw. 560→184**, Bezier/Step, `allowSSBOs`, Collision-/Geometry-/Atlas-/Equipment-Änderungen auf tatsächliche Quell-/Assembly-Eigenschaften mappen.
2. Native ABI `set_dynamic_mesh_pass_v1` (16 B), `get_dynamic_mesh_pass_caps_v1` (OPAQUE only) und `get_frame_id_observation_v1` (40 B) gegen aktuelle Engine-Commits und `ENGINE_HANDOFF_V1` prüfen. `frameId=0` gültig, reservierte TRANSPARENT/SHADOW unverändert `-1` ohne Mutation.
3. Berichtete Native Regression 10/10, Scene `staged=25 presented=25 gl=ABSENT` und konkrete DLL-Hashes von **Live-Game-Receipts** unterscheiden: isolierte Native Scene ≠ VS-Live/GL-Zero-PASS. Gleich nummerierte Run-Serien anhand Run-ID/Session/BuildHash/Backend trennen.
4. Managed-Consumer-`TryGetExport`-Patch + Legacy-Fallback (opaque/sichtbar) als **konkreten kleinen Change-Plan** inklusive Tests materialisieren. Code erst nach OD-03/Owner-Gate; Live-Run erst nach separater Runtime Authorization.
5. Unabhängige QA: Native/Managed Contract und VS-1.23-Risiken prüfen, mit Evidence pro Matrixzelle.

**Ergebnis:** `ENGINE_VS123_FOLLOWUP_V1`, ABI/managed/live Matrix, Patch-Plan, sichere Smoke-Tests und präzise Owner-/Runtime-Gates.

## 3. PT-CLIENT-VS123-02 · Client Thread

**Was:** Versionsparser-Fehlklassifikation und ADR-006-Blocker beseitigen **als überprüfbaren Plan**, bestehende Launcher-Shell nicht neu bauen.

**Wie:**
1. Lokalen Client-Code prüfen: `preReleaseOf()` ist laut Studie implementiert, aber im Compatibility-Pfad unbenutzt. Versionen `1.22.x`, `1.23.0-pre.1`, `1.23.0-rc.*`, `1.23.0` und ungültige Labels als Fixture-Matrix ableiten. Keine Stable-Einstufung nur aus `[1,23,0]`.
2. API-/Mod-Änderungen inklusive Renames, Signaturen, Entity-JSON-/Side-Loading-Diff für `ModLock`, `LaunchPlan`, Server-Join, Provider-Versionen, Import und WorldBinding klassifizieren. Bei UNKNOWN fail closed; keine produktiven Welten öffnen.
3. Den tatsächlichen `PT-BL-CLIENT-02`-Stand (`074c1874` laut Report, Remote `4ffecce2` vor weiterem Push) mit Remote/local abgleichen. Astra F1–F8 nachprüfen, besonders `maybeReplayPendingWorldStart()` nur an init und Async-Mount-Replay; D6 ist nicht unabhängig funktional PASS.
4. ADR-006 D-A..D-D als **Owner-Entscheidungsvorlage** mit Environment-Veto, intentionalem Fallback, `RunRecord`-ID statt Plan-ID, einem Startautoritäts-Owner und Fake-Host-Dynamic-Testplan vorlegen. Keine Implementation ohne OD-04/Owner-Gate.
5. Unabhängige QA für Versionsparser-Fail-Close und Replay/Pending-Race-Testfälle festlegen; vorhandene Launcher-Home-/SP-/MP-UI nicht neu bauen.

**Ergebnis:** `CLIENT_VS123_FOLLOWUP_V1`, Version Compatibility Matrix, ADR-006-/D6-Disposition, Testplan und Owner-/Lab-Gates.

## 4. Lead/QA Integration und Exit

- **Lead-Owner:** keine dritte Produktimplementierung; begrenzte Koordination, Git-/Source-Reconciliation, Entscheidungsvorlage und Contract-Handoffs. Reale Engine-/Client-Threads bleiben getrennt.
- **Independent QA:** verifiziert alle fünf Owner-Gates gegen tatsächliche lokalen Dateien; prüft Klassifikation `IMPLEMENTED` ≠ `TESTED` ≠ `NATIVE_PASS` ≠ `LIVE_PASS` ≠ `OWNER_APPROVED`.
- **Status:** `SOURCE_CONFIRMED` (Artikel verfügbar) und `STUDY_RESEARCH_DONE` laut übermitteltem Report; `IMPACT_PARTIAL` und `TEST_PASS_BLOCKED`. Dieses Repo-Dokument behauptet keine eigenständig wiederholten Tests/Downloads.
- **Next:** Owner entscheidet OD-01..05, dann ggf. Änderungs-/Live-Authorization. Kein automatischer Push/Merge, keine öffentliche Pre1-Version und kein umgehendes Live-Game-Testfenster.

## Referenzen

- [Initial Pre1 Impact Study](PT_VS_1_23_0_PRE1_ENGINE_CLIENT_IMPACT_STUDY_2026-11-19.md)
- [Master Goal Engine/Client](MASTER_GOAL_BISMUT_ENGINE_CLIENT_TWO_SLICES_2026-11-19.md)
- [Q23 Stable-only](ADR_BISMUT_Q23_STABLE_ONLY_MANUAL_UPDATES_2026-11-19.md)
- [Q25 Existing Launcher + Client](ADR_BISMUT_Q25_EXISTING_LAUNCHER_CLIENT_SESSION_2026-11-19.md)
