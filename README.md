[README (3).md](https://github.com/user-attachments/files/31648058/README.3.md)
# Rwanda connect — Rwanda Opportunity Hub

**Rwanda connect** (Kinyarwanda for *guhuza*) is a next-generation job, tender, and internship portal built for the Rwandan labour market. It is designed to sit in the same category as established platforms such as Job in Rwanda — and to feel faster, cleaner, and more product-grade.

This repository ships a **production-quality frontend prototype**: a single `index.html` file that contains the full interface, design system, interaction layer, and a **strong in-memory database** of realistic Rwandan opportunities.

> Open `index.html` in any modern browser. No build step. No server required.

---

## Why this exists

Job boards that dominate Rwanda today are useful but often look like 2014 content-management pages: long text lists, limited filtering, weak visual hierarchy, and almost no sense of product.

Rwanda connect is built to demonstrate what a **higher-tier** portal looks like:

| Capability | Typical local job board | Rwanda connect |
|---|---|---|
| Visual system | Basic CMS theme | Custom design system (Rwanda palette, cards, motion) |
| Search | Keyword only | Live search + 4 filters + result count |
| Listings | Text rows | Featured cards, type badges, deadlines, views, company marks |
| Data | Server HTML | Structured job objects (queryable “database”) |
| Apply flow | External form | In-page detail drawer + application modal |
| Mobile | Partial | Mobile-first, sticky search, touch-friendly filters |
| Extensibility | Theme edits | Swap the JS database for Postgres / Supabase / Firebase |

---

## Product features

### For job seekers
- Hero search across title, company, location, and keywords
- Filters: type (Job / Tender / Internship / Consultancy), sector, location, experience
- Featured and urgent opportunities highlighted
- Job detail panel with description, requirements, and meta
- Demo “Apply now” flow with validation
- Live result counts and empty states
- Keyboard-friendly search (press `/` to focus)

### For the market narrative
- National stats strip (jobs, employers, candidates, districts)
- Sector grid covering Rwanda’s real economy: ICT, Banking & Finance, NGOs, Public sector, Health, Education, Agriculture, Energy, Mining, Tourism & Hospitality
- Employer-facing CTA (post a vacancy)

### Design
- Rwanda-inspired palette: sky blue, verdant green, sun gold
- Inter + Source Serif 4 typography
- Soft elevation, 8px spacing scale, micro-interactions
- Accessible contrast, focus rings, reduced-motion support

---

## The “strong database”

There is no fake placeholder text. `index.html` embeds a structured **Rwanda connect Data Store** — a JavaScript module that behaves like a lightweight database.

Each record looks like this:

```js
{
  id: "UMW-1042",
  title: "DevOps Engineer",
  company: "Development Bank of Rwanda (BRD)",
  initials: "BR",
  location: "Kigali",
  district: "Nyarugenge",
  type: "Job",
  sector: "Banking & Finance",
  experience: "Mid (3–5 years)",
  deadline: "2026-09-06",
  published: "2026-08-20",
  salary: "Competitive + benefits",
  views: 1842,
  featured: true,
  urgent: false,
  tags: ["DevOps", "AWS", "CI/CD", "Linux"],
  summary: "…",
  description: "…",
  requirements: ["…", "…"]
}
```

### What makes it “strong” (for a static prototype)

- **Normalized fields** — every listing shares the same schema, so filters never break.
- **Queryable** — `Rwanda connectDB.query({ q, type, sector, location, experience })` returns ranked results.
- **Ranked search** — title matches beat company matches beat tag matches.
- **Derived fields** — days remaining until deadline, “closing soon” flags, relative published dates.
- **Scale-ready shape** — the same objects can be stored in PostgreSQL, MongoDB, or Supabase without redesign.
- **Realistic corpus** — 42 listings modelled on real Rwandan employers and sectors (BRD, BK, MTN, RwandAir, PIH, CARE, RDB-style public work, agribusiness, mining, energy, universities). Names are illustrative, not official postings.

### Suggested production schema

When you move off the prototype, map the JS objects 1:1:

```sql
CREATE TABLE employers (
  id            UUID PRIMARY KEY,
  name          TEXT NOT NULL,
  slug          TEXT UNIQUE,
  logo_url      TEXT,
  about         TEXT,
  verified      BOOLEAN DEFAULT false,
  created_at    TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE jobs (
  id            UUID PRIMARY KEY,
  public_id     TEXT UNIQUE,          -- e.g. Rw-1042
  employer_id   UUID REFERENCES employers(id),
  title         TEXT NOT NULL,
  summary       TEXT,
  description   TEXT,
  requirements  TEXT[],
  location      TEXT,
  district      TEXT,
  type          TEXT CHECK (type IN ('Job','Tender','Internship','Consultancy')),
  sector        TEXT,
  experience    TEXT,
  salary        TEXT,
  deadline      DATE,
  published_at  TIMESTAMPTZ,
  views         INTEGER DEFAULT 0,
  featured      BOOLEAN DEFAULT false,
  urgent        BOOLEAN DEFAULT false,
  status        TEXT DEFAULT 'live',
  search_vector TSVECTOR
);

CREATE INDEX jobs_search_idx ON jobs USING GIN (search_vector);
CREATE INDEX jobs_deadline_idx ON jobs (deadline);
CREATE INDEX jobs_filters_idx ON jobs (type, sector, location, status);
```

Add tables for `applications`, `candidates`, `saved_jobs`, and `alerts` when you introduce accounts.

---

## File map

```
.
├── index.html      # Entire product: UI + design system + database + logic
└── README.md       # This file
```

Everything is inlined on purpose. You can email the HTML file, drop it on Netlify Drop, or host it on GitHub Pages in under a minute.

---

## Run locally

```bash
# Option A — just open it
open index.html

# Option B — tiny static server (avoids some browser file:// quirks)
npx serve .
# then visit http://localhost:3000
```

---

## How the app works

1. On load, `Rwanda connectDB.all()` hydrates the UI.
2. The search box and filter chips call `Rwanda connectDB.query()`.
3. Results render as cards. Featured jobs are pinned first.
4. Clicking a card opens a detail drawer (no page reload).
5. “Apply” opens a modal. Submissions are validated client-side and stored in `localStorage` under `Rwanda connect_applications` so you can demonstrate persistence without a backend.

### Keyboard
- `/` — focus search
- `Esc` — close drawer or modal

---

## Roadmap to a real platform

The HTML file is the **presentation + contract**. To compete with a national job board you would add:

1. **Auth** — email + OTP (Rwanda-friendly), employer vs candidate roles.
2. **Postgres + full-text search** — or Supabase for a fast start.
3. **Object storage** — CVs (PDF), employer logos.
4. **Application pipeline** — status: submitted → shortlisted → interviewed.
5. **Employer dashboard** — post, feature, export applicants to Excel.
6. **Alerts** — weekly digest SMS/email (MTN / Airtel + Resend).
7. **Kinyarwanda + English + french** — `lang` toggle; store both strings.
8. **SEO** — server-render job pages (`/jobs/devops-engineer-brd`).
9. **Admin** — moderate listings, feature slots, analytics.
10. **Payments** — featured listing plans (mirrors the 100k–300k RWF packages common in the market).

Recommended stack if you graduate this prototype:

- Frontend: this UI extracted into React / Next.js or plain HTML kept as-is
- Backend: Next.js Route Handlers or FastAPI
- Database: PostgreSQL (Neon or Supabase)
- Files: Cloudflare R2
- Hosting: Vercel / Cloudflare Pages
- Observability: Plausible (privacy-friendly, works well in Rwanda)

---

## Brand notes

- **Name:** Umwanya — short, local, memorable.
- **Promise:** “Opportunity, organised.”
- **Voice:** Clear, respectful, ambitious. No slang, no hype that oversells.
- **Do not** present demo listings as live official vacancies. The sample corpus is for product demonstration only.

---

## Comparison intent (honest)

Umwanya is **not** a scraped clone of Job in Rwanda. It is an original interface and data model inspired by the *job the market needs*:

- Job in Rwanda proved demand: hundreds of thousands of monthly visits, mixed jobs + tenders, employer services.
- Umwanya shows a 2026 product layer on top of that demand: structured data, instant filter, modern UX, and a schema you can actually grow into a company.

Treat this file as a design + engineering brief you can put in front of a founder, an NGO digital team, or a student capstone panel.

---

## License

Demo project. Sample employer names are used illustratively. Do not publish this as an official government or Job-in-Rwanda property.

Built as a high-fidelity static product, August 2026.
