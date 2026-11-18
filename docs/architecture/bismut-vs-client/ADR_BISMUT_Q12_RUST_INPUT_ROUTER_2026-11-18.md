# ADR BISMUT Q12 | Rust Input Router

- Datum: 2026-11-18
- Owner: Q12 Option **B**
- Status: **DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Ziel: Bismut VS Client (Rust-EXE, eigenes HWND, .NET Game Runtime in-process, Rust-natives DX12)

## Entscheidung

Rust besitzt im Bismut-Modus die Win32-Event-Pump des nativen Top-Level-HWND, die Erfassung und Sequenzierung physischer Input-Events, Focus/Capture und Cursor-Lifecycle. Ein versionierter C-ABI-/Managed-Compatibility-Adapter übersetzt die Events in die bestehende Vintage-Story-Gameplay-, GUI- und Mod-Hotkey-Semantik. Die native Bismut-UI empfängt ausschließlich ihr zugewiesene Events. **Kein zweiter GLFW-/GL-Input-Owner, keine doppelte Auslieferung.**

## Schnittstellen- und Ownership-Regeln

1. Rust Input Router als **einziger physischer Event-Owner**: event_id, monotone sequence, Timestamp, Device-ID, Event-Type, Focus-Epoch. VS bleibt Owner der Spiel- und Mod-Aktionssemantik.
2. Tastatur-Scancode und Layout/Text-Input separat führen; Raw Mouse/Relative Mode, Scroll, Button-State, Unicode/IME, Key-Repeat und Controller/Hotplug versioniert behandeln. Kein doppeltes Text-/KeyDown-Dispatch.
3. Focus-Arbiter: Modals, Bismut Shell, VS-GUI und Gameplay erhalten eine eindeutige Priorität; Fokusverlust erzeugt Releases und gibt Cursor-Lock kontrolliert frei.
4. Managed VS-Adapter nutzt ausschließlich **qualifizierte** VS-Input-Entrypoints und korrekte Thread-Affinität. Q9 Bootstrap sowie Q10 Threading sind noch offen; bis zur Evidence keine unbelegten API-/Thread-Annahmen.
5. Bismut-Mod-SDK erhält eine **versionierte Input Subscription API**, keine unkontrollierten Win32-/GLFW-Hooks als Standard. Überlast, Event-Drop, Cancel und Shutdown sind auditierbar.

## PASS-/Negativtests

| Test | Erwartung |
|---|---|
| WASD, Mouse-look, Zoom, Inventar, Mod-Hotkeys | Vanilla-Semantik, keine Doppelevents |
| VS-UI / Bismut-UI / Modals | Focus, Z-Order, Cursor-Capture, Text/IME und Key-Repeat korrekt |
| Alt-Tab, Resize, Minimize, Lost Focus, Resume | Keine klemmenden Keys/Buttons oder Pointer-Leaks |
| Controller/Mehrgerätebetrieb | Identifizierbare Events, Hotplug/Disconnect, unbekannte Unterstützung als UNKNOWN |
| Queue overflow, Version mismatch, Cancel, Shutdown | Expliziter Fehlerpfad; keine stillen Datenverluste oder UAF |
| Echte VS-Session + Mods | Nachvollziehbares Input-to-present p50/p95/p99 und Capture-Evidence |

Ein erfolgreiches Synthetic Smoke ist maximal **LAB_QUALIFIED**, kein End-to-End-PASS. **Q12-D Replay** ist eine spätere optionale Diagnosefähigkeit, **nicht** Teil des Beschlusses.

## Abhängigkeiten und Sicherheit

- Q1–Q8 gelten als Owner-Entscheidungen. Q9 Bootstrap A/B/C/D sowie Q10 Threading A/B/C/D und Q11 Memory A/B/C/D sind als **Evaluation Approved**, architektonisch aber **OPEN**.
- Q11-D Rust-Arena ist lediglich ein bevorzugter Hypothesenkandidat.
- Q7-D vollständige VS-Mod-Kompatibilität ist ein noch zu qualifizierendes V1-Ziel, keine Garantie.
- Q8-C OpenGL Zero braucht statische Inspektion, prozessweite Laufzeit-Traces, echte DX12-Frames, Mod-/Negativtests. Atlas testet nur headless Game-/Mod-Verhalten.
- Keine produktiven Saves, Auth-Daten oder fremden Prozesse verändern. Bestehenden C++-DX12-Recovery-Pfad erhalten.

## Dokumentstatus

Dieser ADR ist eine **verbindliche Architekturentscheidung**, keine Implementierungs-Evidence. Er wird im Review-Branch des zentralen Bismut-System-Repos vorgeschlagen. Der konkrete Engine-/Mod-/Launcher-Checkout und dessen Dirty State müssen vor der Umsetzung separat auditiert werden.

Referenz: [Bismut VS Client Decision Register](BISMUT_VS_CLIENT_DECISION_REGISTER_V1_4.md).
