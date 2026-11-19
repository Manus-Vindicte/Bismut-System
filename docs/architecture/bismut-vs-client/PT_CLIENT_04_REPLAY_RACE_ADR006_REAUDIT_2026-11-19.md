# PT-CLIENT-04 | D6 Replay Race-Safety, Pre-Release Gate E2E und ADR-006 Reaudit

- Datum: 2026-11-19
- Owner: Bismut Client Thread + **unabhängige, dynamisch testende QA**
- Status: **HANDOFF / SCOPE_GO_FOR_D6_TESTS_AND_RACE_FIX / ADR006_OWNER_GATE_CLOSED**
- Code-Repo: `Manus-Vindicte/Bismut-Launcher-Dev`
- Remote zum Reconciliieren: `feat/modpack-scope-binding` @ `967c28560aaa041806dfddc0341e38e2d71ee18c`; separater Audit-Branch `lab/pt-bl-lab-04-vs-auth-20261114` @ `fd8be85c0d6a90502c68aeca91efa4bc73b76d52`.
- Nach Live-Code-Lesung: `electron/modpack/gameVersionSemantics.cjs` hat bereits `unverified_lab_only`-Klassifizierung, `frontend/src/services/MockReadModelProvider.jsx` enthält bereits `pending_confirmation`, `intentId`, `worldId`, `expiresAt` und `confirmPendingWorldStart()`. **Nicht PT-CLIENT-03 nochmals bauen.**
- Eingabe: `CLIENT_VS123_FOLLOWUP_V1` + ADR-006 + PT-CLIENT-02 Astra F1/F2 (unabhängiger dynamischer Recheck bislang offen).

## Was / Warum

Prüfe und schließe die tatsächlichen **Verhaltenslücken** des bereits gepushten PT-CLIENT-03, besonders Pending-Intent-Races, stale intent resurrection, Doppelstarts, Provider-/World-Mismatch und Client-Vertrag/Version-Gates. Die vorhandene Launcher-/Singleplayer-/Multiplayer-UI bleibt.

## Wie (max. 5)

1. **Reconciliation & Scope:** lokaler Branch, Remote-HEAD, Dirty State, D6-Fixes und Produkt-Callsites von `evaluateGameVersion` prüfen. Nur fehlende Verhaltens- und Integrationsgates anfassen. ADR-006 D-A..D-D bleibt `PROPOSED`, bis Owner es ausdrücklich genehmigt.
2. **Dynamische Fake-Host-Suite:** T1 Reconnect ohne Autostart; T2 Start nach Bestätigung exakt 1× mit gebundener World-ID; T3 zwei offline Intents, nur neuester; T4 blocked/expired no infinite retry; T5 Mock↔Real-Provider-Wechsel ohne Cross-Instance-Start; T6 `1.23.0-pre.1 >=1.22.0` ⇒ `unverified_lab_only`. Teste auch zwei gleichzeitige Confirm-Calls, bestätigter Start während neuerem Pending und verspätete Fehler-/Success-Callbacks.
3. **Konkrete Async-Race prüfen:** `maybeReplayPendingWorldStart()` setzt `this.pendingWorldStart=null` vor `await this.real.dispatch(...)`, stellt bei Block/Exception aber alte Pending-Daten wieder her. Ein späterer neuer Intent darf **nie** durch einen älteren Request überschrieben werden. Fix mit Intent-ID/Generation/CAS-Guard und single-flight-/idempotency-Regel **im hier freigegebenen engen D6-Sicherheits-Scope**; Änderungen der produktiven Startautorität brauchen weiterhin ADR-006/Owner-GO; Tests vor Fix rot, danach grün dokumentieren. Keine silent retries.
4. **Version Gate end-to-end:** `preReleaseOf` ist **bereits konsumiert** in `evaluateGameVersion()`; verifiziere aber die tatsächlichen Mod-/Launcher-/Update-/Join-Consumers und `unverified_lab_only`-UI-Semantik. Harte Inkompatibilität bleibt inkompatibel, Pre1 muss in Q23 stable-only Pipeline `LAB_ONLY` bleiben. Keine VS-Pre1-Testinstallation oder Save-Migration ohne separates Lab-GO.
5. **QA/Review:** npm Build + echte Semantik-/Join-/Vanilla-/Fake-Host-Tests ausführen; genau benannte Quell-Tests mit `node:test`/existenem Harness statt nur `source.includes()`-Stringchecks. Eine unabhängige QA-Session reproduziert Race/F1/F2 und bewertet Astra `REJECT` neu; erst danach ADR-006 zur Owner-Abnahme vorlegen.

## Erwartung

- `CLIENT_REPLAY_BEHAVIOR_V1`: echte T1–T6 + Races mit reproduzierbarem Fake-Host, clear expected/actual, exact Exit/Counts.
- `CLIENT_VERSION_CONSUMER_AUDIT`: callsite map und Pre-Release-/ModLock-/stable-only Fail-Close-Evidence.
- `ADR006_OWNER_DECISION_PACKET`: F1/F2 Disposition, Environment-Veto, LaunchPlan↔RunRecord-ID, Ready-/Exit-/Crash-Testcases, Gate-Status und getrennte Produkt-/Lab-Scopes.
- Genau **eine** kanonische Startautorität als Ziel, nicht stillschweigend implementieren.
- Unabhängiges QA-Verifikationsreceipt; `LAUNCHER_READY_TO_CONNECT` nur auf Basis der tatsächlich aktuellen PT-BL-010-Evidence und keines alten Chat-Status.

## Problembehandlung / Nicht tun

Bei F1/F2 weiterhin rot: `BLOCKED_ASTRA_REJECT` und minimale reproduzierbare Ursache dokumentieren. Keine neue Launcher-UI, keine zweite Start-Authority, kein nicht autorisierter ADR-006-Rollout, keine Pre1-Stable-Promotion, keine fremden Dirty-Änderungen, kein eigenmächtiger Push/Merge.

**Exit:** `D6_DYNAMIC_BEHAVIOR_PASS` nur nach Testausführung + unabhängigem Reaudit; komplette ClientSession/LaunchPlan/Return-to-Launcher-Integration bleibt eigener ADR-006-Owner-Gate.
