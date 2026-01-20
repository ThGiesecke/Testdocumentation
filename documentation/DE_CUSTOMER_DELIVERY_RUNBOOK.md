# AI Audit Assist – Customer Delivery Runbook (DE)

Dieses Dokument beschreibt den typischen Ablauf, wie die AI Audit Assist Plattform bei einem Kunden eingeführt und technisch ausgerollt wird – von der ersten Anfrage bis zur Übergabe in den Regelbetrieb. Es richtet sich an Vertrieb/Pre-Sales, Projektleitung, technische Ansprechpartner beim Kunden sowie DevOps/Operations.

### Inhaltsverzeichnis
- 1 [Überblick und Rollen](#1-überblick-und-rollen)
- 2 [Phasenmodell der Auslieferung](#2-phasenmodell-der-auslieferung)
- 2.1 [Pre-Sales und Anforderungsaufnahme](#21-pre-sales-und-anforderungsaufnahme)
- 2.2 [Technische Klärung und Architektur-Workshop](#22-technische-klärung-und-architektur-workshop)
- 2.3 [Auswahl des Betriebsmodells (Cloud/SaaS vs. Kunde)](#23--uswahl-des-betriebsmodells-cloud-saas-vs-Kunde))
- 3 [Vorbereitung beim Kunden](#3-vorbereitung-beim-kunden)
- 3.1 [Organisatorische Voraussetzungen](#31-organisatorische-voraussetzungen)
- 3.2 [Technische Voraussetzungen und IT-Requirements](#32-technische-voraussetzungen-und-it-requirements)
- 3.3 [Lizenzierung und Zugriff auf Images](#33-lizenzierung-und-zugriff-auf-images)
- 4 [Deployment-Vorbereitung je Betriebsmodell](#4-deployment-vorbereitung-je-betriebsmodell)
- 4.1 [SaaS / CGS-gehostete Variante](#41-saas--cgs-gehostete-variante)
- 4.2 [Kunden-VM / Docker Compose](#42-kunden-vm--docker-compose)
- 4.3 [Kundeneigenes Kubernetes-Cluster](#43-kundeneigenes-kubernetes-cluster)
- 4.4 [Hybride Szenarien](#44-hybride-szenarien)
- 5 [Test-Deployment (Pilot / Staging)](#5-test-deployment-pilot--staging)
- 5.1 [Zielsetzung des Test-Deployments](#51-zielsetzung-des-test-deployments)
- 5.2 [Durchführung des Test-Deployments](#52-durchführung-des-test-deployments)
- 5.3 [Testprotokoll und Abnahme](#53-testprotokoll-und-abnahme)
- 6 [Go-Live](#6-go-live)
- 6.1 [Go-Live-Checkliste](#61-go-live-checkliste)
- 6.2 [Umschaltstrategie](#62-umschaltstrategie)
- 6.3 [Hypercare-Phase](#63-hypercare-phase)
- 7 [Übergabe in den Regelbetrieb](#7-übergabe-in-den-regelbetrieb)
- 7.1 [Betriebsübergabe an Kunden-IT](#71-betriebsübergabe-an-kunden-it)
- 8 [Anhänge und Verweise](#8-anhänge-und-verweise)

**Weitere projektspezifische Dokumente (z. B. Architekturskizzen, Freigabeprotokolle):**
- `docs/INDEX.md` – Index über weitere technische Dokumente und Tasks
- `docs/README.md` – Gesamtüberblick (Deployment Overview, Production Considerations, Monitoring, Backup)
- `documentation/deployment/Deployment-Anleitung.html` – Schritt-für-Schritt-Anleitung für Kunden
- `documentation/deployment/Azure-Deployment-Customer.md` – Detailleitfaden für Deployment auf Cloud-Server beim Kunden
- `CUSTOMER_DEPLOYMENT_PROCESS.md` – Bestehende Prozessbeschreibung, interne Sicht
- `DEPLOYMENT_GUIDE.md` – Technische Details des Deployments (sofern vorhanden)
- `IT_REQUIREMENTS_AI_AUDIT_ASSIST_DE_EN.md` - technischen Voraussetzungen für die Bereitstellung und den Betrieb
- `technical_requirements/CGS-Assist-Bereitstellung.docx` - CGS Assist technische Bereitstellungsanforderungen
- `COLLECTION_FAQ.md` - Sammlung häufig gestellter Fragen in allen Bereichen des Projektes (z.B. Auslieferung, Bereitstellung, Service)
- `QUICK_START_ONBOARDING.md` - ??
- `ONBOARDING_TEMPLATE_SETUP.md` - ??
- `ONBOARDING_TEMPLATE_COMPLETE.md` - ??
- `ONBOARDING_VISUAL_OVERVIEW.md` - ??

## 1. Überblick und Rollen

  - Ticketsystem / E-Mail-Verteiler für technische Themen
  - Regeltermine (Jour Fixe)
  - Kick-off-Meeting (online/offline)
- **Kommunikation**
  - Kunde: Fachbereich, Projektleitung, IT-Infrastruktur, Netzwerk, Security, IAM/Identity, ggf. Datenschutz.
  - CGS: Vertrieb / Pre-Sales, Projektleitung, Solution Architect, DevOps/Operations, ggf. Security/Compliance.
- **Beteiligte Rollen (typisch)**
- **Ziel**: Gemeinsames Verständnis, wer in welcher Phase welche Aufgaben übernimmt.

## 2. Phasenmodell der Auslieferung

Die Auslieferung folgt typischerweise diesen Phasen:
1. Pre-Sales & Anforderungsaufnahme
2. Technische Klärung & Architektur-Workshop
3. Auswahl des Betriebsmodells (SaaS / Kunde)
4. Vorbereitung beim Kunden
5. Deployment-Vorbereitung & Test-Deployment (Pilot / Staging)
6. Abnahme & Go-Live
7. Hypercare & Übergabe in den Regelbetrieb

Diese Phasen können je nach Kunde zusammengefasst oder iterativ durchlaufen werden, bieten aber eine gute Orientierung.

### 2.1 Pre-Sales und Anforderungsaufnahme

**Verweise**
- Onboarding-Unterlagen wie `QUICK_START_ONBOARDING.md`, `ONBOARDING_TEMPLATE_SETUP.md`, `ONBOARDING_TEMPLATE_COMPLETE.md`, `ONBOARDING_VISUAL_OVERVIEW.md` können für die inhaltliche/onboarding-seitige Planung herangezogen werden.

**Typische Punkte**
- Regulatorische Rahmenbedingungen (z. B. Finanzaufsicht, Aufbewahrungspflichten)
- Erwartete Latenzen/Leistungsanforderungen
- Art der Daten/Dokumente (Volumen, Sensibilität, Sprachen)
- Anzahl der Fachbereiche und Nutzergruppen


**Ziele**
- Erwartete Integrationen und Compliance-Anforderungen einsammeln
- Grobe Nutzungsintensität und Datenquellen klären
- Fachliche Use-Cases verstehen

### 2.2 Technische Klärung und Architektur-Workshop

**Hilfsmittel**
- Bestehende technische Dokumentation in `docs/README.md` (Deployment Overview, Production Considerations)
- IT-Requirements-Dokument (siehe `IT_REQUIREMENTS_AI_AUDIT_ASSIST_DE_EN.md`, sobald vorhanden) als Gesprächsgrundlage

**Inhalte**
- Logging, Monitoring und Backup
- IAM/SSO (z. B. Azure AD/Entra, andere IdPs)
- Netzwerk, Firewall, Proxy, VPN
- Zielumgebung (Cloud, Rechenzentrum, Kubernetes, einzelne VM/Server)
- Betriebsmodell (SaaS / Managed Service vs. Deployment beim Kunden)

**Ziele**
- IT- und Security-Anforderungen des Kunden mit den Plattformmöglichkeiten abgleichen
- Ziel-Betriebsmodell und Architekturvarianten bewerten

### 2.3 Auswahl des Betriebsmodells (Cloud/SaaS vs. Kunde)

Die AI Audit Assist Plattform unterstützt verschiedene Betriebsmodelle:

**SaaS / Managed Service durch CGS**
  - Kunde konsumiert die Anwendung über gesicherte HTTPS-Endpunkte
  - Betrieb der Plattform in einer CGS- oder Partner-Cloud

**Deployment beim Kunden**
  - On-Premise im Rechenzentrum des Kunden
  - Deployment im kundeneigenen Kubernetes-Cluster
  - Docker-Compose auf einer Kunden-VM bzw. in der Kunden-Cloud
  
**Kriterien für die Entscheidung**
  - Skalierungs- und Hochverfügbarkeitsanforderungen
- Interne Prozesse (Change-Management, Betriebsprozesse, Betriebsmodell des Kunden)
- Sicherheitsvorgaben (DMZ, Netztrennung, eigene Security-Tools)
- Regulatorik & Compliance (z. B. Datenstandort, Bank-/Versicherungsaufsicht)

**Verweise**
- `documentation/deployment/Azure-Deployment-Customer.md` – Leitfaden für Deployment auf einem Cloud-Server beim Kunden
- `docs/README.md` – Abschnitt „Deployment Overview“, „Production Considerations“

## 3. Vorbereitung beim Kunden

### 3.1 Organisatorische Voraussetzungen

- Entscheidung, ob Test-/Staging-Umgebung genutzt wird oder direkt im Produktionskontext gestartet wird
- Klärung der internen Freigabe- und Change-Prozesse
- Definition der Kommunikationswege (z. B. zentraler E-Mail-Verteiler, Ticketsystem)
- Benennung der technischen Ansprechpartner (IT, Netzwerk, IAM, Security)

### 3.2 Technische Voraussetzungen und IT-Requirements

- Vorabklärung von Internetzugängen (für SaaS oder externe Services wie LLM-APIs)
  - Backup-/Monitoring-Voraussetzungen
  - Netzwerk, Ports, Firewall-/Proxy-Konfiguration
  - OS, CPU, RAM, Storage
- Bereitstellung der Zielinfrastruktur gemäß IT-Requirements (siehe `IT_REQUIREMENTS_AI_AUDIT_ASSIST_DE_EN.md`):

### 3.3 Lizenzierung und Zugriff auf Images

- Nutzung der Skripte im `customer_deployment_folder/` (z. B. `manage.sh`) entsprechend der Kundenzielumgebung
- Bereitstellung der URL und Zugangsdaten zur Container-Registry (sofern Images dort gehostet werden)
- Ausgabe von Lizenzen/Keys, falls erforderlich
- Abschluss der Lizenz-/Nutzungsverträge




## 4. Deployment-Vorbereitung je Betriebsmodell

### 4.1 SaaS / CGS-gehostete Variante

**Kundenseitige Schritte**
- interne Kommunikation an Nutzer und Fachbereiche
- SSO-Integration (z. B. Azure AD/Entra) nach gemeinsam abgestimmter Konfiguration
- Freigabe von Netzverbindungen zu den CGS-Endpunkten

**CGS-seitige Schritte (High Level)**
- Konfiguration von Authentifizierung, Logging, Monitoring, Backup
- Anlegen von Tenants/Instanzen für den Kunden
- Provisionierung der Plattform in der Ziel-Cloud

### 4.2 Kunden-VM / Docker Compose

**Typischer Ablauf**
  - Verweis auf `documentation/deployment/Deployment-Anleitung.html` (kundengerechte Schritt-für-Schritt-Anleitung)
  - Verweis auf `documentation/deployment/Azure-Deployment-Customer.md`
  - Start des Deployments gemäß Anleitung:
  - Konfiguration kundenindividueller Parameter (URLs, Secrets, Datenbankanbindung, Logging-Ziele etc.)
  - Übermittlung der `docker-compose.yaml` und zugehöriger Konfigurationsdateien (`.env`, `config.yaml`) an den Kunden
  
**Voraussetzungen**
 - Installierter Docker-Engine und Docker Compose
 - Bereitgestellte VM/Server mit unterstütztem OS (siehe IT-Requirements)

### 4.3 Kundeneigenes Kubernetes-Cluster

**Hinweis**
- Detaillierte Kubernetes-Guides können in separaten Dokumenten gepflegt werden und von diesem Runbook referenziert werden.

**High-Level-Schritte**
- Deployment der Applikations-Services in den Cluster
- Anlegen der benötigten Secrets, ConfigMaps und Persistenzkonfiguration (PVCs)
- Vorbereitung von Ingress, Load Balancern und TLS-Zertifikaten
- Definition des Ziel-Namespaces und der Ressourcenzuordnung

Spezifische Hybrid-Architekturen werden projektspezifisch dokumentiert.

### 4.4 Hybride Szenarien

- Kombination mehrerer Tenants/Regionen
- Nutzung kundeneigener Datenbanken/Storage-Dienste
- RAG-Services (Retrieval, Vektorspeicher) on-prem, Frontend/Backend in der Cloud

## 5. Test-Deployment (Pilot / Staging)

### 5.1 Zielsetzung des Test-Deployments

- Überprüfung von Security-, Logging- und Compliance-Anforderungen
- Fachliche Validierung der Use-Cases mit echten oder repräsentativen Testdaten
- Technische Validierung der Plattform in der Kundenumgebung

### 5.2 Durchführung des Test-Deployments

**Schritte (Beispiele)**
- Durchführung von End-to-End Tests mit ausgewählten Pilotnutzern
- Einspielen eines initialen Test-Datenbestands (Dokumente, Konfigurationen)
- Basis-Health-Checks (HTTP-Health-Endpoints, Status der Container/Pods)
- Deployment der Services (Docker Compose / Kubernetes) in der Testumgebung

**Verweise**
- `DEPLOYMENT_GUIDE.md` / `CUSTOMER_DEPLOYMENT_PROCESS.md` (sofern im Projekt als technische Detailanleitungen genutzt)

### 5.3 Testprotokoll und Abnahme

- Schriftliche oder protokollierte Abnahme des Test-Deployments durch den Kunden
- Sammlung offener Punkte und deren Priorisierung
- Dokumentation der Testergebnisse (technisch und fachlich)

## 6. Go-Live

### 6.1 Go-Live-Checkliste

- Endnutzerkommunikation vorbereitet (Einladung, Anleitungen, FAQs)
- Authentifizierung/SSO und Berechtigungskonzept produktiv konfiguriert
- Backup-Strategie implementiert und testweise überprüft
- Monitoring und Alerting für zentrale Komponenten eingerichtet
- Technische Plattform stabil: keine kritischen offenen Bugs im Betrieb

Vor dem finalen Go-Live sollten u. a. folgende Punkte erfüllt sein:

### 6.2 Umschaltstrategie

**Variante 1: Getrennte Test- und Produktionsumgebung**
  - Bereinigung von Testdaten, sofern erforderlich
  - Anpassung von Konfigurationen (z. B. Daten, Nutzer, Logging)
  
**Variante 2: Test-Umgebung wird zur Produktion**
  - Getrennte URLs/Endpunkte
  - Reproduktions-Deployment in Produktivumgebung nach erfolgreichem Test


### 6.3 Hypercare-Phase

- Gemeinsame Bewertung von Optimierungspotenzial und offenen Themen
- Enges Monitoring von Stabilität und Nutzerfeedback
- Erhöhte Bereitschaft auf Seiten von CGS und Kunde
- Definierter Zeitraum (z. B. 2–4 Wochen) nach Go-Live

## 7. Übergabe in den Regelbetrieb

### 7.1 Betriebsübergabe an Kunden-IT

- Klärung der Supportwege und Antwortzeiten
- Vorstellung der Betriebsprozesse (z. B. Patch/Release-Prozess, Incident-Prozess)
- Übergabe aller relevanten Dokumente (IT-Requirements, Deployment-Anleitungen, Betriebsleitfaden, Kontaktdaten)

### 7.2 Wartungs- und Update-Strategie

- Umgang mit sicherheitskritischen Updates
- Beschreibung, wie neue Versionen bereitgestellt und eingespielt werden
- Gemeinsame Festlegung von Wartungsfenstern

### 7.3 Kontinuierliche Verbesserung

- Erfahrungsaustausch zwischen CGS und Kunden (Best Practices)
- Planung zusätzlicher Use-Cases und Integrationen
- Regelmäßige Review-Termine (z. B. quartalsweise) zur Bewertung der Nutzung






