# Customer Deployment Process - CGS Assist

Dieses Dokument beschreibt den vollständigen Prozess zur Auslieferung der CGS Assist Anwendung an einen Kunden.

### Inhaltsverzeichnis
- 1 [Vorbereitung](#1-vorbereitung)
- 2 [Lizenzierung](#2-lizenzierung)
- 3 [Konfiguration](#3-konfiguration)
- 4 [Build-Prozess](#4-build-prozess)
- 5 [Deployment](#5-deployment)
- 6 [Konfiguration und Customization](#6-konfiguration-und-customization)
- 7 [Testing und Validierung](#7-testing-und-validierung)
- 8 [Dokumentation](#8-dokumentation)
- 9 [Übergabe und Training](#9-übergabe-und-training)
- 10 [Post-Deployment](#10-post-deployment)
- 11 [Troubleshooting](#11-troubleshooting)
- 12 [Checkliste für Go-Live](#12-checkliste-für-go-live)
- 13 [Kontakt und Support](#13-kontakt-und-support)

## 1. Vorbereitung

### 1.1 Anforderungsanalyse
- [ ] Kundenanforderungen erfassen
- [ ] Benötigte Module identifizieren (LLM, RAG, MCP-Server, Usecases, etc.)
- [ ] Infrastruktur-Anforderungen klären (On-Premise vs. Cloud)
- [ ] Lizenzmodell festlegen
- [ ] Skalierungsanforderungen bestimmen

### 1.2 Technische Voraussetzungen
- [ ] Hardware-Spezifikationen prüfen
  - CPU: Mindestens 8 Cores
  - RAM: Mindestens 16GB (32GB empfohlen)
  - Speicher: Mindestens 100GB verfügbar
  - GPU: Optional, abhängig von LLM/ImageGen Nutzung
- [ ] Docker & Docker Compose Installation (Version 20.10+)
- [ ] Netzwerk-Konfiguration
  - Port-Freigaben klären
  - Firewall-Regeln definieren
  - SSL/TLS Zertifikate bereitstellen (über Caddyfile)

## 2. Lizenzierung

### 2.1 Lizenz erstellen

license_creation
create_license.py

### 2.2 Lizenz bereitstellen
- [ ] Lizenzdatei generieren
- [ ] Lizenzschlüssel an Kunden übermitteln
- [ ] Lizenz im System hinterlegen (`license/license.json`)

## 3. Konfiguration

### 3.1 Environment-Variablen
Erstellen Sie eine `.env` Datei im Hauptverzeichnis:

```env
DATABASE_URL=sqlite:///cgs_assist_database.db

# Redis (für Celery)
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/1

# API Keys
OPENAI_API_KEY=sk-...
AZURE_OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...

# Sicherheit
SECRET_KEY=<generierter-secret-key>
CORS_ORIGINS=https://customer-domain.com

# Logging
LOG_LEVEL=INFO
LOG_DIR=/var/log/cgs_assist
```

### 3.2 Konfigurationsdateien anpassen
- [ ] `configuration/config.yaml` - Hauptkonfiguration
- [ ] `docker-compose.yaml` - Container-Orchestrierung
- [ ] `Caddyfile` - Reverse Proxy & SSL
- [ ] `alembic.ini` - Datenbank-Migrationen


## 4. Build-Prozess

### 4.1 Docker Images erstellen
```bash
# Alle Images bauen
docker-compose build

# Einzelne Services bauen
docker-compose build cgs_assist
docker-compose build llm_api
docker-compose build rag_api
```

### 4.2 Images taggen und pushen

Verwenden des bereitgestellten Skripts:
```bash
./build_and_push.bat
```

## 5. Deployment

### 5.1 Deployment-Paket vorbereiten
Erstellen eines Deployment-Pakets mit:
- [ ] `docker-compose.yaml`
- [ ] `.env.example` (Template für Umgebungsvariablen)
- [ ] `Caddyfile`
- [ ] `alembic/` (Migrations-Skripte)
- [ ] `license/` (Lizenzverzeichnis)
- [ ] `DEPLOYMENT_GUIDE.md`
- [ ] `QUICK_START_ONBOARDING.md`
- [ ] Alle erforderlichen Konfigurationsdateien

### 5.2 Auf Zielsystem deployen
```bash
# 1. Deployment-Paket auf Server kopieren
scp -r deployment-package user@customer-server:/opt/cgs-assist/

# 2. Auf Server verbinden
ssh user@customer-server

# 3. Ins Verzeichnis wechseln
cd /opt/cgs-assist

# 4. Environment-Variablen konfigurieren
cp .env.example .env
nano .env  # Anpassen

# 5. Services starten
docker-compose up -d
```

### 5.3 Services verifizieren
```bash
# Status prüfen
docker-compose ps

# Logs prüfen
docker-compose logs -f

# Health-Check
curl http://localhost:8000/health
```

## 6. Konfiguration und Customization

### 6.1 Branding anpassen
- [ ] CGS / ARC Version im docker compose setzen

### 6.2 Module konfigurieren
- [ ] **LLM API**: Provider & Modelle konfigurieren
- [ ] **RAG API**: Vector Store & Embeddings einrichten
- [ ] **Workflow Engine**: Automationen einrichten

### 6.3 Benutzer & Rechte
- [ ] Admin-Benutzer anlegen
- [ ] Benutzerrollen definieren
- [ ] Zugriffsrechte konfigurieren

## 7. Testing und Validierung

### 7.1 Funktionale Tests
- [ ] Login & Authentifizierung
- [ ] Alle Module testen
  - LLM Chat-Funktionalität
  - RAG Dokumenten-Upload & Suche
  - ImageGen Bildgenerierung
  - Workflow-Ausführung
- [ ] API-Endpoints testen
- [ ] WebSocket-Verbindungen prüfen

### 7.2 Performance-Tests
- [ ] Last-Tests durchführen
- [ ] Response-Zeiten messen
- [ ] Ressourcen-Auslastung überwachen

### 7.3 Sicherheits-Tests
- [ ] SSL/TLS Verbindung prüfen
- [ ] Authentifizierung validieren
- [ ] CORS-Konfiguration testen
- [ ] Datei-Upload Sicherheit

## 8. Dokumentation

### 8.1 Kundendokumentation bereitstellen
- [ ] `DEPLOYMENT_GUIDE.md` - Installations-Anleitung
- [ ] `QUICK_START_ONBOARDING.md` - Schnellstart für Endbenutzer
- [ ] `ONBOARDING_TEMPLATE_SETUP.md` - Onboarding-Konfiguration
- [ ] API-Dokumentation (Swagger/OpenAPI)
- [ ] Troubleshooting-Guide

### 8.2 Admin-Dokumentation
- [ ] Backup & Restore Prozeduren
- [ ] Monitoring & Logging
- [ ] Update-Prozess
- [ ] Skalierung & Performance-Tuning

## 9. Übergabe und Training

### 9.1 Technische Übergabe
- [ ] Zugang zu allen Systemen bereitstellen
- [ ] Passwörter & Secrets übergeben (sicher!)
- [ ] Backup-Strategie erklären
- [ ] Monitoring-Dashboard einrichten

### 9.2 Benutzer-Training
- [ ] Admin-Training durchführen
- [ ] End-User Training durchführen
- [ ] Dokumentation durchgehen
- [ ] Q&A Session

### 9.3 Support-Vereinbarung
- [ ] Support-Level definieren (SLA)
- [ ] Eskalations-Prozess festlegen
- [ ] Kontaktdaten austauschen
- [ ] Wartungsfenster vereinbaren

## 10. Post-Deployment

### 10.1 Monitoring einrichten
```bash
# Logs überwachen
docker-compose logs -f --tail=100

# Ressourcen-Nutzung
docker stats

# Health-Checks
curl http://localhost:8000/health
```

### 10.2 Backup-Strategie
- [ ] Datenbank-Backups konfigurieren
- [ ] File-Upload Backups
- [ ] Konfigurationsdateien sichern
- [ ] Backup-Restore testen

### 10.3 Wartungsplan
- [ ] Regelmäßige Updates planen
- [ ] Log-Rotation konfigurieren
- [ ] Disk-Space Monitoring
- [ ] Security-Patches anwenden

## 11. Troubleshooting

### Häufige Probleme

#### Services starten nicht
```bash
# Logs prüfen
docker-compose logs cgs_assist

# Container neu starten
docker-compose restart cgs_assist

# Vollständiger Neustart
docker-compose down
docker-compose up -d
```

#### Datenbank-Verbindungsfehler
- [ ] DATABASE_URL in `.env` prüfen
- [ ] Datenbank-Container läuft: `docker-compose ps`
- [ ] Netzwerk-Konnektivität testen

#### Lizenz-Probleme
- [ ] Lizenzdatei vorhanden: `license/license.json`
- [ ] Lizenz nicht abgelaufen
- [ ] Korrekte Module lizenziert

#### Performance-Probleme
- [ ] Ressourcen-Limits erhöhen (RAM, CPU)
- [ ] Celery Worker skalieren
- [ ] Redis Cache optimieren
- [ ] Datenbank-Indizes prüfen

## 12. Checkliste für Go-Live

### Pre-Go-Live
- [ ] Alle Tests erfolgreich
- [ ] Backup-Strategie implementiert
- [ ] Monitoring aktiv
- [ ] SSL/TLS Zertifikate installiert
- [ ] Firewall-Regeln aktiv
- [ ] Dokumentation vollständig
- [ ] Training abgeschlossen

### Go-Live
- [ ] Services starten
- [ ] Health-Checks durchführen
- [ ] Smoke-Tests ausführen
- [ ] Kunden-Freigabe einholen

### Post-Go-Live
- [ ] 24h Monitoring
- [ ] Kunde-Feedback sammeln
- [ ] Performance-Metriken prüfen
- [ ] Support-Verfügbarkeit sicherstellen

## 13. Kontakt und Support

**CGS Software & Data GmbH**
- Website: https://www.cgs-online.de/
- Support-Email: support@cgs-online.de
- Telefon: [Support-Telefonnummer]

**Eskalation**
- Level 1: Kunden-Support
- Level 2: Technische Teams
- Level 3: Entwicklungs-Teams

---

**Version:** 1.0.0  
**Letzte Aktualisierung:** 2026-01-07  
**Autor:** CGS Software & Data

