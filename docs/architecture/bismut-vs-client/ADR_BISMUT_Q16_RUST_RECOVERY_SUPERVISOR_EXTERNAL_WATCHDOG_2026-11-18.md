# ADR BISMUT Q16 | Rust Recovery Supervisor + externer Watchdog

- Datum: 2026-11-18
- Owner-Beschluss: **Q16 Option B + ergänzendes D**
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Ziel: **Bismut VS Client** mit Rust-EXE, in-process Vintage-Story-.NET-Game-Runtime und langfristig Rust-nativem DX12
- Vorangegangene Wahlmöglichkeiten: B = Rust Recovery Supervisor mit Recovery State Machine; D = externer Watchdog/Helper für Prozessabsturzdiagnose und kontrollierten Neustart

## Entscheidung

Bismut verwendet **zwei getrennte Recovery-Ebenen**:

1. **Rust Recovery Supervisor (im Clientprozess)** klassifiziert und behandelt nachweislich kontrollierbare Fehler in Bismut Loader, Game-/Managed-Adapter, Shader-/Asset-Pipeline, Input und D3D12-Renderpfad. Er orchestriert Degradation, Quiesce, gezielte Neuinitialisierung **nur bei validierter Ownership/Lifetime**, kontrolliertes Beenden und die Übergabe von Recovery-Metadaten an den Launcher.
2. **Externer Watchdog (separater minimaler Helper-Prozess)** überwacht Client-Prozessstatus und ausdrücklich definierte Heartbeat-/Readiness-Gates; er sichert Crash-/Hang-Evidence, kann eine Folgestart-Quarantäne anfordern und über den **Bismut Launcher** einen kontrollierten Neustart oder Vanilla-Fallback anbieten. Er ist **kein zweiter Game-Runtime-/DX12-Owner**.

**Nicht beschlossen:** Aus einem Crash automatisch ohne Nutzer-/Policy-Freigabe neu starten; den Game-Prozess per Watchdog zwangsweise fortsetzen; eigene Vintage-Story-Server-/Gameplay-Prozesse für das normale Bismut-Rendering schaffen; beliebigen nativen Mod-Code sicher isolieren.

## Failure Classes und Reaktion

| Klasse | Erkennung | Erlaubte Reaktion | Verbotene Behauptung |
|---|---|---|---|
| Mod-/Loader-Preflight / Dependency / ABI | Manifest-/Capability-/Loader-Prüfung | Mod blockieren, Diagnose, nächste Sitzung gezielt quarantänisieren, Profile nicht zerstören | „Alle Mods kompatibel“ |
| Kontrollierte Managed Exception | Qualifizierter .NET-Adapter, Lifecycle-Kontext | Fehler protokollieren, sichere Transition/Exit, nur geprüfte Reinitialisierung | Wiederanlauf aller VS-Subsysteme ohne Beweis |
| Recoverable Shader/Asset/PSO Error | Vertragliche Pipeline-Fehler | Letzten verifizierten Zustand behalten, betroffene Ressource rollback, Fehleroberfläche | Ungeprüften Hot-Reload oder GPU-Reuse als sicher bezeichnen |
| DXGI_ERROR_DEVICE_REMOVED / RESET / HUNG, Fence-Timeout | D3D12 HRESULT + Device Removed Reason, Traces | GPU-Zugriff stoppen, Ressourcen/Fences quiescen; nur nach nachgewiesener Null-Lifetime reinitialisieren, sonst kontrollierter Exit | Device Lost stets in-process reparierbar |
| Client Crash / UAF / Memory Corruption | Watchdog Prozessstatus, Crash-/Exception-Receipt | Postmortem Evidence, Launcher-Recovery, Profil-/Mod-Quarantäne für Folgestart | Sicherer Recovery im beschädigten Prozess |
| Client Hang / Heartbeat Stale | Watchdog + echte Fortschrittsindikatoren | Hang diagnostizieren, Nutzer-/Policy-gesteuerte Eskalation; keine blinde Terminierung | Stale Heartbeat = sicherer Deadlock |

## State Machine

- **BOOT_PREFLIGHT**: Version-/Hash-/Mod-/Adapter-/Launcher-Pins; Discovery, kein Save-Mutator.
- **RUNNING**: qualifizierter Client-/Game-/GPU-Pfad, Heartbeat mit Fortschrittsphase und Run-ID.
- **DEGRADED**: Fehler klassifiziert, nicht betroffene Subsysteme bleiben nur bei gültigem Vertrag verfügbar.
- **QUIESCING**: keine neuen GPU-/Mod-/Input-Arbeiten; in-flight Frame-/Resource-Handles und Fences sammeln.
- **RECOVERING**: ausschließlich freigegebene, bezeugt zerstörbare Komponenten und versionierte Adapter neu initialisieren; Recovery-Budget und begrenzte Retry-Zahl.
- **RESTART_REQUIRED**: wohldefinierter Exit, Launcher bietet sichere Profilwahl, letzten qualifizierten Adapter/Vanilla und optionale Mod-Quarantäne an.
- **CRASHED/UNRESPONSIVE**: Watchdog postmortem/hang capture, nicht In-process-Fortsetzung.
- **STOPPED**: geordnetes Herunterfahren, Flush nur auf gültiger Spiel-/Save-Ownership.

Übergänge sind protokollierte, versionierte Events mit run_id, profile_id (nicht Geheimnisse), VS-/Client-/Adapter-/Mod-Hashes, Fehlerklasse, GPU-Status, letzter Frame/Tick, Transition, Quelle und Evidence-Pfad. Keine unkontrollierten Restart-Loops; der Watchdog darf keine fremden Prozesse beenden.

## Safety / Ownership

- **Save-Safety:** keine Reparatur produktiver Saves/Profile als implizite Recovery; nur isolierte, kopierte Testwelten für Fault Injection. Recovery darf keine offenen World-Datenbanktransaktionen stillschweigend „reparieren“. Save-/Game-Lifecycle verbleibt bei der qualifizierten VS-Runtime.
- **GPU-Safety:** Ein D3D12-Device/Present-Owner; Generation Handles, Resource Queue und Fence-Retirement aus Q2/Q14. Nach Device Lost alle GPU-Handles als ungültig behandeln, niemals frei gewordene Ressourcen in alten Frames wiederverwenden.
- **Mod-Safety:** Q15 Loader erlaubt Preflight und Quarantäne auf Basis konkreter Belege. Ein In-process Rust/C++-Plugin mit UAF/Out-of-Bounds kann den gesamten Prozess kompromittieren. Panic Catch / Manifest / Capability Claims sind keine OS-Sandbox.
- **Watchdog-Safety:** minimaler Helper mit eigener Prozess-/Heartbeat-Identität, Timeouts und Consent-/Policy-Gates; Crash Dump sanitizen (keine Auth-Tokens, private Save-Inhalte oder Secrets ins Git); Evidence lokal mit begrenzter Retention. Keine eigenmächtigen Mod-Deinstallationen.
- **Fallback:** Vanilla OpenGL bleibt als **separater** qualifizierter Startmodus über den Launcher verfügbar, kein stiller Wechsel mitten im DX12-Frame. Q8 OpenGL-Zero-PASS darf nicht mit Vanilla-Fallback-Frames vermischt werden.
- **Open Dependencies:** Q9 Bootstrap, Q10 Threading, Q11 CPU Memory, Q13 Shader, Q14 GPU Resource Architecture sind nicht entschieden; konkrete Recovery-/Restart-Verträge werden gegen ihre Evidence abgeleitet.

## Qualifikation und Fault-Injection

1. **Preflight:** ungültiges Modmanifest, ABI mismatch, Dependency Cycle, fehlende VS-Adapter-Version, defekte Shader-/Asset-Daten; fehlerklassifizierter Stopp ohne beschädigte Profile.
2. **Render:** bewusst provoziertes GPU-Failure (kontrolliert, keine Gefährdung anderer Prozesse), Device Removed/Reset, Fence-/Descriptor-/Resource-Lifetime-Fehler; gemessener Quiesce/Exit, korrekte Handle-Invalidierung.
3. **Runtime:** gezielt ausgelöste Managed Exception und Loader-Callback-Fehler; kein unqualifizierter Thread-Wechsel und kein falscher Game-World-PASS.
4. **Watchdog:** Child-Prozess testweise beenden/abstürzen lassen, Heartbeat aussetzen, Watchdog selbst beenden, Launcher nicht verfügbar; keine Endlosschleifen oder unbeabsichtigtes Kill anderer Prozesse.
5. **Recovery-Evidence:** State-Transition-Logs, Run-/Build-/VS-/Mod-Hashes, Exit Code, begrenzte Retry-/Timeout-Policy, Crash-/Hang-Receipts, validierte Wiederaufnahme bzw. explizites FAIL/BLOCKED. Atlas-Server-Evidence ist nicht gleich DX12-Client-Evidence.

**Akzeptanz:** isolierte native Lab- und echte VS-Live-Fault-Injection getrennt bewerten. Bis zu tatsächlich ausgeführten Tests bleibt **IMPLEMENTATION_NOT_ATTEMPTED**. Q7 vollständige Mod-Kompatibilität und Q8-C OpenGL Zero bleiben eigenständige, nicht automatisch erfüllte Produktgates.

## Umsetzungsslices

| PT | Was | Exit Gate |
|---|---|---|
| BC-REC-01 | Fehlerklassen, Recovery State Machine und Audit-/Watchdog-IPC-Vertrag | deterministischer Transition-Test + Negativfälle |
| BC-REC-02 | Rust Recovery Supervisor + begrenzte Retries und sichere Quiesce-/Exit-Pfade | Lab PASS ohne Resource-Lifetime-Verletzungen |
| BC-REC-03 | Minimaler externer Watchdog + Crash/Hang-Evidence + Launcher-Handoff | Crash-Harness erkennt Prozessende und Recovery-Option korrekt |
| BC-REC-04 | Mod-Quarantäne und profilbasierter Rollback, ohne Originaldateien zu ändern | VS-/Mod-Adapter-Lab + eigene Client-Receipts |
| BC-REC-05 | Echte isolierte VS-Sessions + DX12 Fault Injection + Q8-Trace-Trennung | Live-QUALIFIED nur bei reproduzierbaren Nachweisen |

## Referenzen

- [Bismut VS Client Roadmap V1.8](BISMUT_VS_CLIENT_ROADMAP_V1_8.md)
- [Q15 Dual SDK + Loader](ADR_BISMUT_Q15_DUAL_SDK_BISMUT_LOADER_2026-11-18.md)
- [Q14 GPU Resources Evaluation](ADR_BISMUT_Q14_GPU_RESOURCE_MANAGEMENT_EVALUATION_2026-11-18.md)

**Provenienz:** Owner-Beschluss aus der Q&A. Dokumentations-Commit im zentralen Git-Review-Branch; keine Engine-Codeänderung, kein tatsächlicher Runtime-/Watchdog-Test, kein lokaler Windows-Checkout verändert.
