# LegacyLeads — OmniLead Nexus

Lead generation, delivery, and data enrichment for realtors, attorneys, and B2B sales teams. FSBO/expired prospecting, neighborhood market analytics, contact enrichment via a skip-trace cascade, TCPA compliance guardrails, and a credit economy — built as part of the RealEstateCRM project family for Excel Legacy Realty Group.

## Status

**Active development, early MVP.** Version 0.5.0 (per `VERSION.md` and `CHANGELOG.md`). Last commit 2026-11-12 (removing a Windows-only SWC dependency that broke Linux installs). Recent work: listings migration plus backend build/start fixes (Nov 2), honest database-down error handling (Nov 1), and removal of all mock data so the app runs on real PostGIS listings and a live skip-trace cascade (Oct 21).

The full product vision is specified in `VISION.md` ("OmniLead Nexus"). The "Status vs Vision" section below tracks exactly what is implemented and what remains.

## What it does

- **FSBO / Expired prospecting** — dead-listing queue ranked by estimated owner equity
- **Neighborhood market analytics** — city-level market slices for micro-trend spotting
- **Contact enrichment** — skip-trace cascade with phone/email discovery
- **TCPA compliance** — quiet-hours enforcement by local timezone before any dialing
- **Credit economy** — basic vs premium data buckets with atomic credit deduction (BullMQ + Redis)
- **AI-sync webhooks** — `/webhooks/ai-sync` accepts agent jobs (AI Sync Queue with compliance guardrails)
- **Custom CRM integration** — REST API (`/listings`, `/augmentpropertyrecord`, `/webhooks/ai-sync`, `/health`)

## Architecture

- **Frontend:** Next.js 14 (App Router) + Tailwind — `frontend/`
- **Backend:** Express + TypeScript — `backend/` (BullMQ queues, Postgres/`pg`, Redis)
- **AI:** DeepSeek V4 (fallback OpenAI) for record augmentation
- **Compliance:** `compliance.ts` — TCPA legal-hours guard on every queued job

### Key backend modules

| Module | Purpose |
|---|---|
| `src/queue.ts` | BullMQ `AI_Sync_Queue` — background lead sync with TCPA guard |
| `src/skiptrace.ts` | Multi-provider skip-trace cascade |
| `src/compliance.ts` | Timezone-aware legal calling hours |
| `src/credits.ts` | Credit deduction / ledger |
| `src/deduplicate.ts` | Cross-account record stitching |

Tests: Jest suite (`app.test.ts`, `compliance.test.ts`, `skiptrace.test.ts`).

## Status vs Vision (OmniLead Nexus)

### Implemented

- Frontend pages: Contacts, FSBO/Expired, Neighborhoods, Help Center, Settings & Sync
- Backend REST API (4 routes), AI Sync job queue with TCPA guardrail, credit deduction, skip-trace cascade, dedup
- DeepSeek-powered record augmentation
- Real PostGIS listings + live skip-trace (mock data removed Oct 21, 2026)

### Remaining (from VISION.md)

- Universal MLS ingestion gateway (RESO Web API abstraction) — per-state MLS credentials
- Interactive map prospecting (Mapbox/Supercluster polygon draws, saved lists)
- Tiered credit purchasing + Stripe billing + top-off UI
- Multi-LLM agent routing (OpenAI/Anthropic/Gemini/Hermes/OpenClaw adapters)
- Daily lead delivery reports (email), delivery schedule UI
- Import from file + custom field mappings, custom dispositions
- Search by phone/address across synced + imported records
- White-label API for major CRMs + custom integration builder

## What's working / what's not

- The core pipeline (listings, skip-trace, compliance, credits, AI sync) is implemented and tested. The Oct 21 mock-data removal means missing keys or a down database now fail honestly instead of producing fake output.
- No production deployment is documented in this repo — `DEPLOY.md` describes a Hetzner deploy (port 3215), but there is no live URL, uptime, or user evidence in the repo itself. Treat as under development, not production-proven.
- The frontend is built by the main RealEstateCRM repo's CI (legacyleads-frontend job), which was green as of Nov 12, 2026 (RealEstateCRM v0.63.6).

## Relationship to RealEstateCRM

This is a satellite component of [RealEstateCRM](https://github.com/RealEstateCRMLLC/RealEstateCRM) — the lead-generation and enrichment service that feeds the main CRM. It is a separate service, not a folder inside the main CRM repo.

## Running it

```bash
# Frontend (port 3005 in prod)
cd frontend && npm install --legacy-peer-deps && npm run dev

# Backend (port 3006 in prod)
cd backend && npm install && npm run dev   # ts-node --esm src/index.ts
```

Environment: `DATABASE_URL`, `REDIS_URL` (optional — degrades gracefully), `DEEPSEEK_API_KEY`. See `DEPLOY.md` and the docs below.

## Documentation

| File | Purpose |
|---|---|
| `VISION.md` | Full OmniLead Nexus product spec |
| `MEMORY.md` | Architecture notes and gotchas |
| `CHANGELOG.md` | Version history |
| `DEPLOY.md` | Deployment (Hetzner, port 3215) |

## Links

- Main CRM: https://github.com/RealEstateCRMLLC/RealEstateCRM
- Org: https://github.com/RealEstateCRMLLC
