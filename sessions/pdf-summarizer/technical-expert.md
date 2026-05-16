# Technical Feasibility Analysis — PDF Summarizer

*Analyst: Senior Software Architect / Integration Blocker Specialist*
*Date: 2026-05-16*

## Score: 9 / 10

This is one of the most technically straightforward SaaS ideas possible in 2026. Every required integration is a mature, well-documented public API. No scrapers, no third-party ToS grey areas, no regulated data pipelines. The build is almost embarrassingly simple — a junior developer could ship an MVP solo in under a week.

---

## Integration Dependency Map

| Dependency | Status | Notes |
|---|---|---|
| Claude API — PDF native support | ✅ Available | Full GA, all active models, 3 upload methods |
| Claude Files API (reusable uploads) | ✅ Available | Beta (`files-api-2025-04-14`), stable and documented |
| Claude Haiku 4.5 (cost model) | ✅ Available | $1.00 input / $5.00 output per MTok |
| Claude Sonnet 4.6 (quality model) | ✅ Available | $3.00 input / $15.00 output per MTok |
| Stripe Checkout (one-time payment) | ✅ Available | Standard product, no restrictions |
| AWS S3 presigned URL upload | ✅ Available | Standard, handles files up to 5 TB |
| AWS Lambda (processing) | ✅ Available | 6 MB sync limit bypassed via S3 presigned URLs |
| React + Vite frontend | ✅ Available | No integration risk |
| PDF parsing (client-side pre-check) | ✅ Available | pdfjs-dist (2M weekly downloads), pdf-parse, unpdf |

---

## 1. Core Integrations

### Claude API — PDF Support
Anthropic natively supports PDF processing in the Messages API with no additional SDK beyond the standard `@anthropic-ai/sdk`. Three delivery methods are supported:
- **URL reference** — simplest, for publicly accessible PDFs
- **Base64 inline** — for user-uploaded files (adds payload weight)
- **Files API** (`file_id`) — upload once, reference forever; best for this use case

Limits confirmed from official docs:
- Max request size: 32 MB
- Max pages: 600 (100 for 200k context models)
- Each page processed as text + image (vision pipeline)
- Token cost per page: ~1,500–3,000 tokens (text) + image tokens per page
- No encryption/password-protected PDFs supported

The Files API (beta header `anthropic-beta: files-api-2025-04-14`) allows uploading once and referencing by `file_id`. Files are retained until explicitly deleted. Organizations get 500 GB storage. This is the right architecture: user uploads → S3 (or direct to Anthropic Files API) → summarize → delete.

Zero Data Retention (ZDR) is available for the PDF feature if the business needs it for enterprise customers.

### Claude Haiku 4.5 — Cost Model
Current pricing (May 2026): **$1.00 / MTok input, $5.00 / MTok output**. Batch processing cuts this by 50%.

**Real cost per summarization (Haiku 4.5):**
- Typical 20-page PDF: ~40,000–60,000 tokens input (text + vision per page)
- Summary output: ~500–1,000 tokens
- Input cost: ~$0.04–$0.06 per 20-page PDF
- Output cost: ~$0.0025–$0.005
- **Total: ~$0.04–$0.07 per summary**

At $0.50/credit (the product's pricing), margin is **85–92%** per summarization on Haiku. Even on Sonnet 4.6 ($3 input / $15 output), cost would be ~$0.12–$0.18 per summary — still 64–76% margin at $0.50. These economics are extremely healthy.

**Prompt caching** can further cut costs by 90% on cached prefixes for repeat document types or system prompts.

### Stripe — One-Time Credit Packs
Stripe Checkout supports one-time payment sessions natively. No subscription complexity required. Pattern: user pays $5 → Stripe webhook → Lambda credits +10 to their account in DynamoDB → user gets summary. This is the simplest Stripe integration pattern that exists. Full Stripe SDK available for Node.js.

### File Upload Architecture
AWS Lambda has a 6 MB synchronous payload limit and 1 MB async limit — well below the 32 MB PDF ceiling. The standard bypass is S3 presigned URLs:
1. Lambda generates presigned PUT URL
2. Browser uploads directly to S3 (bypasses Lambda entirely)
3. S3 event triggers processing Lambda
4. Lambda sends PDF to Anthropic Files API or base64-encodes it inline

This is a solved, well-documented pattern with multiple public tutorials.

---

## 2. Data Availability

No external data sourcing required. The user provides the PDF. The only data the system needs to store:
- User email + credit balance (DynamoDB)
- Stripe customer ID
- Optionally: summary history (nice-to-have)

No scraping, no licensed feeds, no third-party data dependencies.

---

## 3. Regulatory & Compliance

### GDPR
This is the main compliance surface to handle:
- User-uploaded PDFs may contain personal data (PII in contracts, financial docs, medical records)
- GDPR requires documented retention periods, right to deletion, and data processing agreements (DPAs)
- **Anthropic provides a DPA** and supports Zero Data Retention (ZDR), which means PDFs are not stored after the API response returns — this is a strong compliance position
- The product must: (a) delete uploaded files after processing, (b) publish a clear privacy policy, (c) define retention periods, (d) sign a DPA with Anthropic

**Minimum viable GDPR posture:** auto-delete PDFs from S3 immediately after Anthropic call returns, use Anthropic ZDR, add standard privacy policy. This is achievable in the MVP.

### HIPAA
Not required unless explicitly marketing to healthcare. The product should not accept HIPAA-regulated documents without a BAA with Anthropic (available for enterprise). Avoid marketing to healthcare initially.

### PCI DSS
Stripe handles all card data. The product never touches raw card numbers. PCI scope is minimal (SAQ A).

### No financial or sector-specific regulation applies in the base case.

---

## 4. Build Complexity

**MVP components:**
1. React + Vite frontend with PDF drag-and-drop, style selector, Stripe Checkout redirect
2. API Gateway + Lambda (Node.js 20, TypeScript)
3. S3 presigned URL upload flow
4. Anthropic Messages API call with PDF + prompt
5. Stripe webhook handler → DynamoDB credit debit
6. Summary display page

**Hardest parts:**
- Handling scanned/image-only PDFs (no text layer): Claude's vision pipeline handles this, but output quality varies and costs more tokens. No fix needed — just set expectations.
- Large PDFs (100+ pages): must implement chunking or warn users. Straightforward to add a page count check pre-flight.
- Stripe webhook reliability: idempotency keys required to avoid double-crediting on webhook retries. Standard Stripe pattern.

**Time estimate for a 2-person team:** 3–5 days to functional MVP. 1–2 weeks to production-ready with error handling, loading states, and basic analytics. The $200 build cost estimate in the pitch is realistic if counting only API/hosting costs.

**Rate limits to watch:** New Anthropic accounts start at Tier 1 (50 RPM, $100/month spend cap). At $0.05/summary, $100/month = 2,000 summaries before hitting the monthly spend limit. Tier 2 requires a $40 deposit and raises the cap significantly. This is not a blocker — just plan for it at launch.

---

## 5. Key Technical Risk

**Single most likely technical failure mode: scanned PDF quality.**

A large proportion of "PDFs" in the wild are scanned document images with no text layer — old contracts, government forms, academic papers. Claude's vision pipeline can OCR these, but:
- Token costs spike (image-heavy = more tokens per page)
- Quality degrades on low-resolution scans or handwriting
- Processing time increases

At $0.50/credit, a 50-page scanned document could cost $0.20–$0.40 in API costs alone, leaving thin margins. The fix is a pre-flight page count cap (e.g., max 50 pages) and a warning for image-heavy PDFs. This is implementable in the MVP.

**Secondary risk: Anthropic API rate limits at launch.** A viral spike could hit the Tier 1 cap of 50 RPM or $100/month instantly. Mitigation: pre-fund to Tier 2 or Tier 3 before launch, implement a queue with SQS for burst absorption.

---

## Summary: What's Available vs. Restricted

| Component | Status | Detail |
|---|---|---|
| Claude PDF processing | ✅ Available | GA, all models, 3 upload methods, 32 MB / 600 pages |
| Claude Files API | ✅ Available | Beta but stable, 500 GB org storage |
| Claude ZDR (GDPR compliance) | ✅ Available | Files not stored post-response |
| Claude Haiku 4.5 (cost) | ✅ Available | $0.04–$0.07 per summary, 85%+ margin |
| Stripe one-time payments | ✅ Available | Standard Checkout, no restrictions |
| AWS S3 large file uploads | ✅ Available | Presigned URLs, standard pattern |
| PDF parsing (Node.js) | ✅ Available | pdfjs-dist, pdf-parse, unpdf — all mature |
| Encrypted/password PDFs | ❌ Unavailable | Claude does not support these |
| Handwritten/low-res scan quality | ⚠️ Restricted | Works but token cost and quality vary |
| Anthropic Tier 1 spend cap | ⚠️ Restricted | $100/month; upgrade to Tier 2 before launch |

---

## Sources

- [Claude PDF Support — Official Docs](https://platform.claude.com/docs/en/build-with-claude/pdf-support)
- [Claude Files API — Official Docs](https://platform.claude.com/docs/en/build-with-claude/files)
- [Claude API Pricing 2026](https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration)
- [Anthropic API Rate Limits](https://platform.claude.com/docs/en/api/rate-limits)
- [AWS S3 Presigned URL Upload Pattern](https://www.cloudthat.com/resources/blog/simplifying-large-file-uploads-to-amazon-s3-with-presigned-urls)
- [PDF Parsing Libraries for Node.js (2025)](https://strapi.io/blog/7-best-javascript-pdf-parsing-libraries-nodejs-2025)
- [GDPR Compliance for SaaS 2025](https://complydog.com/blog/gdpr-compliance-checklist-complete-guide-b2b-saas-companies)
- [Stripe SaaS Integration](https://docs.stripe.com/saas)
- [Claude Haiku 4.5 Cost Analysis](https://caylent.com/blog/claude-haiku-4-5-deep-dive-cost-capabilities-and-the-multi-agent-opportunity)
