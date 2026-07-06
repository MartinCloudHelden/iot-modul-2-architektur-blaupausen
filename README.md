# IoT-Architektur-Blaupausen: Anbindung von Maschinen an die IT

Vier Architekturmuster für die Kopplung von OT und IT – von der gefährlichen Direktanbindung bis zur Cloud-Plattform. Jedes Muster enthält ein Architekturbild (SVG + PNG), ein bereinigtes Mermaid-Diagramm, eine Beschreibung sowie Stärken, Schwächen und passende Einsatzgebiete.

**RLP-Bezug:** 1.3.2 (Kopplung IT ↔ IoT), 1.4.5 (Netzwerkdesign für Maschinenkommunikation, Netzwerksicherheit, Schutz der Integrität, Verfügbarkeit der Datenübertragung)

## Die vier Muster

| # | Muster | Kernidee | Aufwand | Typischer Einsatz |
|---|---|---|---|---|
| 1 | [Direktanbindung](muster-01-direktanbindung/) | Maschine direkt ins Büronetz | keiner | **Anti-Pattern** – nur als Negativbeispiel |
| 2 | [Edge-Gateway](muster-02-edge-gateway/) | Gateway als einziger Übergabepunkt, Broker in der IT | gering | Retrofit im KMU, PoC, erste Schritte |
| 3 | [Industrie-DMZ](muster-03-industrie-dmz/) | Broker in eigener Pufferzone, zwei Firewalls | hoch | Mittlere/große Produktion, Regulierung |
| 4 | [Cloud-Anbindung](muster-04-cloud-anbindung/) | Managed Broker beim Cloud-Anbieter, nur ausgehend | mittel | Verteilte Standorte ohne lokale IT |

Eine detaillierte Gegenüberstellung: [docs/vergleichsmatrix.md](docs/vergleichsmatrix.md)

## Wie diese Blaupausen gedacht sind

Die Muster bauen aufeinander auf und beantworten jeweils eine Schwäche des vorherigen:

1. **Muster 1** zeigt das Problem (die Ausgangslage in vielen Betrieben).
2. **Muster 2** löst es mit dem kleinstmöglichen Mittel: ein Gateway, eine Firewall, eine Richtung.
3. **Muster 3** beantwortet die verbliebene Schwäche von Muster 2 („Broker steht in der IT") mit einer eigenen Zone.
4. **Muster 4** tauscht Eigenbetrieb gegen Anbieterabhängigkeit – eine andere Antwort auf dieselbe Frage, kein „besseres" Muster 3.

**Es gibt kein generell richtiges Muster.** Die Wahl folgt aus Schutzbedarf, Betriebsgröße, vorhandener Kompetenz und Budget. Genau diese Abwägung ist Systemintegrations-Arbeit.

## Hinweise zur Nutzung der Diagramme

- `diagramm.svg` – Vektorgrafik für Folien und GitHub (skaliert verlustfrei)
- `diagramm.png` – Rastergrafik (2x) für PowerPoint/Word
- `diagramm.mmd` – Mermaid-Quelltext; identischer Codeblock ist im jeweiligen README eingebettet und wird von GitHub direkt gerendert
