# AI Audit Assist – Technische Bereitstellungsanforderungen

Dieses Dokument richtet sich an IT-Infrastruktur und Systemadministratoren und beschreibt die technischen Voraussetzungen für die Bereitstellung und den Betrieb der AI Audit Assist Plattform beim Kunden.

## ⚠️ WICHTIG: LLM-Bereitstellung durch den Kunden

**Die AI Audit Assist Plattform wird OHNE integriertes Large Language Model (LLM) ausgeliefert.**

Der Kunde muss selbst einen LLM-Provider bereitstellen. Folgende Optionen werden unterstützt:

| LLM-Provider | Status | Empfehlung |
|--------------|--------|------------|
| **Azure OpenAI** | ✅ Vollständig unterstützt | **Empfohlen** für Cloud-Deployments |
| **AWS Bedrock** | ✅ Vollständig unterstützt | Für AWS-Kunden geeignet |
| **Lokale Modelle** | ✅ Unterstützt | Für höchste Datenschutzanforderungen |

**Hinweis:** Azure OpenAI ist unsere bevorzugte Empfehlung, aber AWS Bedrock und lokale Modelle werden gleichwertig unterstützt.

---

## 1. Allgemeine Voraussetzungen
 
### 1.1 Infrastruktur (VM oder Server)

Die Lösung wird auf einem **Linux-basierten Server** bereitgestellt (lokale VM oder Cloud-VM). Es ist **keine spezielle Cloud-Integration erforderlich** – jeder Linux-Server mit Docker Compose ist geeignet.

**Wichtig:** Eine VM ist **nicht zwingend erforderlich**. Die Lösung kann auf jedem Linux-Server mit Docker Compose betrieben werden.

### 1.2 Betriebssystem

- **Linux**, z. B. Ubuntu Server LTS (24.04) oder vergleichbare Distributionen
- Getestet mit Ubuntu; andere Linux-Distributionen auf Anfrage

### 1.3 Docker-Umgebung

Die Bereitstellung erfolgt über **Docker Compose**.

**Erforderliche Versionen:**
- **Docker Engine:** Version ≥ 20.x
- **Docker Compose:** Version ≥ 1.29 oder Compose V2

**Installation:** Der Kunde muss Docker und Docker Compose auf dem Server installieren und konfigurieren.

### 1.4 Gemeinsames Verzeichnis (SHARED_FOLDER)

Ein **beschreibbares Verzeichnis** auf dem Server für den Datenaustausch zwischen den Containern und für persistente Daten (Dokumente, Datenbanken, Vector-Store).

**Aufgabe des Kunden:** Bereitstellung eines Verzeichnisses mit ausreichenden Schreib-/Leserechten für die Docker-Container.

### 1.5 Internetzugang / Proxy

**Erforderlich für:**
- Download der Docker-Images (während der Installation)
- Verbindung zum LLM-Provider API (Azure OpenAI, AWS Bedrock - falls Cloud-Provider gewählt wird)

**Proxy-Unterstützung:** Die Lösung unterstützt den Betrieb hinter einem Proxy. Proxy-Konfiguration kann in den Umgebungsvariablen hinterlegt werden.

**Wichtig:** 
- **Bei Cloud-LLM-Providern (Azure OpenAI, AWS Bedrock):** Dauerhafte Internetverbindung erforderlich
- **Bei lokalen Modellen:** Nur für Installation erforderlich (Download der Docker-Images)

### 1.6 Netzwerk / Ports

**Erforderliche Ports:**

| Port | Protokoll | Verwendung | Erforderlich für |
|------|-----------|------------|------------------|
| **80** | HTTP | HTTP-zu-HTTPS-Redirect, Let's Encrypt ACME-Challenge | Produktion mit Caddy |
| **443** | HTTPS | Verschlüsselter Zugriff (TLS) | Produktion (empfohlen) |
| **8000** | HTTP | Direktzugriff auf Anwendung | Test/Entwicklung |

**Empfehlung für Produktion:** Einsatz eines Reverse Proxys (z. B. Caddy, Nginx) mit TLS-Zertifikaten.

**DNS-Anforderungen für Caddy mit Let's Encrypt:**

Wenn Sie Caddy mit automatischen Let's Encrypt TLS-Zertifikaten verwenden möchten:

- ✅ **FQDN erforderlich:** Die Anwendung muss über einen vollständigen Domainnamen erreichbar sein (z.B. `cgs-assist.ihrefirma.de`)
- ✅ **DNS-Auflösung:** Der FQDN muss auf die öffentliche IP-Adresse des Servers zeigen (A-Record oder CNAME)
- ✅ **Port 80 erreichbar:** Muss aus dem Internet erreichbar sein (für ACME HTTP-Challenge)

**Caddyfile-Konfiguration benötigt:**
```
cgs-assist.ihrefirma.de {
    reverse_proxy cgs_assist_backend:8000
}
```

**Alternative DNS‑01 (öffentlich trusted, ohne Freigabe 80/443)**
Zertifikatsanfrage über DNS-Einträge verifiziert anstatt Inhalte über HTTP bereitzustellen 
Beispiel Provider-Syntax Cloudflare Token als Env-Var:
``
texttest.server.de {reverse_proxy cgs_assist_server:8000 {
		header_up X-Forwarded-Proto {scheme}        
		header_up X-Forwarded-Host {host}        
		header_up X-Forwarded-Port {port}        
		header_up X-Real-IP {remote_host}        
		header_up Host {host}    }    
		header {        
		Strict-Transport-Security "max-age=31536000; includeSubDomains" -Server    
		} tls 	{# DNS-01 Challenge        
				dns cloudflare {env.CLOUDFLARE_API_TOKEN}        
				# optional: propagation_timeout 5m        
				# optional: resolvers 1.1.1.1 8.8.8.8
				}
		}
``
**Wichtig:** in der Konfigurationsdatei des Caddy-Server muss das Plugin "caddy-dns" eingebunden werden, sonst wird "dns cloudflare" nicht gefunden.
Beispiel Eigenes Zertifikat (PEM + Key):
Wenn ein Zertifikat existiert (z. B. von eurer Firmen‑PKI, oder manuell erzeugt), wird es direkt eingebunden:
``
texttest.server.de {reverse_proxy cgs_assist_server:8000 header {
	Strict-Transport-Security "max-age=31536000; includeSubDomains" -Server}    
	# Zertifikat + Private Key (PEM)    
	tls /etc/caddy/certs/test.server.de.fullchain.pem /etc/caddy/certs/test.server.de.key
	}
``
**Wichtig:* Die Zertifikatskette muss vollständig sein(typisch “fullchain”) und die SANs (Subject Alternative Names) müssen zum Hostnamen passen.

**Alternative ohne öffentlichen DNS:**
- Verwendung eigener TLS-Zertifikate (z. B. von interner CA)
- Zugriff nur über IP-Adresse: `http://<server-ip>:8000` (nur für Test/Entwicklung empfohlen)

**Aufgabe des Kunden:** 
- Firewall-Freigabe für Ports 80 und 443 (Produktion) oder 8000 (Test)
- DNS-Konfiguration: A-Record für den gewünschten FQDN (z. B. `cgs-assist.ihrefirma.de`) auf Server-IP
- Bereitstellung des FQDN für die Caddyfile-Konfiguration
- Optional: Eigene TLS-Zertifikate, falls Let's Encrypt nicht verwendet werden soll

---

## 2. LLM-Provider – Bereitstellung durch den Kunden 

**WICHTIG:** CGS Assist stellt **kein LLM** bereit. Der Kunde muss einen der folgenden LLM-Provider selbst einrichten und betreiben.

### 2.1 Übersicht der unterstützten LLM-Provider

| Provider | Empfehlung | Vorteile | Setup-Aufwand |
|----------|------------|----------|---------------|
| **Azure OpenAI** | ⭐ **Bevorzugt** | Enterprise-Support, EU-Regionen, einfache Integration | Mittel |
| **AWS Bedrock** | ✅ Gleichwertig | Gute Claude-Modelle, AWS-Integration | Mittel |
| **Lokale Modelle** | ✅ Für hohe Datenschutzanforderungen | Maximaler Datenschutz, keine Cloud | Hoch (GPU-Hardware erforderlich) |

**Wählen Sie einen Provider basierend auf:**
- Vorhandener Cloud-Infrastruktur (Azure oder AWS)
- Datenschutz- und Compliance-Anforderungen
- Budget und Betriebsmodell

---

### 2.2 Option A: Azure OpenAI (Empfohlen)

**Warum Azure OpenAI empfohlen wird:**
- ✅ Bewährte Enterprise-Integration
- ✅ Verfügbarkeit in EU-Regionen (DSGVO-konform)
- ✅ Microsoft Enterprise Support
- ✅ Bewährte GPT Modelle

**Voraussetzungen:**
- Aktives Azure-Abonnement (Subscription)
- Freigabe für Azure OpenAI Service (kann Genehmigung erfordern)
- Ausreichende Berechtigungen zum Erstellen von Ressourcen

#### Schritt-für-Schritt: Azure OpenAI Service aufsetzen

##### Schritt 1: Azure OpenAI Ressource erstellen

1. **Azure Portal öffnen:** [https://portal.azure.com](https://portal.azure.com)
2. **Ressource erstellen:**
   - Klicken Sie auf "+ Ressource erstellen"
   - Suchen Sie nach "Azure OpenAI"
   - Klicken Sie auf "Erstellen"
3. **Konfiguration der Ressource:**
   - **Abonnement:** Wählen Sie Ihr Azure-Abonnement
   - **Ressourcengruppe:** Erstellen Sie eine neue oder wählen Sie eine bestehende
   - **Region:** Wählen Sie eine Region (z. B. West Europe, North Europe)
   - **Name:** Vergeben Sie einen eindeutigen Namen (z. B. `cgs-assist-openai-prod`)
   - **Tarif:** Standard S0 oder höher
4. **Überprüfen und erstellen:** Klicken Sie auf "Überprüfen + erstellen" und dann "Erstellen"

##### Schritt 2: Modelle deployen in Azure AI Foundry

1. **Azure AI Foundry öffnen:** [https://ai.azure.com](https://ai.azure.com)
2. **Modell deploymen:**
   - Navigieren Sie zu "Deployments"
   - Klicken Sie auf "+ Create new deployment"
3. **Empfohlene Modelle:**
   - **Hauptmodell:** `gpt-4o` 
   - **Optional:** `gpt-5` oder höher 

##### Schritt 3: API-Credentials auslesen

1. Im Azure Portal → Ihre OpenAI Ressource → "Keys and Endpoint"
2. **Notieren Sie für die spätere Konfiguration in der Appllikation:**
   - API Key
   - Endpoint (z. B. `https://ihr-ressourcenname.openai.azure.com/`)
   - Region (z. B. `westeurope`)
   - Deployment-Namen der Modelle

**Beispiel:**
```
Endpoint: https://cgs-assist-openai-prod.openai.azure.com/
API Key: 1234567890abcdef...
Region: westeurope
Deployment Name: gpt-4o
```
---

### 2.3 Option B: AWS Bedrock (Gleichwertig unterstützt)

**AWS Bedrock ist eine vollwertige Alternative zu Azure OpenAI.**

**Vorteile:**
- ✅ Exzellente Claude-Modelle von Anthropic
- ✅ Verfügbarkeit in EU-Regionen (Frankfurt, Irland)
- ✅ Gute Integration für AWS-Kunden
- ✅ Häufig kostengünstiger als Azure OpenAI

**Voraussetzungen:**
- Aktives AWS-Konto
- Zugriff auf AWS Bedrock Service
- Ausreichende IAM-Berechtigungen

#### Schritt-für-Schritt: AWS Bedrock aufsetzen

##### Schritt 1: AWS Bedrock aktivieren

1. **AWS Console öffnen:** [https://console.aws.amazon.com](https://console.aws.amazon.com)
2. **Zu Bedrock navigieren:** Suchen Sie nach "Bedrock"
3. **Region wählen:** z. B. eu-central-1 (Frankfurt), us-east-1
4. **Model Access aktivieren:**
   - Navigieren Sie zu "Model access"
   - Klicken Sie auf "Manage model access"
   - Aktivieren Sie Anthropic Claude-Modelle
   - Klicken Sie auf "Save changes"

##### Schritt 2: IAM-Berechtigungen einrichten

Erstellen Sie einen IAM-User mit folgender Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": "arn:aws:bedrock:*::foundation-model/*"
    }
  ]
}
```

##### Schritt 3: Access Keys generieren

1. IAM Console → Users → Ihr User → Security credentials
2. Create access key
3. **Notieren Sie für die spätere Konfiguration in der Appllikation:**
   - AWS Access Key ID
   - AWS Secret Access Key
   - AWS Region
   - Model ID (z. B. `anthropic.claude-3-5-sonnet-20240620-v1:0`)

**Beispiel:**
```
AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG...
AWS Region: eu-central-1
Model ID: anthropic.claude-3-5-sonnet-20240620-v1:0
```
---

### 2.4 Option C: Lokale Modelle (für höchste Datenschutzanforderungen)

**Für Kunden, die keine Cloud-Anbindung wünschen.**

Die Plattform unterstützt lokale LLMs über **OpenAI-kompatible APIs**.

**Unterstützte Tools:**
- **Ollama** (einfachste Lösung) – https://ollama.ai
- **vLLM** (höchste Performance)
- **LM Studio** (GUI-basiert)

**Empfohlene Modelle:**
- **Llama 3.1 70B / Llama 3.3 70B** (sehr gute Qualität)
- **Qwen 2.5 72B** (exzellent, multilingual)
- **Mistral Large 2** (hohe Qualität)

**Hardware-Anforderungen:**

| Modell | VRAM | GPU | RAM |
|--------|------|-----|-----|
| 70B Modelle | 48-80 GB | A100/H100 | 128 GB |
| 8B Modelle | 8-16 GB | RTX 4060 Ti | 16 GB |

**Setup-Beispiel mit Ollama:**
```bash
# Ollama installieren
curl -fsSL https://ollama.ai/install.sh | sh

# Modell herunterladen
ollama pull llama3.1:70b

# API läuft auf http://localhost:11434/v1 (OpenAI-kompatibel)
```

---

### 2.5 Token-Rate-Limits dimensionieren

**Faustregeln für TPM (Tokens per Minute):**

| Nutzerszenario | Empfohlenes TPM | Begründung |
|----------------|-----------------|------------|
| 1-5 Nutzer (Test) | 30.000 - 50.000 | Ausreichend für Tests |
| 10-25 Nutzer | 80.000 - 120.000 | Moderate Nutzung |
| 25-50 Nutzer | 150.000 - 250.000 | Produktivumgebung |
| 50+ Nutzer | 300.000+ | Hohe Last |

**Aufgabe des Kunden:**
- **Azure OpenAI:** Token-Limits in Azure AI Foundry anpassen
- **AWS Bedrock:** Service Quotas in AWS Console prüfen
- **Lokale Modelle:** Ausreichende GPU-Kapazität bereitstellen

---

### 2.6 Kostenübersicht LLM-Provider

**Azure OpenAI (ca., Stand 2026):**
- GPT-4o: $5-15 pro 1M Token
- GPT-4o-mini: $0.15-0.60 pro 1M Token
- **Typisch (50 Nutzer): $100-500/Monat**

**AWS Bedrock (ca., Stand 2026):**
- Claude 3.5 Sonnet: $3-8 pro 1M Input, $15-24 pro 1M Output
- Claude 3 Haiku: $0.25-1 pro 1M Input, $1.25-5 pro 1M Output
- **Typisch (50 Nutzer): $80-400/Monat**

**Lokale Modelle:**
- Keine API-Kosten
- Hardware-Investment: €5.000-50.000+
- Laufende Stromkosten

---

## 3. Serveranforderungen 

### 3.1 Einfache Umgebung (8 GB RAM)

**Geeignet für:** Test- und PoC-Umgebungen

- **RAM:** 8 GB
- **CPU:** 4 vCPUs
- **Storage:** 250 GB SSD
- **Szenarien:** Bis 100 MB Dokumente, 1-5 Nutzer

### 3.2 Mittlere Umgebung (32 GB RAM)

**Geeignet für:** Produktivumgebungen

- **RAM:** 32 GB
- **CPU:** 8–12 vCPUs
- **Storage:** 1 TB SSD
- **Szenarien:** Bis 5 GB Dokumente, 10-50 Nutzer

### 3.3 Große Umgebung (64 GB RAM)

**Geeignet für:** Große Produktivumgebungen

- **RAM:** 64 GB
- **CPU:** 16–32 vCPUs
- **Storage:** 2 TB SSD
- **Szenarien:** Über 5 GB Dokumente, 50+ Nutzer

**Hinweis bei lokalen LLMs:** Die oben genannten Anforderungen gelten für die Anwendung. Lokale LLM-Server benötigen **zusätzliche** GPU-Hardware (siehe Abschnitt 2.4).

---

## 4. Systemkomponenten 

Die Lösung besteht aus folgenden Docker-Containern:

- **Web-/API-Server:** CGS Assist Backend, LLM API, RAG API, Caddy
- **Worker:** Celery-Worker für asynchrone Tasks
- **Redis:** Message-Broker und Caching
- **Datenbank:** SQLite
- **Vector-Store:** ChromaDB für RAG

**Deployment:** Über Docker Compose auf Linux-Server.

## 5. Datenbank und Storage 

**Datenhaltung:**
- **SQLite:** Benutzerdaten, Metadaten, Logs
- **SHARED_FOLDER:** Hochgeladene Dokumente, Konfigurationsdateien und Mount zum Docker
- **ChromaDB:** Embeddings für RAG

**Alle Daten liegen auf dem Kundenserver.**

---

## 6. Authentifizierung 

- Lokale Benutzer-/Passwortverwaltung
- Azure AD/Entra (OIDC/SAML)
- API-Token für Integrationen

---

## 7. Security und Compliance 

### 7.1 TLS/HTTPS

Obligatorisch für Produktion. Zertifikate:
- Let's Encrypt (via Caddy)
- DNS-01 (API-Token Provider)
- Eigene Zertifikate (interne CA)

### 7.2 Firewall

**Ausgehende Verbindungen:**
- **Immer:** Docker Hub / CGS Registry
- **Je nach LLM-Provider:**
  - Azure OpenAI: `*.openai.azure.com`
  - AWS Bedrock: `bedrock-runtime.<region>.amazonaws.com`
  - Lokal: Interne Verbindung

### 7.3 Datenschutz

**Anwendungsdaten:** Ausschließlich auf Kundenserver

**LLM-Provider (Cloud):**
- Azure OpenAI / AWS Bedrock: Anfragen werden übertragen, aber nicht dauerhaft gespeichert
- **Empfehlung:** EU-Region wählen (DSGVO)

**Lokale Modelle:** Höchste DSGVO-Konformität

---

## 8. Backup und Recovery 

**Vom Kunden zu sichern:**
- Datenbank
- SHARED_FOLDER 
- Konfigurationsdateien

---

## 9. Patch-Management 

- **Anwendung:** CGS stellt Updates bereit → Kunde spielt ein
- **Infrastruktur:** Kunde verantwortlich (OS, Docker)

---

## 10. RACI-Matrix 

| Bereich | CGS | Kunde |
|---------|-----|-------|
| **Infrastruktur** | Consult | Responsible |
| **Docker Images** | Responsible | Informed |
| **LLM-Provider Setup** | Consult | Responsible |
| **LLM-Provider Kosten** | Informed | Accountable |
| **Datenbank** | Consult | Responsible |
| **Backups** | Consult | Responsible |
| **Applikations-Updates** | Responsible | Accountable |
| **Security-Patches (OS)** | Consult | Responsible |

---

## 11. FAQ 

### **Wird ein LLM mitgeliefert?**

**Nein.** Kunde muss selbst einen LLM-Provider bereitstellen (Azure OpenAI, AWS Bedrock oder lokales Modell).

### **Welchen LLM-Provider soll ich wählen?**

**Empfehlung:**
1. **Azure OpenAI** (bevorzugt) – wenn Sie Azure nutzen
2. **AWS Bedrock** (gleichwertig) – wenn Sie AWS nutzen
3. **Lokale Modelle** – höchster Datenschutz

**Alle drei werden vollständig unterstützt.**

### **Ist Azure OpenAI zwingend?**

**Nein.** Azure ist bevorzugt, aber **nicht zwingend**. AWS Bedrock und lokale Modelle sind gleichwertige Alternativen.

### **Wie setze ich Azure OpenAI auf?**

Siehe Abschnitt 2.2 für Schritt-für-Schritt-Anleitung.

### **Wie setze ich AWS Bedrock auf?**

Siehe Abschnitt 2.3 für Schritt-für-Schritt-Anleitung.

### **Benötigt die Lösung Internet?**

- **Cloud-Provider (Azure/AWS):** Ja
- **Lokale Modelle:** Nur für Installation

### **Wo liegen die Daten?**

Alle Anwendungsdaten auf Kundenserver. Bei Cloud-LLMs werden Anfragen zur Analyse übertragen (nicht dauerhaft gespeichert).

### **Müssen wir für den LLM bezahlen?**

**Ja** (bei Cloud):
- Azure OpenAI: $100-500/Monat (50 Nutzer)
- AWS Bedrock: $80-400/Monat (50 Nutzer)
- Lokal: €0 API-Kosten, aber Hardware-Investment

---

## 12. Checkliste 

### LLM-Provider Setup (wählen Sie EINEN)

**Option A: Azure OpenAI (empfohlen)**
- [ ] Azure-Abonnement verfügbar
- [ ] Ressource erstellt
- [ ] Modell deployed (gpt-4o)
- [ ] API-Key + Endpoint notiert

**Option B: AWS Bedrock (gleichwertig)**
- [ ] AWS-Konto verfügbar
- [ ] Model Access aktiviert
- [ ] IAM-Credentials erstellt
- [ ] Access Keys + Model ID notiert

**Option C: Lokales Modell**
- [ ] GPU-Server bereitgestellt
- [ ] Ollama/vLLM installiert
- [ ] Modell heruntergeladen
- [ ] API-Endpoint getestet

### Infrastruktur
- [ ] Linux-Server bereitgestellt
- [ ] Docker + Docker Compose installiert
- [ ] SHARED_FOLDER erstellt
- [ ] Firewall-Freigaben (Ports 80, 443)
- [ ] DNS konfiguriert (für Caddy)
- [ ] Ausgehende Verbindungen freigegeben

---

**Version:** 2.0  
**Stand:** Januar 2026  
**Erstellt von:** CGS GmbH

