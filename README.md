# Physis-X System Manager

## Zweck und Architekturrolle

Der System Manager ist die zentrale Verwaltungs- und Orchestrierungskomponente des Physis-X-Systems. Er verwaltet logische Geräteidentitäten, Capabilities, Status und Telemetrie und stellt übergreifende Use Cases bereit.

Die **Simulation ist Bestandteil des System Managers** und wird nicht als separates Repository geführt. Der **Thread Border Router bleibt ein eigenständiges Repository und Firmware-Artefakt**. Auch die ESP32-C6-Node-Firmware bleibt separat.

Dieses Repository soll nicht vor den Hardware-/Kommunikationsnachweisen vollständig implementiert werden. Ein kleiner früher Vertragsspieß ist sinnvoll; die eigentliche Integration folgt nach den C6-/TBR-Interoperabilitätstests.

## Repository-Landkarte

| Repository | Verantwortung | Ausgangsbasis / Fokus |
|---|---|---|
| [physis-x-node-c6](https://github.com/mcuw/physis-x-node-c6) | ESP32-C6 Thread-Endgerät, Actor-Funktionen, CoAP-Geräteendpunkt | [esp32-thread-coap-light](https://github.com/mcuw/esp32-thread-coap-light) |
| [physis-x-tbr](https://github.com/mcuw/physis-x-tbr) | ESP32-S31 Thread Border Router, Thread-/IPv6-Routing und Commissioning | [esp32-thread-br](https://github.com/mcuw/esp32-thread-br) |
| [physis-x-spike-audio-direction](https://github.com/mcuw/physis-x-spike-audio-direction) | Dual-Mikrofon-Capture und Sprecher-Richtungserkennung | ESP32-C6 AMOLED-Hardware |
| [physis-x-spike-translation](https://github.com/mcuw/physis-x-spike-translation) | Übersetzungsdienst, Gesprächslogik und AMOLED-Dialog | Smartphone-Hotspot, externer Übersetzungsdienst |
| **physis-x-system-manager** | Geräteverwaltung, Orchestrierung und Simulation | Gemeinsames logisches Modell |

## Repositoryübergreifende Reihenfolge

### Phase 1 – Netzwerkfundament

1. C6-1: bestehende Node-Firmware analysieren und reproduzierbar bauen.
2. TBR-1: bestehende TBR-Firmware auf dem S31 reproduzierbar bauen.
3. TBR-2 und C6-2: Thread-Netz, Join, IPv6-Routing und Recovery gemeinsam nachweisen.
4. TBR-3 und C6-3: Ende-zu-Ende-CoAP und Verantwortungsgrenzen validieren.

### Phase 2 – unabhängige Funktionsspikes

Parallel zum Netzwerkfundament:
- Audio-1: synchronisierte Dual-Mikrofon-Aufnahme.
- Audio-2: Richtungsalgorithmus und Genauigkeitsgrenzen.
- TR-1: Übersetzungsdienst mit Testaudio und Smartphone-Hotspot.
- TR-2: AMOLED-Dialog zunächst mit simulierten Richtungsereignissen.

### Phase 3 – Systemintegration

1. SM-1: vorläufigen Geräte-/Nachrichtenvertrag mit realen C6-Ressourcen abgleichen.
2. SM-2: Discovery, stabile Identität und Statusmodell definieren.
3. SM-3: Simulation und reales Gerät über denselben logischen Vertrag integrieren.

## System-Manager-Spike-Plan

| Reihenfolge | Issue | Arbeitspaket | Abnahmekriterien |
|---|---|---|---|
| Früh, parallel zu Phase 1 | [SM-1 – Geräte- und Nachrichtenvertrag](https://github.com/mcuw/physis-x-system-manager/issues/1) | Device ID, Hardware/Target, Firmware-Version, Capabilities, Status, Telemetry, Config, Command/Acknowledgement, Versionierung und Fehler modellieren. | C6-Beispiel ist abbildbar; TBR-Netzwerkstatus bleibt getrennt von Actor-API; Simulation kann denselben Vertrag verwenden; offene Felder sind markiert. |
| Nach C6-/TBR-Interop | [SM-2 – Inventar und Discovery](https://github.com/mcuw/physis-x-system-manager/issues/2) | Discovery-Optionen, stabile ID vs. IPv6-Adresse, Online/Offline/Stale, Capability- und Firmware-Inventar entwerfen. | Gerät bleibt bei Adresswechsel logisch identisch; Offline/Stale sind unterscheidbar; Lifecycle und Discovery-Verantwortung sind dokumentiert. |
| Nach SM-1/SM-2 und Interop | [SM-3 – Simulation und reale Geräte](https://github.com/mcuw/physis-x-system-manager/issues/3) | Simulationsmodell, Ports/Adapter, Fehler-/Offline-Simulation und vertikalen Use Case gegen reales C6 und Simulation integrieren. | Derselbe Use Case funktioniert gegen simuliertes und reales Gerät; TBR bleibt separat; Integration ist reproduzierbar dokumentiert. |

## Vorläufiger logischer Gerätevertrag

Als Diskussionsgrundlage für SM-1:

- **Identity:** stabile, transport-unabhängige Device ID
- **Description:** Hardwaretyp, Boardrevision, Firmwareversion
- **Capabilities:** verfügbare Funktionen/Actors und deren Versionen
- **Status:** aktueller Zustand mit Zeitbezug und Datenfrische
- **Telemetry:** Messwerte/Ereignisse
- **Configuration:** unterstützte Konfiguration und deren Validierung
- **Command:** angeforderte Aktion mit eindeutiger ID
- **Acknowledgement/Result:** angenommen, ausgeführt, fehlgeschlagen oder abgelaufen

Der logische Vertrag soll nicht an CoAP, Thread oder eine bestimmte IPv6-Adresse gebunden sein. CoAP kann ein Transportadapter sein. Die konkrete Serialisierung und API-Version sind Ergebnis der Spikes/FSD.

## Abhängigkeiten und Integrationsgates

- SM-1 beginnt früh, muss aber gegen C6-3 und TBR-3 validiert werden.
- SM-2 benötigt SM-1, C6-3 und TBR-3.
- SM-3 benötigt SM-1/SM-2 sowie einen bestandenen realen C6↔TBR-Kommunikationspfad.
- Audio Direction und Translation sind zunächst unabhängige Spikes; ihre Integration in Geräte-/Systemmanagement ist erst nach Klärung ihrer Laufzeit- und Schnittstellenanforderungen vorzusehen.

## Definition of Done für den System-Manager-Spike

Ein realer C6-Node über S31-TBR und ein simuliertes Gerät können über denselben logischen Use Case adressiert werden. Identität, Capabilities, Status, Fehler und Datenfrische sind nachvollziehbar. Die Simulation befindet sich im System Manager; TBR und Node bleiben eigenständige Repositories.

## FSD-Hinweis

Diese README ist eine Roadmap und Sammlung von Architekturhypothesen. Die FSD soll nach Abschluss der Spikes verbindliche Anforderungen, Zustandsmodelle, Schnittstellen, Sequenzdiagramme, Sicherheitsanforderungen, Fehlerfälle und nichtfunktionale Anforderungen festlegen.
