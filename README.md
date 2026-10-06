# ScriptPal NZ

> A patient-facing Progressive Web App that helps New Zealanders understand their
> medications and bring a clean, structured medication summary to a doctor.

**[▶ Live: scriptpal.co.nz](https://scriptpal.co.nz/) &nbsp;·&nbsp; [Try the demo — no account needed](https://scriptpal.co.nz/demo) &nbsp;·&nbsp; [Watch the demo video](./public/how-it-works.mp4)**

Built by **[Dr. Hannah Brotheridge](https://github.com/dr-hannah-brotheridge)**, MBChB (University of Auckland) — a non-practicing registered medical doctor who designs and engineers software for healthcare.

---

## Why it exists

Medication is the most common medical treatment in the world, yet the experience of taking it is often confusing. Patients juggle blister packs, repeats and pharmacy labels; they lose track of what each medicine is for; and at a GP appointment there is rarely time to reconstruct an accurate medication list.

**ScriptPal turns that into a two-minute, structured, plain-language experience** — so patients understand what they're taking and arrive at appointments prepared. Better-informed patients and cleaner summaries mean less clinic time lost to guesswork and safer, higher-quality care.

The app was prototyped in FlutterFlow, then **rebuilt from scratch as a custom-coded web application** to gain full control over privacy, performance, accessibility and cost.

---

## What it does

### 🔍 Find and understand any NZ medicine
Searching across ~3,000 New Zealand reference medicines returns plain-language explanations written the way a doctor would explain them — what the medicine is, why it's prescribed, what it does in your body, common side effects, what to watch out for, and what happens if you stop. Searching a brand name ("Humalog") finds the generic ("Insulin lispro").

### 💊 Build a personal medication list
Add medications with structured dosage, frequency, dates and instructions; attach pill photos; and record custom supplements and natural remedies that aren't in any national database. Finished medicines move automatically into a "Previous Medications" history with notes on why they were stopped.

### 🩹 Mirror a pharmacy blister pack
For patients whose medicines are packed by their pharmacy, a dedicated **Pack View** reproduces the physical blister pack slot by slot (7 days × 4 times of day). Patients can see exactly what to take, and when — and the app remembers their preferred layout.

### 📄 Generate a Doctor Summary in one tap
A clean, print-ready A4 PDF containing the patient's option details, allergies, emergency contact, GP and pharmacy, current medications (including supplements) and a completed-medications history — designed to be shared or printed straight from the phone at an appointment.

### 🧪 A full demo before you sign up
Anyone can explore the entire app against a fictional sample patient at **`/demo`** — no account, no sign-up. It walks through real search results, the blister pack, the educational content and the summary PDF, turning visitors into users without a wall.

### 📱 Install it like an app
ScriptPal is a Progressive Web App: installable to the home screen on iOS and Android, works offline, and needs no app store — keeping it lightweight and instantly updatable.

### 🔒 Privacy and safety as first-class features
- Built for **New Zealand health information**, designed in line with the **Privacy Act 2020** and the **Health Information Privacy Code 2020**.
- **Multi-person accounts** — manage medicines for yourself or family members ("Me", "Mum", "Dad"), each with its own private medication list, blister pack and doctor summary.
- **Optional per-person privacy PIN**, enforced by the database itself — so one family member's information is a hard barrier the account owner cannot override, with a safe recovery path if a PIN is forgotten.
- Data residency in Australia (Sydney), private photo storage via short-lived signed links, and account deletion that permanently removes both records *and* uploaded photos.

### 🩺 Grounded in official sources
Reference medicines come from the national medicines dataset, educational content is aligned to the New Zealand Formulary, and the app links out to **My Medicines** (Te Whatu Ora) official patient leaflets rather than copying them — matching conservatively so it would rather show *no* link than a *wrong* one.

### 📷 Scan a pharmacy label (COMING SOON)
Point the camera at a pharmacy label and ScriptPal reads it with an AI vision model running **inside New Zealand** (Amazon Bedrock, Auckland) and pre-fills the medication form for the patient to review — the label image is processed in memory and never stored.

---

## Skills this project demonstrates
ScriptPal is a single-developer, end-to-end product from clinical problem definition to shipped, hosted software. It demonstrates:

**Product & domain**
- Identifying a real clinical problem and turning it into a usable product
- Healthcare domain expertise and regulatory awareness (Privacy Act, Health Information Privacy Code, medical-disclaimer framing)
- User-centred design: onboarding, accessibility (font-size control, colour never used alone), a "try before you buy" demo, and trust-building consent flows

**Full-stack engineering**
- **Next.js 15 (App Router) + React 19 + TypeScript** — a modern, type-safe web application
- **Tailwind CSS v4** design system with no third-party UI, icon or date dependencies, keeping the bundle lean for mobile
- **Supabase (PostgreSQL + Auth + Storage)** backend, including a substantial relational data model, database functions and SQL migrations
- **Server-side PDF generation** for the Doctor Summary
- **PWA engineering** — service worker, offline handling, install prompts and manifest

**Security & privacy engineering**
- Row-Level Security (RLS) so users can only ever access their own data — with automated tests that *prove* isolation rather than assume it
- Server-only secrets, admin gating, rate limiting against brute-force and spam, strict security headers (CSP, HSTS) and audit logging
- Database-enforced privacy PINs with bcrypt hashing and one-time recovery codes

**AI integration**
- A vision/OCR pipeline using an in-region AWS Bedrock model for pharmacy-label reading
- An LLM-powered content pipeline that generates and quality-checks plain-language educational content, with display-time sanitisation so raw model output is never shown

**Data engineering & quality**
- Ingestion and transformation of a national medicines dataset
- Idempotent, fault-tolerant, concurrency-controlled batch jobs with verification scripts
- CI/CD with automated dependency scanning, linting and build gates on every change

**Cloud & delivery**
- Deployment and hosting on **Vercel**, with **Supabase**, **AWS**, **Upstash** and **Resend** in the stack
- Shipping and maintaining a real, live product used in a regulated domain

---

## Tech stack at a glance

| Layer | Technology |
| --- | --- |
| Frontend | Next.js 15 (App Router), React 19, TypeScript |
| Styling | Tailwind CSS v4 (custom design tokens) |
| Backend | Supabase — PostgreSQL, Auth, Storage |
| Documents | Server-side PDF generation (`@react-pdf/renderer`) |
| AI | AWS Bedrock (in-region vision OCR) + an LLM content pipeline |
| Email | Resend (contact form + transactional auth email) |
| Hosting | Vercel |
| Platform | Installable PWA with offline support |

---

## Project status

ScriptPal NZ is in **beta**. The core app, demo mode and legal documentation are live and continuously improving ahead of public release.

---

## Links

- 🌐 **Live app:** [scriptpal.co.nz](https://scriptpal.co.nz/)
- 🧪 **Interactive demo:** [scriptpal.co.nz/demo](https://scriptpal.co.nz/demo)
- ▶️ **How it works:** [demo video](./public/how-it-works.mp4)
- 💼 **GitHub:** [github.com/dr-hannah-brotheridge](https://github.com/dr-hannah-brotheridge)

---

## License

Proprietary. All rights reserved.

