# Rocketlane stack study — Workstream 2: vendor-published documentation

Scope: what Rocketlane itself publishes. That covers developer.rocketlane.com (ReadMe), help.rocketlane.com (Freshdesk), rocketlane.com/legal/*, the Trust page and its security PDF, and the Beamer changelog. Retrieval date for everything: **2026-11-01**. Collection was passive: public pages only, no login, and robots.txt was checked for all three hosts before fetching. help.rocketlane.com disallows `/support/search`, so articles were found through its sitemap instead.

Grades: **CONFIRMED** = vendor states it · **OBSERVED** = seen in an artefact (spec, header, URL, asset host) · **SECONDARY** = third party · **INFERRED** = my reasoning, shown · **UNKNOWN** = searched, not found.

Evidence IDs (D = developer docs, H = help centre, L = legal/trust, P = security PDF, B = Beamer) point to the ledger in section 5.

---

## 1. Findings table

| Q# | Component | Technology / answer | Grade | Evidence | URL | Date |
|---|---|---|---|---|---|---|
| B11 | API style | REST over HTTPS with some RPC-style action endpoints (`/projects/{id}/archive`); JSON; TLS 1.2 | CONFIRMED | D01 D02 D04 | https://developer.rocketlane.com/docs/overview.md | 2026-11-01 |
| B11 | API versioning | Version in the URL path: `https://api.rocketlane.com/api/1.0/...`. Backward-compatible changes stay on 1.0; breaking changes increment the version | CONFIRMED | D01 | https://developer.rocketlane.com/docs/overview.md | 2026-11-01 |
| B11 | API auth | Static API key in an `api-key` request header, from Settings › API. OpenAPI `securitySchemes` type `apiKey`, in `header` | CONFIRMED (docs) / OBSERVED (spec) | D08 | https://developer.rocketlane.com/reference/get-task.md | 2026-11-01 |
| B11 | API spec | OpenAPI **3.0.1** "Rocketlane API" v1.0 embedded in every ReadMe reference page | OBSERVED | D07 | https://developer.rocketlane.com/reference/get-task.md | 2026-11-01 |
| B11 | API servers | Production `api.rocketlane.com/api`; **Staging `api.nitro.run/api`**; "Orbits" `project.api.orbits.run/api` | OBSERVED | D07 D07b | https://developer.rocketlane.com/reference/get-task.md | 2026-11-01 |
| B11 | Pagination | Cursor/token: `pageSize` (default and max 100), `pageToken`, response `pagination{hasMore,nextPageToken,nextPage,totalRecordCount}`. Token valid 15 min. Sample tokens are UUIDs | CONFIRMED | D05 | https://developer.rocketlane.com/docs/pagination.md | 2026-11-01 |
| B11 | Filtering | Query params `field.op=value` (`eq, gt, lt, ge, le, cn, nc`), custom fields `{type}.field.{id}.value` | CONFIRMED | D06 | https://developer.rocketlane.com/docs/search.md | 2026-11-01 |
| B11 | ID format | Integer IDs, `int64` (e.g. `taskId: 278`). Event IDs are UUIDs | OBSERVED | D10 D17 | https://developer.rocketlane.com/reference/get-task.md | 2026-11-01 |
| B11 | Error shape | `{"errors":[{"errorCode":"NOT_FOUND","errorMessage":"..."}]}`; codes such as `BAD_REQUEST`, `NOT_AUTHORIZED`, `INTERNAL_SERVER_ERROR` | OBSERVED | D09 | https://developer.rocketlane.com/reference/get-task.md | 2026-11-01 |
| B11 | Request tracing | `X-Request-Id` response header on every request | CONFIRMED | D03 | https://developer.rocketlane.com/docs/overview.md | 2026-11-01 |
| B11 | Rate limits | Up to **100 requests/min per user/endpoint**, 60-second rolling window, and rejected calls count against it. Public **search APIs 60 req/min**. Rate-limit response *headers* not documented | CONFIRMED | H14 H15 | https://help.rocketlane.com/support/solutions/articles/67000755761-why-am-i-getting-a-rate-limit-error-during-bulk-updates- | 2026-11-01 |
| B11 | UI uses per-item API calls | Bulk task updates in the UI "send individual API requests for each task", so the web app calls the same rate-limited API per item | CONFIRMED (statement) / INFERRED (architecture: SPA over a REST API, no batch endpoint) | H14 | https://help.rocketlane.com/support/solutions/articles/67000755761-why-am-i-getting-a-rate-limit-error-during-bulk-updates- | 2026-11-01 |
| B12 | Automation engine | Rules built from trigger, condition and action. Triggers: project, task, phase, form, schedule. Run-metered: Essential 50, Standard 200, Premium 500, Enterprise 1000 runs/user/month. Stops at the limit. Per-action Automation Logs. Underlying engine or queue **UNKNOWN** | CONFIRMED (behaviour) / UNKNOWN (tech) | H21 H22 H23 | https://help.rocketlane.com/support/solutions/articles/67000724434-automation-pricing | 2026-11-01 |
| B12 | Queues / schedulers | Not named in any vendor doc | UNKNOWN | — | — | 2026-11-01 |
| B13 | Webhook trigger model | Webhooks are an **automation action** ("Send webhook"), and only task triggers are supported. Payload `resourceType` lists only TASK; the `eventType` list includes PROJECT_* and PHASE_* | CONFIRMED | D12 | https://developer.rocketlane.com/docs/setting-up-webhooks-in-rocketlane.md | 2026-11-01 |
| B13 | Webhook auth / signing | HTTPS only; receiver auth is **Basic Auth only** (optional). **No HMAC signature** documented | CONFIRMED (Basic) / UNKNOWN (signing: none documented) | D11 D16 | https://developer.rocketlane.com/docs/setting-up-webhooks-in-rocketlane.md | 2026-11-01 |
| B13 | Webhook delivery | Success = 200 or 204 only. Failure = any other code or no response in 10 s. **Ordering not guaranteed.** Docs claim "exactly one delivery per event". **Retry policy not documented** | CONFIRMED / UNKNOWN (retries) | D13 D14 D15 | https://developer.rocketlane.com/docs/webhook-delivery.md | 2026-11-01 |
| B13 | Webhook payload | Envelope `{eventId(UUID), resourceType, eventType, version:"1.0", data, changeLog{changedFields,from,to}, source{webhook,actor}, eventOccurredAt, eventTriggeredAt}` (epoch ms) | CONFIRMED | D17 | https://developer.rocketlane.com/docs/event-payload.md | 2026-11-01 |
| B13 | Why this webhook design | Not stated by vendor. *Inferred:* building webhooks as an automation action reused the existing automation runtime (launched Sep 2023) for task webhooks (Feb 2024), at the cost of task-only scope and no signing | INFERRED | B01 B02 | https://app.getbeamer.com/rocketlaneapp/en | 2026-11-01 |
| B14 | Search | Global Search (Ctrl/Cmd+K) covers tasks, projects, chats, files and spaces. The API notes "a couple of seconds" before changes appear in Search, which points to an asynchronously indexed search store (*inferred*). Engine **UNKNOWN** | CONFIRMED (feature) / INFERRED (async index) / UNKNOWN (engine) | H24 D06 | https://help.rocketlane.com/support/solutions/articles/67000753390-using-global-search-in-rocketlane | 2026-11-01 |
| B15 | File storage | All customer data is stored in AWS and encrypted at rest. Files cannot be permanently deleted by users. Object store (S3) and size limits **not stated** | CONFIRMED (AWS) / INFERRED (S3) / UNKNOWN (limits) | P04 H25 | https://help.rocketlane.com/support/solutions/articles/67000737703-can-documents-and-files-sent-in-all-files-and-conversations-be-deleted- | 2026-11-01 |
| C16 | Primary database | Not named. PDF says "backed up every day across all databases" (plural) | UNKNOWN (engine) | P03 | — | 2026-11-01 |
| C17 | Multi-tenancy | "Pure multi-tenant SaaS", accounts "logically isolated" at the data layer. That implies a shared store with a tenant key (*inferred*); schema-per-tenant is not ruled out by the text | CONFIRMED (pooled multi-tenant) / INFERRED (shared schema) | P02 | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | 2026-11-01 |
| C18 | Analytics / reporting store | **ClickHouse**: "Product analytics, dashboards, and reporting" | CONFIRMED | L09 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| C18 | Product analytics | **Mixpanel** | CONFIRMED | L06 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| C18 | Caching layer | Not named | UNKNOWN | — | — | 2026-11-01 |
| C19 | Data residency | Two data centres: **United States DC** and **EU DC**. EU AWS location is Germany, so likely eu-central-1 Frankfurt (*inferred*). Every sub-processor has an EU location except Okta, LogRocket, Mixpanel and Scalekit, which are listed only under US DC. How a customer picks a region is not documented | CONFIRMED (regions) / INFERRED (AWS region) | L01 L15 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| C19 | Backups / retention | Daily backups, last 7 kept, DR simulation yearly (PDF). After termination: reinstate within **30 days**, then deletion | CONFIRMED | P03 L20 | https://www.rocketlane.com/legal/terms | 2026-11-01 |
| D20 | LLM providers | **OpenAI, Anthropic, Google LLC**: "AI model services used by Nitro's AI features" | CONFIRMED | L10 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| D21 | Direct API vs cloud AI | Providers are listed directly as sub-processors (not AWS Bedrock, Azure or Vertex). EU locations: OpenAI "Narvik, Norway", Anthropic "EU Specific Workspace", Google "Germany", which suggests direct vendor APIs with EU data-residency options (*inferred*) | CONFIRMED (vendors) / INFERRED (direct API) | L10 L16 L17 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| D22 | Meeting capture | Own-brand **"Rocketlane AI Notetaker"** bot joins Zoom, Teams and Meet. Separately, **cloud recording retrieval** from Gong, Zoom, Teams (via Microsoft Graph) and Google Meet uses provider APIs with no bot. **No notetaker vendor (e.g. Recall.ai) appears in the sub-processor list** | CONFIRMED / UNKNOWN (bot vendor) | H11 H12 H13 | https://help.rocketlane.com/support/solutions/articles/67000739897-enabling-meeting-intelligence-in-rocketlane-for-admins- | 2026-11-01 |
| D22 | Transcription engine | Cloud sources supply provider transcripts. For the AI Notetaker, the transcription engine is **not named** (no Deepgram or AssemblyAI listed) | UNKNOWN | H12 | — | 2026-11-01 |
| D23 | Vector store / RAG | **Weaviate**: "Vector database to store meetings, emails, pages, and signals for use by AI agents" | CONFIRMED | L12 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| D23 | Agent observability / framework | **LangSmith** for agent tracing. That makes LangChain/LangGraph usage likely (*inferred*), but not stated | CONFIRMED (LangSmith) / INFERRED (LangChain) | L13 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| D23 | Agent sandbox | **E2B**: "Sandbox containers used to run AI agents" | CONFIRMED | L14 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| D23 | Transcript pipeline | Meetings and emails are normalised into one "transcript object". The outputs are AI pages (meeting pages, Signals) with citations | CONFIRMED | H16 | https://help.rocketlane.com/support/solutions/articles/67000753317-rocketlane-ai-for-interaction-data-mental-model-and-behavior | 2026-11-01 |
| D24 | AI credit metering | Credits are charged per **agent run**, "based on the work performed". A run is not each AI call. Monthly chart, CSV export, a role permission, and system-triggered runs tagged "Nitro" | CONFIRMED | H17 H18 | https://help.rocketlane.com/support/solutions/articles/67000755370-nitro-credit-consumption | 2026-11-01 |
| D24 | Training / retention | Terms: will not use Customer Data to train its AI or ML models. **Zero-data-retention with LLM vendors not stated.** Guardrail or evaluation tooling not named | CONFIRMED (no training) / UNKNOWN (ZDR, evals) | L19 | https://www.rocketlane.com/legal/terms | 2026-11-01 |
| D25 | MCP server | Hosted at `https://rocket-mcp.rl-platforms.rocketlane.com/mcp`, **streamable HTTP** (`--transport http`). **OAuth**: Rocketlane sign-in, then an email one-time code, then choose the org. 24 tools (12 read, 12 write), no deletes | CONFIRMED | H07 H08 H09 H10 | https://help.rocketlane.com/support/solutions/articles/67000754219-connect-rocketlane-mcp-to-claude-code | 2026-11-01 |
| D25 | MCP auth provider | **Scalekit**: "Authentication and OAuth 2.0 Identity Management services for MCP" | CONFIRMED | L11 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| E26 | Cloud | **AWS** ("Primary cloud infrastructure provider … where all our applications are hosted"). US and Germany. VPC with private subnets, multi-AZ | CONFIRMED | L01 P05 P06 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| E27 | Containers / IaC | Not named in vendor docs (rolling deployments only) | UNKNOWN | P08 | — | 2026-11-01 |
| E28 | Observability | **Datadog** (APM, alerting, central logs), **LogRocket** (browser error and performance monitoring), **LangSmith** (AI traces) | CONFIRMED | L04 L05 L13 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| E29 | CI/CD | GitHub with protected branches, CI tests, Dependabot. Staging environment, canary rollout to internal accounts, rolling deployments | CONFIRMED (2023-era PDF) | P07 P08 | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | 2026-11-01 |
| E29 | Release cadence (public changelog) | Beamer feed: **30 posts Nov 2025–Oct 2026** (about 2.5 a month, including webinar and event posts); 96 posts back to Jan 2023 | OBSERVED | B03 | https://app.getbeamer.com/rocketlaneapp/en | 2026-11-01 |
| E29 | Secrets | AWS Secrets Manager; user passwords and tokens kept in "a secure Vault" (vendor of Vault not stated) | CONFIRMED / UNKNOWN (Vault vendor) | P01 | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | 2026-11-01 |
| F30 | SSO | SAML SSO with Okta, Google and Azure AD (Entra ID) guides. IdP-initiated login added by the Cognito migration | CONFIRMED | H19 H03 | https://help.rocketlane.com/support/solutions/articles/67000698478-how-to-setup-saml-sso-with-rocketlane-using-okta- | 2026-11-01 |
| F30 | SCIM | Okta and Azure SCIM provisioning guides. SCIM runs independently of authentication | CONFIRMED | H20 H06 | https://help.rocketlane.com/support/solutions/articles/67000745647-setting-up-okta-user-provisioning-integration-with-scim | 2026-11-01 |
| F30 | MFA / magic link | 2FA configurable at account level. Customer magic links are single-use tokens valid 7 days (help) and "secured by JWT" (PDF) | CONFIRMED | H05 P09 | https://help.rocketlane.com/support/solutions/articles/67000745544-2fa-and-magic-links | 2026-11-01 |
| F30 | **Okta → AWS Cognito** | Rocketlane is moving its **internal** authentication layer from Okta to AWS Cognito. It affects only the login layer, and customer IdPs are unaffected. SSO configs are deleted and recreated; downtime is 30–60 min, scheduled with an IST-based team. **Stated reason:** none beyond capability, namely that the new infrastructure "supports identity provider-initiated login". Okta is **still listed** as a sub-processor | CONFIRMED | H01 H02 H03 H04 L07 | https://help.rocketlane.com/support/solutions/articles/67000755973-updating-sso-for-rocketlane-s-authentication-migration | 2026-11-01 |
| F31 | Certifications | Trust page: SOC 2, ISO 27001, ISO 27701 (certified), **ISO 42001 (certified)**, HIPAA, GDPR, CSA STAR, CPRA, CASA, Cyber Essentials Plus. PDF: SOC 2 Type II audited; annual third-party pen test | CONFIRMED | L21 L22 P10 | https://www.rocketlane.com/trust | 2026-11-01 |
| F32 | Encryption | Encrypted at rest in AWS; TLS in transit (API: TLS 1.2). Key management (KMS / customer-managed keys) **not stated** | CONFIRMED / UNKNOWN (KMS, BYOK) | P04 D04 | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | 2026-11-01 |
| G33 | Embedded iPaaS | **Workato** is a sub-processor for "Custom integration between Rocketlane and other systems", in US and Germany. A public Workato connector authenticates with a Rocketlane API token. Whether native integrations run on Workato Embedded is **not stated** | CONFIRMED (Workato) / INFERRED (embedded use for custom integrations) | L08 H26 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| G33 | Salesforce | Native, via an AppExchange **managed package**. The integration creates Apex classes for triggers and data sync (needs Author Apex and Metadata API) | CONFIRMED | H27 H28 | https://help.rocketlane.com/support/solutions/articles/67000711953-getting-started-with-the-rocketlane-salesforce-integration | 2026-11-01 |
| G33 | HubSpot | Native app (Connect app; HubSpot Workflow action "Rocketlane Create Project"); one- or two-way field sync, "real-time" | CONFIRMED | H29 | https://help.rocketlane.com/support/solutions/articles/67000745881-how-does-the-rocketlane-hubspot-integration-handle-data-syncing-and-updates- | 2026-11-01 |
| G33 | Jira | Native: create and sync Jira issues, status, due dates and comments | CONFIRMED | H30 | https://help.rocketlane.com/support/solutions/articles/67000701085-getting-started-with-rocketlane-s-jira-integration | 2026-11-01 |
| G33 | Slack | Native Slack app, public channels only | CONFIRMED | H31 | https://help.rocketlane.com/support/solutions/articles/67000745655-what-are-the-current-capabilities-and-limitations-of-the-slack-integration- | 2026-11-01 |
| G33 | Teams / Graph | Recordings and transcripts via **Microsoft Graph**; no bot needed for cloud retrieval | CONFIRMED | H13 | https://help.rocketlane.com/support/solutions/articles/67000740226-connect-microsoft-teams-to-rocketlane | 2026-11-01 |
| — | Real-time | **Pusher** ("real-time messaging and notifications"), a managed WebSocket service | CONFIRMED | L03 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| — | Email | **SendGrid / Twilio** | CONFIRMED | L02 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| — | Docs/help/marketing platforms | Developer docs on **ReadMe** (`files.readme.io`, `x-readme` in spec); help centre on **Freshdesk** (`/support/solutions/articles/67000…`); marketing site on **Webflow** (`cdn.prod.website-files.com`); changelog on **Beamer**; status page on **Atlassian Statuspage** (`Server: AtlassianEdge`) | OBSERVED | D07 | https://developer.rocketlane.com/reference/get-task.md | 2026-11-01 |
| — | Implementation partners / group entity | Mako IT Lab, SaasGenie (India), Apricity (US). Group entity Rocketlane Technologies Private Limited (India) | CONFIRMED | L18 | https://www.rocketlane.com/legal/sub-processors | 2026-11-01 |
| — | Sitemap leak | www.rocketlane.com/sitemap.xml lists **stage.rocketlane.com** URLs (1,518), which exposes a staging host name | OBSERVED | — | https://www.rocketlane.com/sitemap.xml | 2026-11-01 |

---

## 2. Sub-processor list (verbatim)

Source: https://www.rocketlane.com/legal/sub-processors (retrieved 2026-11-01). The page shows no "last updated" date.

**United States Data Centre (US DC)**

| Name of Sub-processor | Purpose | Data Processing Location |
|---|---|---|
| Amazon Web Services | Primary cloud infrastructure provider for Rocketlane, where all our applications are hosted. | United States |
| SendGrid / Twilio | Used to send email communications within the product. | United States |
| Datadog | Application monitoring and error alerting. Logs are centralized through this service. | United States |
| Pusher | Enables real-time messaging and notifications for platform updates. | United States |
| LogRocket | Error and performance monitoring of browser pages served to customers. | United States |
| Okta | User authentication via SSO / SAML or username and password. | United States |
| Mixpanel | Used to collect and analyze product usage analytics. | United States |
| Workato | Custom integration between Rocketlane and other systems. | United States |
| ClickHouse | Product analytics, dashboards, and reporting. | United States |
| OpenAI | AI model services used by Nitro's AI features (as applicable). | United States |
| Anthropic | AI model services used by Nitro's AI features (as applicable). | United States |
| Google LLC | AI model services used by Nitro's AI features (as applicable). | United States |
| Scalekit | Authentication and OAuth 2.0 Identity Management services for MCP, including user authentication and access management. | United States |
| Weaviate | Vector database to store meetings, emails, pages, and signals for use by AI agents. | United States |
| Langsmith | Tracing and Observability for AI Agents. | United States |
| E2B | Sandbox containers used to run AI agents (as applicable). | United States |
| Mako IT Lab Pvt Ltd | Provides Implementation Services on Behalf of Rocketlane. | India |
| SaasGenie | Provides Implementation Services on Behalf of Rocketlane. | India |
| Apricity | Provides Implementation Services on Behalf of Rocketlane. | United States |

**European union Data Centre (EU - DC)**

| Sub-processor | Purpose | EU Data Processing Location |
|---|---|---|
| Amazon Web Services | Primary cloud infrastructure provider for Rocketlane, where all our applications are hosted. | Germany |
| SendGrid / Twilio | Used to send email communications within the product. | European Union (EU region) |
| Datadog | Application monitoring and error alerting. Logs are centralized through this service. | Germany |
| Pusher | Enables real-time messaging and notifications for platform updates. | Ireland |
| Workato | Custom integration between Rocketlane and other systems. | Germany |
| ClickHouse | Product analytics, dashboards, and reporting. | Germany |
| OpenAI | AI model services used by Nitro's AI features (as applicable). | Narvik, Norway |
| Anthropic | AI model services used by Nitro's AI features (as applicable). | EU Specific Workspace |
| Google LLC | AI model services used by Nitro's AI features (as applicable). | Germany |
| Weaviate | Vector database to store meetings, emails, pages, and signals for use by AI agents. | Germany |
| Langsmith | Tracing and Observability for AI Agents. | Netherlands |
| E2B | Sandbox containers used to run AI agents (as applicable). | Germany |
| Mako IT Lab Pvt Ltd | Provides Implementation Services on Behalf of Rocketlane. | India |
| SaasGenie | Provides Implementation Services on Behalf of Rocketlane. | India |

**Group sub-processors:** Rocketlane Technologies Private Limited, India.

Notable from this list:
- **Missing from the EU list:** Okta, LogRocket, Mixpanel and Scalekit. So EU tenants either don't use those services or rely on US processing, and the page doesn't say which.
- **Missing entirely:** AWS Cognito (covered under AWS), a meeting-bot vendor, a transcription vendor, a search engine, and a support or chat tool. Freshdesk hosts the public help centre but is not listed.

---

## 3. API, webhook and MCP design summary

**REST API (v1.0).**
- Style and transport:
  - "The API follows RESTful principles and uses HTTP methods to perform operations on resources." [D01]
  - "Although some API calls follow an RPC-like structure." [D02]
  - HTTPS with TLS 1.2 [D04].
- Base URL: `https://api.rocketlane.com/api/1.0/<resource>[/<id>[/<action>]]`.
- Auth: `api-key` header. The 401 text says it "has a valid api-key that you acquired from the Rocketlane API Settings tab" [D08].
- Pagination:
  - `pageSize` defaults to 100, which is also the maximum.
  - `pageToken` / `nextPageToken`: "This token is valid for 15 minutes" [D05].
- Errors: `errors[] {errorCode, errorMessage}` [D09].
- Tracing: an `X-Request-Id` header on every response [D03].
- Rate limits, from the help centre, not the dev docs:
  - "up to 100 requests per minute per user/endpoint" [H14]
  - "60 requests per minute for search APIs" [H15]
  - No rate-limit response headers are documented.
- The OpenAPI `servers` block also exposes `https://api.nitro.run/api` (labelled Staging) and `https://project.api.orbits.run/api` (labelled Orbits) [D07, D07b]. "Nitro" is also the brand of the AI product line.

**Webhooks.**
- Configured in Settings, then fired only by an automation's "Send webhook" action.
- "Currently, we only support task triggers for webhooks." [D12]
- Security: HTTPS only [D16]. "Only Basic Authentication is supported currently." [D11]. No payload signature, timestamp-signing or replay protection is documented.
- Delivery:
  - Success is 200 or 204 only.
  - Failure is any other code, or "if no response is received for 10 seconds and the HTTP POST request times out" [D13].
- Ordering: delivery "is not guaranteed to be strictly chronological" [D14].
- Duplicates: "Rocketlane guarantees exactly one delivery per event." [D15]. That is a strong claim, given that no retry or idempotency mechanism is documented; `eventId` is the only dedupe key.
- Payload: the envelope carries `changeLog.from/to` on `_UPDATED` events, plus the actor and two epoch-ms timestamps [D17].
- Launched as "Task webhooks" on Feb 26, 2024 [B01].

**MCP.**
- Endpoint: `claude mcp add --transport http rocketlane https://rocket-mcp.rl-platforms.rocketlane.com/mcp` [H08].
- Auth: "connects to your AI tool through an MCP server URL and uses OAuth" [H07]. The flow is Rocketlane sign-in, then an email OTP ("Enter the one-time code sent to your email to verify your identity." [H09]), then picking the organization.
- The OAuth and identity layer for MCP is Scalekit [L11].
- Tools: "Rocketlane MCP currently supports 24 tools." [H10]
  - 12 read: profile, projects, tasks, users, phases, fields, customer companies, time entries and categories, task, project and conversation comments.
  - 12 write: create and update phase, task, time entry and comments; update project; create project template.
  - No delete tools: "MCP cannot delete a time entry or reassign a time entry to another user." [H10b]
  - Project financial fields are excluded from MCP updates.
- Hosting: the `rl-platforms.rocketlane.com` subdomain suggests a separate platform-services zone (*inferred*).

---

## 4. UNKNOWN list (with what was searched)

| Q# | Unknown | Searched |
|---|---|---|
| B12 | Queue / broker (SQS, Kafka, RabbitMQ, Redis) and scheduler behind Automations | developer docs (llms.txt index, all guides), help automation articles (intro, pricing, logs, triggers), security PDF, sub-processor list |
| B13 | Webhook retry count, backoff, dead-letter, signing (HMAC) | webhooks overview, setup, event payload, delivery and examples pages; help article on Zoho webhook issues (title only) |
| B14 | Search engine (Elasticsearch, OpenSearch, Typesense, Postgres FTS) | developer Search guide, help Global Search article, sub-processors (no search vendor listed) |
| B15 | Object store (S3 implied, not stated) and attachment size limits | help sitemap (no size-limit article), files deletion article, Documents/Spaces articles, PDF |
| C16 | Primary database engine | PDF ("all databases"), DPA, sub-processors, developer docs |
| C17 | Exact tenancy model (shared schema with tenant ID vs schema-per-tenant) | PDF multi-tenant section only; no AWS SaaS Factory material in vendor docs (left to the secondary-sources workstream) |
| C18 | Cache layer; data warehouse beyond ClickHouse | sub-processors, PDF |
| C19 | How customers choose or migrate region; EU coverage for Okta, LogRocket, Mixpanel and Scalekit | sub-processors, help sitemap (no residency article), DPA |
| D22 | AI Notetaker bot vendor (Recall.ai etc.) and transcription engine | sub-processors (none listed), Meeting Intelligence admin, disable-sources, Teams, AI Fills, meeting-matching and bot-recording FAQ articles |
| D23 | Agent framework (LangChain/LangGraph implied by LangSmith, not stated) | sub-processors, Documentation Agents, AI Fills, interaction-data articles |
| D24 | Zero-data-retention agreements with OpenAI, Anthropic and Google; eval or guardrail tooling | Terms, DPA, privacy policy, Nitro credits article, sub-processors |
| E27 | ECS, EKS, Lambda; IaC | PDF (rolling deploys, multi-AZ, VPC only), sub-processors |
| F30 | Business reason for Okta → Cognito beyond "supports identity provider-initiated login" (cost, consolidation on AWS?) | full migration article incl. FAQ and Summary; SSO and SCIM series |
| F32 | KMS or customer-managed keys | PDF, DPA Appendix II (refers to a security policy available on request), Trust page |
| G33 | Whether native Jira, HubSpot, Slack or Salesforce integrations run on Workato Embedded or are in-house | integration setup articles for each, Workato article, sub-processors |
| — | Trust portal vendor | trust.rocketlane.com and security.rocketlane.com are JS-only apps served from S3 (`Server: AmazonS3`); not rendered, so the vendor is not identified |

**Dating caveat.** The security PDF (`…64d0c657…Security and compliance at Rocketlane.pdf`) calls Rocketlane "the leading collaborative customer onboarding platform" and cites "EU-US privacy shield", which points to a 2023-era document. The Trust page lists newer certifications (ISO 42001, ISO 27701). PDF facts (Okta-managed passwords, 7 backups) may be out of date. In particular, Okta-managed passwords are being superseded by the Cognito migration.

---

## 5. Evidence ledger

`check`: `static` = present in a plain HTTP fetch of that URL · `rendered` = needs a browser, or is PDF text (not checkable as HTML) · `inferred` = no verbatim quote.

| ID | Claim | URL | Verbatim quote (<=15 words, no quote marks) | check |
|---|---|---|---|---|
| D01 | REST API | https://developer.rocketlane.com/docs/overview.md | The API follows RESTful principles and uses HTTP methods to perform operations on resources. | static |
| D02 | Some RPC-style calls | https://developer.rocketlane.com/docs/overview.md | Although some API calls follow an RPC-like structure. | static |
| D03 | Request IDs | https://developer.rocketlane.com/docs/overview.md | A request identifier is assigned to each API request. | static |
| D04 | TLS 1.2 | https://developer.rocketlane.com/docs/overview.md | The APIs can be accessed by endpoints using HTTPS through TLS 1.2. | static |
| D05 | Page token TTL | https://developer.rocketlane.com/docs/pagination.md | This token is valid for 15 minutes | static |
| D06 | Search index lag | https://developer.rocketlane.com/docs/search.md | It may take a couple of seconds for a change made in an API | static |
| D07 | Staging API host | https://developer.rocketlane.com/reference/get-task.md | https://api.nitro.run/api | static |
| D07b | Orbits API host | https://developer.rocketlane.com/reference/get-task.md | https://project.api.orbits.run/api | static |
| D08 | api-key header auth | https://developer.rocketlane.com/reference/get-task.md | has a valid api-key that you acquired from the Rocketlane API Settings tab | static |
| D09 | Error message field | https://developer.rocketlane.com/reference/get-task.md | Error message describes in brief about the issue. | static |
| D10 | int64 IDs | https://developer.rocketlane.com/reference/get-task.md | int64 | static |
| D11 | Basic Auth only | https://developer.rocketlane.com/docs/setting-up-webhooks-in-rocketlane.md | Only Basic Authentication is supported currently. | static |
| D12 | Task triggers only | https://developer.rocketlane.com/docs/setting-up-webhooks-in-rocketlane.md | Currently, we only support task triggers for webhooks. | static |
| D13 | 10 s timeout | https://developer.rocketlane.com/docs/webhook-delivery.md | if no response is received for 10 seconds and the HTTP POST request times out | static |
| D14 | No ordering guarantee | https://developer.rocketlane.com/docs/webhook-delivery.md | is not guaranteed to be strictly chronological | static |
| D15 | Exactly-once claim | https://developer.rocketlane.com/docs/webhook-delivery.md | Rocketlane guarantees exactly one delivery per event. | static |
| D16 | HTTPS-only webhooks | https://developer.rocketlane.com/docs/webhooks-overview.md | Rocketlane webhooks use only HTTPS so that communication with the external system is always secure. | static |
| D17 | Payload timestamps | https://developer.rocketlane.com/docs/event-payload.md | Timestamp (in epochmillis) of when the event occurred at in Rocketlane. | static |
| H01 | Okta to Cognito | https://help.rocketlane.com/support/solutions/articles/67000755973-updating-sso-for-rocketlane-s-authentication-migration | Rocketlane is migrating its internal authentication infrastructure from Okta to AWS Cognito. | static |
| H02 | Login layer only | https://help.rocketlane.com/support/solutions/articles/67000755973-updating-sso-for-rocketlane-s-authentication-migration | This change affects only the login layer. | static |
| H03 | IdP-initiated login | https://help.rocketlane.com/support/solutions/articles/67000755973-updating-sso-for-rocketlane-s-authentication-migration | The updated authentication infrastructure supports identity provider-initiated login. | static |
| H04 | Engineering in IST | https://help.rocketlane.com/support/solutions/articles/67000755973-updating-sso-for-rocketlane-s-authentication-migration | Rocketlane's engineering team operates in IST. | static |
| H05 | Magic link token | https://help.rocketlane.com/support/solutions/articles/67000745544-2fa-and-magic-links | generates a unique, single-use, time-sensitive token (valid for 7 days) | static |
| H06 | SCIM independent | https://help.rocketlane.com/support/solutions/articles/67000755973-updating-sso-for-rocketlane-s-authentication-migration | SCIM operates independently of authentication | static |
| H07 | MCP uses OAuth | https://help.rocketlane.com/support/solutions/articles/67000754210-connect-rocketlane-mcp-to-claude | Rocketlane MCP connects to your AI tool through an MCP server URL and uses OAuth | static |
| H08 | MCP URL and transport | https://help.rocketlane.com/support/solutions/articles/67000754219-connect-rocketlane-mcp-to-claude-code | claude mcp add --transport http rocketlane https://rocket-mcp.rl-platforms.rocketlane.com/mcp | static |
| H09 | MCP email OTP | https://help.rocketlane.com/support/solutions/articles/67000754210-connect-rocketlane-mcp-to-claude | Enter the one-time code sent to your email to verify your identity. | static |
| H10 | 24 MCP tools | https://help.rocketlane.com/support/solutions/articles/67000754589-guide-to-using-rocketlane-mcp-tools | Rocketlane MCP currently supports 24 tools. | static |
| H10b | MCP no deletes | https://help.rocketlane.com/support/solutions/articles/67000754589-guide-to-using-rocketlane-mcp-tools | MCP cannot delete a time entry or reassign a time entry to another user. | static |
| H11 | Own-brand notetaker | https://help.rocketlane.com/support/solutions/articles/67000739897-enabling-meeting-intelligence-in-rocketlane-for-admins- | The Rocketlane AI Notetaker joins scheduled Zoom, Microsoft Teams, or Google Meet calls | static |
| H12 | Cloud recording retrieval | https://help.rocketlane.com/support/solutions/articles/67000739897-enabling-meeting-intelligence-in-rocketlane-for-admins- | Rocketlane retrieves completed recordings and transcripts directly from connected providers after a meeting ends. | static |
| H13 | Teams via Graph, no bot | https://help.rocketlane.com/support/solutions/articles/67000740226-connect-microsoft-teams-to-rocketlane | No Rocketlane bot is required for cloud recording retrieval. | static |
| H14 | 100 req/min limit | https://help.rocketlane.com/support/solutions/articles/67000755761-why-am-i-getting-a-rate-limit-error-during-bulk-updates- | up to 100 requests per minute per user/endpoint | static |
| H15 | Search API 60/min | https://help.rocketlane.com/support/solutions/articles/67000747286-my-workato-automation-is-failing-with-a-rate-limit-exceeded-error-what-should-i-do- | our Public API has a limit of 60 requests per minute for search APIs | static |
| H16 | Common transcript object | https://help.rocketlane.com/support/solutions/articles/67000753317-rocketlane-ai-for-interaction-data-mental-model-and-behavior | Rocketlane AI treats different interaction sources (meetings and emails) as a common transcript object | static |
| H17 | Credits per agent run | https://help.rocketlane.com/support/solutions/articles/67000755370-nitro-credit-consumption | Each time a Nitro agent runs, it consumes credits based on the work performed. | static |
| H18 | Run is not an AI call | https://help.rocketlane.com/support/solutions/articles/67000755370-nitro-credit-consumption | A run does not represent each individual AI call made during the execution. | static |
| H19 | SAML with Okta guide | https://help.rocketlane.com/support/solutions/articles/67000698478-how-to-setup-saml-sso-with-rocketlane-using-okta- | How to setup SAML SSO with Rocketlane using Okta? | static |
| H20 | SCIM with Okta guide | https://help.rocketlane.com/support/solutions/articles/67000745647-setting-up-okta-user-provisioning-integration-with-scim | Setting Up Okta User Provisioning Integration with SCIM | static |
| H21 | Automation pricing by runs | https://help.rocketlane.com/support/solutions/articles/67000724434-automation-pricing | Rocketlane's automation feature is priced based on the number of runs per user per month. | static |
| H22 | Stops at limit | https://help.rocketlane.com/support/solutions/articles/67000724434-automation-pricing | automations will stop running once the limit is reached. | static |
| H23 | Trigger/condition/action | https://help.rocketlane.com/support/solutions/articles/67000722958-introduction-to-automations | An automation rule consists of a trigger, a condition, and an action. | static |
| H24 | Global search scope | https://help.rocketlane.com/support/solutions/articles/67000753390-using-global-search-in-rocketlane | Global Search scans your entire Rocketlane instance, including: | static |
| H25 | No permanent file delete | https://help.rocketlane.com/support/solutions/articles/67000737703-can-documents-and-files-sent-in-all-files-and-conversations-be-deleted- | Rocketlane doesn't allow users to permanently delete files | static |
| H26 | Workato connector via API token | https://help.rocketlane.com/support/solutions/articles/67000710283-the-workato-rocketlane-integration | Copy and use the API Token provided here to connect to Workato | static |
| H27 | Salesforce Apex classes | https://help.rocketlane.com/support/solutions/articles/67000711953-getting-started-with-the-rocketlane-salesforce-integration | The integration creates AppEx classes for the triggers and data sync. | static |
| H28 | AppExchange package | https://help.rocketlane.com/support/solutions/articles/67000729101-integrate-rocketlane-and-salesforce-via-appexchange | Rocketlane is now available on the Salesforce AppExchange | static |
| H29 | HubSpot real-time sync | https://help.rocketlane.com/support/solutions/articles/67000745881-how-does-the-rocketlane-hubspot-integration-handle-data-syncing-and-updates- | Rocketlane provides real-time synchronization for all mapped fields within the integration. | static |
| H30 | Jira native sync | https://help.rocketlane.com/support/solutions/articles/67000701085-getting-started-with-rocketlane-s-jira-integration | allows you to create and sync Jira issues for your Rocketlane tasks | static |
| H31 | Slack public channels only | https://help.rocketlane.com/support/solutions/articles/67000745655-what-are-the-current-capabilities-and-limitations-of-the-slack-integration- | only supports the creation of and posting to public Slack channels | static |
| L01 | AWS hosts everything | https://www.rocketlane.com/legal/sub-processors | Primary cloud infrastructure provider for Rocketlane, where all our applications are hosted. | static |
| L02 | SendGrid email | https://www.rocketlane.com/legal/sub-processors | Used to send email communications within the product. | static |
| L03 | Pusher real-time | https://www.rocketlane.com/legal/sub-processors | Enables real-time messaging and notifications for platform updates. | static |
| L04 | Datadog | https://www.rocketlane.com/legal/sub-processors | Application monitoring and error alerting. Logs are centralized through this service. | static |
| L05 | LogRocket | https://www.rocketlane.com/legal/sub-processors | Error and performance monitoring of browser pages served to customers. | static |
| L06 | Mixpanel | https://www.rocketlane.com/legal/sub-processors | Used to collect and analyze product usage analytics. | static |
| L07 | Okta still listed | https://www.rocketlane.com/legal/sub-processors | User authentication via SSO / SAML or username and password. | static |
| L08 | Workato custom integration | https://www.rocketlane.com/legal/sub-processors | Custom integration between Rocketlane and other systems. | static |
| L09 | ClickHouse reporting | https://www.rocketlane.com/legal/sub-processors | Product analytics, dashboards, and reporting. | static |
| L10 | LLM vendors | https://www.rocketlane.com/legal/sub-processors | AI model services used by Nitro's AI features (as applicable). | static |
| L11 | Scalekit for MCP auth | https://www.rocketlane.com/legal/sub-processors | Authentication and OAuth 2.0 Identity Management services for MCP | static |
| L12 | Weaviate vector DB | https://www.rocketlane.com/legal/sub-processors | Vector database to store meetings, emails, pages, and signals for use by AI agents. | static |
| L13 | LangSmith tracing | https://www.rocketlane.com/legal/sub-processors | Tracing and Observability for AI Agents. | static |
| L14 | E2B sandboxes | https://www.rocketlane.com/legal/sub-processors | Sandbox containers used to run AI agents (as applicable). | static |
| L15 | EU data centre | https://www.rocketlane.com/legal/sub-processors | European union Data Centre | static |
| L16 | OpenAI EU location | https://www.rocketlane.com/legal/sub-processors | Narvik, Norway | static |
| L17 | Anthropic EU location | https://www.rocketlane.com/legal/sub-processors | EU Specific Workspace | static |
| L18 | Implementation partners | https://www.rocketlane.com/legal/sub-processors | Provides Implementation Services on Behalf of Rocketlane. | static |
| L19 | No training on customer data | https://www.rocketlane.com/legal/terms | shall not use the Customer Data to train its artificial intelligence | static |
| L20 | 30-day reinstatement window | https://www.rocketlane.com/legal/terms | within thirty (30) days of effective termination of Your Account | static |
| L21 | ISO 42001 | https://www.rocketlane.com/trust | ISO 42001 Certified | static |
| L22 | HIPAA claim | https://www.rocketlane.com/trust | HIPAA Compliant | static |
| L23 | DPA security program | https://www.rocketlane.com/legal/data-processing-agreement | Rocketlane has implemented and shall maintain a security program in accordance with industry standards. | static |
| P01 | Secrets Manager | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | Application secrets are managed through AWS Secrets Manager. | rendered |
| P02 | Pure multi-tenant | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | Rocketlane is built as a pure multi-tenant SaaS application. | rendered |
| P03 | Daily backups | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | Data is periodically backed up every day across all databases. | rendered |
| P04 | Encrypted at rest in AWS | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | All customer data is stored within AWS and encrypted at rest | rendered |
| P05 | Multi-AZ | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | Our application is deployed across multiple availability zones (AZ) in AWS. | rendered |
| P06 | VPC private subnets | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | Only the public-facing application API endpoints are exposed to customers. | rendered |
| P07 | GitHub | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | Our entire codebase is version-controlled in GitHub. | rendered |
| P08 | Canary rollout | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | A subset of internal accounts is chosen for feature trial before a wider | rendered |
| P09 | Magic links JWT | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | authentication using Magic Links which are secured by JWT. | rendered |
| P10 | Annual pen test | https://cdn.prod.website-files.com/6360d3edc696260ba2aecbc7/64d0c65700a997dccc386c70_Security%20and%20compliance%20at%20Rocketlane.pdf | PEN testing is conducted every year by a third party organization | rendered |
| B01 | Task webhooks launch Feb 26 2024 | https://app.getbeamer.com/rocketlaneapp/en | Task webhooks: Stream real-time data wherever you need | rendered |
| B02 | Automations launch Sep 12 2023 | https://app.getbeamer.com/rocketlaneapp/en | The biggest update ever | rendered |
| B03 | 30 posts in last 12 months | https://app.getbeamer.com/rocketlaneapp/en | no verbatim quote; count of post dates from loadMoreNews pages 0-15 (96 posts total) | inferred |
| I01 | EU AWS = Frankfurt | https://www.rocketlane.com/legal/sub-processors | no verbatim quote; inferred from AWS location Germany | inferred |
| I02 | LangChain likely | https://www.rocketlane.com/legal/sub-processors | no verbatim quote; inferred from LangSmith sub-processor | inferred |

Notes on `rendered` rows:
- **P rows:** the quotes come from the PDF's extracted text, read with a PDF reader on 2026-11-01. They can't be checked against HTML.
- **B rows:** Beamer posts load through a `loadMoreNews` POST from the public changelog page. Titles were read from that endpoint's HTML on 2026-11-01.

### Verification (2026-11-01)

- **`static` rows: 73 of 73 verbatim.** A script fetched each URL live and matched the quote against the page text, with HTML tags stripped and entities decoded. No quote is over 15 words.
- **PDF rows (P01–P10): 10 of 10 verbatim.** Checked against text extracted with pypdf.
- **Beamer rows (B01–B02): 2 of 2 found.** Checked in the `loadMoreNews` HTML pages (B01 on page 9, B02 on page 11).
- **`inferred` rows:** carry no quote and were not checked.
- Quotes in the prose tables of sections 1 and 3 are shortened or paraphrased (some use "…") and are **not** ledger evidence. The ledger rows above are the evidence.
