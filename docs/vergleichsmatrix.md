# Vergleichsmatrix der vier Architekturmuster

| Kriterium | 1 Direktanbindung | 2 Edge-Gateway | 3 Industrie-DMZ | 4 Cloud-Anbindung |
|---|---|---|---|---|
| **Sicherheit** | keine – Steuerung für jedes Gerät im Netz erreichbar | gut – ein Übergang, nur ausgehend | sehr gut – zwei Zonenübergänge, auditierbar | gut – nur ausgehend, Zertifikats-Auth; Daten liegen extern |
| **Entkopplung OT/IT** | keine | ja | ja, beidseitig erzwungen | ja |
| **Invest (Hardware/Aufbau)** | 0 € | niedrig (Sensor + Gateway, < 500 €/Maschine) | hoch (DMZ-Systeme, 2 Firewalls, Konzept) | niedrig–mittel (Gateway + Einrichtung) |
| **Laufende Kosten** | 0 € (bis zum Vorfall) | gering | Personal für DMZ-Betrieb | volumenabhängige Cloud-Gebühren |
| **Betriebsaufwand** | keiner | gering, wächst mit Gateway-Anzahl | hoch, braucht klaren Betreiber | gering vor Ort, Anbieter-Management |
| **Nötige Kompetenz** | keine | Linux/Netzwerk-Basis | Netzwerk-Security, Firewall-Betrieb | Cloud-Plattform (z. B. AWS) |
| **Skalierung (viele Maschinen)** | – | organisatorisch mühsam | gut | sehr gut |
| **Verfügbarkeit der Datenkette** | irrelevant (keine Kette) | Gateway = Single Point of Failure | Broker redundant auslegbar | abhängig von Internetleitung; Puffer nötig |
| **Lokale Dashboards ohne Internet** | – | ja | ja | nein (ohne Zusatzkomponenten) |
| **Compliance/Audit (z. B. NIS2)** | Findings garantiert | Basis erfüllbar | Zielbild | möglich, Datenhaltung extern prüfen |
| **Rückkanal (IT → Maschine) verantwortbar?** | technisch trivial, faktisch fahrlässig | nicht vorgesehen (bewusst) | ja, über DMZ kontrollierbar | eingeschränkt (Latenz, Abhängigkeit) |
| **Typischer Nutzer** | ungewollter Ist-Zustand | KMU, PoC | Industrie ab mittlerer Größe | verteilte Standorte, Cloud-affine Betriebe |

## Entscheidungshilfe in drei Fragen

1. **Muss überhaupt etwas ins Netz?** Wenn nein: Air Gap behalten, Muster sparen.
2. **Wo soll die Auswertung stattfinden – lokal oder zentral über Standorte hinweg?** Lokal → Muster 2/3. Zentral/verteilt → Muster 4.
3. **Gibt es regulatorische Anforderungen oder einen Rückkanal-Wunsch?** Ja → Muster 3 (ggf. kombiniert mit 4: DMZ vor Ort, Weiterleitung in die Cloud – in der Praxis häufig).
