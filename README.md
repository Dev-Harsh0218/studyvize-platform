# StudyVize

Study-abroad consultancy platform. Three components across three repos: a **public marketing website** that generates leads, a **corporate admin portal** where consultants manage those leads through the full student-application lifecycle, and a **backend API** that ties them together and handles email notifications.

Built as client work — this is a real product used by consultancy staff.

---

## Architecture

```
                                      ┌──────────────────────────────┐
                                      │      Public Marketing         │
                                      │      (static HTML/CSS/JS)     │
                                      │  Landing pages, contact form  │
                                      │      studyvize-website         │
                                      └──────────────┬───────────────┘
                                                     │
                                             submit contact form
                                                     ▼
                                      ┌──────────────────────────────┐
                                      │       Corp Backend API        │
                                      │       Node.js + Express       │
                                      │  - Lead ingestion             │
                                      │  - Application CRUD           │
                                      │  - Email notifications        │
                                      │    (nodemailer)               │
                                      │      studyvize-api        │
                                      └──────┬──────────────────┬────┘
                                             │                  │
                                             ▼                  ▼
                                      ┌───────────┐    ┌──────────────┐
                                      │ MongoDB   │    │    SMTP      │
                                      │(students, │    │ notification │
                                      │ apps,     │    │   pipeline   │
                                      │ users)    │    └──────────────┘
                                      └─────▲─────┘
                                            │
                                            │ read + write
                                            │
                                      ┌──────────────────────────────┐
                                      │    Corp Admin Portal          │
                                      │    Next.js 16 (auth req.)     │
                                      │  Consultants manage:          │
                                      │  - Student profiles           │
                                      │  - University applications    │
                                      │  - Document checklists        │
                                      │  - Status tracking            │
                                      │      studyvize-portal        │
                                      └──────────────────────────────┘
```

## The three repos

| Repo | Purpose | Tech |
|---|---|---|
| [**`studyvize-website`**](https://github.com/Dev-Harsh0218/studyvize-website) | Public marketing website — course listings, program pages, contact form for prospective students | HTML5, CSS3, vanilla JavaScript |
| [**`studyvize-portal`**](https://github.com/Dev-Harsh0218/studyvize-portal) | Staff-facing admin portal — consultants manage leads, applications, documents, status | Next.js 16, TypeScript, React 19, Tailwind, lucide-react |
| [**`studyvize-api`**](https://github.com/Dev-Harsh0218/studyvize-api) | REST API + email pipeline — receives leads from website, serves the admin portal, sends notification emails | Node.js, Express, MongoDB, Mongoose, Nodemailer, deployed on Vercel |

## The flow

1. **Prospective student** visits [studyvize-website](https://github.com/Dev-Harsh0218/studyvize-website), browses programs, fills out a contact form
2. **Website** POSTs the lead to `studyvize-api` — API creates a `Lead` document in MongoDB, fires an acknowledgement email to the student + notification email to the assigned consultant (via Nodemailer)
3. **Consultant** signs in to [studyvize-portal](https://github.com/Dev-Harsh0218/studyvize-portal), sees the new lead in their inbox, opens the student profile, starts building the university application
4. **All application state** — student personal info, document uploads, university choices, application status — lives in MongoDB, updated via `studyvize-api` from the portal
5. **Email pipeline** fires on state transitions (application submitted, docs approved, offer received, visa granted) — Nodemailer handles delivery

## Design decisions

| Decision | Why |
|---|---|
| **Static HTML for the marketing site (not Next.js)** | The public site is content-heavy but interaction-light — course pages, program descriptions, contact form. Static HTML deploys anywhere for free, has zero JavaScript overhead for SEO crawlers, and is trivially cacheable at any CDN. Overkill would be adding SSR. |
| **Next.js 16 for the admin portal** | Portal is data-heavy, form-heavy, requires auth, needs fast navigation between student profiles. Next.js App Router + React Server Components + client-side interaction is the right tool. |
| **MongoDB (not Postgres) for the backend** | Student applications have variable, unpredictable structure — different countries, different universities, different document requirements. MongoDB's flexibility here saved months of schema-migration work. |
| **Nodemailer for email (not a managed service)** | Volume is low (dozens of emails/day, not thousands), consultants wanted control over templates, and self-hosted SMTP avoided per-email pricing. |
| **Three repos, not one monorepo** | The marketing site's release cadence is monthly (content updates), portal's is weekly, backend's is per-feature. Separate repos = independent deploys, independent code review, minimal cross-team friction. |

## Live URLs

| Component | URL | Status |
|---|---|---|
| Marketing website | _TBD (client-hosted)_ | 🟡 Client hosts |
| Corp portal | _TBD (staff-only)_ | 🟡 Behind auth, not public |
| API | Deployed on Vercel (serverless) | 🟡 Client-hosted |

## Not open-source-able for demo

This is real client work. The three repos are on GitHub as **portfolio / code-sample** — the running infrastructure is on the client's own Vercel + MongoDB Atlas + SMTP setup, not something we can point a public URL at.

The code, architecture, and design decisions are all here for reference.

## Related

Built by [Harsh Bhardwaj](https://github.com/Dev-Harsh0218) — full-stack engineer working across Node.js, Django, React, and Next.js.
