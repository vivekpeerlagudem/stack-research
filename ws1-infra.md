# Rocketlane stack study: Workstream 1 (passive infrastructure and front-end fingerprints)

Retrieval date for every row: **2026-11-01**. Method: public, passive evidence only. No login, no trial, no form input, no scanning. Normal HTTP GETs, DNS over HTTPS (dns.google), TLS handshakes, Certificate Transparency, the Statuspage public JSON, and public static bundles. robots.txt was read first for all four hosts; no disallowed path was fetched. Cookies are recorded by **name only**. Client keys embedded in public bundles (Pusher, Datadog client token, PDFTron/Apryse, Intercom) are recorded by **variable name only**.

Methods are tagged in the Evidence column:
- `[curl]`: plain HTTP or headers.
- `[dns]`: dns.google DoH.
- `[tls]`: node `tls.connect`, peer certificate.
- `[ct]`: CertSpotter CT API. crt.sh returned `502 Bad Gateway` twice.
- `[bundle-grep]`: string scan of downloaded public JS with a local Node script; byte offsets are from `artefacts/bundle-scan-*.txt`.
- `[browser]`: built-in browser, `performance` resource entries, before scope change.
- `[REA]`: REA MCP. Evidence IDs are quoted.

Bundle, browser and REA work was **stopped mid-way** on coordinator instruction; the main session owns REA bundle analysis. Bundle rows below are kept and labelled with their method.

Grades:
- CONFIRMED: vendor states it.
- OBSERVED: seen in headers, DNS, TLS, bundles or network.
- SECONDARY: third-party source.
- INFERRED: reasoning shown.
- UNKNOWN: searched, not found.

---

## 1. Findings table

| Q# | Component | Technology | Grade | Evidence (verbatim string or artefact) | URL | Date |
|---|---|---|---|---|---|---|
| A1 | Web app framework | React 18.3.1 | OBSERVED | [bundle-grep] `version:"18.3.1",rendererPackageName:"react-dom"` (app.js offset 3151266); runtime share scope `{name:"react-dom",version:"18.3.1"` | https://app.rocketlane.com/app.9b934af5ab213e71.js | 2026-11-01 |
| A1 | Login app framework | React 18.3.1, separate app | OBSERVED | [bundle-grep] `version:"18.3.1",rendererPackageName:"react-dom"` in id bundle; HTML `<title>Login</title>` | https://id.rocketlane.com/assets/index-n7c4m12N.js | 2026-11-01 |
| A2 | Main app build tool | webpack 5 with Module Federation, hashed code-splitting | OBSERVED | [bundle-grep] `self.webpackChunkrocket_frontend`; runtime `__webpack_require__`; ~300 named and hashed chunks (`bryntum-gantt`, `sheets`, `customer-portal` …); remotes `https://scarlet-mod.rocketlane.com/remoteEntry.js` (alias `signals`) and `https://agents.rocketlane.com/remoteEntry.js` | https://app.rocketlane.com/runtime.b59c055f275236b0.js | 2026-11-01 |
| A2 | Login app build tool | Vite | OBSERVED | [curl] `<script type="module" crossorigin src="/assets/index-n7c4m12N.js">`, `href="/vite.svg"` | https://id.rocketlane.com/ | 2026-11-01 |
| A2 | Language | TypeScript (compiled helpers) | INFERRED | [bundle-grep] `this&&this.__awaiter`, `__extends` (tslib emit helpers; Babel can also emit similar helpers) | https://app.rocketlane.com/app.9b934af5ab213e71.js | 2026-11-01 |
| A2 | Source maps | Referenced but not public | OBSERVED | [bundle-grep] `//# sourceMappingURL=app.9b934af5ab213e71.js.map`; [curl] HEAD `.map` → `HTTP/1.1 403 Forbidden`, `Server: AmazonS3` | https://app.rocketlane.com/app.9b934af5ab213e71.js.map | 2026-11-01 |
| A2 | Web app hosting | Static SPA on S3 behind CloudFront | OBSERVED | [curl] `Server: AmazonS3`, `X-Amz-Server-Side-Encryption: AES256`, `Via: 1.1 … .cloudfront.net (CloudFront)` | https://app.rocketlane.com/ | 2026-11-01 |
| A3 | State and data fetching | Redux with redux-saga; TanStack React Query 4.36.1 | OBSERVED | [bundle-grep] `redux-saga: Error checking hooks`; runtime `{name:"@tanstack/react-query",version:"4.36.1"` | https://app.rocketlane.com/runtime.b59c055f275236b0.js | 2026-11-01 |
| A3 | UI libraries | Ant Design, styled-components 6.5.3, Radix primitives; internal "ozone" components | OBSERVED | [bundle-grep] chunk `antd`, class `ant-btn`; `{name:"styled-components",version:"6.5.3"`; `window[Symbol.for("radix-ui")]`; sheets chunk `"ozone-select"` (19×) | https://app.rocketlane.com/runtime.b59c055f275236b0.js | 2026-11-01 |
| A3 | Design-system docs | Next.js site on Vercel | OBSERVED | [curl] `Server: Vercel`, `X-Nextjs-Prerender: 1`; [dns] CNAME `a2eab1ff8d4e31f1.vercel-dns-016.com.` | https://ui-docs.rocketlane.com/ | 2026-11-01 |
| A3 | Design system name "Ozone" | Internal design system | INFERRED | `ozone-select` component class plus CT name `ozone.rocketlane.com`; not stated by vendor | (CT) | 2026-11-01 |
| A4 | Rich-text/doc editor | CKEditor 5 with CKEditor Cloud Services real-time collaboration | OBSERVED | [bundle-grep] chunk `cke`; `RealTimeCollaborativeEditing`, `RealTimeCollaborationClient`, `cke-cs.com`, `webSocketUrl`; app.js `uploadAttachmentInCKEditor` → `/attachments/cke-upl…` | https://app.rocketlane.com/cke.e8eb7b03871e61dd.js | 2026-11-01 |
| A4 | Gantt / scheduler | Bryntum Gantt and Scheduler Pro | OBSERVED | [bundle-grep] runtime chunk names `bryntum-gantt`, `bryntum-schedulerpro`; chunk contains `bryntum.com` (35×), `SchedulerPro` (29×), `GanttTaskEditor` | https://app.rocketlane.com/bryntum-gantt.f470147093f5d80c.js | 2026-11-01 |
| A4 | Grid | AG Grid Enterprise (licensed) | OBSERVED | [bundle-grep] chunks `ag-grid-community`, `ag-grid-enterprise`; license string `Please_report_misuse_to_legal@ag-grid.com` | https://app.rocketlane.com/runtime.b59c055f275236b0.js | 2026-11-01 |
| A4 | Sheets engine | In-house sheet module; no 3rd-party spreadsheet engine found in the sheets chunk; AG Grid Enterprise likely underneath | INFERRED | [bundle-grep] sheets chunk: `updateSheetPivot`, `createdFormulas`, `deleteCellsInRange`; zero hits for handsontable/univer/SpreadJS/HyperFormula/luckysheet | https://app.rocketlane.com/sheets.0a06dae913dd59d6.js | 2026-11-01 |
| A4 | PDF and charts | PDFTron/Apryse WebViewer (OEM key), pdf.js, react-pdf, KendoReact PDF export, Highcharts | OBSERVED | [bundle-grep] env var name `REACT_APP_PDFTRON_KEY` (value withheld); chunks `pdfjs-dist`, `react-pdf`, `highcharts`; `@progress/kendo-react-pdf` | https://app.rocketlane.com/runtime.b59c055f275236b0.js | 2026-11-01 |
| A5 | Real-time transport | Pusher Channels (pusher-js, SockJS fallback) via backend-authorized `/pusher/*` endpoints | OBSERVED | [bundle-grep] `this.post("/pusher/heartbeat",e)`; `file:"sockjs"`; env var name `REACT_APP_PUSHER_CLIENT_KEY` | https://app.rocketlane.com/app.9b934af5ab213e71.js | 2026-11-01 |
| A5 | Doc collaboration | CKEditor Cloud Services (socket.io inside CKEditor collab client) | OBSERVED | [bundle-grep] `this._socket.io.off("reconnect_error")` in `cke` chunk at offset 68239 | https://app.rocketlane.com/cke.e8eb7b03871e61dd.js | 2026-11-01 |
| A5 | Vendor-dependent collaboration outage | Spaces document collaboration relies on a vendor | SECONDARY (vendor status text) | [curl] incident update: "Our team worked closely with our vendor to identify and resolved the underlying issu…" | https://status.rocketlane.com/api/v2/incidents.json | 2026-11-01 |
| A6 | Micro-frontends | Module Federation remotes for `signals` (exposes `./NitroPages`) and `agents` | OBSERVED | [bundle-grep] `signals:[{alias:"signals",name:"signals",entry:"https://scarlet-mod.rocketlane.com/remoteEntry.js"`; `name:"./NitroPages",…remoteName:"signals"`; remotes served `Server: AmazonS3` | https://app.rocketlane.com/runtime.b59c055f275236b0.js | 2026-11-01 |
| A6 | Customer portal | Same SPA, lazy `customer-portal` chunk; customer custom domains serve the same S3 object | OBSERVED | [bundle-grep] chunk `customer-portal.26e32866219b3596.js`; [curl] `supportlogic.rocketlane.com` CNAME `customdomain-2.rocketlane.com.` returns `X-Amz-Version-Id: ifNBYJj9SwekecHnQ_Mz5JqTklG976ac` (identical to app.rocketlane.com) | https://supportlogic.rocketlane.com/ | 2026-11-01 |
| A6 | Tenant routing | Per-company subdomain; API host derived from page host | OBSERVED | [bundle-grep] boot.js: `"app"===(i=t.split(".")[0])?t.replace(i,o+"//api")` … `t.replace(i,o+"//"+i+".api")`; bare app page text: "This page is not available" | https://app.rocketlane.com/js/boot.779d16b6f62a7bd174c24073856791c96c908164.js | 2026-11-01 |
| A7 | Mobile | Native-store apps exist (iOS `com.rocketlane.rocketApp` v0.7.1; Android `com.rocketlane.app`); framework UNKNOWN | OBSERVED (existence) / UNKNOWN (framework) | [curl] iTunes Search API `bundleId` `com.rocketlane.rocketApp`, `version` `0.7.1`; Play search `details?id=com.rocketlane.app`; no framework string on either listing | https://apps.apple.com/us/app/rocketlane/id6450273789 | 2026-11-01 |
| A8 | Error / RUM monitoring | Datadog RUM | OBSERVED | [bundle-grep] `REACT_APP_DATADOG_SITE:"datadoghq.com"`, chunk `datadog-rum`; [browser] cookie `_dd_s_v2` | https://app.rocketlane.com/runtime.b59c055f275236b0.js | 2026-11-01 |
| A8 | Session replay | LogRocket (disabled on login page) | OBSERVED | [bundle-grep] `LogRocket is disabled on localhost, login page, or restricted env`; chunk `logrocket` | https://app.rocketlane.com/app.9b934af5ab213e71.js | 2026-11-01 |
| A8 | In-app support / announcements | Intercom; Beamer; Freshdesk portal | OBSERVED | [bundle-grep] env var name `REACT_APP_INTERCOM_APP_ID`; `https://app.getbeamer.com/js/beamer-embed.js`; `https://rocketlanecorp.freshdesk.com` | https://app.rocketlane.com/app.9b934af5ab213e71.js | 2026-11-01 |
| A8 | Feature flags | In-house flags (no 3rd-party flag SDK found) | INFERRED | [bundle-grep] constants `COGNITO_ENABLED`, `partnerRbacEnabled`; zero hits for launchdarkly/split.io/statsig/growthbook/posthog | https://app.rocketlane.com/app.9b934af5ab213e71.js | 2026-11-01 |
| A8 | Product analytics (app) | No Mixpanel/Amplitude/Segment/Heap/Pendo SDK in app bundles; Mixpanel referenced only in Okta CSP | OBSERVED (absence in scanned bundles) | [bundle-grep] zero hits; `amplitude` hit is an SVG attribute name. [curl] login CSP `connect-src … *.mixpanel.com` | https://login.rocketlane.com/ | 2026-11-01 |
| B9 | Main API runtime | Java servlet container (Tomcat/Spring likely) | OBSERVED (servlet) / INFERRED (Spring) | [curl] `HTTP/1.1 404` + `Set-Cookie: JSESSIONID=` on `/` | https://api.rocketlane.com/ | 2026-11-01 |
| B9 | MCP server runtime | Python ASGI (uvicorn) | OBSERVED | [curl] `server: uvicorn` (404 on `/`) on mcp and mcp.eu | https://mcp.rocketlane.com/ | 2026-11-01 |
| B9 | Email service | Node.js Express on AWS App Runner behind Envoy | OBSERVED | [dns] CNAME `sjqrsxrub4.us-east-1.awsapprunner.com.`; [curl] `x-powered-by: Express`, `server: envoy` | https://emails.rocketlane.com/ | 2026-11-01 |
| B10 | Service boundaries (hosts) | Separate services: core API, admin API, conversation-service, marketplace API, sheets API, MCP, agents, emails, Teams bot, migration | OBSERVED | [bundle-grep] `https://admin.api.rocketlane.com/api/v1/accounts/domain`, `https://conversation-service.rl-platforms.rocketlane.com`, `https://api.marketplace.rocketlane.com`; [ct] `api.sheets.eu.rocketlane.com`, `mcp.eu.rocketlane.com`, `agents-eu.rocketlane.com`, `rlteamsbot-eu.rocketlane.com`, `migration-mod.rocketlane.com`, `rocket-vector.rl-platforms-eu.rocketlane.com` | https://app.rocketlane.com/app.9b934af5ab213e71.js | 2026-11-01 |
| B10 | Vector service name | `rocket-vector` on rl-platforms-eu | OBSERVED (name only) | [ct] `rocket-vector.rl-platforms-eu.rocketlane.com` | (CT) | 2026-11-01 |
| B10 | Status-page components | Only two components | OBSERVED | [curl] components.json: `Backend`, `Rocketlane Web Application` | https://status.rocketlane.com/api/v2/components.json | 2026-11-01 |
| B11 | Internal API style | REST, `/api/v1` path versioning, JSON | OBSERVED | [REA ev_5979e158…] and [browser] pre-login GETs `https://api.rocketlane.com/api/v1/accounts/get-meta`, `/api/v1/home/init`, `/api/v1/users/profile` | https://api.rocketlane.com/api/v1/home/init | 2026-11-01 |
| E26 | DNS provider | AWS Route 53 | OBSERVED | [dns] NS `ns-1158.awsdns-16.org.` `ns-1793.awsdns-32.co.uk.` `ns-1021.awsdns-63.net.` `ns-167.awsdns-20.com.` | rocketlane.com (DNS) | 2026-11-01 |
| E26 | Cloud and regions | AWS: US (us-east-1 range) and EU (eu-central-1 range) | OBSERVED (AWS) / INFERRED (exact regions) | [dns] api A `18.209.202.13`, `52.20.144.140`; api.eu A `18.153.234.137`, `52.28.56.47`, `3.125.102.119`; App Runner `us-east-1`; marketing `x-wf-region: us-east-1` is Webflow's | https://api.eu.rocketlane.com/ | 2026-11-01 |
| E26 | Wildcard DNS | `*.rocketlane.com` resolves to one CloudFront distribution | OBSERVED | [dns] `zz9q-nonexistent-7731.rocketlane.com` → `99.86.147.120`; so app.au/uk/ca/in/me names are **not** evidence of regions | (DNS) | 2026-11-01 |
| E26 | CDN | CloudFront for app, id, agents, assets-cdn, status; Cloudflare in front of Webflow (marketing), Freshdesk (help), ReadMe (developer) | OBSERVED | [curl] `X-Amz-Cf-Pop: HYD57-P4` on app; www shows both `Server: cloudflare` and `Via: … (CloudFront)` | https://www.rocketlane.com/ | 2026-11-01 |
| E26 | Apex redirect | CloudFront edge function or Lambda@Edge | OBSERVED | [curl] `X-Cache: LambdaGeneratedResponse from cloudfront`, `301` → `https://www.rocketlane.com/` | https://rocketlane.com/ | 2026-11-01 |
| E26 | TLS | Amazon RSA 2048 M01 (ACM) certificates | OBSERVED | [tls] app/www: issuer `Amazon RSA 2048 M01`, SAN `DNS:*.rocketlane.com, DNS:rocketlane.com`; api: SAN `DNS:api.rocketlane.com`; api.eu: SAN `DNS:api.eu.rocketlane.com` | https://api.rocketlane.com/ | 2026-11-01 |
| E26 | WAF | UNKNOWN | UNKNOWN | No `x-amzn-waf`, `awselb` or WAF block headers seen on any normal GET | — | 2026-11-01 |
| E26 | Email | Google Workspace mail; SES inbound; SendGrid and HubSpot outbound | OBSERVED | [dns] MX `aspmx.l.google.com.`, `inbound-smtp.us-east-1.amazonaws.com.`; SPF `v=spf1 include:_spf.google.com include:sendgrid.net include:25054879.spf01.hubspotemail.net ~all` | rocketlane.com (DNS) | 2026-11-01 |
| E26 | SaaS footprint (TXT verifications) | Anthropic (2 tokens), Notion, Zoom, DocuSign, Canva, Zoho, Microsoft, Apple, HubSpot developer, Webflow, Ahrefs | OBSERVED | [dns] TXT `anthropic-domain-verification-sq1nty=…`, `anthropic-domain-verification-7yy9s5=…`, `hubspot-developer-verification=…`, `_webflow.webflow-sandbox=…` | rocketlane.com (DNS) | 2026-11-01 |
| E27 | Containers / orchestration | App Runner for the emails service; ECS/EKS/Lambda for core API UNKNOWN | OBSERVED (App Runner) / UNKNOWN (rest) | [dns] `awsapprunner.com` CNAME; core API shows no ELB/ECS fingerprint | https://emails.rocketlane.com/ | 2026-11-01 |
| E27 | IaC | UNKNOWN | UNKNOWN | No public IaC artefact in headers, DNS or bundles | — | 2026-11-01 |
| E28 | Observability | Datadog (RUM), LogRocket; backend APM UNKNOWN | OBSERVED (frontend) | see A8 rows | https://app.rocketlane.com/runtime.b59c055f275236b0.js | 2026-11-01 |
| E29 | Release signals | Build-ID-stamped bundles; S3 object versioning | OBSERVED | [bundle-grep] `REACT_APP_BUILD_ID:"779d16b6f62a7bd174c24073856791c96c908164"` (matches boot.js and translations file names); [curl] `X-Amz-Version-Id` | https://app.rocketlane.com/ | 2026-11-01 |
| F30 | Identity provider (current) | Okta custom domain, live | OBSERVED | [dns] CNAME `rocketlane.customdomains.okta.com.`; [curl] `x-okta-request-id`; env var name `REACT_APP_OKTA_ISSUER_DOMAIN` = `https://login.rocketlane.com/oauth2/aus4w8cfckXhQ63eZ5d6` | https://login.rocketlane.com/ | 2026-11-01 |
| F30 | Identity provider (migration) | AWS Cognito code path alongside Okta; feature-flagged | OBSERVED | [bundle-grep] enum `T.COGNITO="COGNITO",T.OKTA="OKTA"`; flag `g="COGNITO_ENABLED"`; AWS SDK `signingProperties:{name:"cognito-idp"`; 56 `cognito` hits | https://app.rocketlane.com/app.9b934af5ab213e71.js | 2026-11-01 |
| F30 | New login front end | id.rocketlane.com (Vite/React) calling an auth broker on id.nitro.run | OBSERVED | [bundle-grep] `https://id.nitro.run/api/v1/authenticate?${new URLSearchParams({clientId:n.clientId,state:n.state` | https://id.rocketlane.com/assets/index-n7c4m12N.js | 2026-11-01 |
| F30 | Login methods | Google, SAML2, OAuth, standard password, magic link, TOTP, SCIM flag | OBSERVED | [bundle-grep] id app: `e.SAML2="SAML2",e.MagicLink="MAGIC_LINK_AUTHENTICATION"`; app: `S.Google="GOOGLE",S.Saml="SAML"`; `/accounts/resend-magic-link`; `{totp:t}`; `n.Scim="scim"` | https://id.rocketlane.com/assets/index-n7c4m12N.js | 2026-11-01 |
| — | Marketing site CMS | Webflow (+ HubSpot EU1 forms, Cookiebot, 6sense, Clarity, Factors.ai) | OBSERVED | [curl] `x-wf-region: us-east-1`, `surrogate-key: stage.rocketlane.com 6360d3edc696260ba2aecbc7`; [browser] `data-wf-site` = `6360d3edc696260ba2aecbc7` | https://www.rocketlane.com/ | 2026-11-01 |
| — | Help centre | Freshdesk | OBSERVED | [dns] CNAME `fdus-lb84-d68.freshdesk.com.`; [curl] cookie `_helpkit_session`; robots `Disallow: /support/tickets/` | https://help.rocketlane.com/ | 2026-11-01 |
| — | Developer portal | ReadMe | OBSERVED | [dns] CNAME `ssl.readmessl.com.`; robots `Disallow: /suggested-edits/`, `# LLM Information: https://developer.rocketlane.com/llms.txt` | https://developer.rocketlane.com/robots.txt | 2026-11-01 |
| — | Status page | Atlassian Statuspage | OBSERVED | [dns] CNAME `b24xs4l6s895.stspg-customer.com.`; [curl] `Server: AtlassianEdge` | https://status.rocketlane.com/ | 2026-11-01 |
| — | Academy | LearnWorlds | OBSERVED | [dns] CNAME `rocketlaneacademy.getlearnworlds.com.` | https://academy.rocketlane.com/ | 2026-11-01 |
| — | Embeds | Iframely (link previews), Statuspage widget | OBSERVED | [REA ev_5979e158…] requests to `cdn.iframe.ly/embed.js`, `status.rocketlane.com/embed/script.js` | https://app.rocketlane.com/ | 2026-11-01 |
| — | Teams embedding | App designed to be framed in Microsoft Teams / SharePoint | OBSERVED | [curl] CSP `frame-ancestors 'self' *.sharepoint.com teams.microsoft.com *.teams.microsoft.com` | https://trust.rocketlane.com/ | 2026-11-01 |
| — | Outage attribution | Cloud-provider incident on 2025-10-20 | SECONDARY (vendor status text) + INFERRED (AWS us-east-1 outage the same day) | [curl] incident title "Degraded performance due to cloud provider" | https://status.rocketlane.com/api/v2/incidents.json | 2026-11-01 |

---

## 2. Raw artefacts appendix

All raw files are in `rocketlane/artefacts/`:

| File | What it holds |
|---|---|
| `robots-all.txt` | robots.txt for www, apex, app, help, developer |
| `headers-<host>.txt` (25 hosts) | Full response header blocks, cookie values replaced with `<redacted>`. CNAME line prepended for CT-derived hosts. |
| `dns-hosts.json`, `dns-apex.json`, `dns-apex-txt.json`, `dns-regions.txt`, `dns-wildcard-test.json` | dns.google answers (A, CNAME, MX, TXT, NS, CAA) |
| `tls-certs.txt` | Issuer, subject, SAN, expiry |
| `certspotter.json`, `ct-names.txt` | CT issuances (100 certs, 165 names). `crtsh.json` holds the crt.sh 502 error body. |
| `status-components.json`, `status-incidents.json`, `status-incidents.txt` | Statuspage v2 API |
| `bundles-index.txt` | 22 downloaded public files with bytes and SHA-256 |
| `bundle-scan-app.txt`, `bundle-scan-id.txt` | Pattern scan output with counts, first byte offset and context |
| `chunks-wanted.txt` | Runtime chunk-name to filename map for fetched chunks |
| `rea-capture-app-prelogin.json` | REA `capture_browser_scenario` Evidence `ev_5979e158af2b8c432e1e4275397465f46357216821d9ad3b1ff61669d95185de` |
| `appstore-search.json`, `play-details.html` | Mobile listings |

### 2.1 Key header blocks (abridged; full blocks are in the files)

```
app.rocketlane.com  /
HTTP/1.1 200 OK
Server: AmazonS3
X-Amz-Server-Side-Encryption: AES256
X-Amz-Version-Id: ifNBYJj9SwekecHnQ_Mz5JqTklG976ac
Via: 1.1 5de6a028df4eb308aab30c20c3edc602.cloudfront.net (CloudFront)
Content-Security-Policy: default-src 'self' 'unsafe-inline' 'unsafe-eval' *  data: blob: filesystem: ; object-src 'none';  upgrade-insecure-requests; block-all-mixed-content;
Strict-Transport-Security: max-age=63072000; includeSubdomains; preload

api.rocketlane.com  /
HTTP/1.1 404
Set-Cookie: JSESSIONID=<redacted>

mcp.rocketlane.com  /           mcp.eu.rocketlane.com  /
HTTP/1.1 404 Not Found          HTTP/1.1 404 Not Found
server: uvicorn                 server: uvicorn

emails.rocketlane.com  /   (CNAME sjqrsxrub4.us-east-1.awsapprunner.com.)
HTTP/1.1 404 Not Found
x-powered-by: Express
server: envoy

login.rocketlane.com  /   (CNAME rocketlane.customdomains.okta.com.)
HTTP/1.1 200 OK
Server: nginx
x-okta-request-id: <id>
x-rate-limit-limit: 1200

www.rocketlane.com  /
Server: cloudflare
x-wf-region: us-east-1
surrogate-key: stage.rocketlane.com 6360d3edc696260ba2aecbc7 …
Via: 1.1 d17697ecd2381c4260bce97d1d6a2232.cloudfront.net (CloudFront)

help.rocketlane.com  /support/home
Server: cloudflare
x-fw-ratelimiting-managed: true
Set-Cookie: _x_w, _helpkit_session, __cf_bm   (names only)
content-security-policy: … *.freshdesk.com/ … *.freshworksapi.com/ … analytics.inlinemanual.com/__profile

status.rocketlane.com  /
Server: AtlassianEdge
Server-Timing: … atl-edge-pop;desc="aws-us-east-1"
```

### 2.2 DNS answers

| Name | Type | Answer |
|---|---|---|
| rocketlane.com | NS | ns-1158.awsdns-16.org., ns-1793.awsdns-32.co.uk., ns-1021.awsdns-63.net., ns-167.awsdns-20.com. |
| rocketlane.com | MX | 1 aspmx.l.google.com.; 5 alt1/alt2.aspmx.l.google.com.; 10 alt3/alt4.aspmx.l.google.com.; 10 inbound-smtp.us-east-1.amazonaws.com.; 15 …mx-verification.google.com. |
| rocketlane.com | TXT (SPF) | v=spf1 include:_spf.google.com include:sendgrid.net include:25054879.spf01.hubspotemail.net ~all |
| _dmarc.rocketlane.com | TXT | v=DMARC1; p=quarantine; rua=mailto:bimi@rocketlane.com; pct=100; |
| rocketlane.com / www | A | 99.86.147.13, .18, .58, .129 (CloudFront) |
| app.rocketlane.com | A | 54.230.27.30, .42, .60, .68 (CloudFront) |
| api.rocketlane.com | A | 18.209.202.13, 52.20.144.140 |
| api.eu.rocketlane.com | A | 18.153.234.137, 52.28.56.47, 3.125.102.119 |
| app.eu.rocketlane.com | A | 3.169.64.31, .93, .97, .108 (cert mismatch: serves `*.rocketlane.com`) |
| help.rocketlane.com | CNAME | fdus-lb84-d68.freshdesk.com. |
| developer.rocketlane.com | CNAME | ssl.readmessl.com. |
| status.rocketlane.com | CNAME | b24xs4l6s895.stspg-customer.com. |
| login.rocketlane.com | CNAME | rocketlane.customdomains.okta.com. |
| emails.rocketlane.com | CNAME | sjqrsxrub4.us-east-1.awsapprunner.com. |
| ui-docs.rocketlane.com | CNAME | a2eab1ff8d4e31f1.vercel-dns-016.com. |
| academy.rocketlane.com | CNAME | rocketlaneacademy.getlearnworlds.com. |
| supportlogic.rocketlane.com | CNAME | customdomain-2.rocketlane.com. |
| sheets / rl-platforms / marketplace | CNAME | `_…acm-validations.aws.` (ACM validation records answered as CNAME for the host) |
| zz9q-nonexistent-7731.rocketlane.com | A | 99.86.147.120 (wildcard) |

TXT verification tokens (values truncated): google-site-verification ×5, `_webflow.webflow-sandbox`, ahrefs-site-verification, MS= ×3, zoom-domain-verification, anthropic-domain-verification-7yy9s5, anthropic-domain-verification-sq1nty, zoho-verification, canva-site-verification, hubspot-developer-verification, notion-domain-verification, docusign, apple-domain-verification ×2, plus one opaque token.

### 2.3 TLS

| Host | Issuer | Subject | SAN | Expires |
|---|---|---|---|---|
| app / www | Amazon, Amazon RSA 2048 M01 | *.rocketlane.com | DNS:*.rocketlane.com, DNS:rocketlane.com | Apr 19 2027 |
| api | Amazon RSA 2048 M01 | api.rocketlane.com | DNS:api.rocketlane.com | Jan 21 2027 |
| api.eu | Amazon RSA 2048 M01 | api.eu.rocketlane.com | DNS:api.eu.rocketlane.com | Feb 19 2027 |
| app.eu | handshake fails hostname check: `Host: app.eu.rocketlane.com. is not in the cert's altnames: DNS:*.rocketlane.com, DNS:rocketlane.com` | | | |

### 2.4 CT names (Rocketlane-owned only)

The CertSpotter result also lists ~110 third-party names (`people.*`, `hrms.*`, `status.*` of other companies). These come from multi-SAN certificates issued to shared SaaS hosts and are not Rocketlane assets.

`*.api.eu`, `*.api`, `*.eu.api`, `*.eu`, `*.marketplace.eu`, `*.marketplace`, `*.proxy.api`, `*.proxy`, `*.rl-platforms-eu`, `*.rl-platforms`, `*.sheets.eu`, `*.sheets`, `<hash>.emails`, `<hash>.emails.eu`, academy, agents-eu, api, api.eu, api.marketplace.eu, api.sheets.eu, askit, assets-cdn, assets-cdn-eu, assets.eu, blog, developer, emails, emails.eu, go, help, id, login, marketplace, maturitymodel, mcp.eu, migration-mod, office, ozone, people, propel, rl-platforms, rlteamsbot-eu, rocket-vector.rl-platforms-eu, scarlet-mod-eu, series, sheets, stage, status, store, supportlogic, supportlogic.api, tv, ui-docs, welisten, plus `id.nitro.run` (all `.rocketlane.com` unless shown).

### 2.5 Bundle file list

See `artefacts/bundles-index.txt` for all 22 files with byte sizes and SHA-256. Cross-check: the boot script `js/boot.779d16b6….js` has SHA-256 `f9da18e5748a0cbd852e256130ba9f8ada4615cc3aed170a320243ad22db0ba8` in both the curl download and the REA export (`ev_31743fb1…`).

### 2.6 Matched strings with byte offsets (from `bundle-scan-app.txt`, app.9b934af5ab213e71.js)

| Pattern | Count | First offset | Context |
|---|---|---|---|
| react-dom version | 1 | 3151266 | `version:"18.3.1",rendererPackageName:"react-dom"` |
| redux-saga | 4 | 141206 | `redux-saga: Error checking hooks detected an inconsistent state` |
| CKEditor | 1 | 438717 | `uploadAttachmentInCKEditor(e){let t=new FormData` |
| ag-grid license | 6 | 672134 | `Please_report_misuse_to_legal@ag-grid.com` |
| pusher | 86 | 564896 | `this.post("/pusher/heartbeat",e)` |
| sockjs | 9 | 2301624 | `file:"sockjs",urls:{getInitial:…"/pusher"` |
| datadog | 10 | 673018 | `REACT_APP_DATADOG_SITE:"datadoghq.com"` |
| logrocket | 6 | 978798 | `LogRocket is disabled on localhost, login page, or restricted env` |
| cognito | 56 | 328526 | `signingProperties:{name:"cognito-idp",region:e.region}` |
| okta | 205 | 133916 | `r=e.oktaAuth,i=e.restoreOriginalUri` |
| saml | 18 | 419382 | `S.Google="GOOGLE",S.Saml="SAML",S),os=((T={}).COGNITO="COGNITO",T.OKTA="OKTA"` |
| magic link | 42 | 422797 | `this.post("/accounts/resend-magic-link"` |
| nitro | 403 | 263859 | `static/media/nitro-claw-logo.…svg` |
| orbits.run | 11 | 673073 | `REACT_APP_API_HOST:"https://july408.api.orbits.run"` (non-prod default baked into the build) |

---

## 3. UNKNOWN list (with what was searched)

| Q# | Item | Searched |
|---|---|---|
| A7 | Mobile framework (native, React Native or Flutter) | iTunes Search API record and description; Play listing HTML for `flutter`, `react native`, `kotlin`, `capacitor`, `cordova`, `ionic`. No hits. App binaries not downloaded (out of passive scope). |
| A5 | CRDT/OT library for Sheets or other modules | Bundle grep for yjs, y-websocket, automerge, sharedb, `EventSource`, `new WebSocket`. No hits outside CKEditor and Pusher. |
| A8 | Feature-flag vendor | launchdarkly, split.io, statsig, growthbook, posthog: zero hits |
| B9 | Core API framework beyond "Java servlet" | Only `/` touched: `404` with `JSESSIONID`; no `X-Application-Context` or error body fingerprint |
| E26 | WAF vendor | Header review on all hosts; no WAF header; no block page triggered (no probing done) |
| E26 | Exact AWS regions | IP ranges suggest us-east-1 (api) and eu-central-1 (api.eu); no reverse lookup or AWS ip-ranges.json join performed |
| E27 | ECS/EKS/Lambda for core services; IaC | Headers and DNS show no `awselb`, `x-amzn-trace-id` or API Gateway fingerprints |
| E28 | Backend APM/logging | No backend-visible headers (`x-datadog-*`, `traceparent`) on unauthenticated responses |
| E29 | CI/CD system and cadence | Only build-ID constants observed; changelog cadence assigned to another workstream |
| — | `app.eu` hosting | Resolves to CloudFront but serves the wildcard cert; EU web app host name not established |
| — | REA bundle analysis | `analyze_javascript_application` on the downloaded directory hit its 240 s per-source deadline (`provider_timeout`; partial Evidence `ev_58645a8b4a287addcaacdb06e31e73198164ecbca2d21cf15733c321f697d859`). REA `export_web_scripts` skipped bodies over 65,536 bytes (`body-size-limit`). Work was handed to the main session on coordinator instruction. |

---

## 4. Evidence ledger

`check` values:
- `static`: the quote appears in a plain HTTP fetch of the URL.
- `rendered`: only after JS or a browser.
- `artefact`: header, DNS, TLS or bundle observation; the quote is the exact observed string and the raw file is in `artefacts/`.
- `inferred`: no verbatim quote.

| ID | Claim | URL | Verbatim quote (<=15 words, no quote marks) | check |
|---|---|---|---|---|
| W01 | Marketing robots blocks /meetings | https://www.rocketlane.com/robots.txt | Disallow: /meetings | static |
| W02 | Help centre robots is Freshdesk-shaped | https://help.rocketlane.com/robots.txt | Disallow: /support/tickets/ | static |
| W03 | Developer portal robots is ReadMe-shaped | https://developer.rocketlane.com/robots.txt | Disallow: /suggested-edits/ | static |
| W04 | Developer portal publishes llms.txt | https://developer.rocketlane.com/robots.txt | # LLM Information: https://developer.rocketlane.com/llms.txt | static |
| W05 | App shell served from S3 | https://app.rocketlane.com/ | Server: AmazonS3 | artefact |
| W06 | App behind CloudFront | https://app.rocketlane.com/ | X-Amz-Cf-Pop: HYD57-P4 | artefact |
| W07 | App HSTS with preload | https://app.rocketlane.com/ | Strict-Transport-Security: max-age=63072000; includeSubdomains; preload | artefact |
| W08 | Core API is a Java servlet container | https://api.rocketlane.com/ | Set-Cookie: JSESSIONID=<redacted> | artefact |
| W09 | MCP server runs on uvicorn | https://mcp.rocketlane.com/ | server: uvicorn | artefact |
| W10 | EU MCP server runs on uvicorn | https://mcp.eu.rocketlane.com/ | server: uvicorn | artefact |
| W11 | Email service is Express | https://emails.rocketlane.com/ | x-powered-by: Express | artefact |
| W12 | Email service on App Runner | https://emails.rocketlane.com/ | sjqrsxrub4.us-east-1.awsapprunner.com. | artefact |
| W13 | Login host is Okta custom domain | https://login.rocketlane.com/ | rocketlane.customdomains.okta.com. | artefact |
| W14 | Okta responds on login host | https://login.rocketlane.com/ | x-okta-request-id | artefact |
| W15 | Marketing site is Webflow | https://www.rocketlane.com/ | x-wf-region: us-east-1 | artefact |
| W16 | Marketing behind Cloudflare | https://www.rocketlane.com/ | Server: cloudflare | artefact |
| W17 | Apex redirect is CloudFront function | https://rocketlane.com/ | X-Cache: LambdaGeneratedResponse from cloudfront | artefact |
| W18 | Help centre is Freshdesk | https://help.rocketlane.com/ | fdus-lb84-d68.freshdesk.com. | artefact |
| W19 | Help centre session cookie name | https://help.rocketlane.com/support/home | _helpkit_session | artefact |
| W20 | Developer portal is ReadMe | https://developer.rocketlane.com/ | ssl.readmessl.com. | artefact |
| W21 | Status page is Atlassian Statuspage | https://status.rocketlane.com/ | Server: AtlassianEdge | artefact |
| W22 | Status components: Backend | https://status.rocketlane.com/api/v2/components.json | Backend | static |
| W23 | Status components: web application | https://status.rocketlane.com/api/v2/components.json | Rocketlane Web Application | static |
| W24 | Cloud-provider incident | https://status.rocketlane.com/api/v2/incidents.json | Degraded performance due to cloud provider | static |
| W25 | Spaces outage involved a vendor | https://status.rocketlane.com/api/v2/incidents.json | Our team is actively working with our vendor to identify the root cause | static |
| W26 | DNS on Route 53 | https://dns.google/resolve?name=rocketlane.com&type=NS | ns-167.awsdns-20.com. | artefact |
| W27 | SPF includes Google, SendGrid, HubSpot | https://dns.google/resolve?name=rocketlane.com&type=TXT | include:_spf.google.com include:sendgrid.net include:25054879.spf01.hubspotemail.net | artefact |
| W28 | SES inbound MX | https://dns.google/resolve?name=rocketlane.com&type=MX | inbound-smtp.us-east-1.amazonaws.com. | artefact |
| W29 | Anthropic domain verification present | https://dns.google/resolve?name=rocketlane.com&type=TXT | anthropic-domain-verification-sq1nty | artefact |
| W30 | Wildcard DNS | https://dns.google/resolve?name=zz9q-nonexistent-7731.rocketlane.com&type=A | 99.86.147.120 | artefact |
| W31 | ACM certificate | https://app.rocketlane.com/ | Amazon RSA 2048 M01 | artefact |
| W32 | Separate EU API with own cert | https://api.eu.rocketlane.com/ | DNS:api.eu.rocketlane.com | artefact |
| W33 | CT lists EU MCP host | https://api.certspotter.com/v1/issuances?domain=rocketlane.com&include_subdomains=true&expand=dns_names | mcp.eu.rocketlane.com | artefact |
| W34 | CT lists vector service host | https://api.certspotter.com/v1/issuances?domain=rocketlane.com&include_subdomains=true&expand=dns_names | rocket-vector.rl-platforms-eu.rocketlane.com | artefact |
| W35 | CT lists Nitro identity host | https://api.certspotter.com/v1/issuances?domain=rocketlane.com&include_subdomains=true&expand=dns_names | id.nitro.run | artefact |
| W36 | React 18.3.1 | https://app.rocketlane.com/app.9b934af5ab213e71.js | version:"18.3.1",rendererPackageName:"react-dom" | artefact |
| W37 | webpack chunk global | https://app.rocketlane.com/app.9b934af5ab213e71.js | self.webpackChunkrocket_frontend | artefact |
| W38 | Module Federation remote for signals | https://app.rocketlane.com/runtime.b59c055f275236b0.js | entry:"https://scarlet-mod.rocketlane.com/remoteEntry.js" | artefact |
| W39 | Login app built with Vite | https://id.rocketlane.com/ | href="/vite.svg" | static |
| W40 | Source map not public | https://app.rocketlane.com/app.9b934af5ab213e71.js.map | HTTP/1.1 403 Forbidden | artefact |
| W41 | React Query version | https://app.rocketlane.com/runtime.b59c055f275236b0.js | name:"@tanstack/react-query",version:"4.36.1" | artefact |
| W42 | styled-components version | https://app.rocketlane.com/runtime.b59c055f275236b0.js | name:"styled-components",version:"6.5.3" | artefact |
| W43 | redux-saga present | https://app.rocketlane.com/app.9b934af5ab213e71.js | redux-saga: Error checking hooks detected an inconsistent state | artefact |
| W44 | CKEditor real-time collaboration | https://app.rocketlane.com/cke.e8eb7b03871e61dd.js | RealTimeCollaborativeEditing | artefact |
| W45 | CKEditor Cloud Services host | https://app.rocketlane.com/cke.e8eb7b03871e61dd.js | cke-cs.com | artefact |
| W46 | Bryntum Gantt chunk | https://app.rocketlane.com/runtime.b59c055f275236b0.js | 5698:"bryntum-gantt" | artefact |
| W47 | AG Grid licensed build | https://app.rocketlane.com/app.9b934af5ab213e71.js | Please_report_misuse_to_legal@ag-grid.com | artefact |
| W48 | KendoReact PDF export | https://app.rocketlane.com/@progress.3a60304898404c70.js | @progress/kendo-react-pdf | artefact |
| W49 | Pusher real-time endpoint | https://app.rocketlane.com/app.9b934af5ab213e71.js | this.post("/pusher/heartbeat",e) | artefact |
| W50 | Datadog RUM site | https://app.rocketlane.com/runtime.b59c055f275236b0.js | REACT_APP_DATADOG_SITE:"datadoghq.com" | artefact |
| W51 | LogRocket gated off login page | https://app.rocketlane.com/app.9b934af5ab213e71.js | LogRocket is disabled on localhost, login page, or restricted env | artefact |
| W52 | Okta and Cognito both in auth enum | https://app.rocketlane.com/app.9b934af5ab213e71.js | (T={}).COGNITO="COGNITO",T.OKTA="OKTA" | artefact |
| W53 | Cognito feature flag | https://app.rocketlane.com/app.9b934af5ab213e71.js | g="COGNITO_ENABLED" | artefact |
| W54 | New login calls Nitro auth broker | https://id.rocketlane.com/assets/index-n7c4m12N.js | https://id.nitro.run/api/v1/authenticate? | artefact |
| W55 | SAML2 and magic-link auth modes | https://id.rocketlane.com/assets/index-n7c4m12N.js | e.SAML2="SAML2",e.MagicLink="MAGIC_LINK_AUTHENTICATION" | artefact |
| W56 | SCIM feature key | https://app.rocketlane.com/app.9b934af5ab213e71.js | n.Scim="scim" | artefact |
| W57 | Conversation service host | https://app.rocketlane.com/app.9b934af5ab213e71.js | https://conversation-service.rl-platforms.rocketlane.com | artefact |
| W58 | Admin API host and versioning | https://app.rocketlane.com/app.9b934af5ab213e71.js | https://admin.api.rocketlane.com/api/v1/accounts/domain | artefact |
| W59 | Tenant subdomain to API host derivation | https://app.rocketlane.com/js/boot.779d16b6f62a7bd174c24073856791c96c908164.js | t.replace(i,o+"//"+i+".api") | artefact |
| W60 | Pre-login REST calls (REA ev_5979e158…) | https://app.rocketlane.com/ | GET https://api.rocketlane.com/api/v1/home/init | rendered |
| W61 | Custom domain serves same app object | https://supportlogic.rocketlane.com/ | X-Amz-Version-Id: ifNBYJj9SwekecHnQ_Mz5JqTklG976ac | artefact |
| W62 | Teams/SharePoint framing allowed | https://trust.rocketlane.com/ | frame-ancestors 'self' *.sharepoint.com teams.microsoft.com | artefact |
| W63 | Design-system docs on Vercel Next.js | https://ui-docs.rocketlane.com/ | X-Nextjs-Prerender: 1 | artefact |
| W64 | Academy on LearnWorlds | https://academy.rocketlane.com/ | rocketlaneacademy.getlearnworlds.com. | artefact |
| W65 | iOS app bundle id | https://itunes.apple.com/search?term=rocketlane&entity=software&limit=10 | com.rocketlane.rocketApp | static |
| W66 | Sheets uses AG Grid underneath | — | no verbatim quote — inferred from ag-grid-enterprise chunk plus pivot code in sheets chunk | inferred |
| W67 | TypeScript source | — | no verbatim quote — inferred from tslib __awaiter/__extends helpers | inferred |
| W68 | Regions us-east-1 and eu-central-1 | — | no verbatim quote — inferred from AWS IP ranges and App Runner hostname | inferred |
