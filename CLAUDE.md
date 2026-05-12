# StudyPair — CLAUDE.md

This file gives Claude Code full context for this project. Read it at the start of every session.

**Then read [docs/INDEX.md](docs/INDEX.md)** — the master hub that lists every rule and doc.

**Then run [docs/prompt-checklist.md](docs/prompt-checklist.md)** — the 60-second routine before responding.

---

## Project Overview

**Platform name:** StudyHive (working title: StudyPair)
**Goal:** Mobile-first academic platform for Malaysian Form 1–5 government secondary school students. Provides past year SPM/PT3 question practice, Claude AI-powered answer analysis, social study partner matching, and a peer tutoring marketplace where high-achieving students earn income.
**Target audience:** Malaysian secondary school students (B40/M40 households), their parents, and platform administrators.
**Monetisation:** Freemium subscription — Individual Plan RM 15/month, Family Plan RM 25/month (up to 3 students). Secondary revenue: 15% platform fee on peer tutoring sessions.

---

## Development Methodology

**Approach: Spec-Driven Development (SDD) + AI-DLC hybrid**

See `docs/methodology.md` for the full guide. In short:
- Every module starts with a written spec (in `docs/specs/`) before any code is written.
- Development runs in short "bolts" (3–5 day intensive iterations), not multi-week sprints.
- Claude acts as an orchestrating agent: given a spec, it generates controllers, models, views, and repository stubs in sequence.
- Specs are the source of truth. If code and spec conflict, fix the code.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Mobile frontend | React Native + TypeScript (Expo managed workflow) |
| Backend API | .NET 8 Web API, C# |
| Database | Microsoft SQL Server (Entity Framework 8, code-first migrations) |
| AI analysis | Anthropic Claude API (claude-sonnet-4-6), prompt caching enabled |
| Payments | Xendit (FPX, DuitNow QR, Touch 'n Go eWallet, card) |
| File storage | Azure Blob Storage + CDN (answer image uploads) |
| Push notifications | Firebase Cloud Messaging |
| Hosting | Azure App Service (.NET API), Azure SQL |

**Do not suggest Firebase Realtime DB or Firestore as a database replacement — SQL Server is a firm decision.**
**Do not suggest Flutter — React Native (Expo) is a firm decision.**

---

## Project Structure

```
D:\StudyPair\
├── StudyPair\                          # .NET 8 Web API project (backend)
│   ├── App_Start\
│   ├── Areas\
│   │   ├── Account\                    # Registration, login, profile, tier management
│   │   ├── QuestionBank\               # Past year questions, browse, attempt, flag
│   │   ├── AIAnalysis\                 # Claude API integration, feedback history
│   │   ├── Pairing\                    # Study partner matching algorithm and sessions
│   │   ├── Tutoring\                   # Mentor marketplace, bookings, earnings
│   │   ├── Subscription\               # Xendit billing, plan management, webhooks
│   │   ├── Notification\               # FCM push, email reminders, notification log
│   │   ├── Admin\                      # User management, reports, moderation
│   │   └── ContentManagement\          # Question bank CRUD, bulk import, flag review
│   ├── Controllers\                    # Shared/base controllers
│   ├── Library\GlobalVariable\         # App constants, config keys
│   ├── Filters\                        # Auth filters, subscription gate filter
│   ├── Views\Shared\
│   ├── Scripts\
│   └── Content\
├── StudyPair.Context\                  # Data Access Layer
│   ├── Models\                         # EF8 entity classes
│   └── Repositories\                   # Repository pattern, one file per entity group
├── StudyPair.Tests\                    # xUnit tests
└── docs\
    ├── methodology.md                  # SDD + AI-DLC development guide
    ├── specs\                          # One spec file per module (created before coding)
    └── db-schema.md                    # SQL Server table definitions
```

Each Area follows this internal layout:
```
Areas\{Module}\
├── Controllers\{Module}Controller.cs
├── Models\                             # ViewModels and DTOs for this area
├── Views\{Module}\                     # .cshtml views
├── Scripts\                            # Area-specific JS
└── Styles\                             # Area-specific CSS
```

---

## Module Responsibilities

| Module | What it owns |
|---|---|
| Account | ss_user table, authentication, JWT tokens, tier (Explorer/Scholar/Mentor), proficiency assessment |
| QuestionBank | qb_question, qb_attempt, qb_tag tables; question rendering with LaTeX; student flagging |
| AIAnalysis | ai_submission, ai_feedback tables; Claude API calls; prompt caching; feedback display |
| Pairing | pair_profile, pair_request, pair_session tables; matching algorithm; study group management |
| Tutoring | tutor_listing, tutor_booking, tutor_review, tutor_earning tables; Xendit session payment |
| Subscription | sub_plan, sub_account tables; Xendit recurring billing; free/premium gate logic |
| Notification | notif_log table; FCM dispatch; email via SendGrid; scheduled reminder jobs |
| Admin | Cross-module read access; user CRUD; broadcast notifications; platform KPI dashboard |
| ContentManagement | Question CRUD; bulk CSV import; Azure Blob upload for question images; flag review queue |

---

## Database Naming Conventions

- Table prefix per module: `ss_` (system/shared), `qb_` (question bank), `ai_` (AI analysis), `pair_` (pairing), `tutor_` (tutoring), `sub_` (subscription), `notif_` (notification), `cm_` (content management)
- All tables have: `Id` (int, PK, identity), `CreatedAt` (datetime2), `UpdatedAt` (datetime2), `IsDeleted` (bit, default 0)
- Foreign keys named `{ReferencedTable}Id` e.g. `UserId`, `QuestionId`
- Columns use PascalCase in EF models, stored as PascalCase in SQL Server

---

## API Conventions (.NET 8 Web API)

- All endpoints return a consistent envelope:
  ```json
  { "success": true, "data": {}, "message": "", "errors": [] }
  ```
- Route pattern: `/api/{module}/{action}` e.g. `/api/questionbank/browse`
- Authentication: JWT Bearer tokens, validated via `[Authorize]` attribute
- Subscription gate: `[RequiresPremium]` custom filter on premium-only endpoints
- All controller actions are async

---

## Claude API Integration Rules

- Model: `claude-sonnet-4-6`
- Always enable prompt caching on system prompts containing marking scheme templates
- Marking scheme templates live in `Library\GlobalVariable\MarkingSchemes\` as `.txt` files, one per subject
- Per-request cost target: under RM 0.05 per analysis
- Never pass the student's full attempt history in a single request — only the current question and answer
- Return feedback structured as JSON with fields: `pointsAwarded`, `missingPoints[]`, `errors[]`, `suggestedCorrection`, `language`

---

## Payment Integration Rules (Xendit)

- Use Xendit Malaysia (licensed by Bank Negara Malaysia)
- Supported methods: FPX, DuitNow QR, Touch 'n Go eWallet, Visa/Mastercard
- Tutoring session payments are held until both parties confirm session completion
- Platform fee: 15% retained, 85% disbursed to Mentor's registered bank account
- Webhook endpoint: `/api/subscription/xendit-webhook` and `/api/tutoring/xendit-webhook`
- Never log full card numbers or full bank account numbers

---

## Student Tier Definitions

| Tier | Label | Eligibility | Can Tutor |
|---|---|---|---|
| Explorer | Needs support | Default for new students | No |
| Scholar | Average performer | Self-selected | No |
| Mentor | High achiever | Pass proficiency assessment (≥80% per subject) | Yes, per verified subject |

Tier is stored per-user in `ss_user.Tier`. Mentor subject badges stored in `ss_mentor_badge (UserId, Subject, VerifiedAt)`.

---

## Free vs Premium Feature Gates

| Feature | Free | Premium (RM 15/month) |
|---|---|---|
| Past year questions | Last 5 years | Full 20-year archive |
| AI answer analyses | 3 per week | Unlimited |
| Study partner matching | Basic | Priority + group matching |
| Progress analytics | None | Full dashboard + PDF export |
| Tutor booking | Yes (pay per session) | Yes (pay per session) |

---

## Key Business Rules

1. A student cannot apply for Mentor status in a subject they have not studied at their current form level.
2. Mentor proficiency assessments enforce a 14-day cooldown on failure before re-attempt.
3. Subscription downgrade to free tier triggers after 3-day grace period following failed payment, not immediately.
4. Past year question images sourced from licensed or original content only — never reproduce MOE exam papers verbatim without a licensing arrangement with Lembaga Peperiksaan Malaysia.
5. All user-generated content (Mentor bios, study group posts, reviews) goes through a moderation queue before first public display.
6. Student data is never sold or shared with third parties. Aggregated analytics shared with educators must be anonymised at topic/region level only.

---

## What NOT to do

- Do not add features beyond the current bolt's spec. Build exactly what the spec says.
- Do not use Dapper — Entity Framework 8 repository pattern is the firm choice for data access.
- Do not store sensitive data (JWT secrets, Xendit API keys, Claude API key) in code. Use environment variables or Azure Key Vault references in `appsettings.json`.
- Do not generate URLs unless you are certain they are correct documentation or API reference URLs.
