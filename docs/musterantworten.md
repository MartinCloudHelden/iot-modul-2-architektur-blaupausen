# Musterantworten zu den Diskussionsfragen

Erwartungshorizonte für die Besprechung der Architekturmuster. Antworten geben den fachlichen Kern vor; Abweichungen der Teilnehmer, die begründet sind, sind zulässig.

---

## Muster 1: Direktanbindung

**1. Der Betrieb hat 12 Mitarbeiter und „keine IT-Abteilung". Ist das Muster damit entschuldbar? Welche minimale Verbesserung kostet unter 100 €?**

Die Betriebsgröße erklärt das Muster, rechtfertigt es aber nicht. Das technische Risiko ist größenunabhängig – ein Ransomware-Befall trifft den 12-Mann-Betrieb genauso, oft härter (kein Backup, keine Wiederanlaufkompetenz). „Klein" senkt die Eintrittswahrscheinlichkeit nicht.

Minimale Verbesserung unter 100 €: ein VLAN-fähiger Smart-Managed-Switch (ab ca. 30–60 €), der Maschine und Büro in getrennte VLANs legt, sodass die Steuerung aus dem Büro-VLAN nicht mehr erreichbar ist. Ehrlich dazu: Ein Smart Switch trennt per VLAN, filtert aber ohne Routing/ACL nicht zustandsbehaftet – das ist ein erster Segmentierungsschritt, keine Firewall. 

Kernaussage: Die billigste wirksame Maßnahme ist Segmentierung (Trennung), nicht „Patchen" – Patches gibt es für alte Steuerungen oft gar nicht.

**2. Welche der Schwächen bleibt bestehen, selbst wenn alle Büro-PCs perfekt gepatcht sind?**

Die strukturelle Schwäche des flachen Netzes: fehlende Authentifizierung der Steuerung und fehlende Isolation. Gepatchte Büro-PCs verhindern nicht (a) einen kompromittierten Fremdgeräte-/Dienstleister-Laptop, USB- oder Phishing-Vektor, (b) ein neu eingestecktes IoT-Gerät oder einen Drucker im selben Netz, (c) Broadcast-/Scan-Traffic, der alte SPS zum Absturz bringt. Perfektes Patchen reduziert eine Eintrittswahrscheinlichkeit, eliminiert aber nicht das Fehlen eines Übergabepunkts und die direkte Erreichbarkeit der Steuerung.

**3. Ordnet die Schwächen den Kategorien aus Modul 2 zu: Sicherheit / Stabilität / Betrieb / Wartbarkeit.**

- **Sicherheit:** ungeschützte Steuerung für jedes Gerät im Netz erreichbar; keine Authentifizierung; Angriffsfläche = gesamtes Netz; Garantie-/Compliance-Risiko.
- **Stabilität:** Scan-/Broadcast-Traffic aus der IT kann alte Steuerungen zum Absturz bringen; keine Lastentkopplung.
- **Betrieb:** kein Monitoring, kein Logging, kein definierter Übergabepunkt – Fehler und Zugriffe bleiben unsichtbar.
- **Wartbarkeit:** keine Protokollgrenze, keine Standardisierung; jede neue Maschine wird wieder direkt reingehängt; kein Rollback bei Kompromittierung.

---

## Muster 2: Edge-Gateway

**1. Der Geschäftsführer will „auch die Stückzahlen sehen". Was ändert sich an diesem Muster – und an seinem Risikoprofil?**

Stückzahlen lassen sich nicht mehr aus dem Stromsensor ableiten – man braucht entweder Prozessdaten aus der Steuerung (Zähler/Register/I/O) oder einen zusätzlichen Sensor am Materialfluss (z. B. Lichtschranke am Auswurf).

- Weg über die SPS: Das nicht-invasive Prinzip fällt weg. Risikoprofil steigt – Anschluss/Zugriff an die Steuerung bringt die Garantie-/Zertifizierungsfrage zurück, der Lesezugriff braucht ein Protokoll (Modbus/OPC UA) und kann die Steuerung belasten, und es entsteht Semantikaufwand (Registerbelegung, Doku).
- Weg über Zusatzsensor: bleibt nicht-invasiv, Risikoprofil bleibt niedrig, aber es ist zusätzliche Hardware und der Zählpunkt muss sauber gewählt sein.

Kernaussage: Der Wunsch nach mehr Daten verschiebt das Muster Richtung invasiv. Vor dem SPS-Zugriff prüfen, ob ein zusätzlicher Sensor reicht – das erhält das risikoarme Grundprinzip.

**2. Warum ist „nur ausgehend" bei der Firewall-Regel so wichtig? Was wäre der Unterschied zu einer eingehenden Freigabe auf das Gateway?**

„Nur ausgehend" heißt: Das Gateway baut die Verbindung zum Broker aktiv auf; von der IT-Seite kann niemand eine Verbindung ins OT-Netz initiieren. Es gibt im OT-Netz keinen lauschenden Dienst, den man von außen erreichen, scannen oder angreifen kann.

Eine eingehende Freigabe auf das Gateway würde einen offenen, erreichbaren Port im OT-Perimeter bedeuten – ein Einfallstor, das gescannt und angegriffen werden kann. Prinzip: Verbindungsrichtung = potenzielle Angriffsrichtung. Ausgehende Verbindungen sind kontrollierbar (feste Ziel-IP/Port, TLS, Zertifikat); eingehende öffnen die Zone. Bild: Das Gateway ruft raus, es nimmt keine Anrufe entgegen.

**3. Das Gateway fällt Freitagabend aus, es merkt niemand bis Montag. Welche Maßnahme aus der Vorlesung fehlt in diesem Bild?**

Monitoring der Datenkette selbst mit Alarmierung – ein Heartbeat/Verfügbarkeitssignal, das Alarm auslöst, wenn Daten ausbleiben. Technisch (wie in der MQTT-Übung): das `verbindung`-Topic mit Last Will, über das der Broker „offline" meldet, plus ein Monitoring-System, das darauf alarmiert. Ohne aktive Überwachung ist der Ausfall unsichtbar (Blindflug). Ergänzend: Pufferung (Store-and-Forward) rettet keine Daten, wenn das Gateway selbst tot ist – dafür braucht es Redundanz oder schnellen Ersatz.

---

## Muster 3: Industrie-DMZ

**1. Warum steht der Broker in der DMZ und nicht in der IT (Muster 2)? Was genau gewinnt man durch den Umzug?**

In Muster 2 steht der Broker in der IT – die OT muss in die IT-Zone publizieren, und eine IT-Kompromittierung erreicht den Broker direkt. In Muster 3 ist der Broker neutraler Boden: Weder OT noch IT sprechen direkt miteinander, beide reden nur mit der DMZ.

Gewinn: (a) eine IT-Kompromittierung endet an FW2/DMZ und erreicht die OT nicht direkt; (b) eine OT-Kompromittierung endet an FW1 und erreicht die IT nicht direkt; (c) ein klarer, auditierbarer Übergabepunkt mit eigenem Eigentümer; (d) die exponierteste Komponente (der Broker) steht weder in der produktiven IT noch in der OT, sondern in einer Zone, die genau für diese Aufgabe gehärtet und überwacht wird. Kurz: Die DMZ ist die kontrollierte Sollbruchstelle zwischen zwei Vertrauenszonen.

**2. FW 1 erlaubt nur „OT → DMZ, Port 8883, ausgehend". Ein Techniker braucht Fernzugriff auf das Gateway. Wie löst dieses Muster das, ohne die Regel aufzuweichen?**

Über den Jump-Host in der DMZ. Der Techniker verbindet sich – mit MFA und protokolliert – auf den Jump-Host; von dort wird der Zugriff auf das Gateway initiiert. Die Verbindungsrichtung bleibt kontrolliert: Der Zugriff kommt aus der DMZ, nicht direkt aus der IT ins OT. FW 1 bekommt dafür eine eng begrenzte, host-spezifische Regel (Jump-Host-IP → Gateway-IP, definierter Port/Protokoll) – keine breite „IT darf ins OT"-Freigabe.

Kernpunkt: Man weicht die Regel nicht auf, sondern kanalisiert den Zugriff über einen einzigen kontrollierten, auditierten Punkt. Der Umweg über die DMZ ist der Sicherheitsgewinn, nicht der Umstand.

**3. Welche der Komponenten in diesem Bild ist am attraktivsten für einen Angreifer – und warum?**

Broker und Jump-Host sind die Scharnierstellen. Der Broker sieht alle Maschinendaten und ist von beiden Seiten erreichbar – seine Kompromittierung bedeutet Zugriff auf alle Telemetrie und potenzielle Manipulation der Daten Richtung IT. Der Jump-Host ist per Definition der legitime Weg in die OT – wer ihn übernimmt, erbt den Wartungszugang zum Gateway.

Der Jump-Host ist der gefährlichere von beiden, weil er (anders als der lesende Broker) einen Pfad Richtung Steuerung eröffnet. Deshalb sind MFA, Protokollierung, minimale Rechte und Härtung genau hier am wichtigsten. Zweitgrößte Angriffsfläche: Fehlkonfiguration der Firewalls – die DMZ schützt nur, solange die Regeln korrekt sind.

---

## Muster 4: Cloud-Anbindung

**1. Die Internetleitung des Standorts fällt für 4 Stunden aus. Was passiert (a) mit der Produktion, (b) mit den Daten, (c) mit den Dashboards? Welche Komponente entscheidet über (b)?**

- **(a) Produktion:** läuft weiter. Maschinen und Steuerung hängen nicht am Internet (Entkopplung) – der Ausfall betrifft nur die Datenanbindung, nicht die Fertigung.
- **(b) Daten:** hängt vom Gateway ab. Mit Store-and-Forward-Puffer werden die Daten lokal zwischengespeichert und nach Rückkehr der Leitung nachgeliefert → kein Verlust, nur Verzögerung. Ohne Puffer sind die 4 Stunden verloren.
- **(c) Dashboards:** nicht verfügbar bzw. veraltet. Sie liegen in der Cloud und werden über dieselbe Leitung abgerufen; vor Ort sieht niemand aktuelle Werte. Lokale Anzeige gäbe es nur mit einer Zusatzkomponente am Standort.

Entscheidende Komponente für (b): das Edge-Gateway mit Pufferung (Store-and-Forward).

**2. Vergleicht die monatlichen Kosten gedanklich: 100 Messwerte/s roh in die Cloud vs. Statusereignisse. Welches Prinzip aus Modul 2 spart hier bares Geld?**

Datenreduktion/Vorverarbeitung am Edge. Cloud-IoT-Plattformen rechnen typischerweise pro Nachricht bzw. Datenvolumen ab. 100 Messwerte/s roh sind 8,64 Mio. Nachrichten pro Tag und Sensor – das schlägt direkt auf die Rechnung durch. Publiziert das Gateway nur Zustandswechsel oder aggregierte Kennwerte, sinkt das Volumen um Größenordnungen (wenige Ereignisse statt Millionen Messpunkte). Prinzip: nur relevante, verdichtete Information übertragen, nicht den Rohstrom.

Ehrliche Einschränkung: Wenn die Rohauflösung fachlich gebraucht wird (z. B. Schwingungs-/Frequenzanalyse), muss man abwägen – dann lokal aggregieren und Rohdaten nur fallweise hochladen.

**3. Wann ist Muster 3 (eigene DMZ) trotz höherem Aufwand die bessere Wahl als dieses Muster?**

Wenn mindestens einer der folgenden Punkte zutrifft: (a) regulatorische oder vertragliche Vorgaben verlangen Datenhaltung im Haus (Datenschutz, Kunden-Audits, NIS2-Kontext); (b) latenzkritische oder lokal autarke Reaktionen sind nötig, die nicht von Internet/Cloud abhängen dürfen; (c) ein schreibender Rückkanal zur OT ist geplant und braucht maximale Kontrolle über den Zugriffspfad; (d) der Standort ist groß genug, um eigenen DMZ-Betrieb zu rechtfertigen, und IT- plus OT-Kompetenz sind vorhanden; (e) Unabhängigkeit von Internetverfügbarkeit und Cloud-Anbieter (Vendor Lock-in, Anbieterausfall) ist gefordert.

Kurzform: sensible/regulierte Daten, Echtzeit/Autarkie vor Ort oder geplanter Rückkanal → DMZ. Verteilte kleine Standorte ohne IT-Personal, zentrale Auswertung, kein Rückkanal → Cloud. In der Praxis oft eine Kombination: DMZ am Standort plus Weiterleitung ausgewählter Daten in die Cloud.
