# ScriptPal NZ

> A patient-facing Progressive Web App for medication management — search NZ medications, understand your scripts, and generate a clean summary PDF for your doctor.

🌐 **Live:** [scriptpal.co.nz](https://scriptpal.co.nz/)
🎨 **Author:** [Dr. Hannah Brotheridge](https://github.com/dr-hannah-brotheridge)

---

## Overview

ScriptPal NZ helps patients understand their medications and bring structured, relevant information to their GP appointments — saving clinic time and improving care quality.

Originally prototyped in FlutterFlow, the app has been rebuilt from the ground up as a custom-coded **Next.js PWA** with a Supabase backend. It is designed to be:

- **Lightweight** — installable on any phone, works offline, no app store required.
- **Patient-centric** — plain-language medication info, pill photos, and a one-tap medical summary.
- **Secure by design** — per-user data isolation via PostgreSQL Row-Level Security (RLS).
- **Clinically grounded** — built by an NZ-registered doctor (MBChB, University of Auckland).

### Motivation

> *"I want to provide solutions to the minute amount of time people have to see their doctor. By allowing people to have lists of their medications in one absolute place and a generated report of the relevant and pertinent info that doctors want — the treatment will be much better."*
> — Dr. Hannah Brotheridge

---

## Key Features

### 🔍 Medication Search
- Lookup from **~2,994 NZ reference medications** (generic names + brand names), sourced from the NZULM monthly CSV.
- Server-side `ILIKE` search across both `medication_name` and `brands` — so typing "Humal" finds "Insulin lispro (Humalog)".
- Plain-language fields: *what it does in your body, what it protects, common dose range, side effects, what symptoms to watch for, when to seek help*.

### 💊 My Medications
- Add medications from the reference database to your personal list.
- Capture structured dosage (strength, unit, quantity, form) and frequency, plus free-text instructions and start/end dates.
- **Pill photos** — upload up to 4 photos per medication (stored in Supabase Storage with signed URLs) so you always know what your pills look like.

### 📄 Doctor Summary PDF
- One-tap generation of a clean, A4 **Medical Summary PDF** (built with `@react-pdf/renderer`).
- Includes: profile info, emergency contact, allergies, GP/pharmacy details, and current medications with dosages.
- Timezone-aware (Pacific/Auckland) generation timestamp.
- Share or print directly from the phone.

### 👤 Profile & Settings
- Patient details: name, DOB, NHI number.
- Allergies, emergency contact, primary GP, and pharmacy info.
- Account management: password reset, legal docs (TOS, Privacy, Medical Disclaimer), account deletion.

### 📱 PWA
- Installable on iOS/Android home screen via `manifest.webmanifest`.
- Offline-capable service worker (`public/sw.js`).
- Custom app icons (192px, 512px, maskable).
- Portrait-locked, standalone display mode.

---

## Tech Stack

| Category | Technology |
| --- | --- |
| **Frontend** | Next.js 15 (App Router), React 19, TypeScript |
| **Styling** | Tailwind CSS v4 (theme tokens in `app/globals.css`, no config file) |
| **Backend** | Supabase (PostgreSQL + Auth + Storage) via `@supabase/ssr` |
| **PDF** | `@react-pdf/renderer` |
| **AI / LLM** | Anthropic SDK (`@anthropic-ai/sdk`), OpenAI SDK (reference data enrichment) |
| **Data Sources** | NZULM / NZ Formulary scrapers (`cheerio`, `pdf-parse`) |
| **Deployment** | Vercel (auto-deploy on Git push) |
| **PWA** | Web App Manifest + Service Worker |

> **No icon or date libraries.** Inline SVGs in `components/icons.tsx`; native `Intl` in `lib/date.ts`.

---

## Architecture

### Data Model

| Table | Scope | Description |
| --- | --- | --- |
| `total_medications` | Read-only (~2,994 rows) | Reference medication database (generics + auto-created generics from NZULM trade rows). Includes `raw_scraped_context` (cached NZF monograph text) and 9 patient-facing educational columns enriched by GLM-5.2. Written to via `import-nzulm.js` and `/admin`. |
| `nzulm_lookup` | Read-only | NZULM trade/brand name lookup. Links `brand_name` → `generic_medication_id` → `total_medications.id`. The `brands` column is backfilled via `refresh_brands_from_lookup()` RPC. |
| `patient_details` | Per-user (1:1 with `auth.users`) | Profile: name, DOB, NHI, allergies, emergency contact, GP, pharmacy. |
| `patient_medications` | Per-user | A user's tracked medications. `medication_id` → `total_medications.id`. |
| `medication_photos` | Per-user | Photo metadata rows; images stored in Supabase Storage bucket `medication-photos`. |

### Auth & Security

- **Middleware** (`middleware.ts`) guards all non-public routes — unauthenticated users are redirected to `/login`.
- **Row-Level Security (RLS)** is enabled on `patient_details` and `patient_medications`. Every policy is scoped to `auth.uid()` — users can only read/write their own rows (SELECT, INSERT, UPDATE, DELETE).
- **Admin access** uses a server-only Supabase client (service role key) in `/admin`. Access is restricted to an email allowlist (`ADMIN_EMAILS` env var).
- The service role key is **never** exposed to the browser — it's used solely in server components and route handlers.

### Route Map

**App routes (authenticated):**
`/home` · `/search` · `/search/[id]` · `/my-meds` · `/my-meds/[id]` · `/summary` · `/settings` · `/account` · `/legal` · `/about`

**Auth routes (public):**
`/login` · `/signup` · `/reset-password` · `/auth/callback` · `/auth/update-password`

**Admin routes (email-allowlisted):**
`/admin/populate` · `/admin/api/sync-row` · `/admin/api/process-row`

---

## Admin & AI Enrichment

The `/admin/populate` panel lets authorized admins review and enrich the ~2,994-medication reference database:

- **Approve & Sync** individual rows or fill blank columns.
- Existing manual text is **never** automatically overwritten — AI enrichment only fills empty fields.
- Reference fields include: drug class, why prescribed, what it does, common dose range, side effects, symptoms to watch for, and when to seek help.
- Powered by LLM-backed prompts (Anthropic API and NeuralWatt GLM endpoint for cost-optimised enrichment).

### Data Pipeline: NZULM CSV → NZF Scrape → GLM Enrichment

The reference medication database is built and maintained through a three-stage pipeline:

**Stage 1 — NZULM CSV Import** (`import-nzulm.js`):
- Parses the NZULM `prescribing_term_selection_list_dump.csv` (monthly release).
- **Pass 1:** Inserts generic medication names (truncated at dosing metrics, capitalised) into `total_medications`.
- **Pass 2:** Inserts trade/brand rows into `nzulm_lookup`. When a trade row's generic parent doesn't exist (e.g. "insulin lispro" has no generic CSV row), it **auto-creates** the missing generic so brands like Humalog aren't lost.
- Backfills `total_medications.brands` via `refresh_brands_from_lookup()` RPC.
- Strict data-cleaning: drops `[obsolete]` entries, de-duplicates, strips manufacturer parenthesised substrings.

**Stage 2 — NZF Scraping** (`scripts/lib/nzf-scraper.js`):
- Scrapes `nzf.org.nz` monograph pages for each medication.
- Extracts labelled clinical section text (Drug action, Cautions, Adverse effects, Dosing, etc.).
- Caches scraped text in `total_medications.raw_scraped_context` — serving as **RAG context** for the LLM enrichment step.
- Handles multi-monograph medications (e.g. methotrexate — oncology + autoimmune).
- Marks non-drug items (devices, supplements) as `NOT_IN_NZF` so GLM uses a safe device-classification prompt.

**Stage 3 — GLM-5.2 Enrichment** (`build-educational-db.js`):
- Loops every medication with blank educational fields and sends cached NZF text as RAG context to **NeuralWatt GLM-5.2** (OpenAI-compatible API).
- GLM generates 9 patient-facing fields per medication at a **6th-grade reading level**: drug class, why prescribed, what it does, what it protects, if you stop it, common dose range, side effects, symptoms to watch for, when to seek help.
- **Idempotent** — only fills blank fields; never overwrites existing human-reviewed content.
- **Cost-optimised**: async concurrency pool (`p-limit`, default 10) keeps GLM's prompt cache hot; smart re-scrape modes skip GLM calls when cached text is unchanged.
- Resilient: per-medication try/catch; truncated JSON responses are auto-repaired.

**Verification scripts** in `scripts/`:
- `verify-rows.js`, `verify-five-rows.js`, `verify-success.js` — data integrity checks.
- `db-diagnostic.js`, `check-column.js`, `check-col-widths.js` — schema/column diagnostics.
- `probe-nzf-*.js` — NZ Formulary endpoint/formatter probe utilities.

---

## Getting Started

### Prerequisites

- Node.js 18+
- A Supabase project (PostgreSQL + Auth + Storage)
- A Vercel account (for deployment)

### Environment Variables

Copy `.env.example` to `.env.local` and fill in the values. See `.env.example` for full details.

| Variable | Scope | Purpose |
| --- | --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | public | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | public | Supabase anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | **server only** | `/admin` sync + account deletion. Never `NEXT_PUBLIC_`. |
| `ADMIN_EMAILS` | server only | Comma-separated emails allowed to access `/admin` |
| `ANTHROPIC_API_KEY` | server only | Claude enrichment in `/admin/populate` (optional) |
| `NEURALWATT_BASE_URL` | server only | GLM endpoint for `scripts/build-educational-db.js` |
| `NEURALWATT_API_KEY` | server only | API key for NeuralWatt |
| `NEURALWATT_MODEL` | server only | Model identifier (e.g. `GLM-5.2`) |

**Supabase Auth config:** Add your redirect URLs in *Dashboard → Authentication → URL Configuration*:
```
https://www.scriptpal.co.nz/auth/callback
https://scriptpal.co.nz/auth/callback
```

### Install & Run

```bash
npm install
npm run dev          # local development (http://localhost:3000)
```

### Build & Deploy

```bash
npm run build        # production build (what Vercel runs)
```

Deploy by pushing to the connected Git branch — Vercel builds and deploys automatically.

### Utility Scripts

```bash
npm run build:edu-db     # rebuild the educational medication database
npm run build:pwa-icons  # regenerate PWA icons from public/icon-source.png
```

---

## Project Structure

```
.
├── app/
│   ├── (app)/            # Authenticated app routes (home, search, my-meds, summary, settings)
│   ├── account/          # Account management + API
│   ├── admin/            # Admin panel (populate, sync-row, process-row APIs)
│   ├── auth/             # Auth callback + password update
│   ├── login/            # Login page
│   ├── signup/           # Signup page
│   ├── reset-password/   # Password reset
│   ├── layout.tsx        # Root layout
│   └── globals.css       # Tailwind v4 theme tokens
├── components/
│   ├── admin/            # Admin data grid
│   ├── forms/            # Medication + profile forms
│   ├── AppChrome.tsx     # App shell + page title
│   ├── BottomNav.tsx     # Mobile bottom navigation
│   ├── MedicationPhotos.tsx  # Photo upload + lightbox
│   ├── OnboardingFlow.tsx    # Pre-auth onboarding
│   └── ...
├── lib/
│   ├── pdf/DoctorSummaryPdf.tsx  # @react-pdf/renderer Medical Summary
│   ├── supabase/         # Server, client, and admin Supabase clients
│   ├── admin-auth.ts     # Server-only admin email check
│   ├── types.ts          # TypeScript interfaces (mirrors live Supabase schema)
│   ├── constants.ts      # Admin emails + reference field definitions
│   └── date.ts           # NZ timezone date formatting
├── scripts/              # Data scraping + DB build utilities
├── supabase/migrations/  # SQL migrations (RLS policies, schema changes)
├── public/               # PWA manifest, service worker, icons
├── middleware.ts         # Auth-guarded routing
└── next.config.ts
```

---

## Engineering Highlights

- **RLS-hardened data isolation** — Every patient table has strict `auth.uid()`-scoped policies for SELECT, INSERT, UPDATE, and DELETE. A dedicated migration (`20260709_000001`) replaced previously permissive policies with properly scoped ones.
- **Paginated server-side search** — Bypasses PostgREST's 1000-row default cap by looping in chunks, with `ILIKE` filtering across both generic and brand names.
- **Zero-dependency UI** — No icon library (inline SVGs) and no date library (native `Intl`). Keeps the bundle lean for mobile.
- **Structured dosage capture** — Dosage is captured as structured fields (strength, unit, quantity, form) then composed into a free-text string, giving both UX and storage flexibility.
- **AI-assisted data pipeline** — Cost-optimised LLM enrichment (model cascading, blank-field-only writes) populates the reference medication database without overwriting human-reviewed content.

---

## About the Author

**Dr. Hannah Brotheridge** is an NZ-registered doctor (MBChB, University of Auckland) and clinical product creator. She designs, architects, and evaluates AI systems and applications for healthcare — bridging the gap between clinical safety protocols and technical system design.

- 🌐 [scriptpal.co.nz](https://scriptpal.co.nz/)
- 🌐 [signalhealth.dev](https://signalhealth.dev/)
- 💼 [GitHub](https://github.com/dr-hannah-brotheridge)

> *"If we can improve the clinical safety and validity, the empathy and rapport of AI; then this tool will be indispensable. We need to be looking upstream at how people can get help."*

---

## License

This project is proprietary. All rights reserved.
