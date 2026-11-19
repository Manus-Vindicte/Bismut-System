# ADR BISMUT Q21 | Einfacher Support, ein Send-Button, alle Cloud-Pfade über Vercel

- Datum: 2026-11-19
- Owner-Entscheidung: **Q21 Option A, erweitert um freiwilligen Supportversand über einen zentralen Vercel-Gate**
- Owner-Präzisierungen: **erst Crash-Panel + lokale Loganalyse + möglicher Lösungsvorschlag, danach freiwillig „Fehlerbericht senden“ für genauere Hilfe**; alle Client-zu-Cloud-Diagnosepfade nur über Vercel; Benutzer müssen die interne Architektur nicht sehen oder verstehen.
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Öffentlicher Produktname: **Bismut Client** (Standard und Extended, gemeinsame Titelleiste). BismutX nur intern.
- Kontext: Q16 Recovery/Watchdog, Q17 Benchmark/Frame-Receipts, Q18 Unified Shell, Q20 Mod Hub. Kein zweites Diagnose-Truth-Modell.

## Entscheidung

Bismut Client bietet im Standard eine **ruhige, verständliche Hilfe-/Diagnoseoberfläche**. Kein Developer Cockpit, keine offenliegende Rust-/DX12-/Shader-/GPU-Architektur, kein Prompt-/Modell-/Provider-Dashboard und keine technische Metrikenflut für normale Benutzer.

- Lokale Diagnose und Wiederherstellung (Logs, erkennbare Mod-/Installationskonflikte, sichere Startoptionen, verständliche Integritäts- und Rollback-Hinweise) funktionieren **offline und ohne Cloud**.
- **Vor jedem Cloudkontakt** fängt der lokale Launcher/Watchdog einen bekannten Crash-/Exit-/Hang-Status ab, erzeugt das **Crash-Panel** und führt eine **lokale, deterministische Log-/RunRecord-/ModGraph-/Version-/GPU-Fehleranalyse** aus. Das Crash-Panel nennt nur mit vorhandener Evidence eine konkrete oder wahrscheinliche Ursache, zeigt eine mögliche Lösung und einen sicheren Bestätigungs-Button. Erkennt die Analyse keine belastbare Ursache, steht dort „Ursache noch unbekannt“.
- **Erst anschließend** wird **„Wenn du genauere Hilfe möchtest, sende den Fehlerbericht“** als sekundäre Aktion angeboten. „Fehlerbericht senden“ löst eine **freiwillige Übermittlung** aus; vorher gibt es eine kurze Datenvorschau und **„Senden“ / „Abbrechen“**. Details bleiben hinter „Welche Daten werden gesendet?“, nicht auf der Hauptoberfläche.
- Nach dem Versand erhält der Nutzer eine **verständliche Ergebnis-/Lösungskarte**: „Was ist passiert?“, „Das kannst du versuchen“, „Falls das nicht hilft“. Konkrete Aktionen („Sicher starten“, „Problematischen Mod deaktivieren“, „Einstellungen zurücksetzen“) nur, wenn lokal qualifiziert und nach expliziter Bestätigung. Bei noch fehlender Antwort: einfacher Status „Bericht empfangen / Wir konnten noch keine Empfehlung ermitteln“ und lokaler Notfallpfad.
- **Keine internen Systembezeichnungen in der normalen UI:** weder Vercel, Dev Client, LLM, AI Gateway, Modellname, RPC, Rust, C-ABI, ETW noch „Receipt“. Intern heißt das Ergebnis `ResolutionReceipt`, **sichtbar** heißt es „Hilfeergebnis“ bzw. „Lösungsvorschlag“. Bei explizit angeforderten erweiterten Details darf ein technischer Export verfügbar sein.
- **Standard und Extended zeigen die gleiche einfache Supportoberfläche.** Extended darf intern zusätzliche DX12-/BCIL-/Loader-Evidenz für den Bericht liefern, ohne sie ungefragt in der Nutzeroberfläche zu präsentieren.

## Crash-Panel und Local-First-Diagnose

**Ablauf:** Game-/Client-/Mod-Crash oder qualifizierter Hang → Q16 Watchdog/Launcher erfasst letzten RunRecord/Crashstatus und erhält die Oberfläche auch dann aufrecht, wenn der Game-Prozess selbst beendet wurde → lokale Regeln analysieren relevante Logs, Exception-/Exit-Codes, Version-/Mod-/Environment-Diffs, fehlende Dateien, Save-Bindings sowie DX12-Fehlercodes (nur falls Extended tatsächlich qualifiziert ist) → **Crash-Panel** zeigt Diagnose und bis zu drei sichere, verständliche Lösungsmöglichkeiten → nur auf Nutzerwunsch **„Fehlerbericht senden“** zum Vercel-Gate.

**Wahrheitsmodell:** `IDENTIFIED` (Fehlerursache durch konkreten bekannten Code und Evidenz belegt), `SUSPECTED` (Indizien, vorsichtig formuliert „könnte“), `UNKNOWN` (keine belegbare Ursache). Ein fehlender Treffer wird niemals als „Mod war schuld“ dargestellt. Lokale Analyse ist **regel-/signature-/heuristikbasiert, ohne notwendige LLM- oder Internetverbindung**; neue Signaturen können als versionierte, vertrauenswürdig signierte Updates kommen.

**Crash-Resilienz:** Stürzt die In-process Game-Runtime ab und beendet den Host, kann dessen UI nicht magisch weiterlaufen. Der **äußere Launcher/Watchdog** zeigt die Hilfe nach einem qualifizierten Neustart/Host-Restore; bei fehlender Evidence bleibt die Aussage UNKNOWN. Ein hängender Prozess wird nicht ohne gesonderte sichere Nutzer-/Policy-Entscheidung beendet.

**Sichere Lösungen:** Erneut starten, Safe Launch, problematische Mods testweise deaktivieren, bekannte Version-/Dateikonflikte prüfen, auf Last Known Good zurückkehren. Mod-/Profil-/Grafikmutationen benötigen Nutzerfreigabe, Snapshot/WorldBinding-Safety und Rollback; keine automatischen Änderungen an Saves, Treiber oder Mod-Binaries.

**Nicht funktional:** Kein Send-Button als primäre Lösung; kein erzwungener Upload; keine Behauptung, dass jede Fehlerursache erkannt werden kann. Die optionale Cloud-Analyse soll **präzisieren**, nicht das lokale Crash-Panel überhaupt erst ermöglichen.

## Architektur: strikter Vercel-Gate

```text
Bismut Client (lokale Diagnose + Redaction + Zustimmung)
    |
    | EIN autorisierter Diagnose-Endpunkt; TLS; kurzlebiger Request-Token
    v
VERCEL GATE (einziger Egress für online Diagnose)
    | Auth/Attestation + Quota + Rate Limits + Dedupe + Redaction-Validation
    | Schema/Size/Policy Gate + Audit + Routing + Kostenlimit
    |---------+------------------------------------+
    v                                              v
Dev Client / Developer Analysis                Kostenloses/guenstiges LLM
(normalisiert Fehler, Regeln,              (menschliche Loesungsformulierung,
Referenzfaelle und bekannte Fixes)           ohne Halluzinations-Promotion)
    |                                              |
    +-------------------> Zusammenführung <---------+
                             |
                       ResolutionReceipt
                             |
                       Vercel Gate
                             |
                       Bismut Client
                       „Hilfeergebnis“
```

**Egress-Invariante:** Für diesen Q21-Diagnoseablauf kommuniziert die lokale Anwendung **ausschließlich mit dem freigegebenen Bismut-Vercel-Endpunkt**. Es gibt **keine direkten Client-Anfragen** an Dev Client, Modellprovider oder externes LLM. Das Gateway entscheidet über Routing und Fallback. Andere bereits bestehende Produkttransporte (z. B. ModDB und Updates) werden durch Q21 nicht pauschal umgeleitet; „alles hinter Vercel“ betrifft **alle Cloud-Pfade der Diagnose- und KI-Hilfefunktion**, nicht den vollständigen Launcher-Netzwerkverkehr.

**Serverseitig** dürfen Dev-Analyse und LLM möglichst parallel auf demselben freigegebenen, redigierten Diagnoseobjekt arbeiten. Ihre Ergebnisse sind getrennte, nicht vertrauliche Datenquellen: Regeln/Evidenz bestimmen den technischen Status, das LLM darf Erklärungen und sichere Vorschläge formulieren, aber **niemals einen ungeprüften Mod-/Save-/Treiber-/GPU-Fix als bewiesenes PASS erklären**. Die Zusammenführung prüft die Plausibilität, Policy und Quell-/Regel-IDs vor dem Abschluss des Receipts.

**Providerwahl:** Eine derzeit „free“ oder günstige LLM-Route ist **ein Budgetziel, kein kostenloser SLA oder garantiert verfügbarer Provider**. Route/Modell/Version nur serverseitig und nach Live-Verfügbarkeits-, Lizenz-/Datenverarbeitungs-, Token- und Kostenprüfung konfigurieren. Vercel AI Gateway kann als serverseitige Modellrouting-Komponente dienen, falls tatsächlich bereitgestellt; Q21 behauptet keine vorhandene Konfiguration. Quotas, Hard-Budgets, Timeouts, Retry-Limits und Cache/Idempotency gelten **vor** teuren Provideraufrufen.

## Minimaler UX-Vertrag

**Crash-Panel:** „Das Spiel wurde unerwartet beendet.“

**Lokale Auswertung:** „Mögliche Ursache erkannt: Ein Mod könnte mit dieser Spielversion inkompatibel sein.“ Wenn belegbar: konkreten Fehler nachvollziehbar nennen, sonst nur als Vermutung kennzeichnen. Bei UNKNOWN: „Wir konnten die Ursache noch nicht erkennen.“

**Vorgeschlagene Lösung:** „Starte das Spiel testweise ohne den betroffenen Mod.“ Primäre Aktion: **„Sicher starten“** oder eine andere belegbar sichere, reversible Lösung. Zweite Aktion: „Schließen“.

**Freiwillige Vertiefung:** „Du möchtest genauere Hilfe? Sende uns deinen Fehlerbericht zur weiteren Analyse.“ Sekundärer Button: **„Fehlerbericht senden“**. Dieser startet erst die Versandfreigabe, nicht sofort den Upload.

**Vor Versand:** „Wir senden technische Fehlerdaten, um dir eine Lösung vorzuschlagen. Deine Spielstände und Zugangsdaten werden nicht mitgeschickt.“ Aktionen: **„Senden“** / **„Abbrechen“**; Link **„Welche Daten werden gesendet?“**. Wenn die Redaction nicht nachweisbar ist, wird **nicht gesendet**.

**Nach Versand:** „Bericht gesendet.“ Danach: „Mögliche Lösung“ mit maximal drei priorisierten, klar formulierten Schritten und je einem Vertrauens-/Vorbedingungsstatus in normalen Worten. Wenn unklar: „Wir konnten die Ursache noch nicht sicher bestimmen.“ Wenn beide Analysezweige ausfallen: „Der Bericht konnte nicht ausgewertet werden. Du kannst die lokalen Reparaturoptionen weiter nutzen.“

**Keine unnötigen Fachlabels oder Roh-JSON im normalen Flow.** Die Report-ID ist im Hintergrund gespeichert und steht nur in „Details / Supportcode“, falls der Benutzer sie für den Support benötigt. Kein Chatbot-Zwang, kein Modell-Chooser, keine prompteingabe, keine Server-/Token-/Endpoint-Einstellungen.

## Sicherheits-/Datenschutzgrenzen

1. **Privacy by default:** ausschließlich nach Zustimmung, minimaler redigierter `DiagnosticBundle` mit ErrorCode, RunRecord-Referenz, Client-/VS-/Mod-/Backend-Versionen, anonymisiertem Hardware-/Capability-Ausschnitt, relevanten Logs/Receipts und Crash-Hashes. **Kein** ungefilterter Minidump-/Memory-Dump-Upload, kein vollständiger Save/World-Upload, keine Access-/Refresh-Tokens, Cookies, Passwörter, private Pfade, Host-/Account-Namen oder vollständigen Chat-Inhalte.
2. **Lokale Redaction:** strukturierte Allowlist und Secret-Scanner; Gateway prüft erneut auf Schema-/Größen-/Secret-Verletzungen. Hashes mit personenbezogenem Bezug und Korrelation-IDs vorsichtig behandeln. Bei unvollständiger Prüfung **BLOCKED**, kein stiller Rohupload.
3. **Gate-Auth:** User-/Installations-gebundene kurzlebige Credentials oder sessionbezogener token, Abwehr von Replay, Quotas, Rate Limits, Abuse- und Größenlimits; keine Gate-/AI-Provider-Schlüssel im Client-Binary. Idempotente Receipt-ID und begrenzte Retention mit konfigurierbarer Löschung.
4. **Vercel-Vendor-Datenfluss:** Modell-/Dev-Provider und mögliche Datenweitergabe müssen vor Zustimmung transparent erklärt werden. Prüfe vor Implementation Datenschutzinformation, AVV/DPA, Regionen, Provider-Retention/Training-Policy, eventuell erforderliche Rechtsgrundlage und Widerruf. Die App muss auch bei Verweigerung vollständig benutzbar bleiben.
5. **LLM-Untrusted:** Logs, Modnamen, Crashtexte und Fehlerausgaben sind nicht vertrauenswürdige Eingaben; keine Prompt-Injection in serverseitige Tool-Aktionen erlauben. LLM bekommt nur freigegebene Daten, ohne Secrets/Repository-Zugriffe, Schreib-/Patch-/Shellrechte oder freie URL-Fetches. Empfehlungen sind advisory, keine automatischen Systemänderungen.
6. **Failure Handling:** kein Netz, Timeout, Vercel-/LLM-Ausfall, Quota/Budget erschöpft, Modellprovider 429 oder ungültiges JSON -> klarer lokaler Status und regelbasierte Hilfe. Kein Cloud-Bedarf für Q16 Safe Launch/Recovery. Kein unbegrenztes Retries/Upload-Replay.

## Receipt-Vertrag (intern)

```json
{
  "schemaVersion": 1,
  "receiptId": "opaque-id",
  "runId": "opaque-run-id",
  "status": "RECEIVED|ANALYZING|RESOLVED|PARTIAL|UNRESOLVED|REJECTED",
  "publicSummary": "Kurz und verständlich",
  "rootCause": {
    "classification": "KNOWN|LIKELY|UNKNOWN",
    "explanation": "faktenbasierte Kurzbeschreibung",
    "evidenceIds": ["diagnostic-check-id"]
  },
  "steps": [
    {
      "title": "Mod testweise deaktivieren",
      "risk": "LOW|MEDIUM|HIGH",
      "needsUserConfirmation": true,
      "prerequisites": ["world-snapshot-available"],
      "actionId": "safe-known-action-or-null"
    }
  ],
  "warnings": [],
  "modelDisclosure": "assisted-by-ai",
  "createdAt": "ISO8601",
  "expiresAt": "ISO8601"
}
```

`rootCause` kann UNKNOWN sein, `steps` auch leer. Die Client-UI filtert interne IDs und technische Statusnamen, zeigt aber keine erfundene Sicherheit an. **Eine unterstützende KI-Formulierung soll als solche erkennbar sein**, ohne den Nutzer mit Providern/Modellen oder dem internen technischen Aufbau zu belasten. Tatsächlich von Dev-Analyse oder LLM erzeugte Aussagen müssen intern unterscheidbar und auditiert sein. Keine Auto-Fix-Aktion durch ein Receipt allein.

## Implementationsslices und Gates

| PT | Inhalt | Exit |
|---|---|---|
| BC-HELP-01 | Q16-Launcher-/Watchdog-Crash-Panel, lokale Rule-/Log-/RunRecord-/Mod-Analyse, Ursache/Suspected/Unknown, Safe-Action-UI, Redaction-Schema | Crash/Process-Exit/Unknown/offline Negativtests, sichere reversible Lösungen und Secret-Fail-Closed |
| BC-HELP-02 | Minimaler Vercel Gate Contract mit authenticated Upload/Download, Consent, Limits, Idempotency und serverseitiger Speicherung | Integrationslab mit invalid payload, 401/429/503, Rate/Budget und Data Retention |
| BC-HELP-03 | Zwei interne Analyse-Adapter: Dev Client und preis-/quotenbewusste LLM-Route; serverseitig parallel, nur über Gate | Normalisierte Fehlerklassen, Fallback ohne LLM, Quoten-/Timeout-Evidence |
| BC-HELP-04 | ResolutionReceipt-Merge, Evidence-Gate, verständliche Hilfekarte im Client, klare KI-Unterstützungsangabe | Regelbasierte vs KI-Hinweise getrennt, keine ungeprüften Auto-Fixes |
| BC-HELP-05 | E2E: echter Launcher-/VS-/Mod-/GPU-Failure, Consent, Send, Receipt, Offline-/Gate-Outage, Rollback/Supportcode | Live-Test nur bei vollständiger Evidence, ansonsten PARTIAL/BLOCKED |

## Exakte Nicht-Ziele

Keine Q21-B/C/D-Entwickler-UI als Standardprodukt. Keine erzwungene Online-Anmeldung, keine erzwungene Datenübermittlung, keine globalen Treiber-/Mod-/Save-Patches. „Kostenloses LLM“ heißt kostenbewusst und opt-in, **nicht** garantiert kostenlos oder immer erreichbar. Q8 OpenGL Zero und Q7 Mod-Kompatibilität bleiben separat zu qualifizieren. **Q22 Distribution/Updates/Komponentenverwaltung bleibt OPEN**.

## Referenzen

- [Roadmap V2.4](BISMUT_VS_CLIENT_ROADMAP_V2_4.md)
- [Q16 Rust Recovery Supervisor + Watchdog](ADR_BISMUT_Q16_RUST_RECOVERY_SUPERVISOR_EXTERNAL_WATCHDOG_2026-11-18.md)
- [Q18A Standard + Extended](ADR_BISMUT_Q18A_PRODUCT_EDITIONS_INSTALLER_2026-11-19.md)
- [Q20 Mod Hub](ADR_BISMUT_Q20_UNIFIED_MOD_HUB_LOCKFILES_BCIL_EDITIONS_2026-11-19.md)
