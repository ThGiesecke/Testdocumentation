# AI Audit Assist – Technical Deployment Requirements

This document is intended for IT infrastructure and system administrators and describes the technical requirements for the deployment and operation of the AI Audit Assist platform at the customer’s premises.

## IMPORTANT: LLM Deployment by the Customer

**The AI Audit Assist platform is delivered WITHOUT a built-in Large Language Model (LLM).**

The customer must provide their own LLM provider. The following options are supported:

| LLM Provider         | Status            | Recommendation                      |
|----------------------|-------------------|-------------------------------------|
| **Azure OpenAI**     | Fully supported   | **Recommended** for cloud deployments |
| **AWS Bedrock**      | Fully supported   | Suitable for AWS customers          |
| **Local Models**     | Supported         | For maximum data privacy requirements |

**Note:** Azure OpenAI is our preferred recommendation, but AWS Bedrock and local models are equally supported.

## 1. General Requirements
 
### 1.1 Infrastructure (VM or Server)

The solution is provided on a **Linux-based server** (local VM or cloud VM). **No specific cloud integration is required** – any Linux server with Docker Compose is sufficient.

**Important:** A VM is **not mandatory**. The solution can run on any Linux server with Docker Compose.

### 1.2 Operating System

- **Linux**, e.g. Ubuntu Server LTS (24.04) or comparable distributions
- Tested with Ubuntu; other Linux distributions available upon request

### 1.3 Docker Environment

Deployment is managed via **Docker Compose**.

**Required versions:**

- **Docker Engine:** Version gt 20.x
- **Docker Compose:** Version gt 1.29 or Compose V2

**Installation:** The customer must install and configure Docker and Docker Compose on the server.

### 1.4 Shared Directory (SHARED_FOLDER)

A **writable directory** on the server for data exchange between containers and for persistent data (documents, databases, vector store).

**Customer responsibility:** Provide a directory with sufficient read/write permissions for the Docker containers.

### 1.5 Internet Access / Proxy

**Required for:**

- Downloading Docker images (during installation)
- Connecting to the LLM provider API (Azure OpenAI, AWS Bedrock – if cloud providers are chosen)

**Proxy support:** The solution supports operation behind a proxy. Proxy configuration can be set using environment variables.

**Important:** 

- **Cloud LLM Providers (Azure OpenAI, AWS Bedrock):** A permanent internet connection is required
- **Local Models:** Internet only needed for installation (Docker image download)

### 1.6 Network / Ports

**Required ports:**

| Port  | Protocol | Usage                                         | Required for              |
|-------|----------|-----------------------------------------------|---------------------------|
| **80**   | HTTP  | HTTP to HTTPS redirect, Let's Encrypt ACME challenge 	| Production with Caddy     |
| **443**  | HTTPS | Encrypted access (TLS)                      			| Production (recommended)  |
| **8000** | HTTP  | Direct access to the application            			| Testing/Development       |

**Production recommendation:** Use a reverse proxy (e.g. Caddy, Nginx) with TLS certificates.

**DNS requirements for Caddy with Let's Encrypt:**

If you want to use Caddy with automatic Let’s Encrypt TLS certificates:

- **FQDN required:** Application must be reachable via a fully qualified domain name (e.g., `ai-audit-assist.yourcompany.com`)
- **DNS resolution:** The FQDN must resolve to the public IP address of the server (A-record or CNAME)
- **Port 80 accessible:** Must be reachable from the internet (for ACME HTTP challenge)

**Caddyfile configuration example:**

~~~json
ai-audit-assist.yourcompany.com {
    reverse_proxy cgs_assist_backend:8000
}
~~~

**Alternative DNS-01 (publicly trusted, without open 80/443):**

Certificate requests are verified via DNS entries instead of HTTP content

Provider syntax example, Cloudflare token as Env-Var:

~~~json
texttest.server.com {reverse_proxy cgs_assist_server:8000 {
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
~~~

**Important:** The "caddy-dns" plugin must be included in the Caddy server's configuration, otherwise "dns cloudflare" will not be found.

Own certificate example (PEM + Key):

If a certificate exists (e.g., from your company PKI or manually generated), integrate it directly:

~~~json
texttest.server.com {reverse_proxy cgs_assist_server:8000 header {
    Strict-Transport-Security "max-age=31536000; includeSubDomains" -Server}    
    # Certificate + Private Key (PEM)    
    tls /etc/caddy/certs/test.server.com.fullchain.pem /etc/caddy/certs/test.server.com.key
    }
~~~

**Important:** The certificate chain must be complete (typically “fullchain”) and the SANs (Subject Alternative Names) must match the hostname.

**Alternative without public DNS:**

Use own TLS certificates (e.g., from internal CA)
Access only via IP address: http://<server-ip>:8000 (recommended only for testing/development)

**Customer responsibility:**

- Firewall rules for ports 80 and 443 (production) or 8000 (testing)
- DNS configuration: A-record for the desired FQDN (e.g., 'ai-audit-assist.yourcompany.com') pointing to server IP
- Provide FQDN for Caddyfile configuration
- Optional: Own TLS certificates if Let's Encrypt is not used

## 2. LLM Provider – Setup by the Customer

*IMPORTANT:* AI Audit-Assist does **not provide an LLM**. The customer must set up and operate one of the following LLM providers.

### 2.1 Overview of Supported LLM Providers
|Provider		|Recommendation		|Benefits											|Setup-Complexity	|
|----- 			|----- 				|-----												|----				|	
|Azure OpenAI	|Preferred			|Enterprise support, EU regions, easy integration	|	Medium			|
|AWS Bedrock	|Equally supported	|Excellent Claude models, AWS integration			|	Medium			|
|Local Models	|For high privacy	|Maximum data protection, no cloud					|	High (GPU required)|

**Select a provider based on:**

- Existing cloud infrastructure (Azure or AWS)
- Data privacy and compliance requirements
- Budget and operating model

### 2.2 Option A: Azure OpenAI (Recommended)

**Why Azure OpenAI is recommended:**

- Proven enterprise integration
- EU-region availability (GDPR-compliant)
- Microsoft enterprise support
- Proven GPT models

**Prerequisites:**

- Active Azure subscription
- Approval for Azure OpenAI Service (may require approval)
- Sufficient permissions to create resources

#### Step-by-step: Setting up Azure OpenAI Service

##### Step 1: Create Azure OpenAI Resource

1. **Open Azure Portal:** https://portal.azure.com
2. **Create resource:**
- Click "+ Create a resource"
- Search for "Azure OpenAI"
- Click "Create"
3. **Configure resource:**
- **Subscription:** Select your Azure subscription
- **Resource group:** Create or select one
- **Region:** Select a region (e.g., West Europe, North Europe)
- **Name:** Assign a unique name (e.g., ai-audit-assist-openai-prod)
- **Pricing tier:** Standard S0 or higher
- **Review and create:** Click "Review + Create" and then "Create"

##### Step 2: Deploy models in Azure AI Foundry
1. **Open Azure AI Foundry:** https://ai.azure.com
- Deploy model:
- Go to "Deployments"
- Click "+ Create new deployment"
2. **Recommended models:**
- Main model: gpt-4o
- Optional: gpt-5 or higher

##### Step 3: Retrieve API credentials
- In Azure Portal → Your OpenAI Resource → "Keys and Endpoint"
- **Note for later app configuration:**
- API Key
- Endpoint (e.g., https://your-resource-name.openai.azure.com/)
- Region (e.g., westeurope)
- Model deployment names

**Example:**

~~~bash
Endpoint: https://ai-audit-assist-openai-prod.openai.azure.com/
API Key: 1234567890abcdef...
Region: westeurope
Deployment Name: gpt-4o
~~~

### 2.3 Option B: AWS Bedrock (Equally Supported)

**AWS Bedrock is a full alternative to Azure OpenAI.**

**Benefits:**

- Excellent Claude models from Anthropic
- Available in EU regions (Frankfurt, Ireland)
- Good integration for AWS customers
- Often less expensive than Azure OpenAI

**Prerequisites:**

- Active AWS account
- Access to AWS Bedrock Service
- Sufficient IAM permissions

#### Step-by-step: Setting up AWS Bedrock

##### Step 1: Enable AWS Bedrock

1. **Open AWS Console:** https://console.aws.amazon.com
2. **Navigate to Bedrock:** Search for "Bedrock"
3. **Select region:** e.g., eu-central-1 (Frankfurt), us-east-1
4. **Model Access:**
- Go to "Model access"
- Click "Manage model access"
- Enable Anthropic Claude models
- Click "Save changes"

##### Step 2: Set up IAM permissions

Create an IAM user with the following policy:

~~~json
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
~~~

##### Step 3: Generate access keys

IAM Console → Users → Your User → Security credentials

1. Create access key
2. **Note for later app configuration:**
- AWS Access Key ID
- AWS Secret Access Key
- AWS Region
- Model ID (e.g., anthropic.claude-3-5-sonnet-20240620-v1:0)

**Example:**

~~~bash
AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG...
AWS Region: eu-central-1
Model ID: anthropic.claude-3-5-sonnet-20240620-v1:0
~~~

##### 2.4 Option C: Local Models (for maximum data privacy)

**For customers who do not want any cloud connectivity.**

The platform supports local LLMs via OpenAI-compatible APIs.

**Supported tools:**

- **Ollama** (easiest solution) – https://ollama.ai
- **vLLM** (highest performance)
- **LM Studio** (GUI-based)

**Recommended models:**

- **Llama 3.1 70B / Llama 3.3 70B** (high quality)
- **Qwen 2.5 72B** (excellent, multilingual)
- **Mistral Large 2** (high quality)

**Hardware requirements:**
|Model		|VRAM		|GPU		|RAM	|
|-----		|-----		|-----		|-----	|
|70B Models	|48-80 GB 	|A100/H100	|128 GB	|
|8B Models	|8–16 GB	|RTX 4060 Ti|16 GB	|

**Setup example with Ollama:**

~~~bash
#Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh
#Download model
ollama pull llama3.1:70b
#API runs at http://localhost:11434/v1 (OpenAI-compatible)
~~~

### 2.5 Sizing Token Rate Limits

**TPM (Tokens per Minute) guidelines:**

|User scenario		|Recommended TPM	|Reason					|
|-----				|-----				|-----					|
|1-5 users (test)	|30,000 – 50,000	|Sufficient for testing	|
|10-25 users		|80,000 – 120,000	|Moderate usage			|
|25-50 users		|150,000 – 250,000	|Production				|
|50+ users			|300,000+			|High load				|

**Customer responsibility:**

- **Azure OpenAI:** Adjust token limits in Azure AI Foundry
- **AWS Bedrock:** Check service quotas in AWS Console
- **Local Models:** Provide sufficient GPU capacity

### 2.6 LLM Provider Cost Overview

**Azure OpenAI (approx., as of 2026):**

- GPT-4o: $5–15 per 1M tokens
- GPT-4o-mini: $0.15–0.60 per 1M tokens
- **Typical (50 users): $100–500/month**

**AWS Bedrock (approx., as of 2026):**

- Claude 3.5 Sonnet: $3–8 per 1M input, $15–24 per 1M output
- Claude 3 Haiku: $0.25–1 per 1M input, $1.25–5 per 1M output
- **Typical (50 users): $80–400/month**

**Local Models:**

- No API costs
- Hardware investment: €5,000–50,000+
- Ongoing electricity costs

## 3. Server Requirements

### 3.1 Simple Environment (8 GB RAM)

**Suitable for:** Test and PoC environments

- **RAM:** 8 GB
- **CPU:** 4 vCPUs
- **Storage:** 250 GB SSD
- **Use case:** Up to 100 MB documents, 1–5 users

### 3.2 Medium Environment (32 GB RAM)

**Suitable for:** Production environments

- **RAM:** 32 GB
- **CPU:** 8–12 vCPUs
- **Storage:** 1 TB SSD
- **Use case:** Up to 5 GB documents, 10–50 users

### 3.3 Large Environment (64 GB RAM)

**Suitable for:** Large production environments

- **RAM:** 64 GB
- **CPU:** 16–32 vCPUs
- **Storage:** 2 TB SSD
- **Use case:** Over 5 GB documents, 50+ users

**Note for local LLMs:** The requirements above apply to the application. Local LLM servers need additional GPU hardware (see section 2.4).

## 4. System Components

The solution consists of the following Docker containers:

- **Web/API Server:** AI Audit-Assist backend, LLM API, RAG API, Caddy
- **Worker:** Celery worker for asynchronous tasks
- **Redis:** Message broker and caching
- **Database:** SQLite
- **Vector Store:** ChromaDB for RAG

**Deployment:** Via Docker Compose on a Linux server.

## 5. Database and Storage

**Data storage:**

- **SQLite:** User data, metadata, logs
- **SHARED_FOLDER:** Uploaded documents, configuration files, Docker mount
- **ChromaDB:** Embeddings for RAG

**All data resides on the customer's server.**

## 6. Authentication

- Local user/password management
- Azure AD/Entra (OIDC/SAML)
- API tokens for integrations

## 7. Security and Compliance

### 7.1 TLS/HTTPS

Mandatory for production. Certificates:

- Let’s Encrypt (via Caddy)
- DNS-01 (API token provider)
- Own certificates (internal CA)

### 7.2 Firewall

**Outgoing connections:**

- **Always:** Docker Hub / CGS Registry
- **Depending on LLM provider:**
	- Azure OpenAI: *.openai.azure.com
	- AWS Bedrock: bedrock-runtime.<region>.amazonaws.com
	- Local: Internal connection
	
### 7.3 Data Privacy

**Application data:** Exclusively on the customer server

**LLM provider (cloud):**

- Azure OpenAI / AWS Bedrock: Requests are transmitted but not stored permanently
- **Recommendation:** Choose EU region (GDPR)

**Local models:** Highest GDPR compliance

## 8. Backup and Recovery

**Customer responsibility to back up:**

- Database
- SHARED_FOLDER
- Configuration files

## 9. Patch Management

- **Application:** CGS provides updates → customer installs them
- **Infrastructure:** Customer responsibility (OS, Docker)

## 10. RACI Matrix

|Area					|CGS		|Customer	|
|----					|----		|----		|
|Infrastructure			|Consult	|Responsible|
|Docker Images			|Responsible|Informed	|
|LLM Provider Setup		|Consult	|Responsible|
|LLM Provider Costs		|Informed	|Accountable|
|Database				|Consult	|Responsible|
|Backups				|Consult	|Responsible|
|App Updates			|Responsible|Accountable|
|Security Patches (OS)	|Consult	|Responsible|

<div class="pagebreak"></div>

## 11. FAQ

### Is an LLM included?

**No.** The customer must provide their own LLM provider (Azure OpenAI, AWS Bedrock, or a local model).

### Which LLM provider should I choose?

**Recommendation:**

1. **Azure OpenAI** (preferred) – if you use Azure
2. **AWS Bedrock** (equivalent) – if you use AWS
3. **Local models** – for maximum data privacy 

**All three options are fully supported.**

### Is Azure OpenAI mandatory?

**No.** Azure is preferred, but not required. AWS Bedrock and local models are equivalent alternatives.

### How do I set up Azure OpenAI?

See section 2.2 for a step-by-step guide.

### How do I set up AWS Bedrock?

See section 2.3 for a step-by-step guide.

### Does the solution require internet access?

- **Cloud providers (Azure/AWS):** Yes  
- **Local models:** Only for installation

### Where is the data stored?

All application data is stored on the customer server. For cloud LLMs, requests are sent for processing but are not stored permanently.

### Do we have to pay for the LLM?

**Yes**(for cloud providers):

- Azure OpenAI: $100-500/month (50 users)
- AWS Bedrock: $80-400/month (50 users)
- Local: €0 API costs, but hardware investment required

<div class="pagebreak"></div>

## 12. Checklist

**LLM Provider Setup (choose ONE)**

**Option A: Azure OpenAI (recommended)**

- [ ] Azure subscription available
- [ ] Resource created
- [ ] Model deployed (gpt-4o)
- [ ] API key + endpoint saved

**Option B: AWS Bedrock (equivalent)**

- [ ] AWS account available
- [ ] Model access enabled
- [ ] IAM credentials created
- [ ] Access keys + model ID saved

**Option C: Local Model**

- [ ] GPU server ready
- [ ] Ollama/vLLM installed
- [ ] Model downloaded
- [ ] API endpoint tested

**Infrastructure**

- [ ] Linux server provided
- [ ] Docker + Docker Compose installed
- [ ] SHARED_FOLDER created
- [ ] Firewall rules (ports 80, 443)
- [ ] DNS configured (for Caddy)
- [ ] Outbound connections allowed

---

**Version:** 2.0  
**Status:** Januar 2026  
**Author:** CGS GmbH