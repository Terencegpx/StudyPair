# StudyPair (StudyHive)

AI-powered social learning and peer tutoring platform for Malaysian secondary school students.

## Project Status — Full Backend Built

| Bolt | Module | Backend | Tests | Mobile UI |
|---|---|---|---|---|
| 0 | Scaffold + foundation | ✅ | — | ✅ |
| 1 | Account | ✅ | 11/11 ✅ | ✅ wired |
| 2 | Subscription | ✅ | 5/5 ✅ | ✅ wired |
| 3 | ContentManagement | ✅ admin endpoints | covered in QB tests | (admin web TBD) |
| 4 | QuestionBank | ✅ | 5/5 ✅ | ✅ wired |
| 5 | AIAnalysis | ✅ Claude (fake) | 3/3 ✅ | ✅ wired |
| 6 | Pairing | ✅ | 3/3 ✅ | ✅ wired |
| 7 | Tutoring | ✅ Xendit (fake) | 3/3 ✅ | ✅ wired |
| 8 | Notification | ✅ FCM (fake) | 3/3 ✅ | ✅ wired |
| 9 | Admin | ✅ | 2/2 ✅ | (admin web TBD) |

**Backend tests: 35/35 passing. Mobile TypeScript: 0 errors.**

External integrations (Claude, Xendit, FCM) are wired behind interfaces with `Fake*` implementations. Swap to real implementations in `Program.cs` when API keys are available — no other code changes needed.

## Quick Start

### 1. Run the backend
```powershell
cd D:\StudyPair\StudyPair
dotnet run
```
Swagger: https://localhost:7050/swagger

### 2. Create the database (one-time)
```powershell
cd D:\StudyPair\StudyPair.Context
dotnet ef migrations add InitialCreate --startup-project ..\StudyPair
dotnet ef database update --startup-project ..\StudyPair
```

### 3. Run the mobile app
```powershell
cd D:\StudyPair\StudyPair.Mobile
npx expo start
```
- Press `a` for Android emulator
- Press `i` for iOS simulator (Mac only)
- Press `w` for web preview

### 4. Run all backend tests
```powershell
dotnet test D:\StudyPair\StudyPair.sln
```
Expected: 35/35 passing.

### 5. Typecheck mobile app
```powershell
cd D:\StudyPair\StudyPair.Mobile
npx tsc --noEmit
```

## Backend Endpoints

| Method | Path | Module | Auth |
|---|---|---|---|
| POST | /api/account/register | Account | none |
| POST | /api/account/login | Account | none |
| GET | /api/account/profile | Account | Bearer |
| PUT | /api/account/profile | Account | Bearer |
| POST | /api/account/mentor-verification | Account | Bearer |
| DELETE | /api/account | Account | Bearer |
| GET | /api/subscription/plans | Subscription | none |
| GET | /api/subscription/status | Subscription | Bearer |
| POST | /api/subscription/subscribe | Subscription | Bearer |
| POST | /api/subscription/cancel | Subscription | Bearer |
| POST | /api/subscription/members | Subscription | Bearer |
| POST | /api/content/questions | ContentManagement | Admin |
| PUT | /api/content/questions/{id} | ContentManagement | Admin |
| POST | /api/content/questions/{id}/retire | ContentManagement | Admin |
| GET | /api/questionbank/browse | QuestionBank | Bearer |
| GET | /api/questionbank/questions/{id} | QuestionBank | Bearer |
| POST | /api/questionbank/attempts | QuestionBank | Bearer |
| GET | /api/questionbank/attempts/me | QuestionBank | Bearer |
| POST | /api/questionbank/flags | QuestionBank | Bearer |
| POST | /api/ai/submit | AIAnalysis | Bearer |
| GET | /api/ai/submissions/{id} | AIAnalysis | Bearer |
| GET | /api/ai/history | AIAnalysis | Bearer |
| POST | /api/ai/submissions/{id}/rate | AIAnalysis | Bearer |
| GET | /api/pairing/profile | Pairing | Bearer |
| PUT | /api/pairing/profile | Pairing | Bearer |
| GET | /api/pairing/discover | Pairing | Bearer |
| POST | /api/pairing/requests | Pairing | Bearer |
| GET | /api/pairing/requests | Pairing | Bearer |
| POST | /api/pairing/requests/{id}/respond | Pairing | Bearer |
| GET | /api/tutoring/listings | Tutoring | Bearer |
| GET | /api/tutoring/listings/{id} | Tutoring | Bearer |
| POST | /api/tutoring/listings | Tutoring | Bearer (Mentor) |
| POST | /api/tutoring/bookings | Tutoring | Bearer |
| GET | /api/tutoring/bookings/me/student | Tutoring | Bearer |
| GET | /api/tutoring/bookings/me/mentor | Tutoring | Bearer |
| POST | /api/tutoring/bookings/{id}/complete | Tutoring | Bearer |
| POST | /api/tutoring/reviews | Tutoring | Bearer |
| GET | /api/tutoring/mentor/dashboard | Tutoring | Bearer |
| GET | /api/notifications | Notification | Bearer |
| POST | /api/notifications/{id}/read | Notification | Bearer |
| POST | /api/notifications/read-all | Notification | Bearer |
| GET | /api/notifications/preferences | Notification | Bearer |
| PUT | /api/notifications/preferences | Notification | Bearer |
| POST | /api/notifications/devices | Notification | Bearer |
| GET | /api/admin/kpis | Admin | Admin |
| GET | /api/admin/users | Admin | Admin |
| POST | /api/admin/users/{id}/disable | Admin | Admin |

## What's Done vs Still Required for Production

✅ Done:
- All 9 modules built with real EF entities, repositories, controllers, tests
- JWT authentication and role-based authorisation
- Premium gating (free tier: 5 most recent years of questions, 3 AI analyses/week)
- 85/15 platform fee split on tutoring sessions
- Soft-delete on every entity
- Global exception middleware
- Swagger UI for API exploration

🚧 Pending (external setup required):
- **Real Claude API key** → swap `FakeClaudeService` for `RealClaudeService` (implementation pending)
- **Real Xendit account + API key** → swap `FakeXenditService` for `RealXenditService`
- **Firebase project + FCM credentials** → swap `InMemoryPushService` for `FcmPushService`
- **Azure SQL connection string** for production DB
- **Azure Blob Storage** for question images and handwritten answer uploads
- **EF migration applied to the local SQL Server** (`dotnet ef database update` — needs LocalDB running)
- **Lembaga Peperiksaan licensing** for SPM/PT3 past year paper content
- **Manual UI test pass** on Android + iOS simulators / physical devices
- **Admin web portal** (Content Management + Admin module currently API-only)
- **Camera upload** for handwritten answer photos

## Documentation

Read in order:
1. [docs/INDEX.md](docs/INDEX.md) — master hub
2. [CLAUDE.md](CLAUDE.md) — firm decisions
3. [docs/architecture.md](docs/architecture.md) — mental model
4. [docs/database.md](docs/database.md) — schema
5. [docs/methodology.md](docs/methodology.md) — bolts

See [docs/INDEX.md](docs/INDEX.md) for the full doc catalogue (28+ MDs).

## License & Legal Note

Past year SPM/PT3 paper distribution requires a licensing arrangement with Lembaga Peperiksaan Malaysia.
