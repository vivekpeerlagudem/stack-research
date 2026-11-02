# Rocketlane tech stack — Workstream 3: secondary and third-party public sources

Retrieval date for every row: **2026-11-01**. Public pages only, no login. Sources include an AWS Partner Network blog post, Rocketlane-run job pages on Kula, job boards (BuiltIn), Rocketlane's public GitHub org and npm packages, and marketplace and app-store listings. Third-party fingerprint sites were also tried.

**Grading** (from the brief): CONFIRMED means Rocketlane states it on its own channels (careers pages on careers.kula.ai/rocketlane, its npm and GitHub org, its vendor-run marketplace listing). SECONDARY means a third party, a job board copy, or an interview published by someone else. OBSERVED means a public artefact seen directly, such as the GitHub fork list or npm dependency manifest. INFERRED means my reasoning, shown. UNKNOWN means not found; §4 lists what was searched.

> **Note on the CTO interview:** the APN post quotes Rocketlane's CTO in his own words, but AWS published it, so it is graded SECONDARY (interview). Its claims match Rocketlane's own 2026 careers pages (ECS, ALB/CloudFront, WAF, PostgreSQL), and the rows below note where they agree.

## 1. Findings

| Q# | Component | Technology | Grade | Evidence | URL | Date |
|---|---|---|---|---|---|---|
| E26 | Cloud provider | **AWS** | SECONDARY (CTO interview) + CONFIRMED (careers) | CTO: "Rocketlane selected Amazon Web Services (AWS)". 2026 security postings: "Review and harden our AWS setup" [L01][L13] | APN blog; careers 54840 | pub 2023-05-30; 2026-09-16 |
| E26 | Marketplace hosting badge | Deployed on AWS | SECONDARY (AWS listing) | Listing shows "Sold by: Rocketlane" and "Deployed on AWS" [L21] | AWS Marketplace prodview-asp3llansbsfq | 2026-11-01 |
| E27 | Container orchestration | **Amazon ECS** | SECONDARY + CONFIRMED | CTO names ECS "for managed container orchestration" [L03]. 2026 posting: "container security on ECS or similar" [L14] | APN; careers 54840 | 2023; 2026 |
| E26 | Edge, WAF, load balancing | AWS WAF, AWS Global Accelerator, ALB, CloudFront | SECONDARY + CONFIRMED | CTO lists WAF, Global Accelerator and ALB [L04]. 2026 posting: "ALB/CloudFront TLS policies and cipher suites" [L15] | APN; careers 54840 | 2023; 2026 |
| C16 | Primary database | **PostgreSQL** (Amazon RDS) | SECONDARY + CONFIRMED | CTO: "Amazon RDS for managed relational database" [L02]. Postings: "Spring, PostgreSQL, Redis, Docker, message brokers, and GitHub" [L09]. GitHub fork "postgresql-action" [L27] | APN; careers 12592; GitHub | 2023; 2025-08-26 |
| C18 / B12 | Cache and job queue | **Redis via Amazon ElastiCache**, used for background-job queueing | SECONDARY + CONFIRMED | CTO: "we use AWS Elasticache for reliable queue time-sensitive background jobs" [L05]. Redis appears in backend and full-stack postings [L09][L11] | APN; careers 12592, 54233 | 2023; 2025–26 |
| B12 | Message broker | "message brokers", product **UNKNOWN** | CONFIRMED (generic term only) | The backend posting says "message brokers" without naming Kafka, RabbitMQ or SQS [L09] | careers 12592 | 2025-08-26 |
| B9 | Backend language and framework | **Java + Spring / Spring Boot** | CONFIRMED | "Familiarity with Java/Spring Boot stacks (our backend)" [L12]. "React frontend and Java backend" [L10]. The CTO is "passionate about ... Spring framework" [L06] | careers 54840, 54233; APN | 2026-09 |
| D21–23 | AI/LLM service language | **Python** | CONFIRMED | "Python (our AI/LLM stack)" [L12] | careers 54840 | 2026-09-16 |
| D20 | LLM provider for Nitro | **Anthropic Claude** | CONFIRMED (careers page) | "Nitro, our AI-native product layer on Claude and MCP" [L16]. Whether the model is called through the Anthropic API directly or through a cloud service (Bedrock or Vertex) is **UNKNOWN** | careers 54908 | 2026-09-17 |
| D25 | Agent integration protocol | **MCP** | CONFIRMED (careers page) | Same quote as above [L16] | careers 54908 | 2026-09-17 |
| D23 | AI gateway (candidate) | **Bifrost** (maximhq/bifrost, written in Go) | OBSERVED (fork exists); use INFERRED | Rocketlane forked Bifrost, "Fastest enterprise AI gateway", on 2026-09-19 and last pushed to the fork on 2026-10-26 [L30]. A fork with recent pushes suggests evaluation or customised use, but **production use is not stated** | github.com/Rocketlane | 2026-10-26 |
| D23 | Agent tooling security | Code execution in agent tools; prompt-injection surfaces | CONFIRMED (careers page) | The posting names "unsafe code execution in agent tooling" as a risk to hunt [L17]. *(Inferred)* Nitro agents can run code or call tools | careers 54840 | 2026-09-16 |
| A1–2 | Frontend | **React**, JavaScript/TypeScript, SASS, Webpack | CONFIRMED | "React and modern JavaScript/TypeScript" [L11]. 2025 frontend posting asks for "Knowledge of module bundlers like Webpack" [L08] | careers 54233, 2230 | 2025-01-29; 2026-09-01 |
| A7 | Mobile | iOS app, publisher ROCKETLANE TECHNOLOGIES PRIVATE LIMITED, version 0.7.1, 75.8 MB | SECONDARY (store listing) | App Store listing [L24]. **Framework UNKNOWN**: the listing has no framework hint. 2026 security postings mention a "mobile app" [L13] | apps.apple.com id6450273789 | 2026-11-01 |
| E29 | CI/CD | **GitHub Actions** with self-hosted EC2 runners; JaCoCo coverage on PRs; release-please; Dependabot | OBSERVED + CONFIRMED | Org forks: "ec2-github-runner — On-demand self-hosted AWS EC2 runner for GitHub Actions custom for rocketlane" [L26], "jacoco-report" [L28], "release-please" [L29]. Posting: "Dependabot ... across dozens of repos" [L18] | github.com/Rocketlane; careers 54840 | 2024-11 → 2026-10 |
| E27 | Infrastructure as code | Named tools: Terraform, Ansible, Chef, CloudFormation. **Which one Rocketlane uses is UNKNOWN** | CONFIRMED (hiring requirement only) | "Terraform, Ansible, Chef, or CloudFormation" [L19] | careers 9833 / BuiltIn 8548456 | 2025-07-10 |
| E28 | Observability | **Prometheus, Grafana** | CONFIRMED (hiring requirement) | "using tools like Prometheus, Grafana, and custom telemetry" [L20]. The lead backend role works on an internal "observability platform" [L09b] | careers 9833; 12592 | 2025 |
| F31 | GRC tooling | **Sprinto** (or similar) | CONFIRMED (hiring requirement) | "Familiarity with GRC tooling (Sprinto or similar)" [L17b] | careers 54908 | 2026-09-17 |
| C17 | Multi-tenancy | Multi-tenant; horizontally scalable at every layer; shared or pooled model **UNKNOWN** | SECONDARY + CONFIRMED | CTO: "multi-tenant architecture that can scale horizontally at every layer" [L07]. Posting: "multi-tenant SaaS environment where tenant isolation is a first-order concern" [L17c]. Whether it is a shared schema, schema-per-tenant or database-per-tenant is not stated | APN; careers 54908 | 2023; 2026 |
| C19 | Data residency | "architected with principles of being global from day one" | SECONDARY | No regions named [L04b] | APN | 2023 |
| G33 | App platform | **Rocketlane Marketplace apps** built with the `@rocketlane/rli` CLI (MIT licence; first published 2025-03-03; 59 versions; latest 1.7.0-beta.29 on 2026-10-21) | CONFIRMED (npm package published by Rocketlane) | "RLI - The Rocketlane CLI for building Apps for the Rocketlane Marketplace" [L25]. Dependencies include webpack, typescript, @babel/preset-react, @tailwindcss/postcss, zod, express, express-ws, ws, swagger-jsdoc, keytar. *(Inferred)* Apps are React + TypeScript bundles built with webpack. Maintainers use @rocketlane.com addresses | registry.npmjs.org/@rocketlane/rli | 2026-10-21 |
| G33 | LLM-assisted app building | "rli.bolt": a fork of bolt.diy for Rocketlane apps | OBSERVED | "Prompt, run, edit, and deploy Rocketlane marketplace applications using any LLM you want!" [L31] | github.com/Rocketlane | 2025-04-17 |
| G33 | Slack integration | **Native Slack app** (ID A031D6SD4ER) with 13 bot token scopes | SECONDARY (Slack directory) | Scopes: channels:join, channels:manage, channels:read, chat:write, chat:write.public, commands, groups:read, groups:write, links:read, links:write, remote_files:write, users:read, users:read.email [L22]. Features include two-way actions from notifications | slack.com/marketplace | 2026-11-01 |
| G33 | Zapier integration | Built on the **Zapier Platform CLI** | OBSERVED | The Zapier app key is "RocketlaneCLIAPI@latest" [L23] | zapier.com/apps/rocketlane | 2026-11-01 |
| G33 | HubSpot | Listed on the HubSpot marketplace (2023 press release); two-way sync | SECONDARY | The marketplace page renders without content when not signed in, so the listing text was not read. The source is a press release | finance.yahoo.com press release | 2023 |
| G33 | Salesforce, Jira, Microsoft Teams | No AppExchange or Atlassian Marketplace listing found. AppSource returned HTTP 403 | UNKNOWN | See §4 | — | — |
| G33 | Customer integration delivery | Forward Deployed Engineers use "iPaaS platforms like Workato/Boomi/Make" | CONFIRMED (hiring requirement) | This is for customer-specific integrations, not evidence of Rocketlane's own embedded iPaaS vendor [L32] | careers 54027 | 2026-08-28 |
| — | Engineering hub | **Chennai** (Perungudi, Global Infocity Park) | CONFIRMED + SECONDARY | All engineering postings are on-site in Chennai. BuiltIn JSON-LD has street address "Rocketlane Technologies Private Limited, 13th Floor, Block-A, Global Infocity Park, Perungudi". YourStory: founded "in 2020 in Chennai" [L33] | careers.kula.ai; builtin.com; yourstory.com | 2022; 2026 |
| — | Third-party fingerprint | BuiltWith (app.rocketlane.com): Google Font API, Web App Manifest, Hotjar | SECONDARY | Free tier shows little; StackShare returned HTTP 429; the Wappalyzer page failed to load | builtwith.com/app.rocketlane.com | 2026-11-01 |

## 2. Job-posting matrix (all careers.kula.ai/rocketlane, Chennai unless noted)

| Posting | URL | datePosted | Languages | Frameworks | Data stores | Queues | Cloud / ops | AI tools |
|---|---|---|---|---|---|---|---|---|
| Senior SWE – Frontend | https://careers.kula.ai/rocketlane/2230-senior-software-engineer-frontend | 2025-01-29 | HTML, CSS, SASS, OO JavaScript | React; Webpack (nice to have) | — | — | — | — |
| Senior SWE – Systems (DevOps/SRE) | https://careers.kula.ai/rocketlane/9833-senior-software-engineer-systems | 2025-07-10 | Python, Java or similar | — | "databases" (generic) | — | AWS preferred; IaC (Terraform/Ansible/Chef/CloudFormation); Prometheus, Grafana; CI/CD; Linux | — |
| Lead SWE – Backend | https://careers.kula.ai/rocketlane/12592-lead-software-enginner-backend | 2025-08-26 | Java | Spring | PostgreSQL, Redis | "message brokers" | Docker; AWS (nice to have); GitHub | — |
| Full Stack Engineer | https://careers.kula.ai/rocketlane/54233-full-stack-engineer | 2026-09-01 | JavaScript/TypeScript, Java | React, Spring | PostgreSQL, Redis | — | AWS, Docker (nice to have) | "AI-powered product experiences" |
| Product Security Engineer | https://careers.kula.ai/rocketlane/54840-product-security-engineer | 2026-09-16 | Java (backend), Python (AI/LLM) | Spring Boot | — | — | AWS WAF, ALB/CloudFront, IAM, ECS; Dependabot; SAST/DAST | LLM/agent security |
| Lead Product Security Engineer | https://careers.kula.ai/rocketlane/54908-lead-product-security-engineer | 2026-09-17 | — | — | — | — | AWS WAF, IAM, ALB/CloudFront; Sprinto | **Claude, MCP** |
| Forward Deployed Engineer (SF) | https://careers.kula.ai/rocketlane/54027-forward-deployed-engineer | 2026-08-28 | Python, TypeScript/JavaScript | — | ETL, data warehouses (customer side) | — | AWS/GCP, Docker, Postman; Workato/Boomi/Make | LLMs, agent orchestration, evals/guardrails; Cursor, Claude Code |
| Customer Engineer | https://careers.kula.ai/rocketlane/51921-customer-engineer | 2026-07-28 | Python, SQL | — | — | — | AWS/GCP, Docker, webhooks | LLM APIs; Cursor, Claude Code |
| GTM Engineer | https://careers.kula.ai/rocketlane/54953-gtm-engineer | 2026-09-18 | — | — | HubSpot/Salesforce CRM | — | — | Clay, Bitscale |
| BuiltIn copies (8548479, 8548456) | https://builtin.com/job/lead-software-enginner-backend/8548479 | 2026-06-22 (reposted) | same as the Kula originals | | | | | |

Not opened: Instahyre (served "Enable JavaScript and cookies" to plain fetches), Cutshort, Wellfound, Naukri. The Kula and BuiltIn texts were enough; the other boards repost the same roles.

## 3. Vendor-stated WHY (H34)

| Choice | Stated reason (verbatim, ≤15 words) | Who and where | Grade |
|---|---|---|---|
| AWS overall | "broad tools and services for builders to quickly innovate" | AWS author, APN post [L01b] | SECONDARY |
| AWS SaaS Factory | "helping us to get to market faster and deliver a delightful experience" | CTO quote, APN [L01c] | SECONDARY |
| Multi-tenant, horizontal scale | "No matter how many customers use the product, our uptime SLAs and performance remain" | CTO, APN [L07b] | SECONDARY |
| ElastiCache for jobs | "reliable queue time-sensitive background jobs" | CTO, APN [L05] | SECONDARY |
| Global from day one | "We have also architected with principles of being global from day one." | CTO, APN [L04b] | SECONDARY |
| Spring and service-oriented architecture | Personal preference: "passionate about service oriented architectures (SOA), distributed systems, Spring framework" | CTO, APN [L06] | SECONDARY |
| Java/Spring team background (INFERRED) | The founders came from Freshworks (Freshchat/Konotor, built by the CTO) [L06b][L33b]. *(Inferred)* An existing Java/Spring team and hiring pool in Chennai | APN; YourStory | INFERRED |
| India-based engineering cost | "$800,000 to $1 million when you have a product and engineering team in India" | CEO, YourStory [L34] | SECONDARY |
| Claude, Bedrock or LLM gateway | No stated reason found | — | UNKNOWN |

## 4. UNKNOWN list and what was searched

| Question | Searched |
|---|---|
| Message broker product (B12): Kafka, RabbitMQ or SQS? | APN post (names only ElastiCache for jobs); all Kula engineering postings; BuiltIn copies; search "Rocketlane Kafka / SQS / RabbitMQ" via job text |
| Search engine (B14) | APN post; postings; GitHub forks. No Elasticsearch, OpenSearch, Algolia or Typesense mention |
| Tenancy model detail (C17) | APN post says multi-tenant only; the lead security posting says tenant isolation matters; no SaaS Factory reference architecture names Rocketlane |
| Analytics store, warehouse or ETL (C18) | Postings mention ETL only on the customer side (FDE); none internally |
| Data residency regions (C19) | APN ("global from day one"); AWS Marketplace listing; no regions found here (the trust centre belongs to another workstream) |
| LLM hosting: direct Anthropic API or Bedrock/Vertex (D21) | Careers pages (Claude named, not the channel); APN 2023 predates Nitro; Nitro launch press (SiliconANGLE, Inc42, TechIntelPro snippets) names no provider; GitHub Bifrost fork (a gateway, not a provider) |
| Meeting capture and transcription vendor (D22) | Postings, press, GitHub: no Recall.ai, Deepgram, AssemblyAI or Whisper mention |
| Agent framework or vector store (D23) | Postings name "agent orchestration" generically; no LangChain, LangGraph, pgvector or Pinecone mention; the only concrete artefact is the Bifrost gateway fork |
| Which IaC tool (E27) | The posting lists four alternatives; GitHub has no Terraform repos (public forks only) |
| Mobile framework (A7) | App Store listing (no framework hint); Google Play search returned no listing; the guessed package URL returned an empty-title page and is not used |
| Salesforce, Jira and Teams integration build (G33) | Salesforce AppExchange search found nothing (rendered page); Atlassian Marketplace search showed only the query echo, no listing; Microsoft AppSource returned HTTP 403 |
| Embedded iPaaS vendor (Paragon, Prismatic, Workato Embedded) | npm (`@rocketlane/*` only RLI), GitHub org, postings: no evidence. *(Inferred)* Rocketlane runs its own app platform (RLI and Marketplace) instead |
| Engineering blog, talks, podcasts by engineering leaders | Searches returned no Rocketlane engineering blog or CTO talk; the only CTO technical statements are in the APN post |
| StackShare / Wappalyzer | StackShare returned HTTP 429; the Wappalyzer lookup page failed to render |
| H35 debt | See below |

**H35, likely constraints or debt** *(INFERRED from evidence)*:
- The **WAF, TLS/cipher, container isolation and IAM** findings described in 2026 security postings as recurring ("get ahead of the recurring findings" [L17d]) suggest the AWS edge and container estate grew faster than its security baseline.
- Running **Java (core) and Python (AI) side by side** means two runtimes, two dependency surfaces and cross-service calls between them. "unauthenticated internal endpoints" is listed as a vulnerability class to hunt [L17].
- **Generic "message brokers"** plus **ElastiCache-backed job queues** suggests queueing built on Redis. Durability and ordering guarantees for webhooks and automations may be weaker than with a log-based broker *(INFERRED; not stated)*.
- The **AI gateway fork** (Bifrost, pushed 2026-10-26) suggests multi-provider routing and guardrails are being added after Nitro launched (Mar 2026) *(INFERRED)*.

## 5. Evidence ledger

| ID | Claim | URL | Verbatim quote (<=15 words, no quote marks) | check |
|---|---|---|---|---|
| L01 | AWS chosen | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | Rocketlane selected Amazon Web Services (AWS) to build their acclaimed customer onboarding SaaS platform | static |
| L01b | Why AWS (AWS author) | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | broad tools and services for builders to quickly innovate | static |
| L01c | Why SaaS Factory (CTO) | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | helping us to get to market faster and deliver a delightful experience | static |
| L02 | RDS | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | Amazon RDS for managed relational database | static |
| L03 | ECS | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | for managed container orchestration | static |
| L04 | WAF, Global Accelerator, ALB | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | Application Load Balancer | static |
| L04b | Global from day one | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | We have also architected with principles of being global from day one. | static |
| L05 | ElastiCache job queue | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | we use AWS Elasticache for reliable queue time-sensitive background jobs | static |
| L06 | CTO preferences | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | passionate about service oriented architectures (SOA), distributed systems, Spring framework, and APIs | static |
| L06b | CTO built Konotor/Freshchat | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | I built a mobile engagement platform called Konotor which was acquired by Freshworks | static |
| L07 | Multi-tenant | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | multi-tenant architecture that can scale horizontally at every layer | static |
| L07b | Why multi-tenant | https://aws.amazon.com/blogs/apn/rocketlane-partners-with-aws-saas-factory-and-aws-marketplace-to-power-its-customer-onboarding-platform-with-saas | No matter how many customers use the product, our uptime SLAs and performance remain | static |
| L08 | Webpack | https://careers.kula.ai/rocketlane/2230-senior-software-engineer-frontend | Knowledge of module bundlers like Webpack | static |
| L09 | Backend stack | https://careers.kula.ai/rocketlane/12592-lead-software-enginner-backend | Spring, PostgreSQL, Redis, Docker, message brokers, and GitHub | static |
| L09b | Observability platform | https://careers.kula.ai/rocketlane/12592-lead-software-enginner-backend | enhancement of core features and functionalities of our observability platform | static |
| L10 | React + Java | https://careers.kula.ai/rocketlane/54233-full-stack-engineer | Design and build features end-to-end across our React frontend and Java backend. | static |
| L11 | Frontend languages | https://careers.kula.ai/rocketlane/54233-full-stack-engineer | React and modern JavaScript/TypeScript | static |
| L12 | Java backend, Python AI | https://careers.kula.ai/rocketlane/54840-product-security-engineer | Familiarity with Java/Spring Boot stacks (our backend) or Python (our AI/LLM stack) | static |
| L13 | AWS setup | https://careers.kula.ai/rocketlane/54840-product-security-engineer | Review and harden our AWS setup | static |
| L14 | ECS | https://careers.kula.ai/rocketlane/54840-product-security-engineer | container security on ECS or similar | static |
| L15 | ALB/CloudFront | https://careers.kula.ai/rocketlane/54840-product-security-engineer | ALB/CloudFront TLS policies and cipher suites | static |
| L16 | Claude and MCP | https://careers.kula.ai/rocketlane/54908-lead-product-security-engineer | building Nitro, our AI-native product layer on Claude and MCP | static |
| L17 | Agent tooling risks | https://careers.kula.ai/rocketlane/54840-product-security-engineer | unsafe code execution in agent tooling, unauthenticated internal endpoints | static |
| L17b | Sprinto | https://careers.kula.ai/rocketlane/54908-lead-product-security-engineer | Familiarity with GRC tooling (Sprinto or similar) | static |
| L17c | Tenant isolation | https://careers.kula.ai/rocketlane/54908-lead-product-security-engineer | multi-tenant SaaS environment where tenant isolation is a first-order concern | static |
| L17d | Recurring findings | https://careers.kula.ai/rocketlane/54908-lead-product-security-engineer | get ahead of the recurring findings | static |
| L18 | Dependabot | https://careers.kula.ai/rocketlane/54840-product-security-engineer | Dependabot and other CI/CD security alerts across dozens of repos | static |
| L19 | IaC options | https://careers.kula.ai/rocketlane/9833-senior-software-engineer-systems | Terraform, Ansible, Chef, or CloudFormation | static |
| L20 | Prometheus/Grafana | https://careers.kula.ai/rocketlane/9833-senior-software-engineer-systems | Prometheus, Grafana | static |
| L21 | Deployed on AWS | https://aws.amazon.com/marketplace/pp/prodview-asp3llansbsfq | Deployed on AWS | rendered |
| L22 | Slack scopes | https://slack.com/marketplace/A031D6SD4ER-rocketlane | Rocketlane has 13 Bot Token Scopes | rendered |
| L23 | Zapier CLI app | https://zapier.com/apps/rocketlane/integrations | RocketlaneCLIAPI@latest | static |
| L24 | iOS publisher | https://apps.apple.com/us/app/rocketlane/id6450273789 | ROCKETLANE TECHNOLOGIES PRIVATE LIMITED | static |
| L25 | RLI CLI | https://registry.npmjs.org/@rocketlane/rli | RLI - The Rocketlane CLI for building Apps for the Rocketlane Marketplace | static |
| L26 | Self-hosted EC2 runners | https://github.com/Rocketlane | On-demand self-hosted AWS EC2 runner for GitHub Actions custom for rocketlane | static |
| L27 | PostgreSQL CI action | https://github.com/Rocketlane | GitHub Action to setup a PostgreSQL database | static |
| L28 | JaCoCo | https://github.com/Rocketlane | publishes the JaCoCo report as a comment in the Pull Request | static |
| L29 | release-please | https://github.com/Rocketlane | generate release PRs based on the conventionalcommits.org spec | static |
| L30 | Bifrost AI gateway fork | https://github.com/Rocketlane | Fastest enterprise AI gateway | static |
| L31 | rli.bolt | https://github.com/Rocketlane | Rocketlane marketplace applications using any LLM you want! | static |
| L32 | FDE iPaaS | https://careers.kula.ai/rocketlane/54027-forward-deployed-engineer | iPaaS platforms like Workato/Boomi/Make | static |
| L33 | Chennai founding | https://yourstory.com/2022/02/rocketlane-saas-startup-unified-workspace-brands-slack-gogle-docs | This gap led the trio to start | rendered |
| L33b | Ex-Freshworks founders | https://yourstory.com/2022/02/rocketlane-saas-startup-unified-workspace-brands-slack-gogle-docs | were colleagues working for Freshworks | rendered |
| L34 | India engineering cost | https://yourstory.com/2022/02/rocketlane-saas-startup-unified-workspace-brands-slack-gogle-docs | $800,000 to $1 million when you have a product and engineering team in India | rendered |
| L35 | Chennai office address | https://builtin.com/job/lead-software-enginner-backend/8548479 | 13th Floor, Block-A, Global Infocity Park, Perungudi | static |
| L36 | BuiltWith Hotjar | https://builtwith.com/app.rocketlane.com | inferred — taken from a WebFetch summary, not verbatim | inferred |

Notes on the `check` column:
- YourStory returns HTTP 403 to curl, so those quotes were read in the browser and are marked `rendered`.
- The Kula, GitHub, App Store, npm-registry, Zapier and BuiltIn quotes were confirmed by `grep -F` against curl downloads on 2026-11-01.
