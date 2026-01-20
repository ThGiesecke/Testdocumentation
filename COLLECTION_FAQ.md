## Sammlung häufig gestellter Fragen

Dieses Dokument ist zur Sammlung häufig gestellter Fragen in allen Bereichen des Projektes (z.B. Auslieferung, Bereitstellung).
Diese FAQ Liste kann je Kunde und Projekt um spezifische Fragen/Antworten erweitert werden.


|  Frage  | Antwort | 
| ------- | ------- | 
|Was passiert bei schwerwiegenden Incidents? | Für schwerwiegende Störungen wird ein definiertes Incident-Management genutzt (z. B. Eskalationspfad, definierte Reaktionszeiten). Die genaue Ausgestaltung erfolgt im Rahmen der vertraglichen Vereinbarungen und SLAs. | 
|Wie läuft ein Update ab |  Updates werden im Vorfeld angekündigt und mit dem Kunden abgestimmt. Je nach Betriebsmodell werden neue Container-Images bereitgestellt und in einem vereinbarten Wartungsfenster ausgerollt. Kritische Sicherheitsupdates können nach Absprache kurzfristig erfolgen. |
|Können wir unsere eigenen Security-Tools (VPN, Proxy, IDS/IPS) nutzen?| Ja, in der Regel werden vorhandene Kundensicherheitsmechanismen integriert. Wichtig ist eine frühzeitige Abstimmung zu Firewall-/Proxy-Regeln und ggf. SSL-Inspection, um die Funktionalität (z. B. Verbindung zu externen LLM-APIs oder Container-Registy) sicherzustellen. |
|Wer ist für den laufenden Betrieb verantwortlich?|Das hängt vom gewählten Betriebsmodell ab. Bei SaaS/Managed Service liegt der technische Betrieb überwiegend bei CGS, beim On-Prem-/Kundenbetrieb ist der Kunde für Infrastruktur, OS, Docker/Kubernetes verantwortlich; CGS unterstützt bei Applikationsupdates und -konfiguration. Details siehe Verantwortlichkeitsübersicht in den IT-Requirements.|
|Wie lange dauert eine Standard-Auslieferung?|  Für einen typischen Pilotbetrieb (PoC) sollten – bei verfügbarer Infrastruktur und schnellen Freigaben – ca. 2–6 Wochen eingeplant werden. Der Übergang in den Produktionsbetrieb hängt stark von internen Prozessen beim Kunden (Change-Management, Security-Reviews) ab.|
|Wie wird die Lösung bereitgestellt?|Über ein Docker Compose-Skript auf einem Linux-Server.|
|Ist eine spezielle Azure-Integration notwendig?|Nein. Nur API-Key und Endpoint werden benötigt.|
|Ist eine VM zwingend erforderlich?|Nein. Jeder Linux-Server mit Docker Compose ist geeignet.|
|Benötigt die Lösung eine dauerhafte Internetverbindung?|Ja, sowohl für die Installation als auch für die Kommunikation mit der Azure OpenAI API.|
|   |   |