# Physis-X – Dokumentation

Dieses Verzeichnis enthält die zentrale Functional System Design Specification (FSD) und verweist auf die repository-spezifischen Spezifikationen.

## Dokumente

- [FSD-001 – Gesamtsystem](FSD-001-system.md): Systemkontext, Architektur, Rollen, Schnittstellen, Datenmodell, Abläufe, Sicherheits- und Qualitätsanforderungen sowie Integrations- und Abnahmekriterien.

## Repository-spezifische Spezifikationen

- [Node C6](https://github.com/mcuw/physis-x-node-c6/tree/main/documents)
- [Thread Border Router](https://github.com/mcuw/physis-x-tbr/tree/main/documents)
- [Audio Direction Spike](https://github.com/mcuw/physis-x-spike-audio-direction/tree/main/documents)
- [Translation Spike](https://github.com/mcuw/physis-x-spike-translation/tree/main/documents)

## Dokumentationsprinzip

Die zentrale FSD definiert systemweite Anforderungen und verbindliche logische Verträge. Die Teil-FSDs konkretisieren diese Anforderungen für das jeweilige Repository. Bei Konflikten hat die zentrale FSD für systemweite Schnittstellen Vorrang; hardware- und implementierungsspezifische Details werden im zuständigen Repository gepflegt.

Der Status dieser Dokumente ist **Entwurf / FSD-Baseline**. Offene Entscheidungen und durch Spikes zu validierende Annahmen sind ausdrücklich gekennzeichnet. Eine Anforderung gilt erst nach festgelegter Verifikation und Abnahme als umgesetzt.
