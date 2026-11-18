# ADR BISMUT Q15 | Dual SDK + Bismut Loader (Forge-inspiriert)

- Datum: 2026-11-18
- Owner-Entscheidung: **Q15 Option B mit ergänzendem Bismut Loader**
- Status: **ARCHITECTURE_DECIDED / IMPLEMENTATION_NOT_ATTEMPTED**
- Produkt: **Bismut VS Client**
- Referenzidee: Forge-artige Loader-/Lifecycle-/Registry-Konzepte, **kein Forge-Code-Fork** und **kein Ersatz des Vintage-Story-Mod-Loaders**

## Entscheidung

Bismut stellt ein **Dual Mod SDK** bereit:

1. **Bismut .NET SDK/Fassade** für Vintage-Story-Mods, die bereits über den VS-eigenen Mod-Loader und seine modinfo.json/ModSystem-/Lifecycle-Semantik laufen. Die Fassade erweitert den Zugriff auf native Bismut-Funktionen, ohne VS-Mod-Loading zu duplizieren.
2. **Bismut Rust SDK (Crates)** für Bismut-native Mods/Erweiterungen, optional mit nativer Plugin-Binary. Öffentliche dynamische Grenzen werden **explizit über eine versionierte C-ABI/API-Tabelle** definiert; Rusts internes ABI ist nicht stabil.
3. **Bismut Loader (Rust)** als Runtime-Komponente im selben Bismut VS Client Prozess. Verantwortlich für Bismut-native Manifest-Discovery, Dependency/Conflict Resolution, Versions- und Capability-Prüfung, erlaubtes Plugin-Loading, Registry- und Lifecycle-Orchestrierung, Diagnose und kontrollierten Shutdown.
4. **Bismut Launcher/Installer** bleibt für Download, Paket- und Modpack-Installation, VS-Version-Matching, Profile, BCIL-Analyse, atomaren Rollback und Startkonfiguration zuständig. Loader lädt/aktiviert und validiert **zur Laufzeit**. Kein zweiter Package Manager in der Engine.

**Explizite Trennung:** Bismut Loader ist nicht dasselbe wie die bestehende Bismut Forge (Creative-/Asset-Editor) und kopiert nicht die Minecraft-Forge-Implementierung.

## Architekturgrenzen

- **VS-Mods:** Bestandsmods bleiben unter dem VS-Mod-Loader, soweit kompatibel. Bismut registriert einen Adapter und erweitert Bismut-Capabilities. Keine ungefragte Veränderung von modinfo.json, VS-Assemblies oder produktiven Mods.
- **Native Bismut-Mods:** nutzen ein separates Bismut-Manifest (vorgeschlagener Name: bismut.mod.json, Format noch zu spezifizieren). Enthält ID, Version, SDK-/ABI-Ranges, Game-Adapter-Version, Dependencies/Conflicts, Side (Client/Server), Capabilities, Publisher/Provenance und Artefakt-Hashes. Eine Mod darf beide Ökosysteme bedienen, aber Discovery und Instantiierung sind dedupliziert.
- **Plugin-Einstieg:** keine C++-/Rust-VTable oder instabile Rust-Symbole als externe ABI. Export/Entrypoint mit struct_size, abi_major/minor, versioniertem Function Table, Feature Bits, Fehlercodes und Generation Handles; spätere Felder nur mit kompatiblen Layout-Regeln. Versionsinkompatibilität ist ein **expliziter Fail-Close** statt eines stillen Load-Versuchs.
- **Bismut Registries:** deklarative Registrierungen für Render-Passes/Materialien, Animation Sources, GUI, Input Subscriptions, Asset Providers und Debug/Telescope APIs, jeweils an Q2 Frame-/Resource-Verträge und Q3/Q4/Q12 Owner gebunden. Kein direkter ungovernter GPU-Device-/Swapchain-/Present-Zugriff über Standard-Mod-SDK.
- **Lifecycle:** Preflight/Resolve → Runtime Ready → Register → Load/Activate → Running → Deactivate/Shutdown. Reihenfolge ist reproduzierbar und geloggt; VS-Thread-Affinity wird erst nach Q9/Q10-Evidence fixiert. Mod-Callbacks erhalten Lifetime-/Reentrancy-Regeln und Abbruchpfade.
- **Update/Hot Reload:** Asset-/Shader-Reload kann nach Gate möglich sein; **Code-Unload und DLL-Hot-Reload sind NICHT automatisch qualifiziert**, insbesondere bei nativen Libraries, .NET AssemblyLoadContexts oder GPU-Fence-gebundenen Ressourcen.
- **Trust-Modell:** In-process-native Plugins sind grundsätzlich vertrauenswürdiger Code mit vollem Prozess-Crash-/Speicherkorruptionspotenzial. Capability-Checks und Manifeste liefern **keine OS-Sandbox**. Launcher-Preflight, Hashes, Nutzerfreigaben, getrennte Testprofile und optional später isolierte WASM-/IPC-Ausführung sind zusätzliche Verteidigungslinien, nicht garantierte Crash-Isolation.
- **V1-Kompatibilität:** Q7-D vollständige VS-Mod-Kompatibilität inklusive Custom-GL ist ein Ziel. Bismut Loader/SDK und Atlas alleine garantieren diese nicht; jeder Custom-GL-Pfad braucht Bismut-Bridge, Adapter und echte Client-/DX12-Evidence. Q8-C OpenGL Zero bleibt ein eigenes Gate.

## Technischer Start-Schnitt

1. **BC-LOADER-01:** VS-Mod-Loader-API + bestehende Bismut-Mod-/Launcher-Verträge inventarisieren; keine Doppel-Discovery; Mod-/Profil-/Version-Matrix dokumentieren.
2. **BC-LOADER-02:** Minimales ABI v1 mit API Table, Major/Minor, struct_size, Feature Bits, opaque Handle-Generations, Ownership/Release, Fehler-/Thread-Verträgen; ein .NET und ein Rust Smoke Client.
3. **BC-LOADER-03:** Manifest-/Dependency Resolver + registrierte Lifecycle-Events + deterministische Ordnung/Diagnostics. VS-Mod-Sidecar-Support nur wenn explizit nötig.
4. **BC-LOADER-04:** **Zwei vertikale Integrations-Slices:** unveränderte Vanilla-VS-Mod plus .NET-Bismut-Fassade; native Rust-Demo-Mod registriert einen ungefährlichen Asset-/UI-/Animator-Provider über Loader und C-ABI.
5. **BC-LOADER-05:** Kompatibilitäts-/Negativmatrix, API-Version-Mismatch, Dependency-Zyklen, doppelte IDs, fehlende Capabilities, Plugin-Ausnahme/fehlgeschlagener Load, Resource-Leaks, Shutdown/Recovery; Atlas für Game-/Server-Mod, Bismut Live-Harness für Render-/Input-/DX12-Client-Pass. Kein Game-Launch ohne bestehende Laufzeit-Gates.

## PASS-Gates

- **Statisch:** deterministischer Resolver und semantische ABI-Kompatibilität (Major/Minor + struct_size, FFI-Panics dürfen Grenze nicht überschreiten).
- **Lab:** Loader bootet Bismut-native Demo-Plugin und registriert korrekte Capabilities; VS-Mod bleibt unter VS-Loader, keine doppelte Mod-Instanz.
- **Live:** VS 1.22.7 mit zwei repräsentativen Mods, funktionierenden .NET-/Rust-SDK-Slices, tatsächlichen Frame-/Resource-Receipts, korrekten Shutdown-/GPU-Retirement-Nachweisen.
- **Sicherheit:** ungültige/neue/alte Manifeste, unsignierte oder unbekannte Plugins gemäß Policy, unerlaubte Capabilities und invalide Handle werden erkennbar abgelehnt. **Arbitrary native memory corruption ist nicht durch Loader isolierbar.**
- **Kompatibilität:** Atlas-Tests sowie echte DX12-Frames und Q8 OpenGL-Zero-Evidence getrennt reporten. Fehlender Qualifikationsnachweis bleibt UNKNOWN, kein erfundenes PASS.

## Abhängigkeiten

- Q1 Rust-first, Q2 versionierte Frame-/Resource-ABI, Q3 Dual Animator, Q4 Hybrid GUI, Q5 Rust Host, Q6 Rust DX12, Q7 BCIL/Launcher/Atlas, Q8 GL Zero, Q12 Rust Input Router gelten.
- Q9 Bootstrap, Q10 Threading, Q11 CPU-Memory, Q13 Shader und Q14 GPU-Resources bleiben evaluierungsbedingt **OPEN**. Ihre Ergebnisse können das Detaildesign von Loader-Lifecycle, ABI-Lifetime und Registry-Callbacks einschränken.
- Bestehende lokale Engine-/Launcher-/Mod-Checkouts und Saves bleiben unangetastet. Dieses ADR ist eine **Entscheidung, keine implementierte Loader-Funktion**.

## Referenzen

- [Roadmap V1.7](BISMUT_VS_CLIENT_ROADMAP_V1_7.md)
- [Q12 Rust Input Router](ADR_BISMUT_Q12_RUST_INPUT_ROUTER_2026-11-18.md)
- [Q14 GPU Resource Evaluation](ADR_BISMUT_Q14_GPU_RESOURCE_MANAGEMENT_EVALUATION_2026-11-18.md)
- Konzepte: Forge Lifecycle, Dependencies, Event Bus (https://docs.minecraftforge.net/en/latest/concepts/lifecycle/; https://docs.minecraftforge.net/en/1.21.x/gettingstarted/modfiles/).
- Vintage Story modinfo und ModInfo (https://wiki.vintagestory.at/Modding:Modinfo/en; https://apidocs.vintagestory.at/api/Vintagestory.API.Common.ModInfo.html).
