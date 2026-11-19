# PT-ENGINE-03 | Managed Consumer für vorhandene Dynamic-Mesh-/Frame-ID-ABI

- Datum: 2026-11-19
- Owner: Bismut Engine Thread + unabhängige QA
- Status: **HANDOFF / CODE_AUTHORIZATION_REQUIRED**, keine Implementierung durch dieses Dokument
- Scope: vorhandene native C++-/DX12-Bridge, **Managed C# Consumer** in der tatsächlich zuständigen `mod-bismut-graphics`-Codebasis; kein Launcher-Code
- Grundlage: Owner-Report `PT-BISMUT-RENDER-DISPATCH-01` mit nativer ABI PASS, Live BLOCKED; `ENGINE_VS123_FOLLOWUP_V1.md` laut Report Commit `79fac8b` (lokaler tatsächlicher Worktree/Branch vor Start verifizieren)
- Git-Truth: Weder lokaler HEAD, Git-Push-Ziel, die >200 Dirty-Änderungen noch der vollständige Engine-Followup sind durch dieses Git-Dokument unabhängig geprüft.

## Was / Warum

Verbinde **ausschließlich** den Managed-Consumer mit den bereits additiv implementierten nativen Funktionen:
`set_dynamic_mesh_pass_v1` (16-B-Descriptor),
`get_dynamic_mesh_pass_caps_v1` (nur `OPAQUE` unterstützt),
`get_frame_id_observation_v1` (40-B-Snapshot).
Damit werden native Capability/Visibility und tatsächliche Frame-Beobachtung erst für den Game-/Mod-Pfad nutzbar, ohne zweite Bridge oder stillen Renderer-Ersatz.

## Wie (max. 5)

1. **Git & Sources:** lokale Engine-/Mod-Worktrees, Repo-/Branch-/Remote-Tip, AGENTS/Owner-Leases, bestehenden `PT-01`-Report und `ENGINE_HANDOFF_V1`, exakte Symbol-/Header-/C#-Signaturen, aktuell geladene DLL/Hash und alle fremden Dirty-Änderungen reconciliieren. Owner-Approval/Scope vor Änderungen bestätigen. VS123-Studie in voller Länge aus dem Client-Thread als versionierte Datei/Hash verlinken; bei fehlender Quelle diese Analyse als `SOURCE_BLOCKED` markieren.
2. **Managed ABI:** per Export `TryGetExport`, kompatible Funktionszeiger, `StructLayout(LayoutKind.Sequential)` mit überprüfbaren 16/40 B, Methoden `GetDynamicMeshPassCaps`, `SetDynamicMeshPass`, `GetFrameIdObservation`; unbekannte ABI-Versionen und reservierte Passes fail-closed. Keine Exporte erfinden.
3. **Legacy:** bei alter DLL/symbolweise fehlenden Exports `FEATURE_UNAVAILABLE`, **unveränderte OPAQUE+sichtbar-Darstellung**; kein harter Startabbruch, kein erzwungenes DX12, keine Transparent-/Shadow-Behauptung.
4. **Frame Semantik:** `frameId=0` gilt als echtes erstes Frame. Nur erfolgreich präsentierte Frames beobachten. ADVANCE/DUPLICATE/REGRESSION anhand signierter nativer Observation-Counters behandeln; Pose-Refresh/Discard strikt aus den geklärten Ownership-/Frame-Gates, kein duplicate Upload oder versehentlicher Leak.
5. **QA/Tests:** native 10/10 als Baseline **neu verifizieren**, C#-Tests für Export vorhanden/fehlend/mixed, Layout, Caps, Visibility/Defaults, Frame=0, Duplicate, Regression und Safe Shutdown. Packaging-DLL-Hash auf Source/Build/Deploy abstimmen. Live-Spiel, Defekt E und AV-06/07 nur mit getrennter Runtime-/Manual-Authorization.

## Erwartung

- `ENGINE_MANAGED_ABI_CONSUMER_V1` + Implementierung in bestehendem Mod-Owner, wenn freigegeben.
- Reale Unit-/Integration-Receipts: Command, Commit, C#/.NET-Build, DLL-SHA, loaded DLL, eindeutig zugeordnete Run-/Frame-/Backend-ID.
- Native `PASS` separat von Managed `PASS` und Game-Live `BLOCKED/PARTIAL/PASS`.
- Unabhängiges QA-Review, keine fälschliche OpenGL-Zero- oder Avatar-Total-PASS-Promotion.
- `ENGINE_HANDOFF_V1` nur evidence-basiert aktualisieren.

## Problembehandlung / Nicht tun

Bei fehlendem lokalen Report, fehlender Freigabe, unklarer ABI-Ownership oder fremdem Dirty-State **STOP und BLOCKED** mit präzisem Grund. Keine zweite Bridge, kein Render-Pass-Neubau, keine ABI-Bit-/Enum-Erweiterung, kein Rust-Komplettrewrite, keine Produktivwelt, kein eigenmächtiger Push/Merge.

**Exit:** `MANAGED_ABI_PASS` nur nach tatsächlichen C#-Tests und Callback-/Ownership-Regressions; `LIVE_GAME_PASS` benötigt zusätzlich eine gesondert autorisierte reale Session.
