# FSD-001 – Physis-X Gesamtsystem

- Dokumenttyp: Functional System Design Specification
- Status: Entwurf / FSD-Baseline
- Version: 0.1
- Geltungsbereich: Gesamtsystem Physis-X
- Führendes Repository: [physis-x-system-manager](https://github.com/mcuw/physis-x-system-manager)

## 1. Zweck und Zielsetzung

Physis-X ist ein modulares IoT-System aus vernetzten Embedded Devices, einem Thread-Netzwerkzugang, einem zentralen System Manager sowie optionalen Audio- und Übersetzungsfunktionen.

Diese FSD beschreibt die logische Gesamtarchitektur, Verantwortungsgrenzen, systemweiten Schnittstellen, Datenflüsse, Qualitätsanforderungen und Integrationsabnahmen. Sie ist die Referenz für die nachfolgenden repository-spezifischen FSDs.

## 2. Dokumenten- und Projektstruktur

| Repository | Verantwortung |
|---|---|
| [physis-x-node-c6](https://github.com/mcuw/physis-x-node-c6) | ESP32-C6 Thread-Endgerät und Gerätefunktionen |
| [physis-x-tbr](https://github.com/mcuw/physis-x-tbr) | ESP32-S31 Function Coreboard-1 Thread Border Router |
| [physis-x-spike-audio-direction](https://github.com/mcuw/physis-x-spike-audio-direction) | Spike zur Sprecher-Richtungserkennung mit zwei Mikrofonen |
| [physis-x-spike-translation](https://github.com/mcuw/physis-x-spike-translation) | Übersetzungsassistent und AMOLED-Dialog |
| [physis-x-system-manager](https://github.com/mcuw/physis-x-system-manager) | Zentrale Verwaltung, Orchestrierung und Simulation |

Die Simulation ist Bestandteil des System Managers und erhält kein separates Repository. Ein Physis-X-Gerät kann auf ESP32-C6 oder ESP32-S31 Function Coreboard-1 basieren. Die konkrete Rolle eines Geräts wird nicht allein aus dem Chiptyp abgeleitet, sondern aus seiner konfigurierten Funktion. Der hier definierte TBR ist ein separates S31-basiertes System.

## 3. Systemkontext

### 3.1 Systemgrenze

Physis-X umfasst:
- Thread-fähige Endgeräte und deren Geräte-/Anwendungsfunktionen.
- Einen Thread Border Router (TBR) auf ESP32-S31.
- Den System Manager zur Geräteverwaltung, Zustandsübersicht, Konfiguration und Orchestrierung.
- Eine Simulation innerhalb des System Managers.
- Optionale Audio-Richtungserkennung und Übersetzungsfunktionen auf geeigneter Hardware.

Externe Systeme:
- Smartphone-Hotspot bzw. lokaler IP-Uplink.
- Externe Speech-to-Text-, Übersetzungs- und gegebenenfalls Text-to-Speech-Dienste.
- Entwicklungs-, Build- und Diagnosewerkzeuge.

### 3.2 Architekturübersicht

```text
                  Internet / externe Dienste
                           |
                    Smartphone-Hotspot
                           |
                    IP-Uplink / LAN
                           |
                 +--------------------+
                 | ESP32-S31 TBR      |
                 | Thread Border     |
                 | Router             |
                 +--------------------+
                           |
                    Thread / IPv6
                           |
             +-------------+-------------+
             |                           |
      +--------------+            +--------------+
      | ESP32-C6     |    ...     | Physis-X     |
      | Thread Node  |            | Device       |
      +--------------+            +--------------+
             |
       CoAP / IPv6
             |
       +-------------------------------+
       | Physis-X System Manager       |
       | - Device Inventory            |
       | - Discovery / Connectivity    |
       | - Status / Telemetry          |
       | - Configuration / Commands    |
       | - Orchestration               |
       | - Simulation                  |
       +-------------------------------+
```

Das Diagramm ist logisch. Der konkrete Hosting-Ort, die Verbindung des System Managers zum IPv6-Thread-Netz und die genaue Management-Topologie sind noch zu entscheiden und in den Netzwerkspikes zu verifizieren.

## 4. Architekturprinzipien und Verantwortungsgrenzen

### 4.1 Thread Border Router

Der TBR:
- verbindet das Thread-Netz mit einem IPv6-fähigen Upstream-Netz;
- übernimmt Thread Border Routing und die dafür notwendige Netzwerkinfrastruktur;
- unterstützt Join-/Commissioning- und Netzwerk-Lifecycle-Prozesse gemäß implementiertem OpenThread-Setup;
- stellt keine fachliche Geräte-Orchestrierung bereit.

Der TBR ist ein eigenständiges Firmware-Repository und darf nicht in den System Manager integriert oder mit dessen Simulation vermischt werden.

### 4.2 Physis-X Node

Ein Node:
- stellt seine Geräteidentität und Fähigkeiten bereit;
- implementiert seine lokalen Funktionen und Aktoren/Sensoren;
- meldet Status und Telemetrie;
- verarbeitet zulässige Konfigurations- und Kommandonachrichten;
- bleibt bei einem Ausfall des System Managers lokal in einem definierten sicheren Zustand funktionsfähig.

Der ESP32-C6-Node ist der erste konkrete Thread-Endgeräte-Spike. Die generische Geräteabstraktion darf nicht auf C6-spezifische Eigenschaften beschränkt werden.

### 4.3 System Manager

Der System Manager:
- verwaltet ein logisches Geräteinventar;
- erfasst Geräteerreichbarkeit und Betriebszustände;
- stellt Fähigkeiten und Telemetrie dar;
- verwaltet Konfigurationen und stößt unterstützte Kommandos an;
- verarbeitet Rückmeldungen und Fehler;
- bietet eine Simulation mit demselben logischen Geräte- und Nachrichtenvertrag wie reale Geräte.

Der System Manager ist kein Thread Border Router und übernimmt keine Funk-/Thread-Netzwerkfunktionen.

### 4.4 Simulation

Die Simulation:
- läuft im Repository und Produktkontext des System Managers;
- bildet Geräte, Fähigkeiten, Statusänderungen, Telemetrie, Kommandos und Fehlerzustände ab;
- verwendet denselben logischen Vertrag wie reale Geräte;
- kennzeichnet simulierte Geräte und Daten eindeutig;
- darf reale Geräte nicht als simuliert ausgeben oder umgekehrt.

## 5. Funktionale Anforderungen

Die IDs sind vorläufig und werden im weiteren FSD-Prozess stabilisiert.

### 5.1 Geräteidentität und Inventar

- SYS-F-001: Der System Manager MUSS Geräte anhand einer stabilen, eindeutigen Geräte-ID unterscheiden.
- SYS-F-002: Ein Geräteobjekt MUSS mindestens ID, Gerätetyp, Hardware-/Firmware-Information, Fähigkeiten und Herkunft (real/simuliert) abbilden können.
- SYS-F-003: Ein Gerät MUSS unabhängig von seiner aktuellen IP-Adresse identifizierbar bleiben.
- SYS-F-004: Der System Manager MUSS den letzten bekannten Zustand und dessen Zeitstempel speichern bzw. darstellen können.
- SYS-F-005: Doppelte Geräte-IDs MÜSSEN erkannt und als Konflikt behandelt werden.

### 5.2 Discovery und Verbindung

- SYS-F-010: Der System Manager MUSS den Verbindungszustand eines Geräts abbilden.
- SYS-F-011: Discovery MUSS von der stabilen Geräteidentität getrennt sein.
- SYS-F-012: Ein vorübergehender Netzwerkausfall DARF nicht zum Verlust der Geräteidentität oder der zuletzt bekannten Metadaten führen.
- SYS-F-013: Die konkrete Discovery-Methode (z. B. Registrierung, CoAP-Discovery, mDNS/Service Discovery oder Kombination) ist durch den Netzwerk-/Interop-Spike festzulegen.

### 5.3 Fähigkeiten, Status und Telemetrie

- SYS-F-020: Geräte MÜSSEN ihre unterstützten Fähigkeiten deklarieren können.
- SYS-F-021: Fähigkeiten MÜSSEN versionierbar sein.
- SYS-F-022: Statusmeldungen MÜSSEN von Telemetrie unterschieden werden.
- SYS-F-023: Messwerte MÜSSEN Einheit, Zeitbezug und – sofern verfügbar – Qualitäts-/Validitätsinformation tragen.
- SYS-F-024: Der System Manager MUSS veraltete Daten als veraltet erkennbar machen.

### 5.4 Konfiguration und Kommandos

- SYS-F-030: Konfigurationsänderungen MÜSSEN von einmaligen Kommandos unterschieden werden.
- SYS-F-031: Kommandos MÜSSEN eine eindeutige Korrelations-ID besitzen.
- SYS-F-032: Ein Gerät MUSS ein Kommando mit Erfolg, Ablehnung oder Fehler quittieren können.
- SYS-F-033: Wiederholte Kommandos MÜSSEN hinsichtlich Idempotenz definiert sein.
- SYS-F-034: Nicht unterstützte Kommandos MÜSSEN eindeutig abgewiesen werden.
- SYS-F-035: Der System Manager MUSS Timeout und Nichterreichbarkeit als getrennte Fehlerzustände darstellen können.

### 5.5 Simulation

- SYS-F-040: Die Simulation MUSS virtuelle Geräte mit Geräte-ID, Fähigkeiten und Zuständen erzeugen können.
- SYS-F-041: Simulierte Geräte MÜSSEN denselben logischen Vertrag wie reale Geräte verwenden.
- SYS-F-042: Die Simulation MUSS Statuswechsel, Telemetrie, Kommandorückmeldungen und Fehlerfälle reproduzierbar auslösen können.
- SYS-F-043: Simulationsdaten MÜSSEN eindeutig als simuliert gekennzeichnet sein.
- SYS-F-044: Ein Integrationstest MUSS wahlweise simulierte und reale Geräte gegen dieselben System-Manager-Funktionen prüfen können.

### 5.6 Audio und Übersetzung

- SYS-F-050: Die Audio-Richtungserkennung MUSS eine Richtung und eine Konfidenz bzw. einen Qualitätswert liefern, sofern der Algorithmus dies unterstützt.
- SYS-F-051: Bei unzureichender Konfidenz MUSS die Anwendung einen unbestimmten Zustand zulassen und darf keine sichere Sprecherzuordnung vortäuschen.
- SYS-F-052: Die Übersetzungsanwendung MUSS Quell- und Zielsprache über die Benutzeroberfläche auswählbar machen.
- SYS-F-053: Die Anzeige MUSS die Textorientierung für die gegenüber sitzende Person um 180 Grad umschalten können.
- SYS-F-054: Die Übersetzungsanwendung MUSS Hotspot-Verbindungsabbrüche und Dienstfehler erkennbar behandeln.
- SYS-F-055: Der konkrete Ort der Audio-/Übersetzungsfunktion und die Integration in das Geräte- und System-Manager-Modell sind nach den Spikes festzulegen.

## 6. Logischer Geräte- und Nachrichtenvertrag

Die folgenden Ressourcen sind ein **Vorschlag für den Spike**, noch kein finaler Wire-Standard:

| Ressource | Zweck |
|---|---|
| `/physis/device` | Identität und Gerätebeschreibung |
| `/physis/capabilities` | Unterstützte Fähigkeiten |
| `/physis/status` | Aktueller Betriebszustand |
| `/physis/telemetry` | Messwerte und Ereignisse |
| `/physis/config` | Konfigurationswerte |
| `/physis/commands` | Kommandos und Ergebnisse |

Zu validierende Protokollentscheidungen:
- CoAP-Methode und Ressourcenmodell;
- JSON oder CBOR;
- CoAP Observe oder alternatives Status-/Eventmodell;
- maximale Payload-Größen und Fragmentierungsverhalten;
- Fehlercodes, Timeouts und Retry-Strategie;
- Kommando-ID, Korrelations-ID und Idempotenz;
- Authentisierung, Autorisierung und Verschlüsselung;
- Schema- und Capability-Versionierung.

### 6.1 Minimales logisches Datenmodell

```json
{
  "schema_version": "0.1",
  "device_id": "stable-device-id",
  "device_type": "physis-node",
  "hardware": {
    "platform": "esp32-c6",
    "board": "board-identifier"
  },
  "firmware": {
    "version": "firmware-version"
  },
  "origin": "real",
  "capabilities": [],
  "status": {
    "state": "online",
    "updated_at": "RFC3339 timestamp"
  }
}
```

Das Beispiel ist ein logisches Modell und keine Festlegung auf JSON als Transportkodierung.

## 7. Kernabläufe

### 7.1 Inbetriebnahme und Beitritt

1. TBR startet und initialisiert sein Netzwerk.
2. Thread-Netzwerkparameter und Border-Routing-Zustand werden persistent bzw. reproduzierbar verwaltet.
3. Node durchläuft den vorgesehenen Join-/Commissioning-Prozess.
4. Node erhält Thread-Konnektivität und eine IPv6-Erreichbarkeit.
5. Node stellt seinen Gerätevertrag bereit.
6. System Manager erkennt bzw. registriert das Gerät und aktualisiert dessen Zustand.

Die konkrete Commissioning-Sequenz und Credential-Verteilung sind im TBR-/Node-FSD zu spezifizieren.

### 7.2 Geräteüberwachung

1. System Manager ermittelt Erreichbarkeit bzw. empfängt Statusmeldungen.
2. Status und Zeitstempel werden aktualisiert.
3. Bei ausbleibenden Meldungen wird der Zustand nach definierter Frist als veraltet/nicht erreichbar markiert.
4. Bei Wiederkehr wird der Zustand aktualisiert, ohne eine neue Geräteidentität zu erzeugen.

### 7.3 Kommandoausführung

1. Ein autorisierter Client fordert ein Kommando an.
2. System Manager validiert Gerät, Fähigkeit, Parameter und Berechtigung.
3. Kommando erhält eine eindeutige ID.
4. Kommando wird an das Gerät übertragen.
5. Gerät quittiert Annahme und später Ergebnis oder meldet einen Fehler.
6. System Manager ordnet die Rückmeldung der ursprünglichen Anfrage zu.

### 7.4 Simulation

1. Benutzer erstellt ein virtuelles Gerät.
2. Simulationsgerät erhält ID, Fähigkeiten und Anfangszustand.
3. System Manager verarbeitet es über dieselben logischen Funktionen wie ein reales Gerät.
4. Simulator liefert reproduzierbare Ereignisse und Kommandoantworten.
5. Oberfläche kennzeichnet das Gerät eindeutig als simuliert.

## 8. Zustände und Fehlerbehandlung

Mindestens folgende logische Gerätezustände sind vorzusehen:
- `unknown`: noch kein belastbarer Zustand bekannt;
- `commissioning`: Inbetriebnahme/Join läuft;
- `online`: erreichbar und aktuell;
- `degraded`: erreichbar, aber eingeschränkt;
- `offline`: nach definierter Frist nicht erreichbar;
- `error`: gemeldeter oder erkannter Fehler.

Die endgültigen Übergänge und Timeout-Werte werden durch die Integrationsversuche bestimmt. Ein Netzwerkfehler darf nicht automatisch als Gerätefehler interpretiert werden.

## 9. Sicherheitsanforderungen

- SYS-S-001: Thread-Netzwerkzugang und Credentials MÜSSEN gegen unbefugten Zugriff geschützt werden.
- SYS-S-002: Geräteidentität und Netzwerkmitgliedschaft MÜSSEN getrennt betrachtet werden.
- SYS-S-003: Management-Kommandos MÜSSEN authentisiert und autorisiert sein.
- SYS-S-004: Vertrauliche Daten und Credentials DÜRFEN nicht in Logs oder Klartext-Diagnoseausgaben offengelegt werden.
- SYS-S-005: Externe Übersetzungsdienste MÜSSEN hinsichtlich Übertragung, Speicherung und Verarbeitung von Audiodaten bewertet werden.
- SYS-S-006: Schlüsselrotation, Geräteentzug und Wiederinbetriebnahme sind zu spezifizieren.

Die Aktivierung von DTLS-/ECJPAKE-Konfigurationsoptionen allein gilt nicht als Nachweis einer durchgängigen Ende-zu-Ende-Sicherheit.

## 10. Nichtfunktionale Anforderungen

Messbare Zielwerte werden nach den Spikes festgelegt. Folgende Qualitätsmerkmale sind verpflichtend zu untersuchen:
- Zuverlässigkeit: Wiederanlauf nach TBR-, Node- und Uplink-Ausfall.
- Latenz: Status-, Kommando- und Audio-/Übersetzungslatenz.
- Ressourcen: RAM, Flash, CPU-Last und Task-Stack auf Embedded-Geräten.
- Skalierbarkeit: Anzahl gleichzeitiger Nodes und Ereignisrate.
- Wartbarkeit: reproduzierbare Builds, gepinnte Abhängigkeiten und dokumentierte Konfiguration.
- Beobachtbarkeit: strukturierte Logs, Fehlercodes und Diagnoseinformationen.
- Datenschutz: Minimierung und transparente Behandlung übertragener Audio-/Textdaten.

## 11. Repository-spezifische Spezifikationen

- Node: [physis-x-node-c6/documents](https://github.com/mcuw/physis-x-node-c6/tree/main/documents)
- TBR: [physis-x-tbr/documents](https://github.com/mcuw/physis-x-tbr/tree/main/documents)
- Audio: [physis-x-spike-audio-direction/documents](https://github.com/mcuw/physis-x-spike-audio-direction/tree/main/documents)
- Translation: [physis-x-spike-translation/documents](https://github.com/mcuw/physis-x-spike-translation/tree/main/documents)

## 12. Abhängigkeiten und Umsetzungsreihenfolge

### Phase 1 – Netzwerkfundament

1. C6-1: reproduzierbare Firmwarebasis.
2. TBR-1: reproduzierbarer S31-Build.
3. TBR-2: Border-Routing-Lifecycle und Netzwerkpersistenz.
4. C6-2: Join, Rejoin und Recovery.
5. TBR-3 / C6-3: reale Interoperabilität und erster Gerätevertrag.

**Gate N1:** Ein realer C6-Node tritt dem TBR-Netz bei, erhält IPv6-Konnektivität und ist über den vereinbarten Anwendungspfad erreichbar. Neustart und kurzzeitiger Ausfall müssen getestet sein.

### Phase 2 – parallele Funktionsspikes

- Audio-1/2: Dual-Mic-Capture und Richtungsalgorithmus.
- TR-1/2: Übersetzungspipeline und AMOLED-Dialog.
- SM-1: vorläufiger Geräte- und Nachrichtenvertrag; Änderungen aus N1 werden eingearbeitet.

**Gate A1:** Richtungserkennung liefert messbare Ergebnisse und einen definierten Unsicherheitszustand.

**Gate T1:** Übersetzungsdialog funktioniert mit Testaudio, Hotspot und simuliertem Richtungssignal; Sprachauswahl und 0°/180°-Anzeige sind nachgewiesen.

### Phase 3 – System Manager und Integration

1. SM-2: Inventar und Discovery.
2. SM-3: Simulation und reale Geräte mit demselben Vertrag.
3. Integration von Audio-/Übersetzungsfunktionen nach bestätigter Hardware- und Schnittstellenentscheidung.

**Gate SM1:** Reales und simuliertes Gerät werden mit gleichem logischen Datenmodell verwaltet; Status, Telemetrie, Konfiguration, Kommandos und Fehler sind testbar.

## 13. Abnahmetests auf Systemebene

| Test-ID | Test | Abnahmekriterium |
|---|---|---|
| SYS-AT-001 | TBR Build | S31-Firmware baut reproduzierbar mit dokumentierter IDF-Version und Konfiguration |
| SYS-AT-002 | Thread Join | C6 tritt dem TBR-Netz bei und erhält IPv6-Konnektivität |
| SYS-AT-003 | Recovery | TBR-/Node-Neustart führt zu definiertem Wiederanlauf ohne manuelle Neuinstallation |
| SYS-AT-004 | CoAP Interop | Node-Ressourcen sind vom vorgesehenen Management-Endpunkt erreichbar |
| SYS-AT-005 | Identität | Geräte-ID bleibt über IP-Wechsel und Neustart stabil |
| SYS-AT-006 | Status | Erreichbarkeit, Veraltung und Fehlerzustand werden korrekt unterschieden |
| SYS-AT-007 | Commands | Kommando-ID, Quittierung, Timeout und Fehlerzuordnung funktionieren |
| SYS-AT-008 | Simulation | Simuliertes Gerät nutzt denselben logischen Vertrag und ist klar markiert |
| SYS-AT-009 | Audio | Richtungserkennung wird gegen eine definierte Winkel-/Distanz-/Störgeräusch-Testmatrix bewertet |
| SYS-AT-010 | Translation | Sprache, Übersetzung, Sprecherwechsel und Display-Orientierung funktionieren im Testszenario |
| SYS-AT-011 | Hotspot Recovery | Wiederverbindung nach Hotspot-/Uplink-Ausfall ist definiert und getestet |

Jeder Test benötigt später einen konkreten Testaufbau, Eingabedaten, erwartete Ergebnisse, Messwerte und einen nachvollziehbaren Testnachweis.

## 14. Offene Architekturentscheidungen

| ID | Entscheidung | Verantwortlich / Nachweis |
|---|---|---|
| ADR-001 | Wo läuft der System Manager und wie erreicht er das Thread-IPv6-Netz? | System Manager / Netzwerk-Interop |
| ADR-002 | Wie erfolgt Geräte-Discovery und Registrierung? | System Manager + Node/TBR |
| ADR-003 | JSON oder CBOR und genaue CoAP-Ressourcen? | Node + System Manager |
| ADR-004 | Observe, Polling oder Event-Push für Status? | Node + System Manager |
| ADR-005 | Sicherheitsmodell für Provisionierung und Management | Systemarchitektur |
| ADR-006 | Welche Hardware führt Audio-/Übersetzungsfunktion aus? | Audio-/Translation-Spikes |
| ADR-007 | Welche STT-/Übersetzungsanbieter und Datenschutzbedingungen gelten? | Translation-Spike |
| ADR-008 | Verbindliche Latenz-, Skalierungs- und Ressourcenbudgets | Messungen aus Spikes |

## 15. Änderungs- und Traceability-Regeln

- Systemweite Anforderungen erhalten stabile IDs im Format `SYS-*`.
- Repository-spezifische Anforderungen erhalten lokale Präfixe und verweisen auf die zugehörigen SYS-IDs.
- Jede Anforderung erhält Verifikationsmethode: Test, Analyse, Inspektion oder Demonstration.
- Entscheidungen, die mehrere Repositories betreffen, werden als Architecture Decision Records (ADR) im System-Manager-Dokumentationsbereich geführt.
- Änderungen am Nachrichtenvertrag müssen gegen Node, TBR (soweit betroffen), System Manager und Simulation geprüft werden.
- Spike-Ergebnisse ersetzen Annahmen erst, wenn Messdaten, Logs oder reproduzierbare Tests dokumentiert sind.

## 16. Referenzen

- [Node C6 Repository](https://github.com/mcuw/physis-x-node-c6)
- [TBR Repository](https://github.com/mcuw/physis-x-tbr)
- [Audio Direction Spike](https://github.com/mcuw/physis-x-spike-audio-direction)
- [Translation Spike](https://github.com/mcuw/physis-x-spike-translation)
- [System Manager](https://github.com/mcuw/physis-x-system-manager)
