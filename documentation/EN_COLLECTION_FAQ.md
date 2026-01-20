## Collection of Frequently Asked Questions

This document is for collecting frequently asked questions covering all areas of the project (e.g., delivery, provisioning). This FAQ list can be extended for each customer and project with specific questions/answers.

| Question|	Answer|
|----|----|
|What happens in case of serious incidents?|	A defined incident management process is used for serious disruptions (e.g., escalation path, defined response times). The exact structure is defined in the contractual agreements and SLAs.|
|How does an update work?|	Updates are announced beforehand and coordinated with the customer. Depending on the operating model, new container images are provided and rolled out during a scheduled maintenance window. Critical security updates may take place at short notice after consultation.|
|Can we use our own security tools (VPN, proxy, IDS/IPS)?|	Yes, existing customer security mechanisms are generally integrated. Early coordination regarding firewall/proxy rules and, if applicable, SSL inspection is important to ensure functionality (e.g., connections to external LLM APIs or container registries).|
|Who is responsible for ongoing operations?|	This depends on the selected operating model. For SaaS/Managed Service, CGS predominantly manages the technical operation; for on-premises/customer operation, the customer is responsible for infrastructure, OS, Docker/Kubernetes. CGS supports with application updates and configuration. For details, see the responsibility overview in the IT requirements.|
|How long does standard delivery take?|	For a typical pilot operation (PoC), plan for about 2–6 weeks, provided infrastructure is available and approvals are swift. Transition to production depends heavily on the customer's internal processes (change management, security reviews).|
|How is the solution provided?|	Via a Docker Compose script on a Linux server.|
|Is special Azure integration required?|	No. Only an API key and endpoint are required.|
|Is a VM required?|	No. Any Linux server with Docker Compose is suitable.|
|Does the solution require a permanent internet connection?|	Cloud provider (Azure/AWS): Yes|
|	|	Local models: Only for installation|
|Is an LLM included?|	The customer must provide an LLM provider (Azure OpenAI, AWS Bedrock, or a local model).|
|Which LLM provider should I choose?|	Recommendation:|
|	|Azure OpenAI (preferred) if you use Azure|
|	|AWS Berock (equivalent) if you use AWS|
|	|Local models – highest data protection|
|	|All three are fully supported|
|Is Azure OpenAI mandatory?|	No. Azure is preferred but not required. AWS Bedrock and local models are valid alternatives.|
|How do I set up Azure OpenAI?|	See IT_REQUIREMENTS_AI_AUDIT_ASSIST_DE_EN.md, section 2.2 for step-by-step instructions.|
|How do I set up AWS Bedrock?|	See IT_REQUIREMENTS_AI_AUDIT_ASSIST_DE_EN.md, section 2.3 for step-by-step instructions.|
|Where is the data stored?|	All application data is on the customer's server. With cloud LLMs, requests are transmitted for analysis (not stored permanently).|
|Do we have to pay for the LLM?|	Yes. The customer must provide the LLM provider for the cloud:|
|	|Azure OpenAI: $100–500/month (50 users)|
|	|AWS Bedrock: $80–400/month (50 users)|
|	|Local: €0 API fees, but hardware investment required|
|How is authentication handled?|	Via local user/password management (customer’s LDAP, customer’s database)|
|	|Via Microsoft Entra ID (formerly Azure AD/Entra) (OIDC/SAML)|
|	|API token for integrations|
|How is access secured?|	Security via certificates (TLS/HTTPS)|
|	|Let’s Encrypt (via Caddy)|
|	|Own certificates (internal CA)|
